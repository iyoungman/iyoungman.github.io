---
layout: post
title: HackerRank. Sherlock and the Valid String
tags: Algorithm HackerRank String Java
categories: Algorithm
---
## Problem
[https://www.hackerrank.com/challenges/sherlock-and-valid-string/problem](https://www.hackerrank.com/challenges/sherlock-and-valid-string/problem)  
  
<br>  

## Think
* 다양한 상황을 고려해야하는 문제이다.  
* 문제를 읽고 조건에 맞게 Valid한 3가지 상황을 고려하여 풀었다.  
  
> `2.1 모든 Character의 Count가 같은 경우`
>
> ex) aabbcc -> a : 2, b : 2, c : 2

> `2.2 한 Character만 Count가 다른데 해당 Count가 1일 경우`
>
> ex) abbbccc -> a : 1, b : 3, c : 3 // a만 지우면 된다.
>
> 아래의 경우는 안된다.
>
> ex) aabbbccc -> a : 2, b : 3, c : 3

> `2.3 한 Character만 Count가 다른데 해당 Count가 가장 클 경우`
>
> ex) aabbccc -> a : 2, b : 2, c : 3 // c를 하나 지우면 된다.
>
> 아래의 경우는 안된다.
>
> ex) aabbcccc -> a : 2, b : 2, c : 4
<br>   

* 위 순서대로 검증을 하면 될것 같다.

<br>  


## Solved Code
```java
// Complete the isValid function below.
static String isValid(String s) {
    char[] sArray = s.toCharArray();
    Map<Character, Integer> countMap = new HashMap();

    for(char c : sArray) {
        int value = countMap.getOrDefault(c, 0);
        countMap.put(c, value + 1);
    }

    List<Integer> values = new ArrayList<Integer>(countMap.values());
    Collections.sort(values);//(1)

    //(2)
    if(isAllSame(values)) {
        return "YES";
    }

    //(3)
    if(values.get(0) != values.get(1) && values.get(0) == 1) {
        values.set(0, 0);
        if(isAllSame(values)) {
            return "YES";
        }
        values.set(0, 1);
    }

    //(4)
    int length = values.size();
    if(values.get(length - 2) != values.get(length - 1)) {
        values.set(length - 1, values.get(length - 1) - 1);
        if(isAllSame(values)) {
            return "YES";
        }
    }

    return "NO";
}

private static boolean isAllSame(List<Integer> values) {
    boolean isResult = true;

    int first = values.get(0) == 0 ? values.get(1) : values.get(0);
    for(int value : values) {
        if(value == 0) {
            continue;
        }

        if(first != value) {
            isResult = false;
            break;
        }
    }

    return isResult;
}
```  
  
<br>  

## Solved Process
1) 계산의 편의성을 위해 정렬을 했다.  
2) [Think](##Think)의 2.1 과정이다.  
3) [Think](##Think)의 2.2 과정이다.  
4) [Think](##Think)의 2.3 과정이다. 
