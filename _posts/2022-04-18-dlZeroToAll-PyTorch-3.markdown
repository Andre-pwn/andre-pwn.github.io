---
title: "모두�? ?��?�� ?��?��?�� 2 - Lab3: Minimizing Cost"
author: Kwon
date: 2022-04-18T14:00:00+0900
categories: [pytorch, study]
tags: [linear-regressoin, cost, gradient-descent]
math: true
mermaid: false
---

[모두�? ?��?�� ?��?��?��](https://deeplearningzerotoall.github.io/season2/lec_pytorch.html) Lab 3: Minimizing Cost 강의�? �? ?�� 공�??�? 목적?���? ?��?��?�� 게시물입?��?��.

***
## Theoretical Overview
?��번에?�� Grdient descent?�� ????�� 조금 ?�� 집중?��?���? ?��?��보기 ?��?�� Hypothesis�? 조금 ?�� ?��?��?���? $ H(x) = Wx $�? 바꾸?�� ?��?��보자.

cost?�� ?��같이 MSE(Mean Square Error)�? ?��?��?���? ?��?��?��?�� ?��?���? 같�?? 공�?? ?���? - ?��?�� ?��?��?���? ?��?��?��?��. ([Lab2 ?��?��?�� 참조](http://qja1998.github.io/2022/04/17/dlZeroToAll-PyTorch-2/))

\\[ MSE = cost(W) = \frac{1}{m} \sum^m_{i=1} \left( H(x^{(i)}) - y^{(i)} \right)^2 \\]

***

## Import
{% highlight python %}
import matplotlib.pyplot as plt
import numpy as np
import torch
import torch.nn as nn
import torch.nn.functional as F
import torch.optim as optim
{% endhighlight %}

***
## Cost by W
W?�� �??��?�� ?���? cost 그래?���? 그려보면 ?��?���? 같�?? 2�? 곡선?�� 그려진다. 그러�?�? cost�? �??�� ?��??? ?��??? 기울�?(미분�?)�? 0?�� 극소?��?��?��.

{% highlight python %}
W_l = np.linspace(-5, 7, 1000)
cost_l = []
for W in W_l:
    hypothesis = W * x_train
    cost = torch.mean((hypothesis - y_train) ** 2)

    cost_l.append(cost.item())

plt.plot(W_l, cost_l)
plt.xlabel('$W$')
plt.ylabel('Cost')
plt.show()
{% endhighlight %}
![](/posting_imgs/images/lab3-1.png)
<br>

***
## Gradient Descent by Hand
cost�? �??�� ?��??? ?��?�� 찾는 것이 ?��리의 목표?��?��, ?��것을 cost?�� 미분값을 ?��?��?��?�� 방식?���? ?��?��?��?���? ?��?��.

cost?�� ?��?���? 같으�?�?
<br><br>
\\[ MSE = cost(W) = \frac{1}{m} \sum^m_{i=1} \left( H(x^{(i)}) - y^{(i)} \right)^2 \\]

$W$?�� ????�� 미분?���? ?��?��??? 같�?? 결과�? ?��?�� ?�� ?��?��.

\\[ \nabla W = \frac{\partial cost}{\partial W} = \frac{2}{m} \sum^m_{i=1} \left( Wx^{(i)} - y^{(i)} \right)x^{(i)} \\]

?��?���? 구한 gradient?�� ?��?�� ?���? 같이 ?��?��?�� ?��?��?���? ?��?��.

\\[ W := W - \alpha \nabla W \,\,\left(\alpha = learning\,\,rate\right)\\]

?��?�� ?��?���? ?��?��?�� ?��?�� ?��?���? ?���? ?��?��보자.
<br><br>
?��?��?�� ?�� gif?�� 각각 극소?��?�� 좌우?��?�� 극소?��?�� ?��근할 ?�� ?��?��?�� �??���? ?��????�� 것이?��.
<br><br>
![](/posting_imgs/images/lab3-2.gif)
<br><br>
먼�?? ?��쪽에?�� ?��근하?�� 경우 기울�?(gradient)�? ?��?��?���? 극소?��?���? ?��?��?���? ?��?��?��?�� $W$�? 커져?�� ?��?��. 그러�?�? ?��?��?�� 기울기�?? 빼주?�� 극소?��?�� �?깝게 ?��?��?�� ?�� ?��?��.
<br><br>
![](/posting_imgs/images/lab3-3.gif)
<br><br>
?��?��?���? ?��른쪽?��?�� ?��근하?�� 경우 기울기�?? ?��?��?���? 극소?��?���? ?��?��?���? ?��?��?��?�� $W$�? ?��?��?��?�� ?��?��. ?�� ?��?�� ?��?��?�� 기울기�?? 빼주?�� 극소?��?�� �?깝게 ?��?��?�� ?�� ?��?��.

결국 ?�� ?��?�� 빼야?���?�? 모두 만족?��?�� ?��?�� $ W := W - \alpha \nabla W $, 기울기의 뺄셈?���? 주어�??�� 것이?��. ?�� ?�� $learning\,\,rate$?�� $\alpha$?�� �? 그�??�? ?��?���?(?�� 번에 ?��?��?�� ?��마나 ?�� 것인�?)?�� ?��????��?�� 것이�?�? ?��?��?�� 맞게 최적?�� ?��?�� ?��?��?��?��.

?���?, ?��?��률이 ?���? ?��?���? ?��?��?�� ?��?���?�?, ?���? ?���? 진동?���? 발산?�� 버리�? ?��문에 ?��?��?�� 범위?�� 값을 ?��?��?��?�� ?��?��.

![](/posting_imgs/images/lab3-4.jpg)

?��?��?�� ?��?�� ?��?��?�� 코드�? ?��?��?���? ?��?���? 같다.

\\[ \nabla W = \frac{\partial cost}{\partial W} = \frac{2}{m} \sum^m_{i=1} \left( Wx^{(i)} - y^{(i)} \right)x^{(i)} \\]

{% highlight python %}
gradient = torch.sum((W * x_train - y_train) * x_train)
print(gradient)

''' output
tensor(-14.)
'''
{% endhighlight %}

\\[ W := W - \alpha \nabla W \,\,\left(\alpha = learning\,\,rate\right)\\]

{% highlight python %}
lr = 0.1
W -= lr * gradient
print(W)

''' output
tensor(1.4000)
'''
{% endhighlight %}

***
## Training
?��?�� 구현?��?�� 것들?�� ?��?��?��?�� ?��?���? ?��?��?��?�� 코드�? ?��?��?�� 보면 ?��?���? 같다.

{% highlight python %}
# ?��?��?��
x_train = torch.FloatTensor([[1], [2], [3]])
y_train = torch.FloatTensor([[1], [2], [3]])
# 모델 초기?��
W = torch.zeros(1)
# learning rate ?��?��
lr = 0.1

nb_epochs = 10
for epoch in range(nb_epochs + 1):
    
    # H(x) 계산
    hypothesis = x_train * W
    
    # cost gradient 계산
    cost = torch.mean((hypothesis - y_train) ** 2)
    gradient = torch.sum((W * x_train - y_train) * x_train)

    print('Epoch {:4d}/{} W: {:.3f}, Cost: {:.6f}'.format(
        epoch, nb_epochs, W.item(), cost.item()
    ))

    # gradient�? H(x) 개선
    W -= lr * gradient

''' output
Epoch    0/10 W: 0.000, Cost: 4.666667
Epoch    1/10 W: 1.400, Cost: 0.746666
Epoch    2/10 W: 0.840, Cost: 0.119467
Epoch    3/10 W: 1.064, Cost: 0.019115
Epoch    4/10 W: 0.974, Cost: 0.003058
Epoch    5/10 W: 1.010, Cost: 0.000489
Epoch    6/10 W: 0.996, Cost: 0.000078
Epoch    7/10 W: 1.002, Cost: 0.000013
Epoch    8/10 W: 0.999, Cost: 0.000002
Epoch    9/10 W: 1.000, Cost: 0.000000
Epoch   10/10 W: 1.000, Cost: 0.000000
'''
{% endhighlight %}

**Hypothesis output 계산 -> cost??? gradient 계산 -> gradient�? hypothesis(weight) 개선**

?��??? 같�?? ?��?���? �? 10 epoch ?��?��?��?�� 코드?��?��. ?��?��?�� ?���? ?�� ?��마다 cost�? 줄어?���?, ?��리�?? ?��각한 ?��?��?��?�� $W$?�� 1?�� ?��?�� �?까워�??�� 것을 ?��?��?�� ?�� ?��?��.

***
## Training with `optim`

**Training**?��?�� ?��?�� 것처?�� ?��리�?? gradient�? 계산?��?�� 코드�? 직접 ?��?��?��?�� ?��?��?�� ?��?�� ?���?�? PyTorch?��?�� ?��공하?�� `optim`?�� ?��?��?��?�� 간단?���? 구현?�� ?��?�� ?��?��.

{% highlight python %}
# ?��?��?��
x_train = torch.FloatTensor([[1], [2], [3]])
y_train = torch.FloatTensor([[1], [2], [3]])
# 모델 초기?��
W = torch.zeros(1, requires_grad=True)
# optimizer ?��?��
optimizer = optim.SGD([W], lr=0.15)

nb_epochs = 10
for epoch in range(nb_epochs + 1):
    
    # H(x) 계산
    hypothesis = x_train * W
    
    # cost 계산
    cost = torch.mean((hypothesis - y_train) ** 2)

    print('Epoch {:4d}/{} W: {:.3f} Cost: {:.6f}'.format(
        epoch, nb_epochs, W.item(), cost.item()
    ))

    # cost�? H(x) 개선
    optimizer.zero_grad()
    cost.backward()
    optimizer.step()

''' output
Epoch    0/10 W: 0.000 Cost: 4.666667
Epoch    1/10 W: 1.400 Cost: 0.746667
Epoch    2/10 W: 0.840 Cost: 0.119467
Epoch    3/10 W: 1.064 Cost: 0.019115
Epoch    4/10 W: 0.974 Cost: 0.003058
Epoch    5/10 W: 1.010 Cost: 0.000489
Epoch    6/10 W: 0.996 Cost: 0.000078
Epoch    7/10 W: 1.002 Cost: 0.000013
Epoch    8/10 W: 0.999 Cost: 0.000002
Epoch    9/10 W: 1.000 Cost: 0.000000
Epoch   10/10 W: 1.000 Cost: 0.000000
'''
{% endhighlight %}

`optim.SGD`�? ?��리�?? 만들?��?�� 구현?��?�� gradient?�� ????�� 처리�? ?��주고 ?��?�� 것을 �? ?�� ?��?��.
{% highlight python %}
optimizer.zero_grad() # gradient 초기?��
cost.backward()       # gradient 계산
optimizer.step()      # 계산?�� gradient�? ?��?�� W, b�? 개선
{% endhighlight %}
???�? 강의?��?��?�� ?��?��?��?�� ?�� 3개의 묶음 코드�? ?��?�� gradient?�� ????�� 계산�? 그에 ?���? ?��?��?�� ?��루어�?�? ?��?��.

?�� ?��?�� 마찬�?�?�? $W$??? cost�? 보면 ?�� ?��?��?�� ?��?�� 것을 �? ?�� ?��?��.