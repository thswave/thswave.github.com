---
layout: post
title: Java remove nulls from list (Java list에서 null 요소 제거)
categories: [java]
tags: [java, list]
description: Java remove nulls from list (Java list에서 null 요소 제거)
---


*본 포스트의 원문은 [http://www.baeldung.com](http://www.baeldung.com/java-remove-nulls-from-list?utm_source=email-newsletter&utm_medium=email&utm_campaign=auto_36_java)에서 확인할 수 있으며 원 저작자의 번역 포스팅 허락을 받았음을 알려드립니다.*

이번 예제는 List에서 null 를 제거하는 방법에 대해 살펴보겠습니다. - 처음에는 plain java를 활요하여, 그리고 Guava를 활용하여 마지막으로 Java 8 의 Lambda 기반 해결법을 살펴보겠습니다. 

이 포스트는 Baeldung 사이트의 “[Java – Back to Basic](http://www.baeldung.com/java-tutorial)” 시리즈입니다. 

# 1. plain java를 사용한 null 요소 제거

Java Collections 프레임워크에서 list의 null 요소를 제거할 수 있는 간단한 방법을 제공하고 있습니다. - while 루프

{% highlight java%}
@Test
public void givenListContainsNulls_whenRemovingNullsWithPlainJava_thenCorrect() {
    List<Integer> list = Lists.newArrayList(null, 1, null);
    while (list.remove(null));
 
    assertThat(list, hasSize(1));
}
{% endhighlight %}

또 다른 간단한 방법도 있습니다. 

{% highlight java%}
@Test
public void givenListContainsNulls_whenRemovingNullsWithPlainJavaAlternative_thenCorrect() {
    List<Integer> list = Lists.newArrayList(null, 1, null);
    list.removeAll(Collections.singleton(null));
 
    assertThat(list, hasSize(1));
}
{% endhighlight %}

두 방법 모두 원래의 리스트가 변경됩니다. 

# 2. Guava를 활용한 null 요소 제거 

Guava에서도 `predicates`를 통해 null을 제거할 수 있는 방법을 제공합니다. 

{% highlight java%}
@Test
public void givenListContainsNulls_whenRemovingNullsWithGuavaV1_thenCorrect() {
    List<Integer> list = Lists.newArrayList(null, 1, null);
    Iterables.removeIf(list, Predicates.isNull());
 
    assertThat(list, hasSize(1));
}
{% endhighlight %}

만약 원래 리스트를 변경하지 않고 null만 제거된 리스트만 별도로 만들고 싶을 경우 Guava를 통해 새로 생성하여 list에 filter를 적용하면 됩니다.

{% highlight java%}
@Test
public void givenListContainsNulls_whenRemovingNullsWithGuavaV2_thenCorrect() {
    List<Integer> list = Lists.newArrayList(null, 1, null, 2, 3);
    List<Integer> listWithoutNulls = Lists.newArrayList(
      Iterables.filter(list, Predicates.notNull()));
 
    assertThat(listWithoutNulls, hasSize(3));
}
{% endhighlight %}

# 3. Apache Commons Collections를 활용한 null 제거

이제는 Apache Commons Collections library에서도 비슷한 기능을 활용하는 방법을 살펴보겠습니다.:

{% highlight java%}
@Test
public void givenListContainsNulls_whenRemovingNullsWithCommonsCollections_thenCorrect() {
    List<Integer> list = Lists.newArrayList(null, 1, 2, null, 3, null);
    CollectionUtils.filter(list, PredicateUtils.notNullPredicate());
 
    assertThat(list, hasSize(3));
}
{% endhighlight %}

위 방법은 원래의 리스트가 변경됩니다.

# 4. Lambdas를 활용한 null 요소 제거(Java 8)

마지막으로 Java 8의 Lambdas의 filter를 활용한 방법을 살펴보겠습니다. 필터링은 parallel이나 serial로 처리할 수 있습니다. 

{% highlight java%}
public void givenListContainsNulls_whenFilteringParallel_thenCorrect() {
    List<Integer> list = Lists.newArrayList(null, 1, 2, null, 3, null);
    List<Integer> listWithoutNulls = 
      list.parallelStream().filter(i -> i != null).collect(Collectors.toList());
}
{% endhighlight %} 

{% highlight java%}
public void givenListContainsNulls_whenFilteringSerial_thenCorrect() {
    List<Integer> list = Lists.newArrayList(null, 1, 2, null, 3, null);
    List<Integer> listWithoutNulls = 
      list.stream().filter(i -> i != null).collect(Collectors.toList());
}
{% endhighlight %}

지금까지 List에서 null 요소를 제거할 수 있는 간단하고 유용한 방법들을 살펴봤습니다. 
