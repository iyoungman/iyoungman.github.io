---
layout: post
title: EntityManagerFactory, EntityManager 그리고 PersistenceContext
tags: JPA EntityManagerFactory EntityManager PersistenceContext
categories: JPA
---

* TOC
{:toc}

## EntityManagerFactory와 EntityManager

> EntityManagerFactory

* EntityManager를 만들어내는 곳.
* 생성 비용이 크므로 애플리케이션 전체에서 딱 한번만 생성하고 공유해서 사용해야 한다.

<br>

> EntityManager

* EntityManager를 이용해 DB와 Connection을 유지하여 CRUD를 한다.
* EntityManagerFactory를 통해 생성되며 비용이 거의 들지 않는다.
* **스레드 간에 공유하거나 재사용하면 안 된다.**

```java
public void createMember() {
    EntityManager em = emf.createEntityManager();
    EntityTransaction transaction = em.getTransaction();

    transaction.begin();

    Member member = new Member("name", "age");
    em.persist(member);

    transaction.commit();
}
```

<!--more-->

<br>

> 관계도

![image](https://user-images.githubusercontent.com/25604495/88453279-7ae2e200-cea0-11ea-8049-26258c5aded1.png)
  
* EntityManager는 DB 연결이 필요한 시점에 Connection을 얻는다.<br>
ex) 트랜잭션이 시작할 때.
* JPA의 구현체들은 **EntityManagerFactory를 생성할 때 커넥션풀을 만든다.**

<br>

## 영속성 컨텍스트

> 정의

* 엔티티를 영구 저장하는 환경.
* EntityManager로 엔티티를 저장하거나 조회하면 영속성 컨텍스트에 엔티티를 보관하고 관리한다.

<br>

> EntityManager와 관계

* 기본적으로 EntityManager를 생성할 때 하나 만들어진다.
* 하지만 컨테이너 환경의 JPA에서는 **여러 EntityManager가 하나의 영속성 컨텍스트를 공유**한다.
* 이는 아래 '트랜잭션 범위의 영속성 컨텍스트'에서 살펴본다.

<br>

## 컨테이너 환경에서 JPA
* 컨테이너를 사용하는 환경(ex. Spring)에서는 개발자가 EntityManager를 직접 생성하지 않고 생성을 **컨테이너에 위임한다.**

<br>

> 스프링이 관리하는 EntityManager의 Thread-Safe를 어떻게 보장할까?

* 결론부터 말하면 **EntityManager를 Proxy를 통해 감싸고 EntityManager를 사용할 때 Proxy를 통해 EntityManager를 생성한다.**
* EntityManger를 직접 사용하는 경우 @PersistenceContext를 사용하면 된다.

<br>

> 확인 1 : 직접 생성한 EntityManager에 @PersistenceContext 선언

![image](https://user-images.githubusercontent.com/25604495/88453808-6e14bd00-cea5-11ea-8832-30c8b656c0e3.png)

```java
entityManager = {$Proxy84@9656} "Shared EntityManager proxy for target factory [org.springframework.orm.jpa.LocalContainerEntityManagerFactoryBean@4f356b98]"
```

* SharedEntityManagerCreator에 의해 Proxy로 만들어진다.

<br>

> 확인 2 : SimpleJpaRepository의 EntityManager


![image](https://user-images.githubusercontent.com/25604495/88453865-df547000-cea5-11ea-9789-75009e7df83d.png)  

```java
em = {$Proxy84@9748} "Shared EntityManager proxy for target factory [org.springframework.orm.jpa.LocalContainerEntityManagerFactoryBean@4f356b98]"
```

* SharedEntityManagerCreator에 의해 Proxy로 만들어진다.

<br>


## 트랜잭션 범위의 영속성 컨텍스트
* 스프링 컨테이너는 **트랜잭션 범위의 영속성 컨텍스트 전략**을 기본으로 사용한다.
* 트랜잭션이 시작할 때 영속성 컨텍스트를 생성하고 트랜잭션이 끝날 때 영속성 컨텍스트를 끝낸다.

![image](https://user-images.githubusercontent.com/25604495/88453538-d4e4a700-cea2-11ea-9453-c3448d088a26.png)  

<br>

> 트랜잭션이 같으면 같은 영속성 컨텍스트를 사용한다

* 여러 EntityManager를 사용해도 한 트랜잭션으로 묶이면 영속성 컨텍스트를 공유한다.

![image](https://user-images.githubusercontent.com/25604495/88453641-c8148300-cea3-11ea-91a9-48f6c5e5885a.png)


<br>

> 트랜잭션이 다르면 다른 영속성 컨텍스트를 사용한다

* 같은 EntityManager를 사용해도 트랜잭션에 따라 접근하는 영속성 컨텍스트가 다르다.
* 따라서 **같은 EntityManager를 호출해도 접근하는 영속성 컨텍스트가 다르므로 멀티스레드에 안전하다.**

![image](https://user-images.githubusercontent.com/25604495/88453617-96032100-cea3-11ea-88e4-99411b2408f3.png)

<br>

## Reference
* [Spring Container는 JPA EntityManager의 Thread-Safety를 어떻게 보장할까?](https://medium.com/@SlackBeck/spring-container%EB%8A%94-jpa-entitymanager%EC%9D%98-thread-safety%EB%A5%BC-%EC%96%B4%EB%96%BB%EA%B2%8C-%EB%B3%B4%EC%9E%A5%ED%95%A0%EA%B9%8C-1650473eeb64)
* [https://stackoverflow.com/questions/11173974/different-ways-of-getting-the-entitymanager](https://stackoverflow.com/questions/11173974/different-ways-of-getting-the-entitymanager)
* https://lng1982.tistory.com/276