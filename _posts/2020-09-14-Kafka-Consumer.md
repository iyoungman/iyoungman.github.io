---
layout: post
title: 카프카 컨슈머
tags: Kafka 카프카데이터플랫폼의최강자
categories: Kafka
---

* TOC
{:toc}

## 컨슈머의 역할
* 파티션을 관리하고 있는 파티션 리더에게 메시지 가져오기를 요청한다.

<!--more-->

<br>

## 컨슈머의 주요 옵션
* 카프카 버전 0.9 이전에는 컨슈머의 오프셋을 주키퍼에 저장한다.
* 카프카 버전 0.9 이후에는 컨슈머의 오프셋을 토픽에 저장한다.

<br>

> bootstrap.servers

* 카프카 클러스터에 연결 하기 위한 호스트와 포트 정보로 구성된 리스트 기입.
* 카프카 클러스터 내의 전체 호스트 정보를 입력하는것이 좋다.

<br>

> group.id

* 컨슈머가 속한 컨슈머 그룹을 식별한다.
* 컨슈머 그룹은 매우 중요하다.

<br>

> request.timeout.ms

* 요청에 대해 응답을 기다리는 최대 시간.

<br>

> session.timeout.ms

* 브로커가 컨슈머가 살아있는것으로 판단하는 시간.
* 컨슈머가 컨슈머 그룹 코디네이터에 해당 시간동안 하트비트를 보내지 않으면 해당 컨슈머가 종료되거나 장애된 것으로 판단한다.
* heartbeat.interval.ms 옵션과 관련이 있다.
* Default = 10초

<br>

> heartbeat.interval.ms

* 컨슈머가 컨슈머 그룹 코디네이터에 얼마나 자주 poll() 메소드로 하트비트를 보낼 것인지 설정한다.
* Default = 3초

<br>

> 그외의 설정

* [https://docs.confluent.io/current/installation/configuration/consumer-configs.html](https://docs.confluent.io/current/installation/configuration/consumer-configs.html)  


<br>


## 자바를 이용한 컨슈머
```java
public class KafkaBookConsumer1 {
  public static void main(String[] args) {
    Properties props = new Properties();
    props.put("bootstrap.servers", "peter-kafka001:9092,peter-kafka002:9092,peter-kafka003:9092");
    props.put("group.id", "peter-consumer");
    props.put("enable.auto.commit", "true");//(1)
    props.put("auto.offset.reset", "latest");
    props.put("key.deserializer", "org.apache.kafka.common.serialization.StringDeserializer");
    props.put("value.deserializer", "org.apache.kafka.common.serialization.StringDeserializer");
    KafkaConsumer<String, String> consumer = new KafkaConsumer<>(props);
    consumer.subscribe(Arrays.asList("peter-topic"));//(2)
    try {
      while (true) {//(3)
        ConsumerRecords<String, String> records = consumer.poll(100);//(4)
        for (ConsumerRecord<String, String> record : records)//(5)
          System.out.printf("Topic: %s, Partition: %s, Offset: %d, Key: %s, Value: %s\n", record.topic(), record.partition(), record.offset(), record.key(), record.value());
      }
    } finally {
      consumer.close();
    }
  }
}
```

* [1] : 자동 커밋은 이후 설명
* [2] : 구독할 토픽 리스트를 기입.
* [3] : 무한 루프. 메시지를 가져오기 위해 카프카에 주기적으로 poll()
* [4] : 컨슈머는 주기적으로 카프카에 폴링.
* [5] : poll()은 레코드 전체를 리턴한다. 레코드에는 토픽, 파티션, 파티션의 오프셋, 키, 값 등을 포함한다. 한번에 여러 메시지를 가져올 수 있으므로 반복문 처리.

<br>

## 파티션과 메시지 순서

> 파티션 여러개로 구성했을 때

* 3개로 구성했다고 가정한다.
* 프로듀서에서 카프카 특정 토픽으로 a,b,c,d,e,1,2,3,4,5 의 순서로 메시지를 보냈다.

<br>

![IMG_1190](https://user-images.githubusercontent.com/25604495/93096979-fb270600-f6df-11ea-9bdd-94fc0187fd87.JPG)

* 그림과 같이 **각각의 파티션에 메시지가 저장된다.**
* 컨슈머는 메시지를 가져올 때 프로듀서가 어떤 순서로 보냈는지 알 수 없다.
* 오직 파티션의 오프셋 기준으로 메시지를 가져온다.

<br>

* 컨슈머는 파티션의 오프셋 순서대로 메시지를 가져온다.
* 따라서 결과는 b,e,2,5,a,d,1,4,c,3 이 된다.
* 토픽의 파티션이 여러개인 경우 메시지의 순서는 보장 할 수 없다.
* 동일한 파티션 내에서만 순서가 유지 된다.


<br>

> 파티션 1개로 구성했을 떄

* 메시지의 순서를 정확하게 보장해야할 때 사용한다.

![IMG_1191](https://user-images.githubusercontent.com/25604495/93096971-f82c1580-f6df-11ea-9cf0-433826a0036e.JPG)  

<br>

## 컨슈머 그룹
* 하나의 토픽에 여러 컨슈머 그룹이 접속해 메시지를 가져올 수 있다.
* 다른 메시징큐와 다른점으로써 하나의 데이터를 다양한 용도로 사용할 수 있다.

<br>


> [1] 기존 상황

![IMG_1196](https://user-images.githubusercontent.com/25604495/93096841-d763c000-f6df-11ea-8fbf-2cbe28f955ac.JPG)  

* 컨슈머가 메시지를 가져가는 속도보다 프로듀서가 메시지를 보내는 속도가 빨라 데이터가 쌓이게 된다면 컨슈머 확장이 필요하다.

<br>

> [2] 컨슈머 그룹에서 컨슈머 추가

* 컨슈머 그룹 안의 컨슈머들은 토픽의 파티션에 대한 정보를 공유한다.
* 컨슈머가 추가되었을 때 소유권이 이동하는 것을 리밸런스라고 한다.
* 리밸런스를 하는 동안 일시적으로 컨슈머 그룹은 메시지를 가져올 수 없다.

<br>

> [3] 파티션 < 컨슈머

* 컨슈머는 놀게된다.
* 파티션에는 하나의 컨슈머만 연결할 수 있다.
* 하나의 파티션에 여러 컨슈머가 연결된다면 메시지 순서를 보장하지 못할수 있다.

<br>

> 컨슈머 그룹내의 컨슈머가 다운되는 경우

![IMG_1194](https://user-images.githubusercontent.com/25604495/93096867-e0549180-f6df-11ea-902f-d86ae8972367.JPG)  

* 특정 컨슈머가 하트비트를 보내지 않으면 해당 컨슈머는 그룹내에서 제거된다.
* 이후 리밸런스가 일어난다.

<br>

## 커밋과 오프셋
* 컨슈머가 poll()을 호출하면 컨슈머 그룹은 카프카의 아직 읽지 않은 메시지를 가져온다.
* **컨슈머들은 각각의 파티션에 자신이 가져간 메시지의 오프셋을 기록하고 있다.**
* 파티션에 현재 위치 정보를 업데이트하는 행위를 커밋 이라고 한다.
* 컨슈머는 파티션에 가장 최근 커밋된 오프셋을 읽고 그 이후의 메시지를 가져온다.

<br>

### 자동 커밋
* enable.auto.commit=true
* 컨슈머는 5초마다 poll()을 호출할 때 가장 마지막 오프셋을 기록한다.

![IMG_1195](https://user-images.githubusercontent.com/25604495/93096858-ddf23780-f6df-11ea-9b66-3e05a796321b.JPG)

* 처음 poll()로 메시지 1,2 를 가져온다.
* 5초가 지난후 마지막 오프셋 2를 커밋한 다음 메시지 3,4 를 가져온다.
* 5초가 지난후 마지막 오프셋 4를 커밋한 다음 이후 메시지를 가져온다.

<br>

> 만약 커밋을 하기 전에 리밸런스가 일어난다면?

* 컨슈머02가 추가되면서 리밸런스가 된다.
* 파티션0에는 오프셋 4까지 커밋되어있다. 따라서 메시지 5,6을 중복으로 가져오게 된다.  
<br>  
* 자동 커밋은 편하지만 중복이 발생할 수 있다.


<br>

### 수동 커밋
* **컨슈머가 메시지를 가져온 이후 특정 처리가 완료될 때까지 커밋을 하면 안될때** 사용한다.
* 자동 커밋은 특정 주기로 poll하면서 커밋을 한다.

```java

public class KafkaBookConsumerMO {
  public static void main(String[] args) {
    Properties props = new Properties();
    props.put("bootstrap.servers", "peter-kafka001:9092,peter-kafka002:9092,peter-kafka003:9092");
    props.put("group.id", "peter-manual");
    props.put("enable.auto.commit", "false");//(1)
    props.put("auto.offset.reset", "latest");
    props.put("key.deserializer", "org.apache.kafka.common.serialization.StringDeserializer");
    props.put("value.deserializer", "org.apache.kafka.common.serialization.StringDeserializer");
    KafkaConsumer<String, String> consumer = new KafkaConsumer<>(props);
    consumer.subscribe(Arrays.asList("peter-topic"));
    while (true) {
      ConsumerRecords<String, String> records = consumer.poll(100);
      for (ConsumerRecord<String, String> record : records)
      {
        System.out.printf("Topic: %s, Partition: %s, Offset: %d, Key: %s, Value: %s\n", record.topic(), record.partition(), record.offset(), record.key(), record.value());
      }
      try {
        //(2) 필요한 동작 정의

        consumer.commitSync();//(3)
      } catch (CommitFailedException e) {
        System.out.printf("commit failed", e);
      }
    }
  }
}
```

* [1] : 수동 커밋 설정
* [2] : 커밋을 하기전에 필요한 동작을 정의
* [3] : 수동으로 커밋

