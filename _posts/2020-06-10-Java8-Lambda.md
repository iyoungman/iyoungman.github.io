---
layout: post
title: 람다 표현식
tags: Java Java8 Lambda ModernJavaInAction
categories: Java
---

* TOC
{:toc}

## 람다란 무엇인가?
* 메서드로 전달할 수 있는 익명함수를 단순화한것.
* 코드를 **간결하게** 표현할 수 있다.

```java
//기존
Comparator<Apple> byWeight = new Comparator<Apple>() {
    @Override
    public int compare(Apple a1, Apple a2) {
        return Integer.compare(a1.getWeight(), a2.getWeight());
    }
};

//람다
Comparator<Apple> byWeight = 
        (Apple a1, Apple a2) -> Integer.compare(a1.getWeight(), a2.getWeight());
```

<!--more-->

<br>  

![image](https://user-images.githubusercontent.com/25604495/84050772-24e9d600-a9e9-11ea-94cc-ca2e0ed1bb00.png)  

<br>  

> 파라미터 리스트  

* Comaparator의 compare 메서드의 파라미터.

<br>  

> 화살표  

* 람다의 파라미터 리스트와 바디를 구분.

<br>  

> 람다 바디  

* 람다의 반환값에 해당하는 표현식.

<br>  

### 람다의 기본 문법

> i. 표현식 스타일  

* 중괄호 생략.
* return 생략.

```java
(parameters) -> expression

//예제
(String s) -> "Iron Man";
```

<br>  

> ii. 블록 스타일  

* 중괄호 사용.
* return 사용.

```java
(parameters) -> { statements; }

//예제
(String s) -> {return "Iron Man";}
```

<br>  

## 어디에, 어떻게 사용할까?
* **함수형 인터페이스라는 문맥에서만 사용할 수 있다.**

<br>  

### 함수형 인터페이스
* 하나의 추상 메서드만 갖는 인터페이스.
* 이때, 디폴트 메서드는 상관없다.  
<br>
* 람다표현식으로 함수형 인터페이스의 추상 메서드 구현을 전달할 수 있으므로<br>
**전체 표현식을 함수형 인터페이스의 인스턴스로 취급 할 수 있다.**

```java
Runnable r1 = () -> System.out.println("Hello");
r1.run();
```

<br>  

### 함수 디스크립터
* 함수 디스크립터는 람다 표현식의 시그니처를 서술하는 메서드.  
<br>
* 함수형 인터페이스의 추상 메서드 시그니처<br>
= 람다 표현식의 시그니처.  

```java
@FunctionalInterface
public interface Runnable {
    public abstract void run();
}
```

> 위의 Runnable 인터페이스는 인수와 반환값이 없는 시그니처.
>
> @FunctionallInterface는 함수형 인터페이스를 가리키는 어노테이션.
>
> 함수형 인터페이스가 아닌 경우 컴파일 에러를 잡는 역할이다.


<br>  

## 람다 활용 : 실행 어라운드 패턴
* 예제를 통해 람다의 활용에 대해 알아본다.

```java
public String processFile() throws IOException {
	try(BufferedReader br = new BufferedReader(new FileReader("data.txt"))) {
		return br.readLine();
	}
}
```

> 자원 처리(DB의 파일 처리)는 일반적으로 
>
> 자원을 열고, 처리한 후에, 자원을 닫는다.
>
> 여기서 '처리'를 제외하고 자원을 열고 닫는 부분은 대부분 비슷하다.

![image](https://user-images.githubusercontent.com/25604495/84054179-f6222e80-a9ed-11ea-9a04-7265319de9aa.png)  


<br>  

### 1단계 : 동작 파리미터화

> 위의 예제를 보면 현재 코드는 파일에서 한번에 한줄만 읽을 수 있다.
>
> 한번에 두 줄을 읽어야한다면?
>
> **처리하는 부분(processFile())만 다른 동작을 하게하면 좋을 것이다.**

<br>  

```java
String result = processFile((BufferedReader br) -> br.readLine() + br.readLine());
```

* 위와같이 procseeFile에 람다를 이용해 원하는 동작을 전달하면 좋을것이다.
* 이를 위해 함수형 인터페이스를 만들어야한다.  

<br>  

### 2단계 : 함수형 인터페이스
* 1단계 나온것처럼 람다를 전달하기 위해서는 함수형 인터페이스를 만들어야한다.
* **BufferReader -> String 반환, IOException**를 던지는 시그니처를 만든다.

```java
@FunctionalInterface
public interface BufferedReaderProcessor {
	String process(BufferedReader b) throws IOException;
}
```

<br>  

* processFile의 인수로 함수형 인터페이스를 전달한다.

```java
public String processFile(BufferedReaderProcessor p) throws IOException {
	
}
```

<br>  

### 3단계 : 동작 실행

```java
public String processFile(BufferedReaderProcessor p) throws IOException {
	try(BufferedReader br = new BufferedReader(new FileReader("data.txt"))) {
		return p.process(br);//실행
	}
}
```

<br>  

### 4단계 : 람다 전달

* 이제 람다를 통해 원하는 방식대로 processFile 메서드에 전달할 수 있다.
* 람다를 이용해 processFile 메서드를 유연하게 만들었다.

```java
//한줄
String oneLine = processFile((BufferedReader br) -> br.readLine());

//두줄
String twoLines = processFile((BufferedReader br) -> br.readLine() + br.readLine());
```

<br>  

## 함수형 인터페이스 사용
* Java 8의 API는 다양한 함수형 인터페이스를 포함하고 있다.
* java.util.function 패키지로 제공.
* 대표적으로 Predicate, Consumer, Function 등이 있다.

<br>  

### Predicate
* 추상 메서드 : test
* 시그니처 : 제네릭 타입의 인자 한개를 받아 boolean 값을 반환.

```java
//Predicate
@FunctionalInterface
public interface Predicate<T> {
    boolean test(T t);
}

//Main
public <T> List<T> filter(List<T> list, Predicate<T> p) {
	List<T> results = new ArrayList<>();
	for (T t: list) {
		if(p.test(t)) {
			results.add(t);
		}
	}
	return results;
}

Predicate<String> nonEmptyStringPredicate = (String s) -> !s.isEmpty();
List<String> nonEmpty = filter(listOfStrings, nonEmptyStringPredicate);
```

<br>  

### Consumer
* 추상 메서드 : accept
* 시그니처 : 제네릭 타입의 인자 한개를 받아 void를 반환.

```java
//Consumer
@FunctionalInterface
public interface Consumer<T> {
    void accept(T t);
}

//Main
public <T> void forEach(List<T> list, Consumer<T> c) {
	for (T t: list) {
		c.accept(t);
	}
}

forEach(
    Arrays.asList(1,2,3,4,5,),
    (Integer i) -> System.out.println(i)
);
```

<br>  

### Function
* 추상 메서드 : apply
* 시그니처 : 제네릭 타입의 인자 한개를 받고 제네릭 타입을 반환.

```java
//Function
@FunctionalInterface
public interface Function<T, R> {
    R apply(T t);
}

//Main
public <T, R> List<R> map(List<T> list, Function<T, R> f) {
	List<R> result = new ArrayList<>();
	for (T t: list) {
		result.add(f.apply(t));
	}
	return result;
}

List<Integer> l = map(
		Arrays.asList("lambdas", "in", "action"),
		(String s) -> s.length()
);
```

<br>  

### 그밖의 함수형 인터페이스

![함수](https://user-images.githubusercontent.com/25604495/84058921-d0e4ee80-a9f4-11ea-876d-f61b98790968.jpg)  


<br>  

## 형식 검사, 형식 추론, 제약
* 람다 표현식 자체에는 람다가 어떤 함수형 인터페이스를 구현하는지 정보가 포함되어 있지 않다.
* 람다의 실제 형식을 파악해본다.

<br>  

### 형식 검사
* 람다가 사용하는 콘텍스트를 통해 람다의 형식 추론.  

![형식](https://user-images.githubusercontent.com/25604495/84173088-bfb0e600-aab7-11ea-8254-fb91d7758811.jpg)

<br>  

### 같은 람다, 다른 함수형 인터페이스
* 대상 형식이라는 특징 때문에<br>
같은 람다 표현식이라도 여러 함수형 인터페이스로 사용될 수 있다.
  

```java
//둘다 인수를 받지 않고 제네릭 형식 T를 반환
Callable<Integer> c = () -> 42;
PrivilegedAction<Integer> p = () -> 42;
```

<br>  


### 지역 변수 사용
* 람다표현식도 익명 함수 처럼 파라미터로 넘겨진 변수가 아닌<br>
외부 변수(=자유 변수)를 활용할 수 있다.
* 이를 **람다 캡처링**이라고 한다.

```java
private Runnable method() {
	int num = 1000;
	return () -> System.out.println(num);
}
```

<br>  

> 하지만, 지역 변수에는 제약이 있다.  

* 명시적으로 final로 선언되어있거나,<br>
실질적으로 final로 선언된 변수와 똑같이 사용되어야한다.  
<br>  
* **즉, 람다표현식은 한 번만 할당된 지역 변수만 캡처할 수 있다.**

```java
//컴파일 에러
private Runnable method() {
	int num = 1000;
	num = 2000;
	return () -> System.out.println(num);
}
```

<br>  

> 외부에서 지역변수 값을 어떻게 참조할까?  

* 위 코드를 예로들면 method가 종료된 이후에는<br>
람다는 반환되고 num 변수는 스택 영역에서 사라질 것이다.<br>
따라서 외부에서 람다를 사용할 때 num 변수에 접근할 수 없을것이다.<br>
이 문제를 해결하기 위해, **자유 변수의 복사본**을 만들어 접근하도록한다.

<br>  

> 왜 final로 지역 변수를 제약해야할까?  

* 리턴된 람다식은 여러 스레드에서 사용할 수 있다.<br>
복사본의 값을 바꿀 수 있도록하면 동기화 처리가 필요할 것이다.<br>
따라서 복사본의 값이 바뀌지 않도록 final로 선언한다.

<br>  

> 인스턴스 변수 or 전역 변수는?  

* 힙 영역에 생성되므로 동일한 변수를 참조할 수 있다.
* 따라서 final로 선언하지 않아도 된다.

<br>

## 메서드 참조  
* 특정 메서들만을 호출하는 람다의 축약형.
* 람다표현식보다 메서드 참조를 통해 가독성을 높일 수 있다.
* 메서드명 앞에 :: 를 붙인다.  

<br>

```java
(Apple a) -> a.getWeight()

Apple::getWeight
```

> Apple 클래스에 정의된 getWeight의 메서드 참조.

<br>


## Reference
* [모던 자바 인 액션](http://www.yes24.com/Product/Goods/77125987?Acode=101)    
* [그림 참조](https://www.slideshare.net/chihwanchoi90/api-76487358)  