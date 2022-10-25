 

# EKS










## Aceess

User accounts versus service accounts 
- user
  - User accounts are for humans. 
  - Service accounts are for processes, which run in pods.
- name
  - User accounts are intended to be global. Names must be unique across all namespaces of a cluster. 
  - Service accounts are namespaced.
- permission
  - Typically, a cluster's user accounts might be synced from a corporate database, where new user account creation requires special privileges and is tied to complex business processes. 
  - Service account creation is intended to be more lightweight, allowing cluster users to create service accounts for specific tasks by following the principle of least privilege.
- Auditing considerations for humans and service accounts may differ.
- A config bundle for a complex system may include definition of various service accounts for components of that system. Because service accounts can be created without many constraints and have namespaced names, such config is portable.













## use case

### Enabling cross-account access to EKS cluster resources 

Often customers manage their AWS environments separated using multiple AWS accounts. 
- They do not want production resources to interact or coexist with development or staging resources. 
- While this provides the benefits of better resource isolation, it increases the access management overhead.

access
- User access to multiple accounts can be managed by leveraging temporary AWS security credentials using `AWS Security Token Service (STS) and IAM roles`. 
- But what if the resources, say, containerized workloads or pods in an EKS cluster hosted in one account wants to interact with the EKS cluster resources hosted in another account? 
  - use fine-grained roles at the pod level using `IAM Roles for Service Accounts (IRSA)`.  

---

**Scenario**

- a customer with multiple accounts – dev, stg, and prod 
- wants to manage the resources from a continuous integration (CI) account. 
- An EKS cluster in this CI account needs to access AWS resources to these target accounts. 
- One simple way to `grant access to the pods in the CI account` to target cross-account resources:
  - Create roles in these target accounts
  - Grant assume role permissions to the CI account EKS cluster node instance profile on the target account roles
  - And finally, trust this cluster node instance profile in the target account’s role(s)


Though this will allow the EKS cluster in the CI account to communicate with the AWS resources in the target accounts, it grants **any pod running on this node** access to this role. 
- At AWS, we always insist to follow the standard security advice of granting least privilege, or granting only the permissions required to perform a task. Start with a minimum set of permissions and grant additional permissions as necessary.

---

**Solution**

![Screen Shot 2022-10-19 at 11.58.46](https://i.imgur.com/Hqu1Por.png)

IAM supports federated users using **OpenID Connect (OIDC)**. 
- Amazon EKS hosts a `public OIDC discovery endpoint` per **cluster** containing the **signing keys** for the `ProjectedServiceAccountToken` JSON web tokens 
- so external systems, like IAM, can validate and accept the OIDC tokens issued by Kubernetes.  


---

#### Prerequisites

- OIDC
- IAM role + IAM policy
- Kubenert service account -> IAM role

 
##### Setup OIDC in CI account

1. Fetch the CI account cluster’s OIDC issuer URL
   1. for EKS cluster version is 1.13+, it will have an OpenID Connect issuer URL. 
   2. get this URL from the Amazon EKS console directly, or use CLI command to retrieve it.

```bash
aws eks describe-cluster \
    --name <CI_EKS_CLUSTER_NAME> \
    --query "cluster.identity.oidc.issuer" \
    --output text \
    --profile ci-env

# Determine whether you have an existing IAM OIDC provider for the cluster.
oidc_id=$(aws eks describe-cluster \
              --name my-cluster \
              --query "cluster.identity.oidc.issuer" \
              --output text | cut -d '/' -f 5)

# Determine whether an IAM OIDC provider with the cluster's ID is already in the account.
aws iam list-open-id-connect-providers | grep $oidc_id
```


1. Create an OIDC provider for the cluster in the CI account
   1. Navigate to the IAM console in the CI account, choose Identity Providers, and then select Create provider.
   2. Select `OpenID Connect` for provider type and paste the OIDC issuer URL for the cluster for provider URL. Enter `sts.amazonaws.com` for audience as shown below.

```bash
# If no output is returned, then you must create an IAM OIDC provider for the cluster.

# Create an IAM OIDC identity provider for the cluster with the following command. Replace my-cluster with the own value.
eksctl utils associate-iam-oidc-provider \
  --cluster my-cluster \
  --approve
```


##### Setup IAM role in CI account

1. Create IAM policy

```bash
cat >my-policy.json <<EOF
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": "config:*",
            "Resource": "*"
        }
    ]
}
EOF

# Create the IAM policy.
aws iam create-policy \
  --policy-name my-policy \
  --policy-document file://my-policy.json
```


1. Create an IAM role
   1. Create an `IAM role` in the CI account, ci-account-iam-role
   2. associate it **IAM policy** 

```json
// Create the role.
aws iam create-role \
  --role-name my-role \
  --assume-role-policy-document file://trust-relationship.json \
  --description "my-role-description"

// Attach IAM policy 
aws iam attach-role-policy \
  --role-name my-role \
  --policy-arn=arn:aws:iam::$account_id:policy/my-policy
```




1. Once the role is created
   1. associate it with **Kubernetes service account**
   2. attach the IAM policy that you want to associate with the serviceaccount. 
   3. the **serviceaccount** must be able to assume role to the target account, so grant the following assume role permissions.
   4. Note that if you haven’t created the target account IAM role, please proceed to step 4 and complete configuring the target AWS account and then finish associating this policy.



```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": "sts:AssumeRole",
            "Resource": "arn:aws:iam::TARGET_ACCOUNT_ID:role/target-account-iam-role"
        }
    ]
}
```





##### Setup Kubernetes serviceaccount in CI account


2. Creat the serviceaccount 
   1. associate **IAM Role** with **Kubernetes service account**
   2. attach the IAM policy that you want to associate with the serviceaccount. 

1. Setup IAM role
   1. associate it with **trust relationship** to the cluster’s OIDC provider
      1. pecify the service-account, namespace to restrict the access. 
      2. specifying ci-namespace and ci-serviceaccount for namespace and serviceaccount respectively. 
      3. Replace the OIDC_PROVIDER with the provider URL saved in the previous step.



```bash
# Set variables 
export cluster_name=dev-medusa-01
export namespace=default
export service_account=medusa-core
export my_role=eks-medusa-core 
export policy_arn=arn:aws:iam::350842811077:policy/medusa-coe-config-policy
# Set AWS account ID 
account_id=$(aws sts get-caller-identity --query "Account" --output text)
# Set cluster's OIDC identity provider 
oidc_provider=$(aws eks describe-cluster \
  --name dev-medusa-01 \
  --query "cluster.identity.oidc.issuer" \
  --output text | sed -e "s/^https:\/\///")



# ============= 1
eksctl create iamserviceaccount \
  --name $service_account \
  --namespace $namespace \
  --cluster $cluster_name \
  --role-name $my_role \
  --attach-policy-arn $policy_arn \
  --approve


# ============= 2 
cat >my-service-account.yaml <<EOF
apiVersion: v1
kind: ServiceAccount
metadata:
  name: $service_account
  namespace: $namespace
EOF
kubectl apply -f my-service-account.yaml


cat >trust-relationship.json <<EOF
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "Federated": "arn:aws:iam::$account_id:oidc-provider/$oidc_provider"
      },
      "Action": "sts:AssumeRoleWithWebIdentity",
      "Condition": {
        "StringEquals": {
          "$oidc_provider:aud": "sts.amazonaws.com",
          "$oidc_provider:sub": "system:serviceaccount:$namespace:$service_account"
        }
      }
    }
  ]
}
EOF
```


1. Configuring the Kubernetes
   1. In Kubernetes, define the IAM role to associate with a service account in the cluster by adding the `eks.amazonaws.com/role-arn` annotation to the service account. 
   2. annotate the service account associated with the cluster in the CI account with the role ARN as shown below.


```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: ci-serviceaccount
  namespace: ci-namespace
  annotations:
    eks.amazonaws.com/role-arn: arn:aws:iam::CI_ACCOUNT_ID:role/ci-account-iam-role
```
The equivalent kubectl command is:

```bash

# Annotate service account with the IAM role Arn that you want the service account to assume. 
# Replace my-role with the name of the existing IAM role. 
# Suppose that you allowed a role from a different AWS account than the account that the cluster is in to assume the role. 
# Then, make sure to specify the AWS account and role from the other account. For more information, see Cross-account IAM permissions.
$ kubectl annotate serviceaccount \
  -n $namespace $service_account eks.amazonaws.com/role-arn=arn:aws:iam::$account_id:role/my-role 
```







##### Confirm the role and service account 


```bash 
# Confirm that the role and service account are configured correctly. 
aws iam get-role \
  --role-name $my_role \
  --query Role.AssumeRolePolicyDocument
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Principal": {
                "Federated": "arn:aws:iam::123:oidc-provider/oidc.eks.region-code.amazonaws.com/id/EXAMPLE"
            },
            "Action": "sts:AssumeRoleWithWebIdentity",
            "Condition": {
                "StringEquals": {
                    "oidc.eks.region-code.amazonaws.com/id/EXAMPLE:sub": "system:serviceaccount:default:my-service-account",
                    "oidc.eks.region-code.amazonaws.com/id/EXAMPLE:aud": "sts.amazonaws.com"
                }
            }
        }
    ]
}

# Confirm the policy is attached to the role.
aws iam list-attached-role-policies \
  --role-name $my_role \
  --query AttachedPolicies[].PolicyArn \
  --output text

# View the default version of the policy.
aws iam get-policy \
  --policy-arn $policy_arn

aws iam get-policy-version \
  --policy-arn $policy_arn \
  --version-id v1

# Confirm that the Kubernetes service account is annotated with the role.
kubectl describe serviceaccount $service_account -n default
# The example output is as follows.
# Name:                my-service-account
# Namespace:           default
# Annotations:         eks.amazonaws.com/role-arn: arn:aws:iam::111122223333:role/my-role
# Image pull secrets:  <none>
# Mountable secrets:   my-service-account-token-qqjfl
# Tokens:              my-service-account-token-qqjfl
```


 


##### configure a pod to use a service account



1. create a deployment manifest that you can deploy a pod to confirm configuration with


```bash

# Set variables 
export cluster_name=dev-medusa-01
export namespace=default
export service_account=medusa-core
export my_app=medusa-core
export my_pod=medusa-core-5bbf4dd447-rshjm
export my_role=eks-medusa-core 
export policy_arn=arn:aws:iam::350842811077:policy/medusa-coe-config-policy
# Set AWS account ID 
account_id=$(aws sts get-caller-identity --query "Account" --output text)
# Set cluster's OIDC identity provider 
oidc_provider=$(aws eks describe-cluster \
  --name dev-medusa-01 \
  --query "cluster.identity.oidc.issuer" \
  --output text | sed -e "s/^https:\/\///")


cat >my-deployment.yaml <<EOF
apiVersion: apps/v1
kind: Deployment
metadata:
  name: $my_app
spec:
  selector:
    matchLabels:
      app: $my_app
  template:
    metadata:
      labels:
        app: $my_app
    spec:
      serviceAccountName: $service_account
      containers:
      - name: $my_app
        image: public.ecr.aws/nginx/nginx:1.21
EOF

# Deploy the manifest 
kubectl apply -f my-deployment.yaml 
```


2. confirm 

```bash
# View the pods that were deployed with the deployment in the previous step.
kubectl get pods | grep $my_app
# The example output is as follows.
# $my_pod   1/1     Running   0          3m28s


# View the ARN of the IAM role that the pod is using.
kubectl describe pod $my_pod | grep AWS_ROLE_ARN:
# The example output is as follows.
# AWS_ROLE_ARN:                 arn:aws:iam::111122223333:role/my-role
# The role ARN must match the role ARN that you annotated the existing service account with. For more about annotating the service account, see Configuring a Kubernetes service account to assume an IAM role.


# Confirm that the pod has a web identity token file mount.
kubectl describe pod $my_pod | grep AWS_WEB_IDENTITY_TOKEN_FILE:
# The example output is as follows.
AWS_WEB_IDENTITY_TOKEN_FILE:  /var/run/secrets/eks.amazonaws.com/serviceaccount/token


# Confirm that the deployment is using the service account. 
kubectl describe deployment $my_app | grep "Service Account"
```

1. Confirm that the pods can interact with the AWS services using the permissions assigned in the IAM policy attached to the role.
   1. When a pod uses AWS credentials from an IAM role that's associated with a service account, the AWS CLI or other SDKs in the containers for that pod use the credentials that are provided by that role. If you don't restrict access to the credentials that are provided to the Amazon EKS node IAM role, the pod still has access to these credentials. For more information, see Restrict access to the instance profile assigned to the worker node.
   2. If the pods can't interact with the services as you expected, complete the following steps to confirm the configured.
      1. Confirm that the pods use an AWS SDK version that supports assuming an IAM role through an OpenID Connect web identity token file. For more information, see Using a supported AWS SDK.
      2. Confirm that the deployment is using the service account. 
      3. If the pods still can't access services, review the steps that are described in Configuring a Kubernetes service account to assume an IAM role to confirm that the role and service account are configured properly.
 


##### Configuring the target account

1. Configuring the target account – IAM role and policy permissions
   1. In target account, create an IAM role named `target-account-iam-role` with a trust relationship that allows AssumeRole permissions to CI account’s IAM role created in the previous step 

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": "sts:AssumeRole",
      "Principal": {
        "AWS": "arn:aws:iam::CI_ACCOUNT_ID:role/ci-account-iam-role"
      },
      "Condition": {}
    }
  ]
}
```

1. Configuring the target account – IAM policy
   1. create an IAM policy with the necessary permissions the service account’s pods in CI account cluster would need to manage.  
   2. Note that if you haven’t created associated the target policy to the CI account, please do the association before proceeding with the next step.

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "VisualEditor0",
            "Effect": "Allow",
            "Action": [
                "s3:ListAllMyBuckets",
                "eks:DescribeCluster",
                "eks:ListClusters"
            ],
            "Resource": "*"
        }
    ]
}
```



1. Verifying the cross-account access to AWS resources
   1. Now that the IAM access roles and policies are configured at both the ends- CI and target accounts, we create a sample Kubernetes pod in the CI cluster and test the cross-account access to AWS resources of the target account.
   2. save the below contents to a file named `awsconfig-configmap.yaml`. 

```bash
# This configmap is to set the AWS CLI profiles used in the deployment pod.
[profile ci-env]
role_arn = arn:aws:iam::CI_ACCOUNT_ID:role/ci-account-iam-role
web_identity_token_file = /var/run/secrets/eks.amazonaws.com/serviceaccount/token.

[profile target-env]
role_arn = arn:aws:iam::TARGET_ACCOUNT_ID:role/target-account-iam-role
source_profile = ci-env
role_session_name = xactarget


# Create a Kubernetes configMap resource with the below Kubectl command.aw
kubectl create configmap awsconfig-configmap \
    --from-file=config=awsconfig-configmap.yaml \
    -n ci-namespace



# Let us create another configMap resource to store the installation scripts. 
# Name a file named script-configmap.yaml and save the below contents into the file.

#!/bin/bash
apt-get update -y && apt-get install -y python curl wget unzip jq nano -y
curl "https://s3.amazonaws.com/aws-cli/awscli-bundle.zip" -o "awscli-bundle.zip"
unzip awscli-bundle.zip
./awscli-bundle/install -i /usr/local/aws -b /usr/local/bin/aws
cp -r aws_config/ ~/.aws/
curl -o kubectl https://amazon-eks.s3-us-west-2.amazonaws.com/1.14.6/2019-08-22/bin/linux/amd64/kubectl
chmod +x ./kubectl
mv ./kubectl /usr/local/bin/kubectl
kubectl version


# Create the second Kubernetes configMap resource with the below kubectl command.
kubectl create configmap script-configmap \
    --from-file=script.sh=script-configmap.yaml \
    -n ci-namespace 
```



Now it’s time to create a deployment and test the cross-account access. Create another file named test-deployment.yaml.
- Note to replace the values for namespace and serviceaccount if you specified different values in step 3 while creating the OIDC trust relationship.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  creationTimestamp: null
  labels:
    app: test-deployment
  name: test-deployment
  namespace: ci-namespace
spec:
  replicas: 1
  selector:
    matchLabels:
      app: test-pod
  template:
    metadata:
      creationTimestamp: null
      labels:
        app: test-pod
    spec:
      containers:
      - image: ubuntu
        name: ubuntu
        command: ["sleep","10000"]
        volumeMounts:
        - name: test-volume
          mountPath: /aws_config
        - name: script-volume
          mountPath: /scripts
      volumes:
      - name: test-volume
        configMap:
          name: awsconfig-configmap
      - name: script-volume
        configMap:
          name: script-configmap
      serviceAccountName: ci-serviceaccount
```


1. test


```bash
# Apply the pod manifest file to create a test container pod 
kubectl apply -f test-deployment.yaml


# Log in to the pod’s bash terminal.
POD_NAME=$(kubectl get pod \
            -l app=test-pod \
            -n ci-namespace \
            -o jsonpath='{.items[0].metadata.name}')
kubectl exec $POD_NAME -it -n ci-namespace -- bash


# Execute the script mounted to the pod to install the required binaries and libraries.
sh /scripts/script.sh


# Verify if the pod is able to assume both the ci-env and target-env roles by issuing the below calls.
aws sts get-caller-identity --profile ci-env
aws sts get-caller-identity --profile target-env



# The pod now has basic list permissions on the AWS resources as defined in the previous step. 
# Below command should output the list of EKS clusters in the target account.
aws eks list-clusters --region <AWS_REGION> --profile target-env



# You can also run a DescribeCluster command to describe the contents of any cluster.

aws eks describe-cluster \
    --name <TARGET_EKS_CLUSTER_NAME> \
    --region <AWS_REGION> \
    --profile target-env 
```




1. Configuring target account’s EKS cluster – Modify the aws-auth configmap

For the CI account cluster pod to access and manage the target cluster’s resources, you must edit the aws-auth configmap of the cluster in the target account by adding the role to the system:masters group. Below is how the configmap should look after the changes.

```yaml            
  mapRoles: |
. . .
    - groups:
      - system:masters
      rolearn: arn:aws:iam::TARGET_ACCOUNT_ID:role/target-account-iam-role
      username: test-user
```




1. Test the access to EKS clusters in the target accounts

```bash
# In the pod created from step 5, update the kubeconfig to test the access to the target account’s EKS cluster.
aws eks update-kubeconfig \
    --name <TARGET_EKS_CLUSTER_NAME> \
    --region <AWS_REGION>  \
    --profile target-env



# The pod should now be able to access the target cluster’s kube resources. Verify by issuing some sample kubectl get calls to access the target account’s EKS resources.
kubectl get namespaces     

kubectl get pods -n kube-system
```

