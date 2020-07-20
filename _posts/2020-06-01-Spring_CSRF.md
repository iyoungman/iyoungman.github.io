---
layout: post
title: CSRF란?
tags: Theory CSRF
categories: Theory
---

* TOC
{:toc}  
> CSRF에 대해 알아본다.
  
<br>  


## At Spring
* **Spring Security**를 사용하면 CSRF Token을 사용하도록 설정되어있다.
* 비활성화 하는 방법은 다음과 같다.

```java
@EnableWebSecurity
public class WebSecurityConfig extends WebSecurityConfigurerAdapter {

	@Override
	protected void configure(HttpSecurity http) throws Exception {
		http
			.csrf().disable();
	}
}
```
CSRF는 결국 CORS를 허용할때

타도메인인데 어떻게 해당 쿠키를 함께 전송하지?
쿠키값을 읽을수는 없지만 함께 전송가능?
HTTP 매개 변수는 못 가져오는 것인지?
<br>  



* 클라이언트에서 CSRF 토큰을 보내도록 한다.
* JSP에서 스프링 MVC가 제공하는 form:form 태그 or 타임리프 2.1+ 버전을 사용할 때 폼에
CRSF 히든 필드가 기본으로 생성 된다
* GET 제외
* https://reiphiel.tistory.com/entry/spring-security-csrf
http://jays1204.github.io/web/node.js/2015/12/14/csrf.html
https://cheese10yun.github.io/spring-csrf/



<br>  

## Reference  
* [https://www.baeldung.com/spring-security-csrf](https://www.baeldung.com/spring-security-csrf)  
* [https://docs.spring.io/spring-security/site/docs/5.0.x/reference/html/csrf.html](https://docs.spring.io/spring-security/site/docs/5.0.x/reference/html/csrf.html)  