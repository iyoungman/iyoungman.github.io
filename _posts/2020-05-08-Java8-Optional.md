---
layout: post
title: Null 대신 Optional 클래스
tags: Java Null Optional ModernJavaInAction
categories: Java
---
  
* TOC
{:toc}  
  
## Summary
* Null의 문제점을 파악한다.
* Java8에서 나온 Optional을 Null을 대체할 방법을 알아본다.
  
<br>  

## Null의 문제점
* Null은 값이 없는 상황을 표현하는 것이다. 
* 하지만 다음과 같은 문제점이 있다.
> NullPointerException이 발생할 수 있다.
>
> NullPointerException은 RuntimeException이기 때문에 대비하지 않을경우 문제가 크다.
>
> 하지만 NullPointerException을 처리하는 코드는 복잡하다.
```java
public String getCarInsuranceName(Person person) {
    if (person != null) {
        Car car = person.getCar();
        if (car != null) {
            Insurance insurance = car.getInsurance();
            if (insurance != null) {
                return insurance.getName();
            }
        }
    }
    return "Unknown";
}
```

<br>  

## Optional 클래스 소개
* 선택형 값을 캡슐화하는 클래스이다.
> 선택형 값은 주어진 값을 가질수도 있고 안 가질 수도 있다.
>
> Optional 객체에 이러한 두 정보를 함께 저장하는 것이다.
>
> 값이 없을 때 Null에 할당하지 않아도 된다.

<br>  

* Null과 Optional.empty()의 차이점은?
> Optional은 NullPointerException에 안전하다.
>
> 또한 Optional은 객체이므로 활용이 가능하다.
  
<br>  

## Optional 적용 패턴

### Optional 객체 만들기
* 기본적으로 Optional 객체를 만드는 방법이다.
```java
Optional<Car> optCar = Optional.empty();
Optional<Car> optCar = Optional.of(car);
Optional<Car> optCar = Optional.ofNullable(car);
```
> 이중 Optional.of 와 Optional.ofNullable의 차이점은
>
> car가 Null일 경우 Optional.of()는 NullPointerException을 반환한다.
>
> 반면 Optional.ofNullable은 Optional.empty()를 반환한다.
>
> 즉, 인자의 객체가 Null인지 아닌지 모르는 상황에 쓰인다.
  
<br>  

### 맵으로 Optional의 값 추출, 변환
* 다음 예제를 보자.
```java
// Before
String name = null;
if(insurance != null) {
	name = insurance.getName();
}

// After
Optional<Insurance> optInsurance = Optional.ofNullable(insurance);
Optional<String> optInsurance = otpInsurance.map(Insurance::getName);
```
> Before와 같은 형태를 After의 형태로 사용할 수 있다.
>
> 이는 Optional과 map을 활용하는 방식이다.
>
> Optional 객체를 최대 요소 개수가 한 개 이하인 데이터 컬렉션으로 생각하면 된다.
>
> Optional이 비어있으면 map에서 아무 일도 일어나지 않는다.
  
<br>  


### Optional과 직렬화
* Optional의 설계 용도는 `선택형 반환값을 지원하는 용도`뿐이다.
* 따라서 Serializable 인터페이스 구현하지 않는다.
* 따라서 도메인 모델에 Optional의 기능과 직렬화를 모두 사용하고 싶다면
* Optional 값을 반환받을 수 있는 메서드를 추가하자.
```java
public class Person {
    private Car car;//Optional<Car> X

    public Optional<Car> getCarAsOptional() {
            return Optional.ofNullable(car);
    }
}
```
  
<br>  

### 디폴트 액션과 Optional 언랩
> get()
* 확실히 존재하는 상황에만 사용한다.
* 그렇지 않으면 Null처리 코드와 크게 다르지 않다.

<br>  

> orElse(T other)
* orElse는 값이 없는경우 인자인 other를 반환한다.

<br>  

> orElseGet(Supplier<? extends T> other)
* orElse의 게으른 버전이다.
* 값이 없는경우에서야 Supplier 를 수행하여 값을 반환한다.

<br>  

> orElseThrow(Supplier<? extends T> exceptionSupplier)
* orElseThrow는 값이 없는 경우 예외를 발생한다.

<br>  

> ifPresent(Consumer<? super T> consumer)
* ifPresent는 값이 존재할때만, 인자의 Consumer를 수행한다.

<br>  

> ifPresentOrElse(Consumer<? super T> action, Runnable emptyAction)
* 자바 9에서 추가된 메서드이다.
* ifPresent와의 차이점은 값이 없는 경우 Runnable을 실행한다.
  
<br>  

### orElse()와 orElseGet() 차이
* 코드로 확인하자.
```java
public static void main(String[] args) {
    String value = "VALUE";

    String result1 = Optional.of(value)
            .orElse(getEmptyStrWhenNull());//getEmptyStrWhenNull() 호출

    String result2 = Optional.of(value)
            .orElseGet(() -> getEmptyStrWhenNull());
}

private static String getEmptyStrWhenNull() {
    System.out.println("Call getEmptyStrWhenNull");
    
    return "EMPTY";
}
```
> result1과 result2 모두 결과가 "VALUE"로 같다.
>
> 하지만 `orElse의 경우 Optional의 값이(value) Null이든 아니든 getEmptyStrWhenNull() 메서드를 항상 호출한다.`
>
> orElse(T other)의 경우 T에대한 Type정보를 알고있어야 하기 때문이다.
> 
> 반면, orElseGet(Supplier<? extends T> other)의 경우 Optional의 값이 Null일 경우에만 Supplier를 실행한다.
>
> 따라서 orElseGet을 orElse의 게으른 버전이라고 하는 것이다.

<br>  

* 결론을 내리자면
> orElse와 orElseGet의 결과는 항상 같다.
>
> 하지만 orElseGet이 성능상의 이점을 가지고 있다.

<br>  

## Reference
* [모던 자바 인 액션](http://www.yes24.com/Product/Goods/77125987?Acode=101)
