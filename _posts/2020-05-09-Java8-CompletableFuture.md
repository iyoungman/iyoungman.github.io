---
layout: post
title: Java8. CompletableFuture
tags: Java Java8 CompletableFuture 비동기 ModernJavaInAction
categories: Java
---
  
## Summary
* Java8, Java9 에서는 코드를 비동기로 실행하는 방법을 제공한다.
* CompletableFuture와 리액티브 프로그래밍이 있다.
* 이중 CompletableFuture를 알아본다.
  
<br>  

## Java5의 Future
* Java5부터 비동기 실행을 위해 Future 인터페이스를 제공한다.
  
<br>  

1) Future의 장점
* 시간이 걸리는 작업을 Future 내부로 설정하여
* 별도의 스레드가 처리하도록 한다.
* 그러면 Future를 호출한 스레드는 별도의 작업을 처리할 수 있다.
* 즉, 동시성을 가질 수 있다.

<img src = "https://user-images.githubusercontent.com/25604495/81470678-fb416180-9226-11ea-9f30-391f9a076435.png" width="600" height="300" />
  
<br>  

* 코드로 확인하자.  

```java
import java.util.concurrent.Callable;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;
import java.util.concurrent.Future;
import lombok.extern.slf4j.Slf4j;

@Slf4j
public class Main {

    public static void main(String[] args) throws Exception {
        ExecutorService executor = Executors.newCachedThreadPool();
        Future<Void> future = executor.submit(new Callable<Void>() {
            @Override
            public Void call() throws Exception {
                return doSomethingLong();
            }
        });

        doSomethingElse();

        future.get();
    }

    private static Void doSomethingLong() {
        log.info("doSomethingLong");

        return null;
    }

    private static void doSomethingElse() {
        log.info("doSomethingElse");
    }
}
```  
  
```java
22:10:27.600 [pool-1-thread-1] INFO completablefuture.step0.Main - doSomethingLong
22:10:27.600 [main] INFO completablefuture.step0.Main - doSomethingElse
```  
  
<br>  

> Log를 통해 확인해보면
>
> Future를 통해 수행한 doSomethingLong() 메서드는 Main 스레드가 아닌
>
> 별도의 스레드를 통해 수행된 것을 확인할 수 있다.
  
<br>  

2) Future의 단점
* 다음과 같은 질의는 어떻게 처리할까?
> 오래걸리는 A라는 작업이 끝나면
>
> 그 결과를 다른 오래 걸리는 B라는 작업으로 전달하라.
>
> 그리고 B의 결과가 나오면 다른 질의 결과와 B의 결과를 조합하라.  
  
* Future를 이용하여 이러한 코드를 구현하는 일은 복잡하다.
* 이러한 복잡한 구현을 `선언적`으로 할 수 있다면 쉽게 구현하며, 코드의 가독성도 높을 것이다.
* Java8에서는 Future의 구현체인 CompletableFuture를 통해 Future를 선언적으로 사용할 수 있다.


<br>  

## 예제로 CompletableFuture 알아보기  
* CompletableFuture를 통한 비동기 API 구현에 대해 살펴볼 것이다.
* 예제는 다음과 같다.
> 상점에서 특정 상품의 가격을 검색하는 애플리케이션을 만들것이다.
>
> 가격 정보를 얻는 과정에서 외부 서비스에 접근해야한다고 가정한다.

<br>  

***  

### Step1. 동기 메서드로 구현
```java
import lombok.extern.slf4j.Slf4j;

@Slf4j
public class Main {

    public static void main(String[] args) {
        Shop shop = new Shop("Shop1");
        double result = shop.getPrice("면도기");

        doSomething();

        log.info("결과 출력 : {}", result);
    }

    private static void doSomething() {
        log.info("do Something");
    }
}
```

```java
import java.util.Random;
import lombok.extern.slf4j.Slf4j;

@Slf4j
public class Shop {

    private final String name;
    private final Random random;

    public Shop(String name) {
        this.name = name;
        random = new Random(name.charAt(0) * name.charAt(1) * name.charAt(2));
    }

    public double getPrice(String product) {
        return calculatePrice(product);
    }

    private double calculatePrice(String product) {
        delay();
        return random.nextDouble() * product.charAt(0) + product.charAt(1);
    }

    //시간이 1초 걸리는 외부 서비스라고 가정한다.
    public void delay() {
        int delay = 1000;

        try {
            log.info("{} 의 외부 서비스 호출", name);
            Thread.sleep(delay);
        } catch (InterruptedException e) {
            throw new RuntimeException(e);
        }
    }
}
```

> Main함수에서는 가격 검색 API(`getPrice()`)의 결과를 끝난후에
>
> 자신이 해야할일(`doSomething()`)을 수행한다.
>
> 가격 검색 API는 임의로 1초가 걸린다고 가정했다.
>
> 하지만 실제 서비스라면 몇초가 걸릴지 모르는 상황이므로
>
> Main에서 동기적으로 무작정 이 API를 기다리는 것은 옳지 못하다.

<br>  

***

### Step2. 동기 메서드를 비동기 메서드로 변환
* `Shop의 getPrice()` 메서드를 다음과 같이 바꾼다.  

```java
public Future<Double> getPriceAsync(String product) {
    CompletableFuture<Double> futurePrice = new CompletableFuture<>();

    new Thread(() -> {
        double price = calculatePrice(product);
        futurePrice.complete(price);
    }).start();

    return futurePrice;//계산 결과가 완료되길 기다리지 않고 Future를 우선 반환
}
```
> CompletableFuture의 complete()
>
> 계산이 완료되면 Future에 값을 설정한다.

```java
public static void main(String[] args) throws Exception {
    Shop shop = new Shop("Shop1");
    Future<Double> result = shop.getPriceAsync("면도기");//즉시 반환

    doSomething();

    log.info("결과 출력 : {}", result.get());//결과를 읽을때까지 블록된다
}
```
> 주석 표시한 부분은 Step1과 비교해 바뀐 부분이다.
>
> Log를 통해 확인해보면 역시 별도의 Thread를 생성해 처리함을 알 수 있다.

```java
00:50:14.484 [main] INFO completablefuture.step2.Main - do Something
00:50:14.484 [Thread-0] INFO completablefuture.step2.Shop - Shop Shop1 의 외부 서비스 호출
00:50:14.489 [main] INFO completablefuture.step2.Main - 결과 출력 : 70141.40833843076
```


<br>  

***

### Step2-1. 에러 처리하기
* Step2에서 Shop의 getPriceAsync()를 통해 별도의 스레드에서 일을 처리하고있다.
* `하지만, 예외가 발생하면 해당 스레드에만 영향을 미칠것이다.`
* 따라서 해당 스레드에서 발생한 문제를
* 호출자인 Main에 전달해야 한다.
* 이는 `CompletableFuture의 completeExceptionally` 메서드를 이용한다.

* Shop의 getPriceAsync()를 다음과 같이 바꾼다.
```java
public Future<Double> getPriceAsync(String product) {
    CompletableFuture<Double> futurePrice = new CompletableFuture<>();
    new Thread(() -> {
        try {
            double price = calculatePrice(product);
            futurePrice.complete(price);
        } catch (Exception ex) {
            //발생한 에러를 Future에 포함시킨 후 Future를 종료
            futurePrice.completeExceptionally(ex);
        }
    }).start();
    return futurePrice;
}
```


<br>  

***

### Step3. 비블록 코드 만들기
> 지금까지의 가정을 조금 바꿔본다.
>
> Step2까지는 Shop 클래스를 직접 비동기 방식으로 바꿨다.
>
> 하지만, Shop의 구현 권한이 우리에게 없으며(외부 API라고 생각)
>
> Shop의 getPrice()가 Step1처럼 동기 방식의 블록 메서드로 되어있다고 가정한다.  

<br>  

* 다음은 Step2와 Step3에서 구현할 코드의 차이점이다.
> 둘다 비동기 방식이다.
>
> 다만, Step2는 호출받는 쪽에서 비동기 방식으로 반환한다.
>
> Step3에서는 호출받는쪽에서 동기 방식으로 반환하지만,
>
> 호출하는 쪽에서 비동기 방식으로 호출 할 것이다.


```java
 public static void main(String[] args) throws Exception {
    //방법1 List라면 parrallelStream()

    Shop shop = new Shop("Shop1");
    //방법2 CompletableFuture로 비동기 호출
    CompletableFuture<Double> result = CompletableFuture.supplyAsync(() -> shop.getPrice("면도기"));

    doSomething();

    log.info("결과 출력 : {}", result.get());//join()
}

private static void doSomething() {
    log.info("do Something");
}
```

> Shop의 코드는 Step1과 같다.
>
> CompletableFuture의 supplyAsync()를 이용한다.

<br>  

### Step3.1 병렬 스트림 vs CompletableFuture
* 만약 List 형태로 여러 Shop을 처리한다고 생각해보자
* 이때 `병렬 스트림을 사용하는 방식과 CompletableFuture로 비동기 처리하는 것의 차이점이 있을까?
> 결론부터 말하면 CompletableFuture는 작업에 이용할 수 있는 Executor를 지정할 수 있다.
>
> Step3.2에서 확인하자.
  
<br>  

### Step3.2 CompletableFuture와 커스텀 Executor 사용하기

```java
public class Main {

    private static final int shopSize = 1;

    //Executor 생성
    private static final Executor executor = Executors.newFixedThreadPool(Math.min(shopSize, 100), (Runnable r) -> {
        Thread t = new Thread(r);
        t.setDaemon(true);
        return t;
    });

    public static void main(String[] args) throws Exception {
        Shop shop = new Shop("Shop1");
        CompletableFuture<Double> result = CompletableFuture.supplyAsync(() -> shop.getPrice("면도기"), executor);//Executor 적용

        doSomething();

        log.info("결과 출력 : {}", result.get());
    }

    private static void doSomething() {
        log.info("do Something");
    }

}
```

> 주석 표시한 부분은 Step3과 비교해 바뀐 부분이다.
>
> Executor를 커스텀하게 생성해서 CompletableFuture.supplyAsync에 적용했다.
>
> 주목할 부분은 스레드 풀의 개수를 Math.min(shopSize, 100)만큼 생성했다.
>
> `이는 1개의 Shop을 처리하든 100개의 Shop을 처리하든 같은 시간이 소요된다는 의미이다.`
  
<br>  

* 적절한 스레드 풀 크기 조절 참고  
> [https://12bme.tistory.com/368](https://12bme.tistory.com/368)
  
<br>  

### 병렬 스트림 vs CompletableFuture 비교 

1) 스레드 풀  
> 병렬 스트림을 이용하면
>
> 스레드 풀의 Default 개수인 Runtime.getRuntime().availableProcessors()

> CompletableFuture는
>
> 스레드 풀 개수를 커스텀하여 적용 할 수 있다.
  
<br>  

2) 어디에 사용할까?
> 병렬 스트림의 경우
>
> I/O가 포함되지 않은 계산 중심의 동작 실행.
>
> 모든 스레드가 계산 작업을 수행할 때는 프로세서 코어 수 이상의 스레드를 가질 필요가 없다.
>
> 또한 구현도 간편하다.

> CompletableFuture는
>
> I/O를 기다리는 작업을 병렬로 실행할때.
>
> 스레드의 유연성을 설정할 수 있기 때문이다.


<br>  

## Reference  
* [모던 자바 인 액션](http://www.yes24.com/Product/Goods/77125987?Acode=101)  
* [그림 참조](http://blog.engineering.publicissapient.fr/wp-content/uploads/2020/03/Future.png)  

<br>  

* TOC
{:toc}  
