---
title: Dockerfile 모범 사례 소개
author: Jongin Kim
date: 2020-10-01 00:00:00 +0900
categories: [docker]
tags: [docker]
---
## 빌드시간

### 캐싱을 위한 순서문제
![](https://images.velog.io/images/manofbell/post/73b2a80a-2f79-4adf-81c9-990f73d1182a/Untitled.png)
- 가장 적게 변경되는 부분 부터 → 가장 자주 변경되는 단계의 순서대로 배치

### 필요한 것만 복사하기
![](https://images.velog.io/images/manofbell/post/26512851-c925-4549-ac59-e6ad2d734c4f/Untitled%20(1).png)
- 관련있는 파일만 캐시가 되도록 복사할 대상이 특정되어야 한다.
- 위 사진에서 우리가 필요한 파일은 jar파을 하나인데 빨간색 `copy . /app` 코드처럼 모든 파일을 복사하는 경우 현재 위치에 있는 파일이 하나라도 변경되는 경우 캐시가 깨지기 때문에 비효율 적이다. 
- 그러므로 꼭 필요한 파일만 COPY 한다.

### apt-get update & install 같은 캐시가능한 단위를 확인하자
![](https://images.velog.io/images/manofbell/post/02b3f5ca-0da8-41ae-827b-752b0095a670/Untitled%20(2).png)
- RUN 은 캐시가 가능한 실행 단위이다. 
- 너무 많은 명령어가 불필요할 수 있으며, 모든 명령어들을 하나의 RUN 명령으로 체이닝하면 쉽게 캐시가 파손된다. 
- 고로 효율적인 캐시를 하기위해 중요한점은 한 RUN 명령어 단위에 캐시를 태우는 점 + 항상 동일한 패키지를 설치하는 점 이라고 할 수 있다. 즉, 위 도커파일 처럼 패키지 매니저에서 패키지를 설치할 때 항상 패키지 인덱스 정보를 업데이트 하고 ( ex. apt-get update ) 동일한 RUN 실행 단위에서 패키지를 설치하는게 좋다.
- 이렇게하면 캐시가능한 하나의 유닛이 될 뿐더러 오래된 패키지를 설치할 위험도 사라진다.

## 이미지 크기 축소

### 불필요한 의존성 제거
![](https://images.velog.io/images/manofbell/post/3b4db4f7-b7ef-4d11-af5f-6f2c0171b590/Untitled%20(3).png)
- 가끔 우리가 개발하다 보면 도커 컨테이너에 들어가서 소스를 확인하고 싶은 경우가 종종 생기는데 이런경우가 잦아지다 보니 그냥 아예 이미지에 `vim` 을 설치해버리는 경우가 종종 있다.
- 이런 디버깅을 위해 필요한 의존성은 언제든지 컨테이너에서 설치할 수 있으니 이런 의존성은 도커파일에서 제거하자
- 또 Apt에는 실제로 필요하지 않은 추천 종속성이 설치되지 않도록하는`--no-install-recommends`플래그가 있고. 필요한 경우 명시 적으로 추가하는게 좋다.

### 패키지 매니저 캐시 제거
![](https://images.velog.io/images/manofbell/post/0963dab1-f103-4d8f-9025-fb881a024d51/Untitled%20(4).png)
- 패키지 관리자는 자체 캐시를 관리한다.
- 도커파일에 캐시가 있기 때문에 큰 의미가 없을 뿐 더러 이미지 크기만 잡아먹는다. 그래서 지워버린다.
- 하지만 여기서 주의해야 할 점은 꼭 같은 RUN 실행 단위에서 지웠을 때 이미지 크기가 줄어든다. 다른 RUN 에서 지워봐야 이미지 크기가 줄어들지 않는다.

## 유지보수

### 가능한 공식 이미지를 사용해라
![](https://images.velog.io/images/manofbell/post/2d69964e-cb12-4b99-b8c7-d6f050a4da7e/Untitled%20(5).png)
- 당연한 이야기 일 수 있지만 ..

### 구체적인 이미지 태그를 사용해라
![](https://images.velog.io/images/manofbell/post/6abeb6ed-163f-4616-aef7-5ccbc33bf815/Untitled%20(6).png)
- 이것도 당연한 이야기 일 수 있지만..
- 시간이 지남에 따라 이미지가 변경된다면.. 빌드에 실패하는 경우가 생기니 구체적인 이미지 태그를 사용하는게 좋다.
- 실무에서 수많은 방식의 도커 이미지의 tag를 사용해봤지만 역시 누가봐도 확실한게 제일 좋다.

### 가능한 최소한의 flavors를 찾아라
![](https://images.velog.io/images/manofbell/post/471bd50b-4afd-43d6-be40-7a84eda46687/Untitled%20(7).png)
- 제일 크기가 작은 알파인을 사용하는 것이 좋겠지만...
- 알파인은 musl libc를 사용하는데, 비록 훨씬 작지만 호환성 문제가 발생할 수 있으니 주의하자.
- 추가로 openjdk의 경우 jre flavor가 sdk가 아닌 Java 런타임 만 포함해 이미지 크기가 크게 줄어 드니 이점도 참고하자.

> 원문, 참고 - [https://www.docker.com/blog/intro-guide-to-dockerfile-best-practices/](https://www.docker.com/blog/intro-guide-to-dockerfile-best-practices/)