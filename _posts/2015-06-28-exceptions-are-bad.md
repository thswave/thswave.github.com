---
layout: post
title: Java Exception 생성 비용이 비싸다.
categories: [java, exception]
tags: [java, exception]
description: Java Exception 생성 비용이 비싸다.
---


### Creating an exception in Java is very slow

* Exception이 성능에 영향을 주는 것에 대해 살펴보겠습니다.   

* 예외를 적절하게 쓰면 비지니스 명확성을 높여주기 때문에 각 서비스 성격에 맞게 Custom Exception 클래스를 만들게 됩니다. 저 역시 웹 서비스를 개발하면서 Controller나 Service Layer 입력 값을 검사 할 때 조건에 맞는지 확인하기 위해 if 문으로 검사하고 비정상 입력 값일 경우 더이상 하위 로직을 수행하지 않고 벗어나기 위해 예외 클래스를 만들고 이를 활용했습니다. 스프링의 경우 어디에서 Exception을 던지더라도 전역적으로 잡아주는 `@ControllerAdvice`가 유용했습니다.  

* 비지니스 로직이 커짐에 따라 상황에 따른 예외 케이스가 많아지고 Exception 발생 횟수도 점점 많아졌는데 딱히 문제는 없어 보였는데 시니어 분께서 `예외 생성 비용이 비싸다` 고 조언해 주셔서 이와 관련된 포스팅을 읽어보게 되었습니다.

* [http://java-performance.info/throwing-an-exception-in-java-is-very-slow/](http://java-performance.info/throwing-an-exception-in-java-is-very-slow/)

* 위 포스팅에서 성능에 영향을 미치는 큰 요소로  예외의 발생 경로`trace`가 성능에 영향을 미친다는 걸 알 수 있습니다. 예외를 만들면서 1~5 ms를 소비한다고 하니 이는 매우 큰 수치라고 생각이 들었습니다. 

* `Filling in the stack trace is slow`

```
at java.lang.Throwable.fillInStackTrace(Throwable.java:-1)
at java.lang.Throwable.fillInStackTrace(Throwable.java:782)
- locked <0x6c> (a sun.misc.CEStreamExhausted)
at java.lang.Throwable.<init>(Throwable.java:250)
at java.lang.Exception.<init>(Exception.java:54)
at java.io.IOException.<init>(IOException.java:47)
at sun.misc.CEStreamExhausted.<init>(CEStreamExhausted.java:30)
at sun.misc.BASE64Decoder.decodeAtom(BASE64Decoder.java:117)
at sun.misc.CharacterDecoder.decodeBuffer(CharacterDecoder.java:163)
at sun.misc.CharacterDecoder.decodeBuffer(CharacterDecoder.java:194)
```

<br><br>

##### Overriding `fillInStackTrace` method

* 예외라는 것은 NullPointException이나 OutOfMemory 와 같은 예외나 비정상적인 상태나 값에 대한  예외가 있지만 우리가 만든 Custom Exception은 값을 검사해 하위 비지니스 로직을 수행할 수 없어 이를 방지하기 위한 용도로 만들었기 때문에 사실 현재 값이 어떤 call stack을 가지는지에 대한 trace는 필요하지 않았습니다. 

* exception stack trace는 `Throwable.fillInStackTrace` 메소드를 통해 생성되므로 이를  아무런 trace도 가지지 않게 Override해둘 수 있습니다.

{% highlight java %}
@Override 
public synchronized Throwable fillInStackTrace() {
	return this;
}
{% endhighlight %}  

<br><br>

##### Caching an exception

* stack trace를 가지지 않도록 Overriding 해둔 Exception이라면 `static final` 로 선언하고 일종의 상수 값 형태로 예외를 캐싱해두고 쓰는것이 매번 new로 생성하는 것보다 효율적입니다. new 로 매번 같은 종류의 예외를 생성하는건 비효율적이니까요.
  
  
{% highlight java %}
public class CustomException extends RuntimeException {
	public static final CustomException INVALID_NICKNAME = new CustomException(ResponseType.INVALID_NICKNAME);
	public static final CustomException INVALID_PARAMETER = new CustomException(ResponseType.INVALID_PARAMETER);
	public static final CustomException INVALID_TOKEN = new CustomException(ResponseType.INVALID_TOKEN);
	//생략
}
{% endhighlight %}
  
* Exception 클래스에 예외 상황에 대한 적당한 응답 메세지나 코드를 담도록 한 뒤 예외 발생 상황에서 `new` 키워드 없이 `throw` 합니다.
  
{% highlight java %}
if (StringUtils.isBlank(parameter)) {
	throw WebtoonCoreException.INVALID_PARAMETER;
}
{% endhighlight %}
  
* 이렇게 던진 예외를 프레임워크에서 핸들링 해주는 영역에서 처리하거나 호출 클래스로 넘겨 처리할 수 있습니다.