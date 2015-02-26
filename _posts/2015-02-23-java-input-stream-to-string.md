---
layout: post
title: Java InputStream to String
categories: [java]
tags: [java, guava, inputstream]
description: Java InputStream to String
---

*본 포스트의 원문은 [http://www.baeldung.com](http://www.baeldung.com/convert-input-stream-to-string)에서 확인할 수 있으며 원 저작자의 번역 포스팅 허락을 받았음을 알려드립니다.*

*Guava에 대한 설명은 다음주에 다시 설명드리겠습니다.*

## 1. Overview

이번 튜토리얼에서는 [Guava](https://code.google.com/p/guava-libraries/), [Apache Commons IO library](http://commons.apache.org/proper/commons-io/), plain Java를 활용하여 InputStream을 String으로 변환하는 방법에 대해 살펴볼 것입니다.

## 2. Convert with Guava

Guava를 활용한 변환 방법부터 살펴 보겠습니다. - InputSupplier 기능을 활용한 방법 :

{% highlight java%}
@Test
public void givenUsingGuava_whenConvertingAnInputStreamToAString_thenCorrect() 
  throws IOException {
    String originalString = randomAlphabetic(8);
    InputStream inputStream = new ByteArrayInputStream(originalString.getBytes());
 
    InputSupplier<InputStream> inputSupplier = new InputSupplier<InputStream>() {
        @Override
        public InputStream getInput() throws IOException {
            return inputStream;
        }
    };
    InputSupplier<InputStreamReader> readerSupplier = 
      CharStreams.newReaderSupplier(inputSupplier, Charsets.UTF_8);
 
    String text = CharStreams.toString(readerSupplier);
 
    assertThat(text, equalTo(originalString));
}
{% endhighlight %}


다음과 같은 순서로 진행됩니다.

1. InputStream을 InputSupplier<InputStream>으로 감싼다. - 이 방법이 가장 쉬운 방법이기 때문에 이와 같은 방식으로 진행.

2. 그리고 InputStream을 파라미터로 활용한 reader를 사용. - character stream을 활용하기 위해.

3. 마지막으로 Guava CharStreams 기능을 활용해 String으로 변환합니다. - 마지막으로 String으로의 변환 작업은 `Charsets.toString` 그리고 InputStream을 close 해준다.

Guava를 통해 변환하는 더 간단한 방법이 있습니다. 자동으로 리소스를 close 하는 것을 제외하면 다음과 같습니다. 

{% highlight java%}
@Test
public void givenUsingGuavaAndJava7_whenConvertingAnInputStreamToAString_thenCorrect() 
  throws IOException {
    String originalString = randomAlphabetic(8);
    InputStream inputStream = new ByteArrayInputStream(originalString.getBytes());
 
    String text = null;
    try (final Reader reader = new InputStreamReader(inputStream)) {
        text = CharStreams.toString(reader);
    }
 
    assertThat(text, equalTo(originalString));
}
{% endhighlight %}

위 예제에서 CharStreams.toString 메서드는 InputStream을 close하지 않습니다. 대신 Java 7의 try-with-resources 을 통해 자동으로 자원이 반환 됩니다. 

## 3. Apache Commons IO 활용한 변환

이제는 Commons IO library를 활용하는 방법에 대해 살펴보겠습니다. 

주요 차이점은 Guava와는 달리 아래 예제들에서 InputStream이 close 되지 않습니다. 그래서 개인적으로 Guava를 활용한 방법을 더 선호합니다. 

{% highlight java%}
@Test
public void givenUsingCommonsIo_whenConvertingAnInputStreamToAString_thenCorrect() 
  throws IOException {
    String originalString = randomAlphabetic(8);
    InputStream inputStream = new ByteArrayInputStream(originalString.getBytes());
 
    String text = IOUtils.toString(inputStream, StandardCharsets.UTF_8.name());
    assertThat(text, equalTo(originalString));
}
{% endhighlight %}

그리고 StringWriter를 활용한 변환할 수 있습니다. 

{% highlight java%}
@Test
public void givenUsingCommonsIoWithCopy_whenConvertingAnInputStreamToAString_thenCorrect() 
  throws IOException {
    String originalString = randomAlphabetic(8);
    InputStream inputStream = new ByteArrayInputStream(originalString.getBytes());
 
    StringWriter writer = new StringWriter();
    String encoding = StandardCharsets.UTF_8.name();
    IOUtils.copy(inputStream, writer, encoding);
 
    assertThat(writer.toString(), equalTo(originalString));
}
{% endhighlight %}

## 4. plain Java, InputStream 변환

plain java를 활용하는 lower level 접근법을 살펴보겠습니다. - InputStream과 간단한 StringBuilder 활용 :

{% highlight java%}
@Test
public void givenUsingJava5_whenConvertingAnInputStreamToAString_thenCorrect() 
  throws IOException {
    String originalString = randomAlphabetic(DEFAULT_SIZE);
    InputStream inputStream = new ByteArrayInputStream(originalString.getBytes());
 
    StringBuilder textBuilder = new StringBuilder();
    try (Reader reader = new BufferedReader(new InputStreamReader
      (inputStream, Charset.forName(StandardCharsets.UTF_8.name())))) {
        int c = 0;
        while ((c = reader.read()) != -1) {
            textBuilder.append((char) c);
        }
    }
    assertEquals(textBuilder.toString(), originalString);
}
{% endhighlight %}

## 5. plain Java, Scanner를 활용한 변환

마지막으로 plain Java를 활용한 또 다른 방법을 살펴보겠습니다.  - 표준 text Scanner 활용 :

{% highlight java%}
@Test
public void givenUsingJava7_whenConvertingAnInputStreamToAString_thenCorrect() 
  throws IOException {
    String originalString = randomAlphabetic(8);
    InputStream inputStream = new ByteArrayInputStream(originalString.getBytes());
 
    String text = null;
    try (Scanner scanner = new Scanner(inputStream, StandardCharsets.UTF_8.name())) {
        text = scanner.useDelimiter("\A").next();
    }
 
    assertThat(text, equalTo(originalString));
}
{% endhighlight %}

InputStream은 Scanner 를 close하면서 같이 close니다. 

위 예제는 Java 7 를 가정하고 있기 때문에 Java 5 에서는 try-with-resources 문은 사용할 수 없습니다.
일반적인 try-finally문을 사용해야 Java 5 에서 정상적으로 compile되고 실행됩니다. 

## 6. Conclusion

After compiling the best way to do the simple conversion – InputStream to String – in a correct and readable way – and after seeing so many wildly different answers and solutions – I think that a clear and concise best practice for this is called for.
위 예제들을 통해 다양한 답안과 해결 방법을 살펴봤습니다.  InputStream을 String으로 간편하게 변환하는 최선의 방법은 정확하면서 가독성이 좋은 방법이어야 합니다. 

모든 예제 코드들은 [github 프로젝트](https://github.com/eugenp/tutorials/tree/master/core-java#readme)에서 확인할 수 있습니다. - Eclipse 기반 프로젝트로 쉽게 import해서 실행해볼 수 있습니다.
