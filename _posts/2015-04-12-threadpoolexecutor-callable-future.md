---
layout: post
title: ThreadPoolExecutor + Callable + Future Example
categories: [mac]
tags: [test, mac, osx]
description: ThreadPoolExecutor + Callable + Future Example
---

### ThreadPoolExecutor + Callable + Future Example

*본 포스트의 원문은 [http://howtodoinjava.com/2015/03/25/threadpoolexecutor-callable-future-example/](http://howtodoinjava.com/2015/03/25/threadpoolexecutor-callable-future-example/)에서 확인할 수 있으며 원 저작자의 번역 포스팅 허락을 받았음을 알려드립니다.*


Executor framework의 장점 중 하나는 task 처리가 끝난 다음 결과를 return 받아 task들을 병렬로(concurrent) 실행할 수 있다는 점입니다. Java Concurrent API의 Callable, Future 두 인터페이스를 활용하는 방법을 살펴보도록 하겠습니다.   
  
먼저 두 인터페이스들을 살펴보겠습니다.


Callable : 이 인터페이스에는 call() 메서드가 있습니다. 이 메소드는 여러분이 task의 로직을 구현해야 합니다. Callable 인터페이스는 parameterized 인터페이스입니다. 즉 call() 메서드가 return 하는 data 타입을 지정해줘야 합니다. 


Future : 이 인터페이스는 Callable 객체에서 생성된 결과를 얻을 수 있는 메서드들과 상태를 관리하기 위한 메서드들이 있습니다.  



#### Callable Future Example   
<br>
이 예제는 Callable 타입의 FactorialCaculator를 만들고 call() 메서드를 override하여 계산을 하고 call() 메서드의 결과를 return 해줍니다. 이 결과는 나중에 메인 프로그램의 Future 레퍼런스에서 찾아 사용하게 됩니다.   
<br>
{% highlight java %}
class FactorialCalculator implements Callable<Integer>
{
 
    private Integer number;
 
    public FactorialCalculator(Integer number) {
        this.number = number;
    }
 
    @Override
    public Integer call() throws Exception {
        int result = 1;
        if ((number == 0) || (number == 1)) {
            result = 1;
        } else {
            for (int i = 2; i <= number; i++) {
                result *= i;
                TimeUnit.MILLISECONDS.sleep(20);
            }
        }
        System.out.println("Result for number - " + number + " -> " + result);
        return result;
    }
}
{% endhighlight %}
<br>
<br>
이제 factorial 계산기를 2개의 thread와 4개의 임의의 숫자로 테스트해보겠습니다.  
<br>
<br>
{% highlight java %}
package com.howtodoinjava.demo.multithreading;
 
import java.util.ArrayList;
import java.util.List;
import java.util.Random;
import java.util.concurrent.Callable;
import java.util.concurrent.ExecutionException;
import java.util.concurrent.Executors;
import java.util.concurrent.Future;
import java.util.concurrent.ThreadPoolExecutor;
import java.util.concurrent.TimeUnit;
 
public class CallableExample
{
      public static void main(String[] args)
      {
          ThreadPoolExecutor executor = (ThreadPoolExecutor) Executors.newFixedThreadPool(2);
           
          List<Future<Integer>> resultList = new ArrayList<>();
           
          Random random = new Random();
           
          for (int i=0; i<4; i++)
          {
              Integer number = random.nextInt(10);
              FactorialCalculator calculator  = new FactorialCalculator(number);
              Future<Integer> result = executor.submit(calculator);
              resultList.add(result);
          }
           
          for(Future<Integer> future : resultList)
          {
                try
                {
                    System.out.println("Future result is - " + " - " + future.get() + "; And Task done is " + future.isDone());
                }
                catch (InterruptedException | ExecutionException e)
                {
                    e.printStackTrace();
                }
            }
            //shut down the executor service now
            executor.shutdown();
      }
}
{% endhighlight %}
<br>

<br>
``` 
Output:
 
Result for number - 4 -> 24
Result for number - 6 -> 720
Future result is -  - 720; And Task done is true
Future result is -  - 24; And Task done is true
Result for number - 2 -> 2
Result for number - 6 -> 720
Future result is -  - 720; And Task done is true
Future result is -  - 2; And Task done is true
```

<br>
<br>
Callable 객체를 실행시키기 위해 executor의 submit() 메서드에 전달하여 실행하였습니다. 이 메서드는 Callable 객체를 파라미터로 받아 Future 객체를 리턴으로 돌려주는데 이는 두가지 목적이 있습니다. 
<br>
<br>
a) task의 상태(status) 제어: task를 취소할수 있고 끝났는지 체크할 수 있습니다. isDone() 메서드를 통해 task들이 끝났는지 확인할 수 있습니다. 
<br>
<br>
b) call() 메서드의 return 결과를 얻을 수 있습니다. get() 메서드를 사용하여 Callable 객체의 call() 메서드의 실행이 끝날때까지 기다려 return되는 결과를 받을 수 있습니다. 만약 thread가 get() 메서드로 결과를 기다리고 있는 도중에 interrupt 되면 InterruptedException 예외를 던집니다.
<br>
<br>
c) Future 인터페이스는 또다른 버전의 get() 메서드를 제공해 줍니다. i.e. get(long timeout, TimeUnit unit). 이 버전의 get 메서드는 task의 결과를 얻을 수 없다면 특정 시간만큼 기다립니다. 만약 특정 시간이 지난 뒤에도 결과를 여전히 얻을수 없다면 메서드는 null 값을 return합니다. 

