---
layout: post
title: 트랜잭션과 잠금
tags: MySQL InnoDB Transaction Lock IsolationLevel
categories: DB
---

* TOC
{:toc}

> 회사에서 AWS Aurora MySQL 을 사용한다.
> Aurora MySQL의 [default_storage_engine](https://docs.aws.amazon.com/ko_kr/AmazonRDS/latest/AuroraUserGuide/AuroraMySQL.Reference.html#AuroraMySQL.Reference.Parameters.Cluster) 은 InnoDB 이다.  
> 따라서 InnoDB 를 기준으로 정리해본다.

<!--more-->

<br>

## 트랜잭션과 잠금
* 트랜재션은 데이터 정합성을 보장하기 위한것.
* 잠금은 동시성을 제어하기 위한것.
* 잠금이 없다면 하나의 테이블을 여러 커넥션이 동시에 변경하여 레코드의 값을 예측할 수 없을 것이다.

<br>

## 트랜잭션
* Partial Update 를 막기위한것.
* 트랜잭션의 범위는 가능하면 작게 가져가는 것이 좋다.

![IMG_1795](https://user-images.githubusercontent.com/25604495/99869643-134f6000-2c10-11eb-9138-1a8ad7e05524.JPG)


<br>

## InnoDB 스토리지 엔진의 잠금
* MyISAM과 다르게 레코드 기반의 잠금 방식을 사용한다.
* 따라서 동시성 처리 능력이 MyISAM 보다 뛰어나다.

<br>

### InnoDB의 잠금 방식
* 비관적 잠금 방식을 사용한다.

<br>

> 비관적 잠금

* 트랜잭션이 변경하고 하는 레코드에 잠금을 획득한 후에 변경 작업을 수행한다.

<br>

> 낙관적 잠금

* 트랜잭션간에 같은 레코드를 변경할 가능성이 희박하다고 가정한다.
* 따라서 변경 작업을 수행하고 마지막에 충돌이 있는지 확인하여 문제가 있으면 롤백하는 방식이다.

<br>

## InnoDB의 잠금 종류

![IMG_1796](https://user-images.githubusercontent.com/25604495/99869644-15192380-2c10-11eb-911f-c4405ee4284e.JPG)

> 레코드 락

* 다른 DBMS의 레코드 락과 동일한 역할을 하지만 차이점이 있다.
* InnoDB는 레코드 자체가 아니라 인덱스의 레코드를 잠금한다.
* 만약 인덱스가 하나도 없는 테이블이라도 내부적으로 자동 생성된 클러스터 인덱스를 통해 잠금을 설정한다.  
<br>
* 프라이머리 키 또는 유니크 인덱스에 의한 변경 작업은 레코드 자체에만 락을 건다.
* 보조 인덱스를 통한 변경 작업은 (넥스트 키 락 or 갭 락)을 사용한다.

<br>

> 갭 락

<br>

> 넥스트 키 락


<br>

> 자동 증가 락

* 자동 증가하는 숫자 값을 추출하기 위해 AUTO_INCREMENT라는 칼럼 속성을 사용한다.
* AUTO_INCREMENT 칼럼이 사용된 테이블에 동시에 INSERT 되는 경우, AUTO_INCREMENT 락이라고 하는 테이블 수준의 잠금을 수행한다.
* AUTO_INCREMENT 락은 INSERT 와 같이 새로운 레코드를 저장하는 쿼리에서만 발생한다.
* INSERT 문장에서 AUTO_INCREMENT 값을 가져오는 순간에만 락이 걸렸다가 즉시 해제된다.
* MySQL 5.1 이상에서는 자동 증가 락의 작동 방식을 설정할 수 있다.(innodb_autoinc_lock_mode)

<br>

* innodb_autoinc_lock_mode=0
* innodb_autoinc_lock_mode=1
    * insert 하는 레코드 건수를 예측할 수 있을때는 한번에 여러 자동 증가값을 할당받아 INSERT 되는 레코드에 사용한다.
    * insert 하는 레코드 건수를 예측할 수 없을때는 mode=0 과 같이 동작한다.
    * 만약 한번에 할당받은 자동 증가값이 남으면 폐기되므로 레코드 자동 증가 값은 누락될 수 있다.
* innodb_autoinc_lock_mode=2


<br>

### 인덱스와 잠금
* InnoDB의 잠금은 레코드가 아닌 인덱스를 잠근다.

![IMG_1797](https://user-images.githubusercontent.com/25604495/99869648-15b1ba00-2c10-11eb-9638-3b59fc6bb012.JPG)


* UPDATE employees SET hire_date=NOW() WEHRE first_name='Georgi' AND last_name='Klassen';
* 위 문장을 실행하면 1건이 UPDATE 되지만 253건의 레코드가 잠긴다.

* 추가적으로 인덱스를 조건절로 넣지 않는다면 풀스캔한다.

<br>

### 트랜잭션 격리 수준과 잠금
* 불필요한 레코드 잠금은 InnoDB의 넥스트 키 락 때문에 발생한다.
* 갭 락이나 넥스트 키 락을 줄이는 것이 중요하다.
* 아래 조건으로 줄일 수 있다.

![IMG_1798](https://user-images.githubusercontent.com/25604495/99869646-15b1ba00-2c10-11eb-95d5-dcbbfac62336.JPG)

* 인덱스만을 비교하여 일치하는 레코드를 찾아 배타적 잠금을 걸지만, 나머지 조건을 비교해 일치하지 않는 레코드는 잠금을 해제한다.
* 위 예제에서 1차 비교 단계에서는 first_name='Georgi'인 레코드를 모두 잠그지만, 나머지 조건 일치 여부를 판단하는 2차 비교 단계에서 UPDATE 조건이 아닌것을 잠금에서 해제한다.

<br>

## MySQL의 격리 수준
* 동시에 여러 트랜잭션 처리될 때, 특정 트랜잭션이 다른 트랜잭션에서 변경되거나 조회하는 데이터를 볼 수 있게 허용할지 말지 결정하는 것이다.
* InnoDB의 default 는 REPEATABLE_READ
* InnoDB에서는 Repeatable Read 수준에서도 Phantom Read가 발생하지 않는다.

![image](https://user-images.githubusercontent.com/25604495/99869586-a6d46100-2c0f-11eb-8d75-838f9b973651.png)



