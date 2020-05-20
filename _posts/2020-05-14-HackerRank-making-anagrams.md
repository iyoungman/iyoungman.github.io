---
layout: post
title: HackerRank. Making Anagrams
tags: Algorithm HackerRank String Java
categories: Algorithm
---
## Problem
[https://www.hackerrank.com/challenges/ctci-making-anagrams/problem](https://www.hackerrank.com/challenges/ctci-making-anagrams/problem)  
  
<br>  

## Think
* 단순한 String 문제이다.  
  
<br>  


## Solved Code
```java
// Complete the makingAnagrams function below.
static int makingAnagrams(String s1, String s2) {
    char[] s1Array = s1.toCharArray();
    char[] s2Array = s2.toCharArray();

    int[] s1Count = getCount(s1Array);
    int[] s2Count = getCount(s2Array);

    int result = 0;
    for(int i = 0; i < 26; i++) {
        result = result + Math.abs(s1Count[i] - s2Count[i]);
    }

    return result;
}

private static int[] getCount(char[] array) {
    int[] count = new int[26];
    for(char c : array) {
        count[c - 97] = count[c - 97] + 1;
    }
    
    return count;
}
```  

