---
layout: post
title: 스프링 프레임워크 핵심 기술. @Autowired
tags: Spring @Autowired
categories: Spring
---

* TOC
{:toc}
## @Autowired
* 필요한 의존 객체의 **타입**에 해당하는 빈을 찾아 주입한다.
* 선언한 빈을 못 찾으면 애플리케이션 구동 안된다.   

```java
boolean required() default true;
```
  
<br>

## 사용할 수 있는 위치
1) 생성자
> 스프링 4.3 부터는 생략 가능


2) 세터

3) 필드

<br>
 

## 경우의 수
1) 해당 타입의 빈이 없는 경우
* 구동이 안된다.
* 굳이 구동시키고 싶다면 required = false.

<br>
 
2) 해당 타입의 빈이 한 개인 경우
* 구동 된다.

<br>
 
3) 해당 타입의 빈이 여러 개인 경우

```java
public interface AllService {
}

@Service
public class AService implements AllService {
}

@Service
public class BService implements AllService {
}

@Service
public class OtherService {

    //찾을 수 없다.
    @Autowired
    private AllService allService;
}
```

* 구동이 안된다.

***

> `해결 방법1. @Primary`
>
> 우선 순위를 정한다.
>
> AService 혹은 BService에 선언한다.

***

> `해결 방법2. @Qualifier`
>
> 빈의 이름을 기입한다.
>
> 기본적으로 빈의 camelCase가 이름이다.

```java
@Service
public class OtherService {

    @Autowired @Qualifier("aService")
    private AllService allService;
}
```


<br>
 
## 동작 원리
* BeanPostProcessor라는 LifeCycle Interface의 구현체에 의해 동작.

<br>
 
### BeanPostProcessor
* 빈의 인스턴스를 만드는 LifeCycle이 있다.
* BeanPostProcessor는 `빈의 initialization Life Cycle 이전과 이후`에 또 다른 부가적인 작업을 할 수 있는 LifeCycle CallBack.

<br>

1) @PostConstuct  

```java
@Service
public class OtherService implements InitializingBean {

    @Autowired
    private AllService allService;

    @PostConstruct
    public void setUp() {
        //do...
    }
}
```

<br>

2) InitializingBean

```java
@Slf4j
@Service
public class OtherService implements InitializingBean {

    @Autowired
    private AllService allService;

    @Override
    public void afterPropertiesSet() throws Exception {
        //do...
    }
}

```

<br>

***

### AutowiredAnnotationBeanPostProcessor
* BeanPostProcessor를 상속받고 있다.
![1_상속구조](https://user-images.githubusercontent.com/25604495/82553370-d93ccd00-9b9e-11ea-828e-7c07cc2a4688.PNG)

<br>


* 다음은 빈 초기화 LifeCycle 일부이다.
![image](https://user-images.githubusercontent.com/25604495/82553608-536d5180-9b9f-11ea-9ae7-cfda63fbe667.png)  

<br>
 
* BeanPostPorcessor는 이중 두가지 메서드를 제공한다.(11, 14)
![image](https://user-images.githubusercontent.com/25604495/82553138-70555500-9b9e-11ea-84b2-009c3889dcc0.png)  

<br>
 
* BeanPostProcessor의 postProcessBeforeInitialization() LifeCycle에 
* AutowiredAnnotationBeanPostProcessor가 동작해서@Autowired 라는 Annotation을 찾아 해당 타입의 Bean을 주입해 준다.

<br>


## Next
* 추후에 AutowiredAnnotationBeanPostProcessor가 동작을 더 자세하게 알아봐야겠다.

<br>

## Reference
* https://www.inflearn.com/course/spring-framework_core/lecture/15508
* https://javaslave.tistory.com/48
