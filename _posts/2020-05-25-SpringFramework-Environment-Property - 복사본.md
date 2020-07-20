---
layout: post
title: 스프링 부트 개념과 활용 - 스프링 웹 MVC 2부:HttpMessageConverter
tags: Spring Environment Property
categories: Spring
---

* TOC
{:toc}
> 강의 내용을 바탕으로 정리.  
>
> Spring의 외부 설정과 관련된 Property에 대해 알아본다.
  
<br>  

## HttpMessageConverter란?
* HTTP **요청** 본문 -> 객체로 변경
* 객체 -> HTTP **응답** 본문으로 변경
* ResponseBody와 함께 사용된다.


ResponseBody
RequestBody와 함께 사용

요청은 Content Type
json 이면 Json타입으로 변경해준다(JsonMessageConverter)

응답할때 어떤 방식으로 변환할것인가?
기본 Type -> JonsMessageConverter
String -> StringMessageConverter

응답시
MessageConverter는 @ResponseBody에서 적용된다
없으면 ViewResolver를 탄다




Accept-Header
Client가 서버로부터 어떠한 타입을 받기를 원한다.
이것에 따라서 리턴 타입 정해진다


  
<br>  

## Reference
* [https://www.inflearn.com/course/spring-framework_core/lecture/15512](https://www.inflearn.com/course/spring-framework_core/lecture/15512)  
* [https://stackoverflow.com/questions/44745261/why-do-jvm-arguments-start-with-d](https://stackoverflow.com/questions/44745261/why-do-jvm-arguments-start-with-d)