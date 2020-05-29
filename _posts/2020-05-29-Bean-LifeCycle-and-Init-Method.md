---
layout: post
title: Bean LifeCycle을 통한 초기화 메서드가 필요한 이유
tags: Spring Bean LifeCycle DI
categories: Spring
---
* TOC
{:toc}
> Bean LifeCycle과 DI를 알아본다.
>
> 초기화 메서드가 필요한 이유를 알아본다.

<br>    

## Bean의 LifeCycle  
![image](https://user-images.githubusercontent.com/25604495/82725638-38bee800-9d19-11ea-8d2e-7c3f374d0145.png)  

<br>    

### Constructor

> 생성자 주입

* 해당 LifeCycle에 일어난다.

```java
@Service
public class HelloService {

    private ByeService byeService;

    public HelloService(ByeService byeService) {
        this.byeService = byeService;
    }
}
```

***  

### Setter Methods

> Setter Method 주입

* 해당 LifeCycle에 일어난다.

```java
@Service
public class HelloService {

    private ByeService byeService;

    @Autowired
    public void setByeService(ByeService byeService) {
        this.byeService = byeService;
    }
}
```

***  


### BeanPostProcessor.postProcessBeforeInitialization()  

> @Autowired 주입

* 해당 LifeCycle에 일어난다.

```java
@Service
public class HelloService {

    @Autowired
    private ByeService byeService;
}
```

***

### 초기화 LifeCycle  
> 초기화 메서드란?
* **DI 작업을 마친 후 실행되는 메서드**

<br>  

> 아래 3가지 방법으로 Bean의 초기화를 할 수 있다.

1) @PostConstruct
2) InitializingBean.afterPropertiesSet()
3) init-method 

<br>  

> 의문 : 초기화 메서드의 역할은 무엇일까? 

* 일반적인 Java에서는 Consturctor에서 초기화를 진행한다.
* 하지만 Spring에서는 어떠한 Bean의 Consturctor 시점에<br>의존하는 Bean들이 주입되지 않는 경우가 있다.<br>
setter, field 주입을 사용하는 경우로 예를 들수 있다.
* 즉, 이시점에 의존 Bean을 사용할 수 없다.  

```java
@Service
public class HelloService {

    @Autowired
    private ByeService byeService;

    public HelloService() {
        System.out.println(byeService);//null
    }

    @PostConstruct
    public void init() {
        System.out.println(byeService);//not null
    }
}
```

* 따라서 @Autowired로 의존성 주입이 일어난 **BeanPostProcessor.postProcessBeforeInitialization() Lifecycle 이후에**
* 초기화 메서드를 실행한다.

<br>  

## Reference
* [https://www.concretepage.com/spring/spring-bean-life-cycle-tutorial](https://www.concretepage.com/spring/spring-bean-life-cycle-tutorial)  
* [https://whiteship.tistory.com/791](https://whiteship.tistory.com/791)  
* https://madplay.github.io/post/spring-bean-lifecycle-methods
* https://zorba91.tistory.com/223
* https://stackoverflow.com/questions/11380558/why-does-spring-dependency-injection-have-init-method