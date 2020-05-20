---
layout: post
title: HackerRank. New year chaos
tags: Algorithm HackerRank Array Sort Java
categories: Algorithm
---

* TOC
{:toc}  

<br>

## Problem
[https://www.hackerrank.com/challenges/new-year-chaos/problem](https://www.hackerrank.com/challenges/new-year-chaos/problem)  
  
<br>  

## Think
* 처음에 푼 코드가 TimeOut이 발생하여 고민하다 다른 Solution을 참고했다.

* 다시 풀어볼 문제.  
  
<br> 

## Solved Code
```java
// Complete the minimumBribes function below.
static void minimumBribes(int[] q) {
    int length = q.length;

    //(1)
    for(int i = 0; i < length; i++) {
        int sub = q[i] - (i + 1);
        if(sub > 2) {
            System.out.println("Too chaotic");
            return;
        }
    }

    int count = 0;
    for(int i = 0 ; i < q.length ; i++) {
        if (i + 1 == q[i]) {
            continue;
        }

        //(2) 원래 순서가 아닐때
        int origin = i + 1; 
        for(int j = i + 1; j < q.length ; j++) {
            if(q[j] == origin) {
                swap(j - 1, j, q);
                
                count = count + 1;
                i--;//(3)
                break;
            }
        }
        
    }

    System.out.println(count);
}

private static void swap(int i, int j, int[] q) {
    int temp = q[i];
    q[i] = q[j];
    q[j] = temp;
}
```  
  
<br>   

## Solved Process
1) 뒷번호가 앞에 있는지 검증하는 과정에서 2번 초과로 뇌물을 줬으면 `Too chaotic`을 출력하고 종료한다.
> 핵심은 뒷번호만 앞번호에게 뇌물을 줄 수 있다는 것이다.
>
> 따라서 `int sub = q[i] - (i + 1);`와 같은 코드가 나온다.
>
> 나의 경우 처음에 `int sub = Math.abs((i + 1) - q[i])`로 해서 틀렸다.
>
> 앞번호는 뒤로 밀리다보면 충분히 2 초과로 떨어진 위치로 이동할 수 있다.

<br>

2) 원래 순서가 아닐경우 Swap 횟수를 세는 과정이다.
> 주의할점은 앞으로 한칸씩만 이동할 수 있다는 사실이다.  

<br>

3) 코드의 가독성을 위해 `i--`로 하였다.
> j에서 i까지 반복문을 돌며 count를 세도 될것같다.
