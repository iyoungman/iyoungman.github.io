---
layout: post
title: 상속(Inheritance) vs 컴포지션(Composition)
tags: DesignPattern Inheritance Composition Java Spring
categories: DesignPattern
---
* TOC
{:toc}
> 코드를 재사용하기 위해 상속 or 컴포지션을 사용한다.
>
> 상속보다는 컴포지션이 좋은 이유를 알아본다.
  
<br>  

## 상속의 단점  

> [1] 캡슐화를 위반한다.

* 상위 클래스가 어떻게 구현되느냐에 따라 하위 클래스의 동작에 이상이 생길 수 있기 때문이다.
* 상위 클래스의 내부 구현이 달라지면 하위 클래스를 고쳐야할 수 있다.
  
<br>  

> [2] 설계가 유연하지 못하다.

* 컴파일 시점에 객체의 Type이 정해지기 때문이다.
  
<br>  

> [3] 다중상속

* 자바는 다중상속이 불가능하다.
* 따라서 다른 클래스를 상속받고있다면 추가적으로 상속을 받을 수 없다.
  
<br>  

## 컴포지션이란?  

* **다른객체의 인스턴스를 자신의 인스턴스 변수로 포함해서 메서드를 호출**하는 기법이다.
* 해당 인스턴스의 내부 구현이 바뀌더라도 영향을 받지 않는다.
* 또한, 다른객체의 인스턴스이므로 인터페이스를 이용하면 Type을 바꿀 수 있다.
  
<br>  

## 적용 예제
> [이전 예제](https://iyoungman.github.io/designpattern/DesignPattern-Factory-Method/)를 컴포지션으로 바꿔본다.
  
<br>  

```java
//ConnectionMaker
public interface ConnectionMaker {
    public Connection makeConnection() throws ClassNotFoundException, SQLException;
}

//DConnectionMaker
public class DConnectionMaker implements ConnectionMaker {

    public Connection makeConnection() throws ClassNotFoundException, SQLException {
        // D사 DB Connection 생성코드
        return null;
    }
}

//NConnectionMaker
public class NConnectionMaker implements ConnectionMaker {

    public Connection makeConnection() throws ClassNotFoundException, SQLException {
        // N사 DB Connection 생성코드
        return null;
    }
}

//UserDao
public class UserDao {

    private ConnectionMaker connectionMaker;//컴포지션 사용

    public UserDao(ConnectionMaker connectionMaker) {
        this.connectionMaker = connectionMaker;
    }

    public void add(User user) throws ClassNotFoundException, SQLException {
		Connection c = connectionMaker.makeConnection();
        //생략
    }

    public User get(String id) throws ClassNotFoundException, SQLException {
		Connection c = connectionMaker.makeConnection();
        //생략
    }
}

//Client
ConnectionMaker connectionMaker = new DConnectionMaker();
UserDao userDao = new UserDao(connectionMaker);
```
  
<br>  

## 언제 상속을 사용할까?
* 상위 클래스와 하위 클래스의 관계가 정말 **is-a 관계**일때 사용한다.<br>
확신할 수 없다면 컴포지션을 사용하자.  
<br>

> HashTable과 Properties  

* 자바 플랫폼 라이브러리에서 위의 원칙을 위반한 예시이다.<br>
설명에 필요한 Properties API의 일부 코드만 가져왔다.

```java
//Properties
public class Properties extends Hashtable<Object,Object> {

    @Override
    public synchronized Object put(Object key, Object value) {
        return map.put(key, value);
    }

    public synchronized Object setProperty(String key, String value) {
        return put(key, value);
    }

    public void store(Writer writer, String comments)
        throws IOException
    {
        store0((writer instanceof BufferedWriter)?(BufferedWriter)writer
                                                 : new BufferedWriter(writer),
               comments,
               false);
    }
}

//Client
public class Main {

    public static void main(String[] args) {
        File path = new File("./src/main/java/inheritance_composition/TEST_FILE");
        try (FileWriter file = new FileWriter(path)) {
            Properties p = new Properties();
            p.put(1, "id");//상위 클래스의 메서드
            p.setProperty("id", "id");//Properties의 메서드
            p.setProperty("pw", "pw");
            p.store(file, "user");//Runtime Exception!
        } catch (Exception e) {
            System.out.println("fail));
        }
    }
}
```

> Properties는 키와 값으로 문자열만 허용하도록 설계되었다.

* 따라서 Properties의 setProperty 메서드의 인자인 Key, Value는 String 값이다.
 
<br>  


> 하지만 상위 클래스의 메서드를 호출하면 위와 같은 불변식을 깨버린다. 

* 상위 클래스를 오버라이드한 put 메서드를 호출하면 메서드의 인자인 Key, Value는 Object가 된다.
 
<br>  

> 불변식이 깨지기때문에 store와 같은 Properties API의 메서드를 사용할 수 없다.

* 위 예제에서 Properties의 Key, Value에 String 이외의 값이 들어갔기 때문에<br>
store 메서드를 호출했을때 Exception이 발생한다.
 
<br>  

> 예제의 결론

* 상속보다는 컴포지션을 사용했으면 좋았을 것이다.



  
<br>  


## Reference
* [토비의 스프링](http://www.yes24.com/Product/Goods/7516911)  