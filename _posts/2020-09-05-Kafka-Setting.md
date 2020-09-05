---
layout: post
title: 카프카 환경설정
tags: Kafka 카프카데이터플랫폼의최강자
categories: Kafka
---

* TOC
{:toc}

## 카프카 관리를 위한 주키퍼

> 주키퍼

* 최근들어 하둡, 나이파이, 에이치베이스 등 분산 애플리케이션이 개발되고 있다.<br>
분산 애플리케이션 관리를 위한 코디네이션 애플리케이션이 필요한데 대표적으로 주키퍼를 사용한다.
* 분산되어 있는 각 애플리케이션 정보를 중앙에 집중하고 구성 관리, 그룹 관리, 동기화 등 서비스를 제공한다.

<!--more-->

![IMG_1167](https://user-images.githubusercontent.com/25604495/92304767-ebab0d00-efbb-11ea-960d-ef0a00a9edbc.JPG)

<br>

> 지노드

* 상태 정보는 주키퍼의 지노드에 Key-Value 형태로 저장한다.
* 지노드는 데이터를 저장하기 위한 공간으로 파일, 폴더 개념으로 생각하면 된다.<br>
따라서 계층형 구조이다.  

![IMG_1166](https://user-images.githubusercontent.com/25604495/92304768-ecdc3a00-efbb-11ea-81d0-d2f499eae695.JPG)  

<br>

> 앙상블(클러스터)

* 앙상블로 구성되어 있는 주키퍼는 과반수 방식에 따라 살아 있는 노드 수가 과반수 이상 유지되면 서비스를 유지할 수 있다.

![IMG_1164](https://user-images.githubusercontent.com/25604495/92304770-ee0d6700-efbb-11ea-88a9-5cc7909c0154.JPG)

<br>

## 카프카 설치
* 카프카 클러스터의 서버 대수는 홀수가 아니어도 상관없다.
* 주키퍼와 카프카는 동일한 서버가 아닌 별도로 구성하는 것이 좋다.

<br>

> 주키퍼와 카프카

![IMG_1165](https://user-images.githubusercontent.com/25604495/92304769-ed74d080-efbb-11ea-9dcb-9aca57bbe249.JPG)

* [1] 앙상블 내 3대 서버는 모두 동일한 정보를 갖고있다. <br>
따라서 카프카 설정 파일내 주키퍼 정보 입력시 주키퍼 앙상블 중 3대 중 하나만 입력해도 사용할 수는 있다.

* [2] 만약 카프카가 바라보고 있는 1대의 주키퍼 서버가 다운되었다고 해보자.

* [3] 주키퍼 서버 3대 중 다운된 1대를 제외하고 과반수 이상인 2대의 서버는 살아있다.<br>
따라서 주키퍼 앙상블은 정상 상태로 유지되고 있다.<br>
하지만, 카프카 환경설정에서 주키퍼 서버 하나만 입력했기 때문에 카프카 클러스터에는 장애가 발생한다.

<br>

* 따라서 **카프카 설정에서 모든 주키퍼 서버 리스트를 입력하는 것이 좋다.**

```yml
zookeeper.connect=peter-zk001:2181,peter-zk001:2182,peter-zk003:2181
```

<br>

> 카프카 환경 설정 server.properties 파일의 주요 옵션

[https://docs.confluent.io/current/installation/configuration/broker-configs.html](https://docs.confluent.io/current/installation/configuration/broker-configs.html)



