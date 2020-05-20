---
layout: post
title: HackerRank. Fraudulent Activity Notifications
tags: Algorithm HackerRank Sort Java
categories: Algorithm
---
## Problem
[https://www.hackerrank.com/challenges/fraudulent-activity-notifications/problem](https://www.hackerrank.com/challenges/fraudulent-activity-notifications/problem)  
  
<br>  

## Think
* 시간복잡도를 고려해야한다.
* 조건을 잘 봐야한다.  
![image](https://user-images.githubusercontent.com/25604495/82432563-3404f480-9acb-11ea-9095-862b01291cc0.png)  

> 시간복잡도를 고려하지 않으면 Before Code와 같이 풀 수 있다.
>
> 하지만 n과 d의 범위를 고려해야한다.
>
> `0 <= expenditure[i] <= 200` 의 조건을 갖고 있기 때문에
>
> expenditure[i]에 대한 counting 배열을 두고 매번 검사한다.
>
> 그러면 O(n) = 2 x 10의 5승 x 200 기 때문에 시간은 충분하다.  

* 나중에 다시한번 풀어보자.


<br>  


## Before Code
```java
// Complete the activityNotifications function below.
static int activityNotifications(int[] expenditure, int d) {
    int result = 0;

    int[] temp = new int[d];
    int start = d;
    for(int i = d; i  < expenditure.length; i++) {
        int index = 0;
        for(int j = i - d; j < i; j++) {
            temp[index++] = expenditure[j];
        }
        Arrays.sort(temp);

        if((double) expenditure[i] >= 2 * getMedian(temp)) {
            result++;
        }
    }

    return result;
}

private static double getMedian(int[] temp) {
    int mid = temp.length / 2;

    if(temp.length % 2 == 1) {
        return temp[mid];
    } else {
        return ((double)temp[mid] + temp[mid - 1]) / 2;
    }
}
```  

<br>

## Solved Code
```java
static int activityNotifications(int[] expenditure, int d) {
    int[] counts = new int[200 + 1];
    for(int i = 0; i < d; i++) {//(1)
        counts[expenditure[i]]++;
    }

    int result = 0;
    for(int i = d; i  < expenditure.length; i++) {
        double median = getMedian(counts, d);//(2)
        if(expenditure[i] >= (int)(median * 2)) {
            result++;
        }

        //(3)
        counts[expenditure[i - d]]--;
        counts[expenditure[i]]++;
    }

    return result;
}

private static double getMedian(int[] counts, int d) {
    double result = 0;
    int allCount = 0;
    int midIndex = d / 2;

    if (d % 2 == 0) {//짝수
        int mid1 = 0;
        int mid2 = 0;

        for(int i = 0; i < counts.length; i++) {
            if(counts[i] > 0) {
                allCount = allCount + counts[i];

                if(allCount > midIndex - 1 && mid1 == 0) {
                    mid1 = i;
                }
                if(allCount > midIndex && mid2 == 0) {
                    mid2 = i;

                    result = ((double) mid1 + mid2) / 2;
                    break;
                }
            }
        }
    } else {//홀수
        for(int i = 0; i < counts.length; i++) {
            if(counts[i] > 0) {
                allCount = allCount + counts[i];

                if(allCount > midIndex) {
                    result = (double) i;
                    break;
                }
            }
        }
    }

    return result;
}
```
  
<br>  

## Solved Process
1) 주어진 변수 d 범위 만큼 count를 구한다.

2) 중간값을 구한다.
> 이때 counts 배열을 이용한다.
>
> 문제의 조건에 맞게 d가 짝수일때, 홀수일때의 로직이 달라진다.

<br>

3) counts를 조정한다.
> 맨 앞에있는 index의 count를 -1
>
> 맨 뒤에있는 index의 count를 +1
