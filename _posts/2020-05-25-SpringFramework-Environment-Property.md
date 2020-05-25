---
layout: post
title: 스프링 프레임워크 핵심 기술 - Environment 2부. 프로퍼티
tags: Spring Environment Property
categories: Spring
---

* TOC
{:toc}
> 강의 내용을 바탕으로 정리.  
>
> Spring의 외부 설정과 관련된 Property에 대해 알아본다.
  
<br>  

## Property 란?
* 애플리케이션에서 사용하는 여러가지 설정 값들을<br>애플리케이션 밖이나 안에 정의하는 기능.

<br>  

## Property는 우선순위가 있다.
* 같은 설정값이라도 우선순위가 높은 설정이 적용된다.
* 아래는 [Spring Docs](https://docs.spring.io/spring-boot/docs/current/reference/html/spring-boot-features.html#boot-features-external-config)에 나온 우선순위이다.

```java
1. Devtools global settings properties in the $HOME/.config/spring-boot directory when devtools is active.

2. @TestPropertySource annotations on your tests.

3. properties attribute on your tests. Available on @SpringBootTest and the test annotations for testing a particular slice of your application.

4. Command line arguments.

5. Properties from SPRING_APPLICATION_JSON (inline JSON embedded in an environment variable or system property).

6. ServletConfig init parameters.

7. ServletContext init parameters.

8. JNDI attributes from java:comp/env.

9. Java System properties (System.getProperties()).

10. OS environment variables.

11. A RandomValuePropertySource that has properties only in random.*.

12. Profile-specific application properties outside of your packaged jar (application-{profile}.properties and YAML variants).

13. Profile-specific application properties packaged inside your jar (application-{profile}.properties and YAML variants).

14. Application properties outside of your packaged jar (application.properties and YAML variants).

15. Application properties packaged inside your jar (application.properties and YAML variants).

16. @PropertySource annotations on your @Configuration classes. Please note that such property sources are not added to the Environment until the application context is being refreshed. This is too late to configure certain properties such as logging.* and spring.main.* which are read before refresh begins.

17. Default properties (specified by setting SpringApplication.setDefaultProperties).
```

<br>  

***  

### 예제
* 위의 우선순위 중 **4, 9, 14** 방법을 이용해 우선순위 적용을 확인해보겠다.
* 3가지 방법 **동시에 동일한 설정 Key 값을 적용**하여 확인한다.
* 설정 Key 값은 아래와 같다.  

```java
app.name
```

<br>  

* 우선순위 4 - Command line arguments.  

```properties
--app.name="Command Line"
```

![캡처 - 복사본 (2)](https://user-images.githubusercontent.com/25604495/82790654-5fab2480-9ea7-11ea-985b-97b1209f23fc.PNG)

<br>  

* 우선순위 9 - Java System properties (System.getProperties()).  

```properties
-Dapp.name="VM Options"
```
![캡처 - 복사본](https://user-images.githubusercontent.com/25604495/82790659-633eab80-9ea7-11ea-9b30-5ddc808e8649.PNG)  

<br>  

* 우선순위 14 - application.properties.  

```properties
app.name="Properties File"
```

<br>  

***  

### 확인

* 코드

```java
@Slf4j
@Component
public class EnvironmentRunner implements ApplicationRunner {

    @Autowired
    private ApplicationContext applicationContext;

    @Override
    public void run(ApplicationArguments args) throws Exception {
        Environment environment = applicationContext.getEnvironment();

        log.info("app.name : {}", environment.getProperty("app.name"));
    }
}
```

* Log 확인  

```java
c.i.s.c.environment.EnvironmentRunner    : app.name : Command Line
```

<br>  

* 우선순위가 가장 높은 "Command Line" 이 출력된다.

  
<br>  

## Reference
* [https://www.inflearn.com/course/spring-framework_core/lecture/15512](https://www.inflearn.com/course/spring-framework_core/lecture/15512)  
* [https://stackoverflow.com/questions/44745261/why-do-jvm-arguments-start-with-d](https://stackoverflow.com/questions/44745261/why-do-jvm-arguments-start-with-d)