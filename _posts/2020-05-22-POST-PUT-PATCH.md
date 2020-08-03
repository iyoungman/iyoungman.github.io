---
layout: post
title: POST, PUT, PATCH
tags: Network HTTP Idempotent
categories: Network
---

* TOC
{:toc}

## Summary
* 멱등성(Idempotent) 에 대해 알아본다.
* POST vs PUT 의 차이를 알아본다.
* PUT vs PATCH 의 차이를 알아본다.

<!--more-->
  
<br>  

## 멱등성(Idempotent)
* 연산을 여러번 수행해도 결과가 같은 경우.

> Ex) a = a + 1 -> 멱등하지 않다.
> 
> Ex) a = 1 -> 멱등하다.
  
<br>  

## POST vs PUT
* POST는 Create 연산, PUT은 Update 연산에 사용된다.
* 차이점을 좀 더 알아보자.

<br>

### 1. 멱등성
* **POST**의 경우 멱등하지 않다.  
* 요청시마다 새로운 게시글이 생성된다.  

<br>

* **PUT**의 경우 멱등하다.  
* 여러번 요청해도 같은 결과를 반환한다.


<br>

### 2. 요청시 식별자를 알고있는가?
* **POST**의 경우 요청할때 식별자를 보내지 않는다.
* 따라서 서버에서 생성한다.

```json
POST /board HTTP/1.1
{
    ...
}
```

<br>

* **PUT**의 경우 요청할때 식별자를 보낸다.
* 단, 해당 식별자가 존재하지 않을 경우.
    * 넘어온 식별자를 id로 사용할 수 있으면 POST와 같이 Create 한다. 
    * 넘어온 식별자를 id로 사용할 수 없으면 에러코드를 응답한다.  

```json
PUT /board/1 HTTP/1.1
{
    ...
}
```


<br>

## PUT vs PATCH
* 둘다 Update 성격을 갖고있다.  
<br>
* 하지만 아래와 같은 차이점이 있다.
* **PUT**은 자원의 모든 필드를 교체한다.
* **PATCH**는 자원의 일부를 교체한다.

<br>

* 요청 전  

```json
{
    title : "title",
    content : "content";
}
```

<br>

* PUT 요청 & 결과  

```json
PUT /board/1 HTTP/1.1
{
    title : "put";
}

// 결과
{
    title : "put",
    content : null
}
```

<br>

* PATCH 요청 & 결과  

```json
PATCH /board/1 HTTP/1.1
{
    title : "patch";
}

// 결과
{
    title : "patch",
    content : "content"
}
```

<br>  

## PATCH는 Idempotent 하지 않다

![image](https://user-images.githubusercontent.com/25604495/82633447-13a57900-9c36-11ea-9b3a-f75a1ecaffba.png)  

* 스펙에 의하면 PATCH는 멱등하지 않다고 되어있다. 왜 그럴까?
* PUT의 경우 전체를 Update 하지만
* PATCH의 경우 부분을 Update 하는 성격과 관련이 있다.

<br>  

### 예제

* 요청 전  

```json
{
    title : "title",
    content : "content";
}
```

<br>  

> Client X와 Client Y가 있다고 가정한다.

<br>  

* Client X 요청 & 결과  

```json
PATCH /board/1 HTTP/1.1
{
    title : "patchTitle";
}


// 결과
{
    title : "patchTitle",
    content : "content"
}
```

<br>  

* Client Y 요청 & 결과  

```json
PATCH /board/1 HTTP/1.1
{
    content : "patchContent";
}

// 결과
{
    title : "patchTitle",
    content : "patchContent"
}
```

<br>  

* Client X 다시 같은 요청 & 결과  

```json
PATCH /board/1 HTTP/1.1
{
    title : "patchTitle";
}

// 결과
{
    title : "patchTitle",
    content : "patchContent"
}
```

<br>  

* Client X는 같은 요청에 대해 다른 결과를 받았다.


<br>  


## Reference
* [https://multifrontgarden.tistory.com/245](https://multifrontgarden.tistory.com/245)
* [https://feel5ny.github.io/2019/08/16/HTTP_003_02/#8](https://feel5ny.github.io/2019/08/16/HTTP_003_02/#8)
* [https://stackoverrun.com/ko/q/8021222](https://stackoverrun.com/ko/q/8021222)
