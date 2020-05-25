---
layout: post
title: 스프링 프레임워크 핵심 기술 - Environment 1부. 프로파일
tags: Spring Environment Profile
categories: Spring
---

* TOC
{:toc}
> 강의 내용을 바탕으로 정리.  
>
> Spring의 환경별 설정과 관련된 Profile에 대해 알아본다.

## Environment
* Profile 과 Property를 다루는 인터페이스.

```java
public interface ApplicationContext extends EnvironmentCapable {

}

public interface EnvironmentCapable {

	Environment getEnvironment();
}
```

<br>  

## Profile 이란?
* Bean들의 그룹.
* 업무 환경에서 다양한 개발 환경별 설정이 필요하다.  

> test, alpha, beta, real..

<br>  

## Profile 설정  
> 아래 3가지 중 하나로 하면 된다.  

<br>  

1) Active profiles  

![2](https://user-images.githubusercontent.com/25604495/82731877-b9470e00-9d44-11ea-8772-6de83e3c87a4.PNG)

<br>  

2) VM options  

![1](https://user-images.githubusercontent.com/25604495/82731878-ba783b00-9d44-11ea-8feb-4a21e82dbaef.PNG)

<br>  

3) application.properties 파일  

```properties
spring.profiles.active=test
```

<br>  

## Profile에 따른 Bean 설정
* **@Profile** 어노테이션을 이용한다.

<br>  

* 'test' Profile 환경에서만 적용되는 특정 Bean을 사용하고 싶다.

```java
public class TestBookRepository {

}

@Configuration
@Profile("test")
public class TestConfiguration {

    @Bean
    public TestBookRepository testBookRepository() {
        return new TestBookRepository();
    }
}

@Slf4j
@Component
public class EnvironmentRunner implements ApplicationRunner {

    @Autowired
    private ApplicationContext applicationContext;

    @Autowired
    private TestBookRepository testBookRepository;

    @Override
    public void run(ApplicationArguments args) throws Exception {
        Environment environment = applicationContext.getEnvironment();
        log.info("Profile : {}", environment.getActiveProfiles());//확인
    }
}
```

```java
"Profile : test"
```

> 'test' Profile로 설정되어있지 않은경우, TestBookRepository를 주입 받을 수 없다.
  
<br>  

## Reference
* [https://www.inflearn.com/course/spring-framework_core/lecture/15511](https://www.inflearn.com/course/spring-framework_core/lecture/15511)  