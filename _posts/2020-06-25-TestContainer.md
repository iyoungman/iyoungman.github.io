---
layout: post
title: TestContainers
tags: TestContainers Docker Test
categories: Spring
---
* TOC
{:toc}
> 객체 어댑터 패턴을 알아본다.
  
<br>  

## TestContainers 소개
* 테스트 코드를 이용해 도커 컨테이너를 실행하도록 하는 라이브러리.

> 왜 사용할까?

* 테스트에서는 운영 환경과 **동일한** 개발 환경을 테스트하는것이 중요하다.
* DB를 예를 들면 테스트 환경에서 H2를 사용하고, 운영 환경에서 Postgresql을 사용할 수 있겠지만<br>
* 하지만 H2와 Postgresql은 분명히 차이점이 있다.  

<br>

* 도커를 이용하면 쉽게 운영 환경과 동일한 환경을 쉽게 구축할 수 있는데
* 사용자가 직접 도커 컨테이너를 띄우고 설정하고 삭제하는 것이 아니라<br>
**테스트 코드 내에서 컨테이너를 띄우고 제거**하는 기능을 TestContainers가 제공해준다.

<br>  

## TestContainers 설치

> [1] 예제는 JUnit5에서 TestContainers를 실행할 것이다.
>
> [https://www.testcontainers.org/test_framework_integration/junit_5/](https://www.testcontainers.org/test_framework_integration/junit_5/

> [2] 또한 여러 모듈중에서 Database/Postgresql 모듈을 사용해볼 것이다.
>
> [https://www.testcontainers.org/modules/databases/Postgresql/](https://www.testcontainers.org/modules/databases/Postgresql/)

<br>  

1) TestContainers 의존성 추가

```gradle
testCompile "org.testcontainers:Postgresql:1.14.3"
```

<br>  

2) Postgresql 의존성 추가

```gradle
testCompile "org.testcontainers:Postgresql:1.14.3"
```

3) JDBC 연결 설정


4) 테스트 컨테이너 공유








```gradle
testCompile "org.testcontainers:Postgresql:1.14.3"
```



<br>  

## Reference
