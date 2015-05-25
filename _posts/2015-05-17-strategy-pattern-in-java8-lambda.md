---
layout: post
title: Strategy Pattern using Lambda Expressions in Java 8
categories: [mac]
tags: [mac]
description: Strategy Pattern using Lambda Expressions in Java 8
---


### Strategy Pattern 

* 전략 패턴 : 어떤 알고리즘을 위한 전략을 정의하는 인터페이스를 정의한다. 상호 교환 가능한 클래스군이 인터페이스를 구현하며, 하나의 클래스는 하나의 알고리즘을 나타낸다. 

* Strategy : 알고리즘으로의 접근을 허용하는 인터페이스

* Concrete Strategy : Strategy 인터페이스를 따라 특정 알고리즘을 구현한다.

* Context : Strategy 인터페이스를 통해 알고리즘을 사용한다.


![0517_1.png](/assets/media/0517_1.png)  


JDK 에도 전략 패턴을 찾아볼 수 있다. Collection.sort(List, Comparator) 메서드에서 Comparator 가 전략에 해당하고 Collection.sort() 가 Context에 해당한다.  
또 다른 예는 java.util.Arrays#sort(T[], Comparator < ? super T> c) 메서드가 Collection.sort() 메서드와 List라는 점만 제와하고 유사하다.



{% highlight java %}
interface Strategy{
  public void strategy();
}


class FirstStratgey implements Strategy{

  @Override
  public void strategy() {
    System.out.println("strategy 1");
  }
  
}

class SecondStratgey implements Strategy{

  @Override
  public void strategy() {
    System.out.println("another strategy");
  }
  
}

public class StartegyPatternOldWay {

  public static void main(String[] args) {
    List<Strategy> strategies = 
        Arrays.asList(
          new FirstStratgey(), 
          new SecondStratgey()
        );
                      
    for(Strategy stg : strategies){
      stg.strategy();
    }
  }
}
{% endhighlight %} 

<br>
<br>

### Strategy Pattern in Java 8

{% highlight java %}
import java.util.Arrays;
import java.util.List;

public class StrategyPattern {
  public static void main(String[] args) {
    
    List<Strategy> strategies = 
      Arrays.asList(
        () -> {System.out.println("strategy 1");},
        () -> {System.out.println("strategy 2");}
      );
    
    strategies.forEach((elem) -> elem.strategy());
  }
  
}
{% endhighlight %} 

<br>
<br>
기존 자바 에서는 매개변수로 원시 값, 객체만 넘길 수 있었기 때문에 어떤 메서드(동작)는 넘길 수 없다.
예를 들어 데이터를 정렬하기 위해 특성에 따라 다른 정렬 알고리즘을 적용하고 싶을 경우 전략패턴을 사용할 수 있을것이다. 전략패턴의 핵심은 인터페이스로 메서드를 추상화하는 것이라 볼 수 있다. 

Java 8에서 동작(메서드)를 정의하고 이를 매개변수로 넘길 수 있는 lambda 가 소개 되어 경우에 따라 인터페이스 생성을 생략할 수 도 있다. 람다식으로 특정 행위를 매개변수로 넘기기만 하면 Context에서 이를 활용할 수 있기 때문이다. 

### How Scala killed the Strategy Pattern

스칼라에서 전략패턴은 사실상 아주 간결하게 표현할 수 있다. 아래 예제에서는 알고리즘(여기서는 간단한 사칙연산)을 매개변수로 넘겨준다. 

{% highlight scala %}
object DeathToStrategy extends App {
 
  def add(a: Int, b: Int) = a + b
  def subtract(a: Int, b: Int) = a - b
  def multiply(a: Int, b: Int) = a * b
   
  def execute(callback:(Int, Int) => Int, x: Int, y: Int) = callback(x, y)
 
  println("Add:      " + execute(add, 3, 4))
  println("Subtract: " + execute(subtract, 3, 4))
  println("Multiply: " + execute(multiply, 3, 4))
 
}
{% endhighlight %} 



<br>
### DI는 전략 패턴?
<br>
의존성을 외부(클라이언트)에서 주입하는 측면에서 볼 때 전략 패턴과 DI(Dependency Injection)은 닮았다고 볼 수 있다. 전략 패턴은 의존성 객체의 lifetime에 걸쳐 상호 교환되도록 의도 되었고, DI의 경우 한 의존 instance를 주입하곤 한다. 


##### 참조
(http://en.wikipedia.org/wiki/Dependency_injection)[http://en.wikipedia.org/wiki/Dependency_injection]

