---
layout: post
title: Java Volatile 의미
categories: [java]
tags: [java, volatile]
description: Java Volatile
---

*원 글 [Java's Volatile Keyword](http://tutorials.jenkov.com/java-concurrency/volatile.html) 을 참고하여 번역하였습니다.*


Java volatile 키워드는 자바 변수를 "메인 메모리에 저장 할" 표식으로 사용합니다. 좀 더 정확하게 말하자면 모든 `volatile 변수를 읽어 들일 때 CPU 캐시가 아니라 컴퓨터의 메인 메모리로 부터 읽어들입니다`. 그리고 volatile 변수를 쓸 때에도(write) CPU 캐시가 아닌 메인 메모리에 기록합니다. 


java 5 이래로 volatile 키워드는 volatile 변수들을 메인 메모리로 부터 읽고 쓰는걸 것 보다 더 큰 의미를 가지는데 이는 곧 다시 설명 하겠습니다. 

# Java volatile은 변수의 가시성(Visibility)을 보장한다. 


Java volatile 키워드는 여러개의 쓰래드들 에서 사용되는 변수의 변화(값의 변화) 에 대한 가시성의 보장합니다. 이 말이 좀 추상 적으로 느껴질 수 있는데 자세하게 설명하자면 


non-volatile 변수들로 운영되는 멀티 쓰래드 어플리케이션에서 각 쓰래드들은 성능적이 이유로 메인 메모리로 부터 변수를 읽어 CPU 캐시에 복사하고 작업하게 됩니다. 만약 여러분의 컴퓨터에 하나 이상의 CPU로 구성되어 있고 각 쓰래드들이 서로 다른 CPU에서 실행 될수 있습니다. 이 말은 각 쓰래드들이 서로 다른 CPU들의 CPU 캐시에 값을 복사할 수 있다는 것으로 아래의 다이어그램이 이를 설명해주고 있습니다.

![0308.jpg](/assets/media/0308.jpg)

non-volatile 변수들은 어느 시점에 Java Virtual Machine(JVM)이 메인 메모리로 부터 데이터를 읽어 CPU 캐시로 읽어 들이거나 혹은 CPU 캐시들에서 메인 메모리로 데이터를 쓰는지(write) 보장해 줄 수 없습니다.  이럴 경우 어떤 문제가 발생 할 수 있는지 예를 들어 보겠습니다. 

두 개 혹은 그 이상의 쓰래드가 접근하고 사용하는 공용 오브젝트에 counter 라는 변수가 아래와 같이 선언된 상황을 가정합니다 : 

```
public class SharedObject {

    public int counter = 0;

}
```


1번 쓰래드가 공유 오브젝트의 counter 값인 0 을 CPU 캐시로 읽어 갑니다. 그리고 1 로 증가 시킨 후 메인 메모리에는 아직 쓰지 않았습니다. 2번 쓰래드가 같은 공유 오브젝트의 counter 값을 메인 메로리로 부터 읽어 가는데 이 때 아직 값은 0 입니다. 그리고 이를 CPU 캐시에 복사하고 2번 쓰래드 역시 이 값을 1로 증가 시키고 역시 메인 메모리로 쓰지 않았습니다. 이 경우 1번과 2번 쓰래드의 동기화(sync)가 깨진 상태입니다. 실제 공유 오브젝트의 counter값은 실질적으로 2 여야 하는데 각 쓰래드에서 CPU 캐시들에 1 로 기록한 상태입니다. 메인 메모리에는 아직 0 인 상태입니다. 쓰래들이 공유 counter 변수를 메인 메모리로 다시 쓸 경우 결국 이 값은 잘못 된 값입니다. 

공유 counter 변수에 volatile을 선언함으로써 JVM은 해당 변수에 대한 모든 읽기 연산을 항상 메인 메모리에서 부터 읽어가도록 보장해 줍니다.  그리고 변수에 대한 모든 write 역시 항상 메인 메모리에 기록되도록 해 줍니다. 
volatile 선언은 다음과 같이 할 수 있습니다. 

```
public class SharedObject {

    public volatile int counter = 0;

}
```

멀티 쓰래드에서 접근하는 변수의 가장 최근에 기록된 값을 보기(see)때문에 변수에 간단하게 volatile을 선언 해주는 것이 충분한 경우도 있습니다. 어떤 경우에 volatile 선언해야 하는지 좀 더 뒤에 살펴보겠습니다. 


이번에도 두 개의 쓰래드에서 같은 변수를 읽고 쓰는 상황을 가정하겠습니다. 변수에 간단하게 volatile 을 선언하는 것으로 충분하지 않은 상황으로 1번 쓰래드에서 counter 변수 값 0을 CPU 1의 CPU 레지스터로 읽고, 이와 동시에(또는 바로 이어서) 2번 쓰래드에서 counter 변수 1을 CPU 2의 CPU 레지스터에 읽었습니다. 두 쓰래드들이 메인 메모리로 부터 바로 변수의 값을 읽었습니다. 이제 두 변수들에 값을 증가시켜 메인 메모리로 다시 쓰기 연산이 실행됩니다. 두 레지스터에서 counter의 값을 1로 증가시켰기 때문에 메인 메모리로 기록할 때 1로 기록합니다. 실제로는 두번의 증감으로 값이 2 가 되어야 합니다.


멀티 쓰래드의 문제점은 다른 쓰래드에서 메인 메모리로 아직 기록하지 않은 값을 보지 못했기 때문입니다. 이를 "가시성" 문제라고 불립니다. 한 쓰래드에서의 업데이트는 다른 쓰래드에서는 볼수 없습니다.


# The Java volatile Guarantee


Java 5의 volatile 키워드는 단순히 변수를 메인 메모리로 부터 읽고 쓰는것을 이상울 보장(guarantees) 해 주는데 실제로 volatile 키워드가 보장해주는 것은 :


만약 쓰래드 A가 volatile 변수에 쓰기 작업을 하고 쓰래드 B가 바로 직후에 같은 volatile 변수를 읽을 경우, volatile 변수를 쓰기 직전에 모든 변수를 쓰래드 A에서 볼 수(visible) 있습니다. 쓰래드 B 역시 볼 수 있습니다.


volatile 변수들의 읽고 쓰기 연산은 JVM에 의해 재배치(reorder) 되지 않습니다.(JVM은 프로그램이 reordering에 의해 동작이 바뀌지 않는 한 성능상의 이유로 JVM이 instructions을 reorder 할 수 있습니다.) volatile 이전, 이후의 명령(Instructions)들은 재배치 될 수 있습니다. 하지만 volatile 읽기 혹은 쓰기는 이 연산들과 섞이지 않습니다. volatile 변수에 대한 읽기 혹은 쓰기 연산 뒤에 실행되는 연산들은 volatile 읽기/쓰기 작업 이후에 실행됩니다. 

아래 예제를 보면 :

```
Thread A:
    sharedObject.nonVolatile = 123;
    sharedObject.counter     = sharedObject.counter + 1;

Thread B:
    int counter     = sharedObject.counter;
    int nonVolatile = sharedObject.nonVolatile;
```


쓰래드 A가 volatile인 sharedObject.counter에 쓰기 연산 전에 sharedObject.nonVolatile non-volatile 변수에 기록합니다. 그러면 sharedObject.nonVolatile과 sharedObject.counter 두 변수는 메인 메모리에 기록 됩니다. 


왜냐하면 쓰래드 B가 volatile sharedObject.counter를 읽으면서 시작했기 때문에 두 sharedObject.counter와 sharedObject.nonVolatile 변수는 메인 메모리로 부터 읽었기 때문입니다.


non-volatile 변수의 읽기와 쓰기는 volatile 변수의 읽기/쓰기 전과 후에 위치하면 재배치(reordered) 되지 않습니다.  

# 언제 volatile이 적합한가?


앞서 언급하였든 두 쓰래드가 공유 변수에 대한 읽기와 쓰기 연산이 있을 경우 volatile 키워드로는 충분하지 않습니다. 이 경우 `synchronization` 를 통해 변수의 읽기 쓰기 연산의 원자성(atomic)을 보장해 줘야합니다. 

하지만 한 쓰래드에서 volatile 변수의 값을 읽고 쓰고, 다른 쓰래드에서는 오직 변수 값을 읽기만 할 경우, 그러면 읽는 쓰래드에서는 volatile 변수의 가장 최근에 쓰여진 값을 보는 것을 보장할 수 있습니다. volatile 없이는 이를 보장해 줄 수 없습니다.


# volatile 의 성능 고려사항

volatile 변수에 대한 읽기와 쓰기는 변수를 메인 메모리로 부터 읽거나 쓰게 됩니다. 메인 메모리에 읽고 쓰는것은 CPU 캐시보다 더 비싸다고 할 수 있습니다. 또한 volatile 변수는 성능을 개선 기법인 명령(instruction)들의 재배치를 방지하기 때문에 변수의 가시성을 강제할 필요가 있는 경우에만 volatile 변수를 사용하는 것이 좋습니다. 

