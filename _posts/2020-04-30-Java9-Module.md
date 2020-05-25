---
layout: post
title: Java9. 모듈 시스템 알아보기
tags: Java9 Module ModernJavaInAction
categories: Java
---  

* TOC
{:toc}
> 모듈을 사용하는 주요 동기를 이해한다.
>
> 모듈을 어떻게 사용할지 살펴본다.
  
<br>  

## 왜 모듈을 만들었을까?

1) 관심사 분리
* SoC(Separation of Concerns)
* 각각의 부분을 기능 중심으로 분리한다.
* 이를 클래스를 그룹화한 모듈을 이용한다.  

> 회계 애플리케이션은 개발한다고 가정하자.
>
> SoC를 적용하여 파싱, 분석, 레포트 기능을 각각의 모듈로 분리한다.

<br>  

* <U>패키지로 클래스를 그룹화 할 수 있지 않나?</U>
* 자바9 모듈은 클래스가 어떤 다른 클래스를 볼 수 있는지를
* **컴파일 시간에 제어할 수 있다.**
* 패키지는 모듈성을 지원하지 않는다.

<br>  

2) 정보 은닉
* 세부 구현을 숨겨 어떤 부분이 변경되었을 떄
* 다른 부분까지 미칠 영향을 줄일 수 있다.
* 자바에서는 이러한 캡슐화를 pivate 접근제어자로 제공한다.
* 하지만 **자바9 이전에는 클래스와 패키지가 의도된 대로 공개되었는지** 알 수 없다.

<br>  

## 자바 모듈 시스템을 설계한 이유

### 자바9 이전 모듈화의 한계
1) 제한된 가시성 제어
* 자바는 클래스, 패키지 등의 코드 그룹화를 제공한다.
* 클래스는 가시정을 제어를 위해 접근 제어자를 이용한다.  

> public, protected, default, private  

<br>
 
* 하지만, 패키지간의 가시성 제어는 없다.


<br>

2) 클래스 경로
  
<br>

## 자바 모듈 사용해보기
> 간단하게 구현해보며 개념을 정리한다.
>
> 자바 모듈과 빌드툴인 Maven을 조합해서 사용해볼것이다.
>
> Maven을 통해 빌드, 외부 종속성 등을 주입받는다.
>
> 그리고 자바의 모듈을 통해 패키지를 캡슐화 할것이다.


<br>  

1) 테스트 할 2개의 모듈을 module1, module2 라고 정의한다.

***

2) 두 패지키를 모듈로 등록한다.
* Project Structure -> Project Settings -> Modules -> `+` -> New Module Click

<img src = "https://user-images.githubusercontent.com/25604495/80907604-64226880-8d53-11ea-8ef9-f7f9dc64cf3a.png" width="600" height="250" />
   
<br>  
<br>  

* Maven 선택 후 GroupId, ArtifactId 지정

<img src = "https://user-images.githubusercontent.com/25604495/80907551-232a5400-8d53-11ea-99a4-31dac2009a84.png" width="600" height="250" />

<img src = "https://user-images.githubusercontent.com/25604495/80907569-405f2280-8d53-11ea-9ecc-5298a24b2e9a.png" width="600" height="250" />  

<br>  
<br>  
<br>  

* module1, module2가 모듈로 등록되었다.

<img src = "https://user-images.githubusercontent.com/25604495/80907588-510f9880-8d53-11ea-95a8-7bbf40dd8345.png" width="400" height="200" />  

<br>  
<br>  
<br>  

***


3) 간단한 예제 코드를 작성한다.
* Module1의 구조와 코드  

<img src = "https://user-images.githubusercontent.com/25604495/80907823-f8d99600-8d54-11ea-9b7d-c43dd4c8d6d4.png" width="400" height="300" />

```java
public class Module1Api {

    private Module1Service module1Service = new Module1Service();

    public void method() {
        module1Service.method();
    }

}

public class Module1Service {

    public void method() {
        System.out.println("Hello Im Module1");
    }
}
```
  
<br>  


* Module2의 구조와 코드   

<img src = "https://user-images.githubusercontent.com/25604495/80907777-9a141c80-8d54-11ea-8ba5-95a47821b1cb.png" width="400" height="300" />  

```java
public class Main {

    public static void main(String[] args) {
        //Module1에 속해있는 Module1Api을 가져오기 위해 모듈 디스크립터 정의가 필요하다.
        Module1Api api = new Module1Api();
        api.method();
    }

}
```

  
<br>  

***



4) 모듈 디스크립터를 정의한다.
* 모듈 디스크립터는 크게 두가지 역할을 한다.
* **자신 모듈에 속한 패키지 가시성을 제어하거나**
* **다른 모듈을 가져온다.**

> 예제에서는 Module2에서 Module1의 기능을 가져와서 사용할 것이다.
>
> 다만, Module1에서는 외부 인터페이스 역할인 api 패키지만 외부로 노출하고
>
> 내부 구현인 service 패키지를 숨기고 싶다고 가정한다.
  
<br>  

* 일단 Module1와 Module2에 각각 모듈 디스크립터인 **module-info.java**를 추가한다.
* 각 모듈의 Root에 위치해야한다.  

> 3)의 그림을 참조하자.  

```java
//module1의 module-info.java
module module1 {
    exports com.iyoungman.module1.api;
}

//module2의 module-info.java
module module2 {
    requires module1;
}
```

> 다양한 모듈 구문이 있지만 크게 `exports`와 `requires`가 있다
> 
> exports 패키지명 : 외부로 노출할 패키지를 정의한다.
>
> requires 모듈명 : 가져올 모듈을 정의한다.
>
> exports는 패키지명, requires는 모듈명이라는 것을 주의하자.


  
<br>  

***

5) 중간 확인  
  
* 이제 Module2에서 Module1을 가져와 사용할 수 있다.
* 또한, Module1에서 정의한대로 패키지 가시성을 제어한다는 것을 확인할 수 있다.
* Module1의 api 패키지에 있는 Module1Api는 사용할 수 있지만
* Module1의 service 패키지에 있는 Module1Service는 사용할 수 없다.  

<img src = "https://user-images.githubusercontent.com/25604495/80679593-1f4ec580-8af8-11ea-919f-f1e905e5b40c.png" width="600" height="250" />  

<br>  

***

6) 외부 종속성 주입
* 만약 외부 라이브러리를 주입받고 싶다면
* Maven/Gradle 등과 함께 사용하면 된다.
* pom.xml에 필요한 의존성 기입 후 
* module-info.java에서 **require** 해준다.

<br>  

* Module2의 pom.xml  

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
        xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.iyoungman</groupId>
    <artifactId>module2</artifactId>
    <version>1.0-SNAPSHOT</version>

    <dependencies>
        <dependency>
            <groupId>com.iyoungman</groupId>
            <artifactId>module1</artifactId>
            <version>1.0-SNAPSHOT</version>
        </dependency>

         <!-- 외부 라이브러리 -->
        <dependency>
            <groupId>org.apache.httpcomponents</groupId>
            <artifactId>httpclient</artifactId>
            <version>4.5.12</version>
        </dependency>
    </dependencies>

    <properties>
        <maven.compiler.target>11</maven.compiler.target>
        <maven.compiler.source>11</maven.compiler.source>
    </properties>


</project>
```  
  
<br>  

* Module2의 module-info.java  

```java
module module2 {
    requires module1;

    //추가
    requires org.apache.httpcomponents.httpclient;
}
```



<br>  

## 모듈 정의와 구문들
* requires  

> 다른 모듈을 가져올 때
>
> requires 모듈;

<br>  

* exports  

> 다른 모듈로 패키지를 내보낼 때
>
> exports 패키지;

<br>  

* requires transitive  

> requires를 전이한다.

<br>  

* exports to  

> 사용자에게 공개할 기능을 제한한다.

  
<br>  

## Code  
* [https://github.com/iyoungman/java-practice](https://github.com/iyoungman/java-practice)  

  
<br>  

## Reference
* [모던 자바 인 액션](http://www.yes24.com/Product/Goods/77125987?Acode=101)
* [https://www.baeldung.com/java-9-modularity](https://www.baeldung.com/java-9-modularity)
* [https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-9-jigsaw](https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-9-jigsaw)  