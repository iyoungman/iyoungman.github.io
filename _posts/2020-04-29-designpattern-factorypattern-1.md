---
layout: post
title: Factory 패턴 (1)
tags: DesignPattern FactoryPattern 생성사용분리
categories: DesignPattern
---

## Factory 패턴
* 객체의 `생성과 사용을 분리`하는 디자인 패턴
* 
* 왜 생성과 사용을 분리해야할까?
* 아래 예제를 통해 설명한다.

<br>  
  
## SimpleFactory
  
### Entity
* 치킨을 만든다고 생각해보자.  
* 치킨의 종류는 여러가지가 있으므로 아래와 같이 구성할 수 있다. 
  
```java
public interface Chicken {
    void bake();
}
```

```java
public class FriedChicken implements Chicken {

    @Override
    public void bake() {
        System.out.println("후라이드 굽기");
    }
}
```

```java
public class SeasoningChicken implements Chicken {

    @Override
    public void bake() {
        System.out.println("양념 굽기");
    }
}
```

```java
public class GrilledChicken implements Chicken {

    @Override
    public void bake() {
        System.out.println("그릴 굽기");
    }
}
```

<br>  
  
### Before
* 치킨을 만들어 사용하는 쪽을 `Client`라고 하자.
* 아래와 같이 Switch문 or If-Else문을 이용해서 생성하고
* 사용할 수 있을 것이다.

```java
public class Client {

    public static void main(String[] args) {
        Chicken chicken = null;

        //생성
        String type = "fried";
        switch (type) {
            case "fried":
                chicken = new FriedChicken();
                break;
            case "seasoning":
                chicken = new SeasoningChicken();
                break;
            case "grilled":
                chicken = new GrilledChicken();
                break;
            default:
                throw new RuntimeException("No Chicken");
        }

        //사용
        chicken.bake();
    }
}
```

* 하지만, Client가 여러개 있다고 생각해보자.
* 각 Client에서 Chicken을 생성해서 사용해야한다.  
<br>
* 어찌어찌해서 각 Client에서 Chicken을 생성했다고 가정하자.
* 새로운 종류의 치킨이 추가/삭제 되었을때
* `각 Client마다 수정을 해야한다.`  
> 위의 예제에서 GrilledChicken이 없어졌다고 생각해보자.
>
> 각 Client에서 Switch문을 수정해줘야 할 것이다.  
  
<br>
  
* 이는 생성의 책임을 분리하면 해결할 수 있다.
* Client는 가져와서 사용하는 책임만 가지면 되기 때문이다.
* 따라서 Factory 패턴을 이용하면 변경에 따른 비용을 최소화할 수 있다.
* 이외에도 인스턴스화 로직을 Factory 클래스 내부로 숨길 수 있다는 장점을 가질 수 있다.

<br>  
  
### After1 : SimpleFactory 구현
```java
public class SimpleChickenFactory {

    public static Chicken createChicken(String type) {
          switch (type) {
            case "fried":
                return new FriedChicken();
            case "seasoning":
                return new SeasoningChicken();
            case "grilled":
                return new GrilledChicken();
            default:
                throw new RuntimeException("No Chicken");
        }
    }

}
```

```java
public class Client {

    public static void main(String[] args) {
        //생성
        String type = "fried";
        Chicken chicken = SimpleChickenFactory.createChicken(type);

        //사용
        chicken.bake();
    }

}
```

<br>  
  
### After2 : 람다로 코드 리팩토링
* 람다 표현식을 이용하면 Switch문을 제거해서
* 더 유연한 코드로 리팩토링 할 수 있다.
* Client 부분은 After1과 같다.

```java
public class SimpleChickenFactory {

    private static Map<String, Supplier<Chicken>> chickenTypeMap = new HashMap<>();

    static {
        chickenTypeMap.put("fried", FriedChicken::new);
        chickenTypeMap.put("seasoning", SeasoningChicken::new);
        chickenTypeMap.put("grilled", GrilledChicken::new);
    }

    public static Chicken createChickenWithLambda(String type) {
        Supplier<Chicken> s = chickenTypeMap.get(type);
        if (s != null) {
            return s.get();
        }
        throw new RuntimeException("No Chicken");
    }
}
```

<br>  
  
## Next
* 위에서 간단한 Factory 패턴을 구현했다.
* 다음 포스팅에서는 구체화된 Factory 패턴의 2가지 종류를 알아본다.
> Factory 메서드 패턴  
> 추상 Factory 패턴

<br>  

## 참고
