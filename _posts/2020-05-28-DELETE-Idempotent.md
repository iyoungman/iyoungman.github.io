---
layout: post
title: HTTP DELETE는 멱등성을 가질까?
tags: HTTP DELETE Idempotent
categories: Network
---
* TOC
{:toc}

## Question
* 서버로 아래와 같은 DELETE 요청을 보낸다고 생각하자.

```json
DELETE /account/123 HTTP/1.1
```  

<br>  

* 서버에 존재하는 자원이라는 가정하에
* 첫번째 응답은 HTTP 200 번대가 오지만<br>재요청하면 응답은 400 번대가 올것이다. 
* 해당 자원이 없기 때문이다.  
<br>  
* **과연 HTTP Spec에 나온대로 DELETE가 멱등한것이 맞을까?**

<!--more-->

<br>  

## Answer
* DELETE 멱등하다.
* 멱등성에 대해 정확히 이해해야한다.

<br>  

> idempotency - you can send the request more than once without additional changes to the state of the server.

* Stackoverflow의 [답변](https://stackoverflow.com/questions/4088350/is-rest-delete-really-idempotent)중 일부는 위와 같다.  

<br>  

> Opinion

* 위 답변에서 **서버의 상태(state of server)** 가 중요하다고 생각한다.
* **Client가 아닌 Server의 입장에서 생각**하면 된다.
* Server의 경우 첫번째 요청이 와서 해당 자원이 삭제되었든
* 재요청이 왔을때 해당 자원이 없어 Client에게 에러코드를 응답해줬든
* 동일한 요청에 대한 상태는 같다.
