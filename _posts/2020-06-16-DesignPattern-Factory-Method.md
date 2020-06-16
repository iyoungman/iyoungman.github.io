---
layout: post
title: 팩토리 메소드 패턴
tags: DesignPattern FactoryMethodPattern Java
categories: DesignPattern
---
* TOC
{:toc}
> 예제를 통해 팩토리 메소드 패턴을 알아본다.
  
<br>  

## 팩토리 메소드 패턴
* 상속을 이용하여 변하지 않는 기능은 상위 클래스에 만들고<br>
하위 클래스에서 구체적인 오브젝트 생성 방법을 결정하는 디자인 패턴.  

![팩토리](https://user-images.githubusercontent.com/25604495/84584886-b4a5df00-ae44-11ea-865e-0e822ffd5964.jpg)  

<br>

## 적용 예제

> D사와 N사는 각기 다른 종류의 DB를 사용하고 있다.
>
> 따라서 각자의 방법으로 DB 커넥션을 만들고 싶다.
>
> 이때 팩토리 메소드 패턴을 적용한다.

<br>

```java
//UserDao
public abstract class UserDao {

    public void add(User user) throws ClassNotFoundException, SQLException {
		Connection c = getConnection();
        ...
    }

    public User get(String id) throws ClassNotFoundException, SQLException {
		Connection c = getConnection();
        ...
    }

    public abstract Connection getConnection() throws ClassNotFoundException, SQLException;
}

//DUserDao
public class DUserDao {
	public Connection getConnection() throws ClassNotFoundException, SQLException {
		// D사 DB Connection 생성코드
		return null;
	}

}

//NUserDao
public class NUserDao extends UserDao {
	public Connection getConnection() throws ClassNotFoundException, SQLException {
		// N사 DB Connection 생성코드
		return null;
	}
}
```

<br>

## 장점
* 하위 클래스를 통해 독립적으로 기능을 확장할 수 있다.
* 상위 클래스를 사용하는 기존 코드에 영향을 주지 않고 구현체를 변경할 수 있다.  

<br>

## 단점
> 상속을 사용했기 때문에 상속의 단점을 그대로 갖는다.

* 상위 클래스와 하위 클래스의 결합도가 높아진다.
* 상위 클래스의 변경이 있을 때 하위 클래스를 함께 수정해야할수도있다.

<br>

## Reference
* [토비의 스프링](http://www.yes24.com/Product/Goods/7516911)  