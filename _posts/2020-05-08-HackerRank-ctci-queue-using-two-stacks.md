---
layout: post
title: HackerRank. Queues - A Tale of Two Stacks
tags: Algorithm HackerRank Queue Stack
categories: Algorithm
---

* TOC
{:toc}
## Problem
[https://www.hackerrank.com/challenges/ctci-queue-using-two-stacks/problem](https://www.hackerrank.com/challenges/ctci-queue-using-two-stacks/problem)    
  
<br>  

## Think
* Stack 2개를 이용해 Queue의 기능을 구현하는 문제이다.
* 시간복잡도를 고려해서 풀어야한다.

<br>  

## Solved Code
  
```java  
class MyQueue<T> {
    Stack<Integer> one  = new Stack();
    Stack<Integer> two  = new Stack();

    public void enqueue(int num) {
        one.push(num);
    }

    public int dequeue() {
        if(two.isEmpty()) {
            while(!one.isEmpty()) {
                two.push(one.pop());
            }
        }
        return two.pop();
    }

    public int peek() {
        if(two.isEmpty()) {
            while(!one.isEmpty()) {
                two.push(one.pop());
            }
        }
        return two.peek();
    }
}
```  

<br>  

## Solved Process  
1) Stack one은 데이터를 넣는 Stack, Stack two는 데이터를 빼는 Stack으로 정했다.  
2) Queue는 FIFO, Stack은 LIFO 이기 때문에 Stack one의 데이터를 Stack two로 옮기는 과정이 필요하다.   
3) `이때, 옮기는 시점이 중요하다.`  
> MyQueue에 enqueue()할때마다 Stack two를 재정렬하면 효율성에 문제가 발생한다.  
>
> Stack two가 비었을 시점에만 Stack one의 데이터를 옮기면 된다.  
