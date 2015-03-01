---
layout: post
title: java remove duplicates from list. (Java list에서 중복 요소 제거)
categories: [java]
tags: [java, list]
description: java remove duplicates from list (Java list에서 중복 요소 제거)
---

*본 포스트의 원문은 [http://www.baeldung.com](http://www.baeldung.com/java-remove-duplicates-from-list?utm_source=email-newsletter&utm_medium=email&utm_campaign=auto_36_java)에서 확인할 수 있으며 원 저작자의 번역 포스팅 허락을 받았음을 알려드립니다.*

이번 예제는 List에서 중복 요소를 제거하는 방법에 대해 살펴보겠습니다. - 처음에는 plain java를 활요하여, 그리고 Guava를 활용하여 마지막으로 Java 8 의 Lambda 기반 해결법을 살펴보겠습니다. 

이 포스트는 Baeldung 사이트의 “[Java – Back to Basic](http://www.baeldung.com/java-tutorial)” 시리즈입니다. 

# 1. plain java를 사용한 중복 요소 제거

Java Collections 프레임워크 중 List에서 중복 요소를 제거하는것은 Set을 활용하면 됩니다.
(Set 이 중복 요소를 저장하지 않기 때문에)

{% highlight java%}
public void givenListContainsDuplicates_whenRemovingDuplicatesWithPlainJava_thenCorrect() {
    List<Integer> listWithDuplicates = Lists.newArrayList(0, 1, 2, 3, 0, 0);
    List<Integer> listWithoutDuplicates = new ArrayList<>(new HashSet<>(listWithDuplicates));
 
    assertThat(listWithoutDuplicates, hasSize(4));
}
{% endhighlight %}

원래 리스트(listWithDuplicates)의 값은 중복값이 남아있습니다. 

# 2. Guava를 활용한 중복 요소 제거 

Guava를 통해 같은 방법으로 중복 요소 제거 가능합니다. 

{% highlight java%}
public void givenListContainsDuplicates_whenRemovingDuplicatesWithGuava_thenCorrect() {
    List<Integer> listWithDuplicates = Lists.newArrayList(0, 1, 2, 3, 0, 0);
    List<Integer> listWithoutDuplicates = Lists.newArrayList(Sets.newHashSet(listWithDuplicates));
 
    assertThat(listWithoutDuplicates, hasSize(4));
}
{% endhighlight %}

역시 원래의 리스트 값은 변화 없습니다. 

# 3. Java 8 Lambdas 를 활용한 중복 요소 제거 

이제 마지막으로 `Lambdas` 를 활용한 새로운 방법을 살펴보겠습니다.;  Stream API 중 *고유한(distinct) 요소들로 구성된 stream을 반환 해주는 `distinct()` 메서드를 활용*한 방법을 살펴 보겠습니다. 

{% highlight java%}
public void givenListContainsDuplicates_whenRemovingDuplicatesWithJava8_thenCorrect() {
    List<Integer> listWithDuplicates = Lists.newArrayList(1, 1, 2, 2, 3, 3);
    List<Integer> listWithoutDuplicates = 
      listWithDuplicates.parallelStream().distinct().collect(Collectors.toList());
}
{% endhighlight %}

위 예제에서 살펴본 3가지 간단한 방법을 통해 list의 중복 요소들을 제거할 수 있습니다. 


