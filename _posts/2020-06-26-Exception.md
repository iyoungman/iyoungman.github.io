---
layout: post
title: 예외
tags: Spring Exception Java
categories: Spring
---
* TOC
{:toc}
> 예외와 처리 방법에 대해 알아본다.
>
> 스프링의 예외처리 전략을 통해 얻는 이점을 알아본다.
  
<br>  

## 잘못된 예외 처리

1) 예외를 잡고 아무것도 하지 않는 것  

2) 예외를 화면에 출력
* 운영서버에 올라가면 찾기 힘들다.  

3) 시스템을 종료

<br>  

## 예외의 종류와 특징

![image](https://user-images.githubusercontent.com/25604495/49819560-20733b80-fdb9-11e8-81f9-eec8abbc2007.png)  

<br>  

> Error

* 시스템 레벨에서 발생.
* 애플리케이션에서 이런 에러에 대한 처리는 대응할수 없기 때문에 신경쓸 필요 없다.

<br>

> Exception

1) Checked Exception
* 컴파일 단계에서 확일할 수 있는 예외.
* 컴파일 단계에서 처리해줘야한다.
* Checked Exception이 발생한 메서드에서 try catch로 잡거나<br>
  throws를 이용하여 메서드 밖으로 던져야한다.  

<br>  


2) Unchecked Exception
* 컴파일 단계에서 확인할 수 없는 예외.
* 컴파일 단계에서 처리를 반드시 할 필요는 없다.

<br>  

## 예외처리 방법

> [1] 예외 복구

* 예외상황을 파악하고 문제를 해결해서 정상 상태로 돌려놓는 것.
* 예를 들면 사용자가 요청한 파일을 읽지 못햇을 때<br>
사용자에게 상황을 알려주고 다른 파일을 이용하도록 유도한다.
* 애플리케이션이 정상적으로 설계된 흐름을 따르도록 하는 것이다.

<br>  

> [2] 예외처리 회피

* 예외처리를 자신을 호출한 쪽으로 던지는 것.
* 예외를 회피하는 것은 의도가 명확해야한다.<br>
자신을 사용하는 쪽에서 예외를 다루는게 최선일 때 회피해야한다.

<br>  

> [3] 예외전환

* 예외처리와 회피하게 메서드 밖으로 던지는 것이지만<br>
발생한 예외를 그대로 넘기지 않고 **적절한 예외로 전환**하여 던진다.  
<br>
* 크게 2가지 목적이 있다.
* (1) 의미를 분명하게 해줄 수 있는 예외로 전환.

```java
public void add(User user) throws DuplicateUserIdException, SQLException {
    try {

    } catch (SQLException e) {
        if (e.getErrorCode() == MysqlErrorNumbers.ER_DUP_ENTRY) {
            throw new DuplicateUserIdException(e).initCause(e);
        } else {
            throw e;
        }
    }
}
```

<br>  


* (2) 처리하기 쉽도록 포장한다.<br>
주로 **예외처리를 강제하는 체크 예외를 런타임 예외로 바꿀 때** 사용.

```java
catch (SQLException e) {
    ...
    throw new DuplicateUserIdException(e).initCause(e);
}
```


<br>  

> 복구 가능한 예외 vs 복구 불가능한 예외

* 복구 가능한 경우는 체크 예외를 사용하여 적절한 대응을 한다.<br>
try catch를 강제하기 때문이다.  
<br>
* 복구 불가능한 경우는 런타임 예외를 사용하여 해당 메소드 밖으로 던진다.<br>
해당 예외를 받은 메서드는 불필요하게 또다시 throws를 던질 필요가 없으며<br>
그냥 예외를 발생시킨다.

<br>

> 서버환경에서의 예외 처리

* 대부분의 서버환경에서는 예외 처리를 일괄적으로 다룰 수 있는 기능을 제공한다.
* 복구하지 못할 예외라면 런타임 예외를 포장하여 던지고 예외처리 서비스를 이용하면 편리하다.  
<br>
* 서버 환경에서는 쳬크 예외의 활용도가 떨어지고 있다.

<br>

## 예외처리 전략

> 체크 예외를 런타임 예외로 전환

```java
//DuplicateUserIdException
public class DuplicateUserIdException extends RuntimeException {

    public DuplicateUserIdException(Throwable cause) {
        super(cause);
    }
}

//add 메서드
public void add(User user) throws DuplicateUserIdException, SQLException {
    try {

    } catch (SQLException e) {
        if (e.getErrorCode() == MysqlErrorNumbers.ER_DUP_ENTRY) {
            throw new DuplicateUserIdException(e);
        } else {
            throw new RuntimeException(e);
        }
    }
}
```

* add 메소드를 사용하는 오브젝트는 SQLException을 처리하기 위한 불필요한 throws를 선언할 필요가 없다.
* 런타임 예외를 일반화하면 장점이 많지만<br>
컴파일 타임에 예외를 잡을 수 없기때문에 예외상황을 고려하지 못할 수도 있다.

<br>

### 스프링의 예외처리 전략
* 스프링의 JdbcTemplate는 위와 같은 전략을 따르고 있다.
* JdbcTemplate는 템플릿과 콜백 안에서 발생하는 SQLException을 런타임 예외인 DataAccessException을 포장해서 던진다.
* 따라서 JdbcTemplate을 사용하는 객체에서 DataAccessException을 **처리해도 되고 처리를 안해도 된다.**

<br>

> SQLErrorCodeSQLExceptionTranslator.class에서 SQLException을 DataAccessException로 포장.  

![Exception](https://user-images.githubusercontent.com/25604495/85713845-c761c300-b724-11ea-8e38-72e1f50e3d3a.PNG)  

<br>  

## 예외 전환
* 예외 전환은 2가지 목적을 갖고있다.
* [1] 위에서 SQLException -> DataAccessException으로 포장한 것처럼 런타임 예외로 바꾸는 용도.
* [2] **추상화된 예외로 변환.**

<br>  

### 추상화된 예외로 변환하여 얻는 효과는?

> 호환성 없는 SQLException의 DB 에러정보

* DB에 따라서 SQL뿐만 아니라 에러의 종류와 원인도 다르다.
* JDBC는 데이터 처리중에 발생하는 예외를 SQLException 하나에 모두 담고 던진다.
* 예외 발생 원인은 SQLException에 담긴 에러 코드와 상태정보를 참조해야하는데 이 역시 DB별로 다르다.
* DB에 독립적으로 에러 코드 처리를 하려면 어떻게 해야할까?

<br>  

> DB 에러 코드 매핑을 통한 전환

* DataAccessExcetpion은 SQLExcetpion을 단순히 런타임 예외로 대체할 뿐만 아니라
**DataAccessExcetpion의 서브클래스로 세분화된 예외 클래스를 정의하고 있다.**
* 스프링은 DB별 다른 에러 코드를 분류해서 DataAccessExcetpion의 서브클래스와 매핑한다.<br>
아래는 MySQL 에러 코드 매핑 파일이다.(sql-error-codes.xml)  

![image](https://user-images.githubusercontent.com/25604495/85816409-33d0d680-b7a6-11ea-845f-7b1173efe332.png)  



* 따라서 JdbcTemplate는 DB의 종류와 상관없이 동일한 예외를 받을 수 있다.

<br>  

## 데이터 액세스 예외 추상화와 DataAccessException 구조
* 데이터 액세스 기술에는 JDBCTemplate(SQL Mapper)을 이용한 방식 이외에도 여러 방법이 존재한다.
* 예를 들면 MyBatis(SQL Mapper)나 JPA(ORM) 등이 있다.  

<br>  

> ORM 방식  

![image](https://user-images.githubusercontent.com/25604495/85820178-a34bc380-b7b0-11ea-9226-911d1c7c6168.png)  

<br>  

* 이런 경우 각 데이터 액세스 기술마다 예외가 다를 수 있다.<br>예를 들면 JPA와 같은 ORM에서는 발생하지만 JDBC에서는 없는 예외가 있다.
* DataAccessException에서는 일부 기술에서만 발생할 수 있는 예외도 추상화하고 있다.





<br>  

## Reference
* [토비의 스프링](http://www.yes24.com/Product/Goods/7516911)  
* [https://terasolunaorg.github.io/guideline/5.1.0.RELEASE/en/ArchitectureInDetail/DataAccessJpa.html](https://terasolunaorg.github.io/guideline/5.1.0.RELEASE/en/ArchitectureInDetail/DataAccessJpa.html)
