---
layout: post
title: Java8. 새로운 날짜와 시간 API
tags: Java Java8 Date Calendar LocalDate LocalTime
categories: Java
---
## 개요
* Java8이전의 날짜 API의 문제점을 알아본다.
* Java8의 새로운 날짜와 시간 라이브러리를 알아본다.
  
<br>  

## 새로운 API를 제공하는 이유
* Java8 이전에 날짜, 시간 관련 기능을 제공하는
* `Date`와 `Calendar`는 문제를 가지고 있었다.  

<br> 

### Date 클래스 문제점
```java
Date date = new Date();//Wed Apr 22 18:18:50 KST 2020

LocalDate date = LocalDate.now();//2020-04-22
```
1. Java8의 LocalDate와 비교했을 때, 결과가 직관적이지 않다.
> Wed Apr 22 18:18:50 KST 2020
  
<br>  

2. Date는 가변클래스이다.
> 반면에 Java8의 LocalDate는 불변클래스이다.

<br> 

### Calendar 클래스 문제점
```java
Calendar calendar = Calendar.getInstance();
calendar.set(2020, 4 , 22);//2020년 5월 22일
```
1. Month의 인덱스
> Calendar의 Month는 상수필드로써 0부터 시작한다.
>
> 즉, 0 -> 1월, 1 -> 2월과 같은 형태이다.  
  
<br>  

* 다음은 Calendar 클래스에 정의된 상수값이다.  
  
```java  
/**
* Value of the {@link #MONTH} field indicating the
* first month of the year in the Gregorian and Julian calendars.
*/
public static final int JANUARY = 0;

/**
* Value of the {@link #MONTH} field indicating the
* second month of the year in the Gregorian and Julian calendars.
*/
public static final int FEBRUARY = 1;

/**
* Value of the {@link #MONTH} field indicating the
* third month of the year in the Gregorian and Julian calendars.
*/
public static final int MARCH = 2;
```  
  
<br>  

2. 상수 필드 남용  

```java  
//현재 시간에서 두 시간을 더한다.(HOUR_OF_DAY)
Calendar calendar = Calendar.getInstance();
calendar.add(Calendar.HOUR_OF_DAY, 2);

//엉뚱한 상수가 들어갔지만 위와 결과가 같다.(DECEMBER)
Calendar calendar2 = Calendar.getInstance();
calendar.add(Calendar.DECEMBER, 2);
```
  
> Calendar의 add(int field, int amount) 메소드는 일정 시간 만큼 +, - 해주는 역할이다.
>
> 하지만 첫번째 파라미터에 Calendar.DECEMBER과 같이 엉뚱한 상수가 들어가도 
>
> 컴파일 시점에 확인 할 수 없을뿐더러, 결과가 같을 수도 있다.
  
<br>  

3. Calendar 역시 가변클래스이다.
  
<br>  

## LocalDate와 LocalTime
* LocalDate는 날짜를 표현하며 
* LocalTime은 시간을 표현한다.
* 장점은 크게 두가지라고 생각한다.
> 1. 불변객체이다.
>
> 2. 내장 메서드를 이용해 편리하게 사용할 수 있어 직관적이다.

```java
LocalDate date = LocalDate.of(2020, 5, 22); //2020-05-22
int year = date.getYear(); // 2020
Month month = date.getMonth(); // MAY
int monthValue = date.getMonthValue() //5
int day = date.getDayOfMonth(); // 5

LocalTime time = LocalTime.of(13, 45, 20); //13:45:20
int hour = time.getHour(); // 13
int minute = time.getMinute(); // 45
int second = time.getSecond(); // 20
```
  
<br>  

* 그밖에 참고 : ~~1.1. LocalDate, 1.2. LocalTime~~ 주요 메서드
> [자바8의 java.time 패키지(LocalDate, LocalTime, LocalDateTime 등)](http://blog.eomdev.com/java/2016/04/01/%EC%9E%90%EB%B0%948%EC%9D%98-java.time-%ED%8C%A8%ED%82%A4%EC%A7%80.html)

* 그밖에 참고 : ~~날짜 변환하기~~
> [Java 8 날짜와 시간 계산](https://madplay.github.io/post/java8-date-and-time)

<br>  

## LocalDateTime  
* LocalDate + LocalTime  

```java
LocalDateTime dt = LocalDateTime.of(2014, Month.MARCH, 18, 13, 45, 20);
  
LocalDate date = dt.toLocalDate();  
LocalTime time = dt.toLocalTime();
```

<br>  

## 날짜의 속성 바꾸기
* withAttribute 메서드로 기존의 LocalDate를 간단하게 바꿀 수 있다.
* 물론 LocalDate는 불변객체이므로
* 기존 객체를 바꾸지 않고 
* 바뀐 속성을 포함하는 새로운 객체를 반환한다.  

```java
//절대적인 방식 
LocalDate date1 = LocalDate.of(2017, 5, 11); // 2017-05-11
LocalDate date2 = date1.withYear(2010);      // 2010-05-11
LocalDate date3 = date2.with(ChronoField.MONTH_OF_YEAR, 1); // 2010-01-11 
LocalDate date4 = date3.withDayOfMonth(1);   // 2010-01-01

//상대적 방식
LocalDate date1 = LocalDate.of(2017, 5, 11); // 2017-05-11
LocalDate date2 = date1.plusWeeks(1);        // 2017-05-18
LocalDate date3 = date2.minusYears(3);       // 2014-05-18
LocalDate date4 = date3.plus(3, ChronoUnit.MONTHS); // 2014-08-18
```
  
<br>  

* 그밖에 참고 : ~~특정 시점을 표현하는 날짜 시간 클래스의 공통 메서드~~
> [[자바 8] CH12 - 새로운 날짜와 시간 API](http://blog.naver.com/PostView.nhn?blogId=hehe5959&logNo=221003414774&parentCategoryNo=&categoryNo=20&viewDate=&isShowPopularPosts=true&from=search)



<br>  

## 참고
* [모던 자바 인 액션](http://www.yes24.com/Product/Goods/77125987?Acode=101)
* [https://d2.naver.com/helloworld/645609](https://d2.naver.com/helloworld/645609)
* [https://www.geeksforgeeks.org/date-class-java-examples/](https://www.geeksforgeeks.org/date-class-java-examples/)
* [https://www.geeksforgeeks.org/calendar-class-in-java-with-examples/](https://www.geeksforgeeks.org/calendar-class-in-java-with-examples/)
