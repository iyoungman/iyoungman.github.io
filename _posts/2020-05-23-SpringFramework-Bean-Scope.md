---
layout: post
title: 스프링 프레임워크 핵심 기술 - 빈의 스코프
tags: Spring Scope
categories: Spring
---

* TOC
{:toc}

## Summary
* Spring 빈의 Scope에 대해 알아본다. 
* Singleton Scope 빈에서 Prototype 빈을 참조하는 법을 알아본다.

<!--more-->
  
<br>  

## Scope  
* Singleton
    * 스프링 컨테이너 전역에서 하나의 인스턴스만 사용.  
* Prototype
    * 해당 빈을 가져올때마다 새로운 인스턴스 생성.
    * 즉, 해당 빈을 의존성 주입할때마다 새 인스턴스가 생성된다.
* Request
* Session  
<br>
* 기본적으로 Singleton Scope로 빈이 생성된다.  

<br>

## Prototype과 Singleton
* Prototype 빈이 Singleton 빈을 참조
    * 문제 없다.
* Singleton 빈이 Prototype 빈을 참조
    * **하나의 Prototype 빈만 가져온다.**  

<br>  

* 예제 코드  

```java
@Component
@Scope("prototype")
public class ProtoTypeBean {

}

@Component
public class SingletonBean {

    @Autowired
    private ProtoTypeBean protoTypeBean;

    public ProtoTypeBean getProtoTypeBean() {
        return protoTypeBean;
    }
}

@Slf4j
@Component
public class Runner implements ApplicationRunner {

    @Autowired
    private SingletonBean singleToneBean;

    @Autowired
    private SingletonBean singleToneBean2;

    @Override
    public void run(ApplicationArguments args) throws Exception {
        log.info("prototype of singleton 1 : {}", singleToneBean.getProtoTypeBean());
        log.info("prototype of singleton 2 : {}", singleToneBean2.getProtoTypeBean());
    }
}
```

```java
prototype of singleton 1 : com.iyoungman.spring.core.scope.ProtoTypeBean@17ba57f0
prototype of singleton 2 : com.iyoungman.spring.core.scope.ProtoTypeBean@17ba57f0
```

> Log를 확인해보면 같은 Prototype 인스턴스를 가져온다.

<br>

## Sigleton에서 Prototype 답게 참조하려면?  
* **ProxyMode** 설정만 바꿔주면된다.
* 나머지 코드는 위와 동일하다.

```java
@Component
@Scope(value = "prototype", proxyMode = ScopedProxyMode.TARGET_CLASS)
public class ProtoTypeBean {

}
```

```java
prototype of singleton 1 : com.iyoungman.spring.core.scope.ProtoTypeBean@53ed80d3
prototype of singleton 2 : com.iyoungman.spring.core.scope.ProtoTypeBean@48e8c32a
```

> Log를 확인해보면 다른 Prototype 인스턴스를 가져온다.


<br>  

### Why Proxy?
* ScopedProxyMode를 설정해주지 않으면 Proxy가 생성되지 않는다.
* ScopedProxyMode.TARGET_CLASS
    * 해당 Class를 Proxy로 감싼다.
    * Class 기반 Proxy를 가능하게 해주는 **cglib**를 이용한다.
    * Sigleton이 Proxy가 아닌 Prototype을 직접 참조하면
    * 주입시마다 해당 빈을 바꿔줄 수 없을 것이다.  
  
![image](https://user-images.githubusercontent.com/25604495/82722857-51250780-9d05-11ea-86e0-942202e5a596.png)  

<br>  

## Reference
* [https://www.inflearn.com/course/spring-framework_core/lecture/15510](https://www.inflearn.com/course/spring-framework_core/lecture/15510)
