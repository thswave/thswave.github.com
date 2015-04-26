---
layout: post
title: Partition a List in Java
categories: [guava, java]
tags: [guava, java, list]
description: Partition a List in Java
---

*본 포스트의 원문은 [http://www.baeldung.com](http://www.baeldung.com/java-list-split)에서 확인할 수 있으며 원 저작자의 번역 포스팅 허락을 받았음을 알려드립니다.*


### 1. Overview

이번 예제에서는 List를 어떻게 sublist로 분할(split) 하는지 살펴보겠습니다.
  
상대적으로 간단한 연산임에도 표준 Java Collection API에서 지원하지 않습니다. 운 좋게도 Guava나 Apache Commons Collections 에서 이런 연산을 비슷한 방법으로 구현 해 두었습니다.  


### 2. Use Guava to partition the List

Guava의 Lists.partition 연산으로 List를 특정 사이즈의 sublist로 분할(partitioning)하기 수월합니다.  
  
{% highlight java %} 
@Test
public void givenList_whenParitioningIntoNSublists_thenCorrect() {
    List<Integer> intList = Lists.newArrayList(1, 2, 3, 4, 5, 6, 7, 8);
    List<List<Integer>> subSets = Lists.partition(intList, 3);
 
    List<Integer> lastPartition = subSets.get(2);
    List<Integer> expectedLastPartition = Lists.<Integer> newArrayList(7, 8);
    assertThat(subSets.size(), equalTo(3));
    assertThat(lastPartition, equalTo(expectedLastPartition));
}
{% endhighlight %}
<br>
<br>

### 3. Use Guava to partition a Collection

Collection 분할도 Guava를 통해 가능합니다:
  

{% highlight java %} 
@Test
public void givenCollection_whenParitioningIntoNSublists_thenCorrect() {
    Collection<Integer> intCollection = Lists.newArrayList(1, 2, 3, 4, 5, 6, 7, 8);
 
    Iterable<List<Integer>> subSets = Iterables.partition(intCollection, 3);
 
    List<Integer> firstPartition = subSets.iterator().next();
    List<Integer> expectedLastPartition = Lists.<Integer> newArrayList(1, 2, 3);
    assertThat(firstPartition, equalTo(expectedLastPartition));
}
{% endhighlight %}
<br>
<br>
중요한 것은 분할된 sublist들은 원래 collection을 바라본다는 점인데 즉, 원래 collection을 변경하면 분할 partition들에도 반영된다는 뜻 입니다. 

{% highlight java %} 
@Test
public void givenListPartitioned_whenOriginalListIsModified_thenPartitionsChangeAsWell() {
    // Given
    List<Integer> intList = Lists.newArrayList(1, 2, 3, 4, 5, 6, 7, 8);
    List<List<Integer>> subSets = Lists.partition(intList, 3);
 
    // When
    intList.add(9);
 
    // Then
    List<Integer> lastPartition = subSets.get(2);
    List<Integer> expectedLastPartition = Lists.<Integer> newArrayList(7, 8, 9);
    assertThat(lastPartition, equalTo(expectedLastPartition));
}
{% endhighlight %}
<br>
<br>
### 4. Use Apache Commons Collections to partition the List

Apache Commons Collection의 최근 릴리즈에도 리스트 분할을 지원하는 기능이 최근 추가되었습니다:
  

{% highlight java %} 
@Test
public void givenList_whenParitioningIntoNSublists_thenCorrect() {
    List<Integer> intList = Lists.newArrayList(1, 2, 3, 4, 5, 6, 7, 8);
    List<List<Integer>> subSets = ListUtils.partition(intList, 3);
 
    List<Integer> lastPartition = subSets.get(2);
    List<Integer> expectedLastPartition = Lists.<Integer> newArrayList(7, 8);
    assertThat(subSets.size(), equalTo(3));
    assertThat(lastPartition, equalTo(expectedLastPartition));
}
{% endhighlight %}
<br>
<br>
Commons Collection에는 Guava의 Iterables.partition과 같은 기본(raw) Collection을 분할할 수 있는 옵션기능은 없습니다.

마지막으로 결과 분할된 리스트가 원래의 리스트를 바라본다는 같은 조건이 적용됩니다. 

<br>
<br>
5. Conclusion

위 포스트에서 제안한 방법들은 Guava나 Apache Commons Collections 와 같은 추가 라이브러리를 사용했습니다. 두 라이브러리는 가볍고 강력하면서도 매우 유용합니다. 그래서 이들 중 하나 정도는 classpath에 추가해 두면 편리합니다. 
자바를 이용한 해결방법은 [여기](http://www.vogella.com/articles/JavaAlgorithmsPartitionCollection/article.html) 에서 확인 할 수 있습니다. 

