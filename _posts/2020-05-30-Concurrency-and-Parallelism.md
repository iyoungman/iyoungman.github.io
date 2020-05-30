---
layout: post
title: 동시성 vs 병렬성
tags: Theory Concurrency Parallelism
categories: Theory
---

* TOC
{:toc}
> 동시성과 병렬성의 차이점을 알아본다.

<br>  

## 동시성 vs 병렬성

<img src = "https://user-images.githubusercontent.com/25604495/82318976-b8903e00-9a0b-11ea-8428-3f44bfa0dffa.png" width="600" height="350" />

<br>  

### 동시성  
* 싱글코어에서 멀티 쓰레드를 동작시키는 것.  
* 동시에 실행되는 것처럼 보이지만, 여러 쓰레드가 **번갈아가면서** 실행되는 것이다.  

<br>  

### 병렬성  
* 멀티코어에서 멀티 쓰레드를 동작시키는 것.  
* 실제로 **동시에** 실행되는 것이다.  
* 병렬성은 데이터 병렬성과 작업 병렬성으로 나뉜다.  

<br>  

> 데이터 병렬성

* 하나의 작업을 병렬 처리하는 것.
* Ex) Java8의 병렬스트림.

<br>  

> 작업 병렬성

* 다른 작업을 병렬 처리하는 것.
* Ex) 웹서버에서 각 요청을 개별 스레드로 할당하여 처리.


<br>  

## At Java
* 동시성은 논리적인 소프트웨어 스레드와 관련이 있다.
* Java에서도 싱글코어에서 메모리가 허용하는 한 얼마든지 스레드를 만들 수 있다.  
<br> 
* 병렬성은 물리적인 하드웨어 스레드와 관련이 있다.
* 즉 CPU의 코어와 관련이 있다.
* 아래는 컴퓨터의 사양이다.  

<img src = "https://user-images.githubusercontent.com/25604495/82319746-17a28280-9a0d-11ea-812b-e5b6b7cacef4.PNG" width="600" height="400" />  

> 나의 경우, 쿼드 코어 + 한 코어당 2개의 스레드(하이퍼스레딩)를 가지므로 논리 프로세서는 8이다.

<br>  

* 따라서 Java8의 병렬스트림은 기본적으로 논리 프로세서의 수만큼<br>병렬로 작업을 실행한다.

> Runtime.getRuntime().availableProcessors()

<br>  

## 정리

![image](https://user-images.githubusercontent.com/25604495/83323561-ac716f80-a29a-11ea-9eb6-03864f324984.png)

<br>  

## Reference
* [https://www.codeproject.com/Articles/1267757/Concurrency-vs-Parallelism?msg=5575168](https://www.codeproject.com/Articles/1267757/Concurrency-vs-Parallelism?msg=5575168)  
* [https://atin.tistory.com/567](https://atin.tistory.com/567)