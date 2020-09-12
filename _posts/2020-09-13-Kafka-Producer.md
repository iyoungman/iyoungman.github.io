---
layout: post
title: 카프카 프로듀서
tags: Kafka 카프카데이터플랫폼의최강자
categories: Kafka
---

* TOC
{:toc}

## 프로듀서의 역할
* 각각의 메시지를 토픽 파티션에 매핑한다.
* 파티션의 리더에 요청을 보낸다.

<!--more-->

<br>

## 자바를 이용한 프로듀서

```java
public class KafkaBookProducer1 {
  public static void main(String[] args) {
    Properties props = new Properties();//(1)
    props.put("bootstrap.servers", "peter-kafka001:9092,peter-kafka002:9092,peter-kafka003:9092");//(2)
    props.put("key.serializer", "org.apache.kafka.common.serialization.StringSerializer");//(3)
    props.put("value.serializer", "org.apache.kafka.common.serialization.StringSerializer");

    Producer<String, String> producer = new KafkaProducer<>(props);//(4)
    producer.send(new ProducerRecord<String, String>("peter-topic", "Apache Kafka is a distributed streaming platform"));//(5)
    producer.close();
  }
}
```

* [1] : Property 생성
* [2] : 브로커 리스트 정의
* [3] : Key, Value Serializer 설정
* [4] : 프로듀서 생성
* [5] : send()를 이용해 토픽에 메시지를 전송

<br>

### 1.메시지를 보내고 확인하지 않기

```java
Producer<String, String> producer = new KafkaProducer<>(props);
try {
    producer.send(new ProducerRecord<String, String>("peter-topic", "Apache Kafka is a distributed streaming platform"));
} catch (Exception exception) {
    exception.printStackTrace();
} finally {
    producer.close();
}
```

* 메시지 손실 가능성이 있어 사용하지 않는다.

<br>

### 2.동기 전송

```java
Producer<String, String> producer = new KafkaProducer<>(props);
try {
    RecordMetadata metadata = producer.send(new ProducerRecord<String, String>("peter-topic", "Apache Kafka is a distributed streaming platform"))
            .get();
    System.out.printf("Partition: %d, Offset: %d", metadata.partition(), metadata.offset());
} catch (Exception exception) {
    exception.printStackTrace();
} finally {
    producer.close();
}
```

* send() 메소드의 Future 객체를 리턴한다.
* Future의 get() 메소드를 통해 기다린다. 이를 통해 성공/실패 여부를 알 수 있다.
* 메시지가 보내기 전, 보내는 동안의 Exception 을 잡을 수 있다.

<br>

### 3. 비동기 전송

```java
Producer<String, String> producer = new KafkaProducer<>(props);
try {
    producer.send(new ProducerRecord<String, String>("peter-topic", "Apache Kafka is a distributed streaming platform"),
            new PeterCallback());
} catch (Exception exception) {
    exception.printStackTrace();
} finally {
    producer.close();
}

//콜백
class PeterCallback implements Callback {
	@Override 
    public void onCompletion(RecordMetadata metadata, Exception exception) {
		if (metadata != null) {
			System.out.println("Partition : " + metadata.partition() + ", Offset : " + metadata.offset() + "");
		} else {
			exception.printStackTrace();
		}
	}
}
```

* send() 메소드를 콜백과 같이 호출한다.
* 비동기 전송을 통해 빠른 전송이 가능하다.

<br>

### 특정 파티션에 전송하기

```java
if (i % 2 == 1) {
    producer.send(new ProducerRecord<String, String>(testTopic, oddKey, "message"));
} else {
    producer.send(new ProducerRecord<String, String>(testTopic, evenKey, "message"));
}
```

* Key를 지정해 testTopic의 특정 파티션으로 메시지를 보낼 수 있다.
* Key를 지정하지 않으면 라운드 로빈 방식으로 메시지를 보낸다.

<br>

## 프로듀서 주요 옵션

> bootstrap.servers

* 카프카 클러스터에 연결 하기 위한 호스트와 포트 정보로 구성된 리스트 기입.
* **카프카 클러스터 내의 전체 호스트 정보를 입력하는것이 좋다.**

<br>

> acks

* 프로듀서가 카프카 토픽의 리더에게 메시지를 보낸 후 요청을 완료하기 전 ack의 수.

<br>

* [acks=0] 프로듀서는 서버로부터 어떠한 ack도 기다리지 않는다.<br>
서버의 응답을 기다리지 않기 때문에 빠르지만 메시지가 손실될 수 있다.

* [acks=1] 클러스터의 리더는 데이터를 기록하지만, 모든 팔로워는 확인하지 않는다. <br> 
데이터 일부 손실이 있을 수 있다.

* [acks=all 또는 -1] 클러스터의 리더는 ISR의 팔로워로부터 데이터에 대한 ack를 기다린다.<br> 데이터의 무손실을 보장한다.

<br>

> retries

* 전송에 실패한 데이터를 다시 보내는 횟수를 의미.

<br>

> batch.size

* 프로듀서는 같은 파티션으로 보내는 여러 데이터를 함께 배치로 보낸다.<br>
성능 향상에 도움이 될수 있다.<br>
해당 옵션의 Value에 배치 크기 바이트를 기입한다.

<br>

> 그외의 설정

* [https://docs.confluent.io/current/installation/configuration/producer-configs.html](https://docs.confluent.io/current/installation/configuration/producer-configs.html)

<br>

## 메시지 전송 방법

* 프로듀서의 옵션 중 **acks 옵션에 따라 메세지 손실 여부, 처리량이 달라 진다.**

<br>

### 1. 메시지 손실 가능성 높지만 빠른 속도의 전송
* acks=0
* 프로듀서는 메시지를 전송할 때 카프카 서버의 응답을 기다리지 않는다.
* 따라서 속도는 빠를 수 있지만 메시지 손실 가능성이 비교적 크다.

<br>

### 2. 메시지 손실 가능성이 적고 적당한 속도의 전송
* acks=1
* Default 값이다.
* 프로듀서는 카프카의 응답을 기다린다.  

![IMG_1183 2](https://user-images.githubusercontent.com/25604495/93000195-a067a000-f561-11ea-862b-640d001af751.JPG)

![IMG_1184](https://user-images.githubusercontent.com/25604495/93000194-9f367300-f561-11ea-9407-966817cfad7c.JPG)


<br>

> 그렇다면 acks=1에서는 메시지가 손실이 제로일까?

* 아니다. 손실이 나는 경우가 있다.

![IMG_1185](https://user-images.githubusercontent.com/25604495/93000191-9e054600-f561-11ea-8545-bf1e760a60c8.JPG)

![IMG_1186](https://user-images.githubusercontent.com/25604495/93000190-9c3b8280-f561-11ea-8d75-63c1cf4d8438.JPG)  

* [1] 리더는 acks를 보내고 장애 발생.
* [2] 팔로워는 메시지를 가져올 수 없다.
* [3] 리더가 다운되었으므로 팔로워중 한명이 리더가 된다.
* [4] 따라서 메시지 손실이 발생한다.

<br>

### 3. 메시지 손실이 없지만 느린 속도의 전송
* acks=all
* 팔로워까지 메시지를 받았는지 확인 후 acks를 보낸다.





