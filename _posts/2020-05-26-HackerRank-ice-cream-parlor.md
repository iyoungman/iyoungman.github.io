---
layout: post
title: HackerRank. Ice Cream Parlor
tags: Algorithm HackerRank HashMap Java
categories: Algorithm
---

* TOC
{:toc}
> HashMap 문제.

<br>  

## Problem  
[https://www.hackerrank.com/challenges/ctci-ice-cream-parlor/problem](https://www.hackerrank.com/challenges/ctci-ice-cream-parlor/problem)  
  
<br>  

## Think  
* 단순 반복으로 풀면 시간복잡도 O(n²) 이 나올것이고<br> 아래 조건에 따라 Timeout이 될 것이다.  

![image](https://user-images.githubusercontent.com/25604495/82860473-b6bb0300-9f54-11ea-8ed7-c9280530bca3.png)  
<br>
* 처음에는 별도의 배열을 만들어 index를 저장했다.(Before Code)

> indexOfCost[cost[i]] = i + 1;

* Test Case는 맞았지만, 배열 범위가 money이기 때문에 <br>Heap 관련 Exception이 발생했다.

* 따라서 배열을 Map으로 바꿔 해결했다.(Solved Code)  
<br> 
* 직접 해결한 문제.

<br>  

## Before Code

```java
// Complete the whatFlavors function below.
static void whatFlavors(int[] cost, int money) {
    int[] indexOfCost = new int[money];

    for(int i = 0; i < cost.length; i++) {
        if(cost[i] < money) {
            if(indexOfCost[cost[i]] != 0 && cost[i] * 2 == money) {
                System.out.println(indexOfCost[cost[i]] + " " + (i + 1));                      return;
            }
            indexOfCost[cost[i]] = i + 1;
        }
    }

    for(int i = 1; i <= money / 2; i++) {
        if (indexOfCost[i] != 0 && indexOfCost[money - i] != 0) {
            int minIndex = Math.min(indexOfCost[i], indexOfCost[money - i]);
            int maxIndex = Math.max(indexOfCost[i], indexOfCost[money - i]);
            
            System.out.println(minIndex + " " + maxIndex);
            return;
        }
    }
}
```
  
<br>  


## Solved Code  

```java
// Complete the whatFlavors function below.
static void whatFlavors(int[] cost, int money) {
    Map<Integer, Integer> indexOfCostMap = new HashMap();

    for(int i = 0; i < cost.length; i++) {
        if(cost[i] < money) {
            if(indexOfCostMap.containsKey(cost[i]) && cost[i] * 2 == money) {//(1)
                System.out.println(indexOfCostMap.get(cost[i]) + " " + (i + 1));                      
                return;
            }
            indexOfCostMap.put(cost[i], i + 1);
        }
    }

    Set<Integer> keySet = indexOfCostMap.keySet();
    for(int i : keySet) {
        if (indexOfCostMap.containsKey(i) && indexOfCostMap.containsKey(money - i)) {//(2)
            int minIndex = Math.min(indexOfCostMap.get(i), indexOfCostMap.get(money - i));
            int maxIndex = Math.max(indexOfCostMap.get(i), indexOfCostMap.get(money - i));
            
            System.out.println(minIndex + " " + maxIndex);
            return;
        }
    }
}
```  
  
<br>  

## Solved Process  

1) 같은 Key 값에 대해서 다른 Value 값이 나올 경우, 해당 Key 값을 * 2 한 값이 money인지 검사한다.  

> 어차피 답은 2개의 index를 출력하는 것이기 때문에
>
> 동일한 Key 값에 대해 여러 Value 값을 저장할 필요가 없다.
>
> Map<Integer, Integer> 이면 충분하다.

<br>

2) 정답의 조건에 만족하면 출력한다.
