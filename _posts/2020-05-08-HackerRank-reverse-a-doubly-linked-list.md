---
layout: post
title: HackerRank. Reverse a doubly linked list
tags: Algorithm HackerRank LinkedList Java
categories: Algorithm
---

* TOC
{:toc}

## Problem
[https://www.hackerrank.com/challenges/reverse-a-doubly-linked-list/problem](https://www.hackerrank.com/challenges/reverse-a-doubly-linked-list/problem)    

<!--more-->
  
<br>  

## Think
* LinkedList의 Reserve를 직접 구현하는 문제이다.

<br>  

## Solved Code
  
```java  
// Complete the reverse function below.

static DoublyLinkedListNode reverse(DoublyLinkedListNode head) {
    DoublyLinkedListNode prevNode = null;
    DoublyLinkedListNode curNode = head;
    DoublyLinkedListNode nextNode = head.next;
    curNode.next = prevNode;

    while(nextNode != null) {
        prevNode = curNode;
        curNode = nextNode;
        nextNode = curNode.next;

        curNode.next = prevNode;
    }

    return curNode;
} 
```  
