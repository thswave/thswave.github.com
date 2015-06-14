---
layout: post
title: future vs promise
categories: [java, netty]
tags: [java, netty, promise, future]
description: future vs promise
---

### Future VS  Promise

*  netty 기반 프로젝트를 진행하면서 java 8의 lamdba와 더불어 netty/play framework에서 제공하는 `future, promise`를 사용하고 있습니다. 막연하게 눈으로 코드를 봐오다 둘 의 차이를 알아야할 시점이 찾아와 조사해 보았습니다. future와 promise는 다른 언어에서 널리 사용되고 있으니 본 글에 나오는 설명이 부족하거나 틀린 부분이 있다면 의견 부탁드리겠습니다.
<br>
* future와 promise는 netty에 종속적인 기법/방법이 아닌 concurrent programming 언어에서 동기화에 사용되고 있습니다. 
* 좀 더 상세한 내용은 wiki [https://en.wikipedia.org/wiki/Futures_and_promises](https://en.wikipedia.org/wiki/Futures_and_promises)에 있는 정의를 참고 하시면 될 것 같습니다.

> Futures and promises
> In computer science, future, promise, and delay refer to constructs used for synchronization in some concurrent programming languages (생략)

<br><br>


* scala에서도 역시 `future와 promise` 를 제공해주고 있습니다. 

> Futures provide a nice way to reason about performing many operations in parallel - in an efficient and non-blocking way. The idea is simple, a Future is a sort of a placeholder object that you can create for a result that does not yet exist. Generally, the result of the Future is computed concurrently and can be later collected. Composing concurrent tasks in this way tends to result in faster, asynchronous, non-blocking parallel code

* 요약하자면 `performing many operations in parallel` 병렬로 연산을 수행.  `non-blocking way` 
* Future는 `create for a result that does not yet exist`지금은 존재하지 않는 값/결과를 만들어준다. 고 하는데 헷갈릴 수 있는데  즉, 병렬로 연산을 수행하고 그 결과가 생성된다고 보시면 될 것 같습니다.
  
* java에서도 future와 promise는 `java.util.concurrent` package 에서 찾아볼 수 있습니다.
  
* 두 용어의 의미가 비슷하고 모호해서 다시 검색해보았습니다.  
  
> a Future is essentially a read-only reference to a yet-to-be-computed value. A Promise is a pretty much the same except that you can write to it as well. In other words, you can read from both Futures and Promises, but you can only write to Promises. You can get the Future associated with a Promise by calling the future method on it, but conversion in the other direction is not possible (because it would be nonsensical).

* 참조: [http://programmers.stackexchange.com/questions/207136/what-is-the-difference-between-a-future-and-a-promise](http://programmers.stackexchange.com/questions/207136/what-is-the-difference-between-a-future-and-a-promise)

* Future : `a read-only reference`(읽기 전용) to a yet-to-be-computed value 
* Promise : pretty much the same except that you `can write to it`(쓰기 가능) 
* 공통점은  `yet-to-be-computed value` 아직 계산되지 않은 값에 대한 레퍼런스 라는 점입니다.

* 또 다른 답변을 살펴보면 좀 더 이해가 잘 될 것 같습니다.

> The Promise and Future are complementary concepts. The Future is a value which will be retrieved, well, sometime in the future and you can do stuff with it when that event happens. It is, therefore, the read or out endpoint of a computation - it is something that you retrieve a value from.

> A Promise is, by analogy, the writing side of the computation. You create a promise which is the place where you'll put the result of the computation and from that promise you get a future that will be used to read the result that was put into the promise. When you'll complete a Promise, either by failure or success, you will trigger all the behavior which was attached to the associated Future.  

* 참조 : [http://stackoverflow.com/questions/13381134/what-are-the-use-cases-of-scala-concurrent-promise](http://stackoverflow.com/questions/13381134/what-are-the-use-cases-of-scala-concurrent-promise)

* Future : 이벤트 발생 시 사용할 수(읽을 수) 있는 어떤 값
* Promise : 연산의 결과를 넣어두고(쓰고) 이 결과 값을 읽을 수 있는 Future를 얻을 수 있습니다. 즉, Promise가 끝나고 나면 성공, 혹은 실패에 연결되어 있는 Future를 통해 특정 동작을 유발시킬 수 있다.



