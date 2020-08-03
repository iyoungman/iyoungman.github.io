---
layout: post
title: 의존성 주입(Dependency Injection)
tags: DesignPattern DI
categories: DesignPattern
---

* TOC
{:toc}

## Summary
* 의존성 주입에 대해 알아본다.
* 간단한 예제를 통해 필요성을 알아본다.

<!--more-->
  
<br>  

## 의존성 주입이란?
* Dependency Injection(DI)
* 객체는 혼자서 모든 일을 처리하기 힘들기 때문에 다른 객체와 협력한다.
* 이때 객체 내부에서 다른 객체의 인스턴스를 생성하지 않고
* 외부에서 생성된 다른 객체의 인스턴스를 주입 받는것을
* 의존성 주입이라고 한다.  

> DI는 **인스턴스를 생성하는 책임과 사용하는 책임을 분리**하자는 의미와 같다.
>
> 이를 통해 유연한 설계를 할 수 있다.
>
> 또한, **OCP(개방-폐쇄 원칙)**과도 연관이 깊다.
  
<br>  


## DI 사용 안한 예제  

```java
import java.util.Calendar;

public class DateMessageProvider {
    public String getDateMessage() {
        Calendar now = Calendar.getInstance();
        int hour = now.get(Calendar.HOUR_OF_DAY);

        if (hour < 12) {
            return "오전";
        }
        return "오후";
    }
}

public class DateMessageProviderTest {
    @Test
    public void 오전() throws Exception {
        DateMessageProvider provider = new DateMessageProvider();
        assertEquals("오전", provider.getDateMessage());
    }

    @Test
    public void 오후() throws Exception {
        DateMessageProvider provider = new DateMessageProvider();
        assertEquals("오후", provider.getDateMessage());
    }
}
```

* Test를 제대로 할 수 없다.
* DateMessageProvider에 있는 Calendar의 시간을 변경할 수 있는 방법이 없기 때문이다.
> DateMessageProvider와 Calendar를 `강결합` 관계라고 말한다.
  
<br>  

## DI 사용한 예제  

```java
import java.util.Calendar;

public class DateMessageProvider {
    public String getDateMessage(Calendar now) {
        int hour = now.get(Calendar.HOUR_OF_DAY);

        if (hour < 12) {
            return "오전";
        }
        return "오후";
    }
}


public class DateMessageProviderTest {
    @Test
    public void 오전() throws Exception {
        DateMessageProvider provider = new DateMessageProvider();
        assertEquals("오전", provider.getDateMessage(createCurrentDate(11)));
    }

    @Test
    public void 오후() throws Exception {
        DateMessageProvider provider = new DateMessageProvider();
        assertEquals("오후", provider.getDateMessage(createCurrentDate(13)));
    }

    private Calendar createCurrentDate(int hourOfDay) {
        Calendar now = Calendar.getInstance();
        now.set(Calendar.HOUR_OF_DAY, hourOfDay);
        return now;
    }
}
```

* DateMessageProvider가 의존하고 있는 Calendar 인스턴스 생성을 DateMessageProvider가 결정하지 않고 외부에서 전달받는다.
* 이 처럼 좀 더 유연한 구조로 개발하는 것이 DI의 핵심이다.