---
layout: post
title: Minor GC 와 Major GC
tags: Java
categories: Java GC 
---

<aside markdown="1">
* TOC
{:toc}
</aside>
## Summary
* JVM Heap의 Young, Old 영역에 대해 알아본다.
* Minor GC와 Major GC에 대해 알아본다.  
* GC 방식에 대해 알아본다.
  
<br>  

## Heap의 Young, Old 영역
* 가비지 콜렉터는 Heap 영역에서 사용하지 않는 객체를 메모리에서 제거한다.
* Heap에는 크게 2개의 물리적 공간으로 나뉜다.
    * Young Generation 영역(이하 Young 영역)
    * Old Generation 영역(이하 Old 영역)  

![image](https://user-images.githubusercontent.com/25604495/82313897-f5583700-9a03-11ea-9ba8-23f7b8e21268.png)  
  
<br>  

### Young 영역
* 새롭게 생성한 객체가 위치하는 영역이다.
* 이 영역에서 객체가 사라질 때 `Minor GC`가 발생한다고 말한다.
* Young 영역은 3개의 영역으로 나뉜다.
    * Eden 영역
    * Survivor 영역(2개)

<br>  

### Old 영역
* Young 영역에서 살아남은 객체가 복사되는 영역이다.
* 대부분 Young 영역보다 크게 할당하기 때문에 크기가 큰 만큼 Young 영역보다 GC는 적게 발생한다. 
* 이 영역에서 객체가 사라질 때 `Major GC`가 발생한다고 말한다.

<br>  

## GC 과정
* 아래 예제를 통해 GC 과정에 대해 알아본다.  
  
1) Eden 에서 Survivor 영역 이동
> 새로 생성한 대부분의 객체는 Eden 영역에 위치한다.
>
> 여러 객체를 생성했다고 하자.
>
> A, B, C, D, E, F 라고 부르겠다.  

![image](https://user-images.githubusercontent.com/25604495/82351378-7aa90f00-9a37-11ea-8d93-fc477b3227ec.png)  

<br>

***

> Eden 영역이 꽉 찼으므로 Minor GC가 발생한다.
>
> Minor GC가 발생한 후 살아남은 객체는 Survivor 영역 중 하나로 이동한다.
>
> A, B, C가 살아남았다고 가정한다.

![image](https://user-images.githubusercontent.com/25604495/82351635-ca87d600-9a37-11ea-9ba3-f70d4ffba936.png)

<br>

***

> Eden 영역에 새로운 객체 H, I, J가 생성되었다.

![image](https://user-images.githubusercontent.com/25604495/82352199-952fb800-9a38-11ea-82fc-291919556757.png)  

<br>

***  
> Eden영역이 꽉 차거나, Survivor영역이 꽉 차면 Minor GC가 발생한다.
>
> Minor GC가 발생하면 Eden과 S0에 Alive된 객체를 S1에 복사한다.
>
> Eden에서는 I 가 Alive, S0에서는 A,B가 Alive라고 가정한다.  

![image](https://user-images.githubusercontent.com/25604495/82352588-2868ed80-9a39-11ea-8474-6b2ffce895ee.png)


<br>

***

> Minor GC후에 Eden과 S0에 남은 객체는 죽은 것이므로 Clear한다.
>
> 즉, 더이상 참조되지 않는 객체라는 의미이다.

![image](https://user-images.githubusercontent.com/25604495/82352696-4c2c3380-9a39-11ea-9fe3-08bc43823bdb.png)

<br>

***

> 다음번 Minor GC가 발생하면 같은 원리로 Eden과 S1 영역에 Alive된 객체를 S0에 복사한다.
>
> 이러한 과정을 반복한다.
>
> 따라서, Survivor 영역 중 하나는 반드시 비어있는 상태여야한다.

***

<br>



3) Survivor 에서 Old 영역으로 이동
* Minor GC에서 살아남아 Survivor로 이동할때마다 객체의 Age가 증가한다.
* Age가 일정 이상이 되면 Old 영역으로 이동한다.
* Age는 `-XX:MaxTenuringThreshold` 옵션으로 설정할 수 있다고 한다.

> 아래 그림에서 A,B가 Alive 되어있고 Age가 일정 이상 되었다고 하자.
>
> Minor GC가 발생하면 A,B는 Old 영역으로 이동한다.
>
> I는 Alive 지만 Age가 일정 이상 되지 않아 S0으로 이동한다.

![image](https://user-images.githubusercontent.com/25604495/82356088-19d10500-9a3e-11ea-8edd-6f0d3d6414cc.png)


***

<br>

4) Old 영역
> Old 영역은 기본적으로 데이터가 가득 차면 GC를 실행한다.
>
> Full GC, Major GC가 발생하면 GC를 한다.
  
<br>  

## GC의 방식

### 1. Serial GC
* Young 영역에서는 위의 GC 과정에서 설펴본 방식을 사용한다.
* Old 영역의 GC는 `Mark-Sweep-Compaction` 알고리즘을 사용한다.  

> 1) Old 영역에 살아 있는 객체를 표시한다.(Mark)
>
> 2) Old 영역의 Mark된 객체를 제외하고 제거한다.(Sweep)
>
> 3) 살아있는 객체를 모은다.(Compaction)  
  
<img src = "https://user-images.githubusercontent.com/25604495/82413363-268e4100-9ab0-11ea-9de4-95336641b73e.png" width="500" height="300" />  
 

***

<br>

### 2. Parallel GC
* Serial GC와 기본적인 알고리즘은 같다.
* 차이점은 
    * Serial GC는 처리하는 스레드가 하나인 것에 비해
    * Parallel GC는 GC를 처리하는 쓰레드가 여러개이다.
* 따라서 Serial GC보다 빠르게 객체를 처리한다.

***

<br>

### 3. Parallel Old GC
* Parallel GC와 비교해 Old 영역의 GC 알고리즘만 다르다.
* Old 영역에서 `Mark-Summary-Compaction` 알고리즘을 사용한다.  

> Mark-Sweep-Compaction 연산과는 Sweep과 Summary의 차이다.
>  
> Sweep은 단일 스레드가 Old 영역을 훑어 객체만 찾아내는 방식이지만
>
> Summary는 여러 스레드가 Old 영역을 분리하여 훑는다.
>
> 효율을 위해 이전 GC에서 Compaction된 영역을 별도로 훑는다.

***

<br>

### 4. CMS GC
* Old영역은 GC는 다음 흐름과 같다.  

> 초기 Initial Mark 단계에서 클래스 로더에서 가장 가까운 객체 중 살아있는 객체를 찾는다.
>
> Concurrent Mark 단계에서 방금 살아있다고 확인된 객체에서 참조하고 있는 객체들을 따라가며 확인한다.
>
> Remark 단계에서 Concurrent Mark 단계에서 새로 추가되거나 끊긴 객체를 확인한다.
>
> Concurrent Sweep 단계에서는 Mark된 쓰레기 객체를 정리한다.

<br>

* 장점은 stop-the-world 시간이 짧다.  

> 즉, 전체 스레드를 멈추고 수행되는 것이 아니라
>
> 다른 스레드가 실행 중인 상태에서 동시에 수행되는 연산이 많기 때문이다.
>
> Inital Mark 단계를 제외하고 위와 같이 수행된다.

<br>

* 단점은 다른 GC 방식보다 메모리와 CPU를 더 많이 사용한다.
* 또한, Compaction 단계가 기본적으로 제공되지 않는다.

***

<br>

### 5. G1 GC
* Young 영역과 Old 영역을 나누지 않고
* 바둑판 모양 각 영역에 객체를 할당한다.
* 모든 GC 방식중에 가장 빠르다.  

<img src = "https://user-images.githubusercontent.com/25604495/82412694-0b6f0180-9aaf-11ea-9a30-9f9fe6f2f722.png" width="600" height="300" />

<br>

## Next
* Young과 Old 영역, GC 방식에 대해 간단하게 알아봤다.
* 추후에 실제로 사용할 일이 생긴다면 각 GC 방식에 대해 더 자세히 알아봐야겠다.

<br>


## Reference
* [https://d2.naver.com/helloworld/1329](https://d2.naver.com/helloworld/1329)  
* [https://5dol.tistory.com/183](https://5dol.tistory.com/183)  
* [https://johngrib.github.io/wiki/java-gc-eden-to-survivor/](https://johngrib.github.io/wiki/java-gc-eden-to-survivor/)  
* [https://deveric.tistory.com/64?category=346694](https://deveric.tistory.com/64?category=346694)  
* [https://preamtree.tistory.com/118](https://preamtree.tistory.com/118)  
