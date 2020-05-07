---
layout: post
title: Equals and HashCode
tags: Java Eqauls HashCode HashMap
categories: Java
---
  
* TOC
{:toc}  
  
## Summary
* Java의 equals()와 hashCode()에 대해 알아본다.
* equals()와 hashCode()를 함께 재정의하는 이유를 알아본다.
  
<br>  

## equals()와 hashCode() 메서드
* 둘다 기본적으로 Object 클래스에 정의되어있다.
* 따라서 객체에 대한 동등 처리(equals())와
* 객체에 대한 해시값(hashCode())를 커스텀하고 싶을 때
* 재정의한다.

### equals()
* Object 클래스의 equals() 메서드이다.
```java  
public boolean equals(Object obj) {
        return (this == obj);
}
```
> 객체관의 동등 처리를 `주소값`으로 비교하고 있다.
>
> 단순 '값'으로 비교하고 싶다면 재정의(Override)한다.
  
<br>  

* 아래는 String 클래스에서 재정의한 equals() 메서드이다.
```java
 public boolean equals(Object anObject) {
        if (this == anObject) {
            return true;
        }
        if (anObject instanceof String) {
            String aString = (String)anObject;
            if (coder() == aString.coder()) {
                return isLatin1() ? StringLatin1.equals(value, aString.value)
                                  : StringUTF16.equals(value, aString.value);
            }
        }
        return false;
    }
```  

> 따라서, 우리는 String의 equals()를 이용할 때 주소값이 달라도 `값`이 같으면 true를 반환한다.
  
<br>  

### hashCode()
* 해시 코드란?
> 객체를 구분하기 위한 정수이다.

* Object 클래스에서는 객체의 메모리 번지를 이용해 해시코드를 반환한다.
  
<br>  

## equals를 재정의하려거든 hashCode도 재정의하라
* equals()를 재정의해서 두 객체가 같을경우 논리적으로 동등하다는 의미이다.
* 따라서 hashCode()도 재정의해서 동일한 해시코드가 반환되도록 해야한다.
  
<br>  

* 다음 예제를 보자.  

```java
public class Main {

    public static void main(String[] args) {

        //(1) oneValue = null;
        Map<One, String> oneMap = new HashMap<>();
        oneMap.put(new One(1), "Hello");
        String oneValue = oneMap.get(new One(1));

        //(2) twoValue = null;
        Map<Two, String> twoMap = new HashMap<>();
        twoMap.put(new Two(1), "Hello");
        String twoValue = twoMap.get(new Two(1));

        //(3) threeValue = "Hello"
        Map<Three, String> threeMap = new HashMap<>();
        threeMap.put(new Three(1), "Hello");
        String threeValue = threeMap.get(new Three(1));
    }

}

class One {
    private int number;

    public One(int number) {
        this.number = number;
    }
}

class Two {
    private int number;

    public Two(int number) {
        this.number = number;
    }

    @Override
    public boolean equals(Object o) {
        if (o instanceof Two) {
            Two other = (Two) o;
            if(number == other.number) {
                return true;
            }
        }
       return false;
    }
}

class Three {
    private int number;

    public Three(int number) {
        this.number = number;
    }

    @Override
    public boolean equals(Object o) {
        if (o instanceof Three) {
            Three other = (Three) o;
            if(number == other.number) {
                return true;
            }
        }
       return false;
    }

    @Override
    public int hashCode() {
        return number;
    }
}
```

> (2)의 경우 Key객체의 equals()만 재정의했다.
>
> 이때, twoMap.get(new Two(1))을 하면 "Hello"라는 value를 가져오지 못한다.
>
> 반면 Key객체의 equals(), hashCode()를 재정의한 (3)은 "Hello"를 가져온다.
  
<br>  

* HashMap, HashSet, HashTable 같이 해시 함수를 이용하는 자료구조에서는 아래와 같은 과정을 거친다.  
![image](https://user-images.githubusercontent.com/25604495/81193180-06fa1180-8ff6-11ea-826f-ad905a1b0c86.png)  
  
<br>  

* 이중 HashMap의 코드를 살펴보면 아래와 같다.

<img src = "https://user-images.githubusercontent.com/25604495/81195390-b2a46100-8ff8-11ea-8fe4-ebce006a6de5.PNG" width="500" height="50" />  
  
<img src = "https://user-images.githubusercontent.com/25604495/81195386-b1733400-8ff8-11ea-90b2-973ace700163.PNG" width="500" height="200" />  


> key를 통해 value를 가져오는 get() 메서드이다.
>
> 아래 네모로 표시해놓은 부분을 보면
>
>  해시 코드를 먼저 확인하고 equals()로 확인한다.
>
> 따라서 Key에 대한 hashCode()와 equals()의 결과가 모두 같아야 Value를 가져올 수 있다.

<br>  

## Next
* 참고로 HashMap은 하나의 해시 버킷에 여러 값이 저장될 수 있어 LinkedList 형태로 저장한다.
* 따라서 위 HashMap의 코드에서 Key의 해시 버킷을 찾고(hashCode())
* LinkedList에서 Key와 같은지 검증한 것이다.(equals())
* 다음 포스팅에서는 HashMap에 대해 좀 더 자세히 알아보겠다.



<br>  

## Reference
* [한빛미디어-이것이 자바다 그림 참조](https://slidesplayer.org/slide/14091546/)
