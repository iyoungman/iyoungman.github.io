---
layout: post
title: Java8. 병렬 데이터 처리와 성능
tags: Java Java8 ParallelStream ForkJoinPool ModernJavaInAction
categories: Java
---

* TOC
{:toc}  
> 병렬 스트림에 대해 알아본다.
>
> 포크/조인 프레임워크를 알아본다.
  
<br>  

## Java7 이전 vs 이후
* Java7 이전에는 데이터 컬렉션을 병렬 처리하기 힘들었다.
* 다음은 Java7 이전에 데이터 컬렉션을 병렬 처리하는 과정이다.

> 데이터를 서브 파트로 분할한다.
>
> 분할된 서브 파트를 각각의 스레드로 할당한다.
>
> 스레드로 할당한 후 적절한 동기화 처리를 한다.
>
> 마지막으로 부분 결과를 합친다.
 
<br>  


* Java7 부터는 위의 기능을 지원하는 **포크/조인 프레임워크** 를 기능을 제공한다.
* 또한 Java8의 병렬 스트림은 내부적으로 포크/조인 프레임워크를 사용한다.
 
<br>  

## 병렬 스트림

### 정의
* 스트림 요소를 분할하여 각각의 스레드에서 처리하도록 하는것.

![image](https://user-images.githubusercontent.com/25604495/83319428-504a2380-a279-11ea-8d75-ee229043c61a.png)

<br>

### 사용
* 컬렉션에 **parallel()** 혹은 **parallelStream()** 을 호출한다.  

```java
public long parallelSum(long n) {
    return Stream.iterate(1L, i -> i +1)
            .limit(n)
            .parallel()
            .reduce(0L, Long::sum);
}
```

<br>

> parallel을 호출하면 병렬 불리언 플래그 설정된다.
>
> 최종연산에서 플래그를 확인하여 처리된다.

![image](https://user-images.githubusercontent.com/25604495/83319281-0c0a5380-a278-11ea-84e8-9879cae9a6fb.png)  

  
<br>

* 만약 parallel과 sequential 두개를 모두 사용했을 때에는<br>최종적으로 호출된 메서드로 전체 스트림 파이프라인에 영향이 미친다.
  
<br>


### 내부
* 병렬스트림으로 사용하는 스레드는 ForkJoinPool을 사용한다.
* ForkJoinPool은 **CPU 코어 개수**와 관련있다.
* 정확하게는 **하이퍼스레딩**과 관련된 가상 프로세서 개수이다.

```java
Runtime.getRuntime().availableProcessors()
```

  
<br>


### 효과적으로 사용하기

> 병렬 스트림은 순차 스트림보다 성능이 떨어질 수도 있다.
>
> 아래와 같은 요소를 고려하자.

1) 직접 성능을 측정하라.  

<br>  

2) 박싱을 주의  

<br>  

3) 요소의 순서와 관련있는 연산 피하기
* limit, findFirst  

<br>  

4) 소량의 데이터의 경우 사용하지 말자.  

<br>  

5) 스트림을 구성하는 자료구조 확인


<br>


## 포크/조인 프레임워크
* Java7에 추가된 기능.
* Java8 병렬스트림에서는 내부적으로 포크/조인 프레임워크로 처리.

<br>  

### RecursiveTask 활용
* ForkJoinPool을 이용하려면 RecursiveTask<R>의 서브클래스를 만든다.
* RecursiveTask의 추상 메서드 compute를 구현한다.  
<br>  
* compute의 의사코드이다.

```java
if (태스크가 충분히 작거나 더 이상 분할 할 수 없으면) {
	순차적으로 태스크 계산
} else {
    태스크를 두 서브태스크로 분할
    태스크가 다시 서브태스크로 분할되도록 이 메서드를 재귀적으로 호출함
    모든 서브태스크의 연산이 완료될 때까지 기다림
    각 서브태스크의 결과를 합침
}
```

<br>  

* long[] 의 sum을 구하는 것을 포크/조인 프레임워크를 사용한 예제이다.

```java
public class ForkJoinSumCalculator extends RecursiveTask<Long> {

    public static final ForkJoinPool FORK_JOIN_POOL = new ForkJoinPool();
    public static final long THRESHOLD = 10_000;

    private final long[] numbers;
    private final int start;
    private final int end;

    public ForkJoinSumCalculator(long[] numbers) {
        this(numbers, 0, numbers.length);
    }

    private ForkJoinSumCalculator(long[] numbers, int start, int end) {
        this.numbers = numbers;
        this.start = start;
        this.end = end;
    }

    @Override
    protected Long compute() {
        int length = end - start;
        if (length <= THRESHOLD) {
            return computeSequentially();
        }
        ForkJoinSumCalculator leftTask = new ForkJoinSumCalculator(numbers, start, start + length / 2);
        leftTask.fork();
        ForkJoinSumCalculator rightTask = new ForkJoinSumCalculator(numbers, start + length / 2, end);
        Long rightResult = rightTask.compute();
        Long leftResult = leftTask.join();
        return leftResult + rightResult;
    }

    private long computeSequentially() {
        long sum = 0;
        for (int i = start; i < end; i++) {
            sum += numbers[i];
        }
        return sum;
    }

    public static long forkJoinSum(long n) {
        long[] numbers = LongStream.rangeClosed(1, n).toArray();
        ForkJoinTask<Long> task = new ForkJoinSumCalculator(numbers);
        return FORK_JOIN_POOL.invoke(task);
    }

}
```

<br>  

* 이를 그림으로 나타내면 아래와 같다.  

![Untitled](https://user-images.githubusercontent.com/25604495/83319519-3ceb8800-a27a-11ea-8734-4eadbfe5bbb6.png)

<br>  


### 포크/조인 프레임워크를 제대로 사용하는 방법
1) 두 서브태스크가 시작된 다음에 join 호출.
* join은 태스크의 결과가 처리될 때까지 기다린다.  
<br>  
2) RecursiveTask 내에서는 invoke() 메서드 사용하지 말아야한다.
* 순차 코드에서 병렬 계산을 시작할때만 invoke()를 사용한다.  
* invoke()는 Task를 Pool에 제출하는 메서드이다.  
<br>  
3) 한쪽 작업에는 fork, 한쪽 작업에는 compute 가 효율적.
* compute 쪽에서는 같은 스레드를 재사용.  

<br>  

### 작업 훔치기
* 일반적으로 멀티 스레드로 처리를 하면 각 작업의 완료 시간이 다르다.
* 이 경우, 어떤 스레드는 바쁜 상태인데<br>어떤 스레드는 할일이 없는 상태가 될 수 있다.  
<br>  
* 포크/조인 프레임워크는 **작업 훔치기** 기법으로 이 문제를 해결한다.
* 작업 훔치기는 각각의 태스크에 작업을 공정하게 분할한다.  

![image](https://user-images.githubusercontent.com/25604495/83319723-e0896800-a27b-11ea-80ef-d0aef15362ed.png)



<br>  

## Spliterator 인터페이스
* Java8부터 제공하는 인터페이스.
* **스트림을 분할하여 처리할때 사용.**
* 스트림의 요소를 커스텀하게 분할하고 싶을때 구현하면된다.  

<br>

### 메서드

* 다음은 Spliterator 인터페이스에서 필수로 구현해야하는 메서드이다.

```java
public interface Spliterator<T> {
    boolean tryAdvance(Consumer<? super T> action);
    Spliterator<T> trySplit();
    long estimateSize();
    int characteristics();
}
```

<br>

> boolean tryAdvance(Consumer<? super T> action);

* Spliterator의 요소를 하나씩 순차적으로 소비하면서 탐색해야 할 요소가 남아 있으면 참을 반환.

<br>

> Spliterator<T> trySplit();

* Spliterator의 일부 요소를 분할해서 두 번째 Spliterator를 생성하는 메서드.

<br>

> long estimateSize();

* 탐색해야 할 요소 수 정보를 제공.

<br>

> int characteristics();

* Spliterator에서 정의한 int형으로 Spliterator의 특성을 나타낸다.  

![KakaoTalk_20200530_195210607](https://user-images.githubusercontent.com/25604495/83326459-30355700-a2af-11ea-9936-3834e8322d8a.jpg)



<br>


### 분할 과정

* trySplit()의 결과가 null을 반환할때까지 재귀적으로 반복.
* 분할 과정은 characteristics() 메서드로 정의되는 Spliterator의 특성에 영향.  

![image](https://user-images.githubusercontent.com/25604495/83326258-7db0c480-a2ad-11ea-92b0-408724055d9a.png)




<br>  

## Reference  
* [모던 자바 인 액션](http://www.yes24.com/Product/Goods/77125987?Acode=101)  
* [그림 참조](https://yongho1037.tistory.com/705)  
* [https://stackoverflow.com/questions/34132326/forkjoinpool-invoke-and-forkjointask-invoke-or-compute](https://stackoverflow.com/questions/34132326/forkjoinpool-invoke-and-forkjointask-invoke-or-compute)
* [https://livebook.manning.com/book/java-8-in-action/chapter-7/140](https://livebook.manning.com/book/java-8-in-action/chapter-7/140)
