---
layout: post
title: Java Guava – Maps
categories: [java]
tags: [java, guava]
description: Java Guava – Maps
---

*본 포스트의 원문은 [http://www.baeldung.com](http://www.baeldung.com/guava-maps?utm_source=email-newsletter&utm_medium=email&utm_campaign=auto_45_guava)에서 확인할 수 있으며 원 저작자의 번역 포스팅 허락을 받았음을 알려드립니다.*

#  1. Overview

이번 튜토리얼에서 Guava를 통해 Java Map을 유용하게 사용할 수 있는 방법들에 대해 설명할 것입니다.

Guava를 통해 new 연산자 없이 HashMap 을 만드는 아주 간단한 방법을 살펴보도록 합시다. 


{% highlight java%}
Map<String, String> aNewMap = Maps.newHashMap();
{% endhighlight %}


# 2. ImmutableMap

다음으로 Guava를 통해 ImmutableMap 을 만드는 것을 살펴보겠습니다:


{% highlight java%}
@Test
public void whenCreatingImmutableMap_thenCorrect() {
    Map<String, Integer> salary = ImmutableMap.<String, Integer> builder()
      .put("John", 1000)
      .put("Jane", 1500)
      .put("Adam", 2000)
      .put("Tom", 2000)
      .build();
 
    assertEquals(1000, salary.get("John").intValue());
    assertEquals(2000, salary.get("Tom").intValue());
}
{% endhighlight %}


# 3. SortedMap

이제 SortedMap을 만들고 사용하는 방법에 대해 살펴보겠습니다.

다음 예제는 - Guava builder 를 통해 sorted map을 만들겠습니다:


{% highlight java%}
@Test
public void whenUsingSortedMap_thenKeysAreSorted() {
    ImmutableSortedMap<String, Integer> salary = new ImmutableSortedMap
      .Builder<String, Integer>(Ordering.natural())
      .put("John", 1000)
      .put("Jane", 1500)
      .put("Adam", 2000)
      .put("Tom", 2000)
      .build();
 
    assertEquals("Adam", salary.firstKey());
    assertEquals(2000, salary.lastEntry().getValue().intValue());
}
{% endhighlight %}


# 4. BiMap

다음으로 어떻게 BiMap을 사용하는지 살펴보겠습니다. BiMap은 map 의 value로 key 값을 찾을 수 있습니다.(value들이 유니크한 값을 가져야 합니다)

다음 예제는 - BiMap 을 만들고 inverse() 사용 예제를 살펴보겠습니다.


{% highlight java%}
@Test
public void whenCreateBiMap_thenCreated() {
    BiMap<String, Integer> words = HashBiMap.create();
    words.put("First", 1);
    words.put("Second", 2);
    words.put("Third", 3);
 
    assertEquals(2, words.get("Second").intValue());
    assertEquals("Third", words.inverse().get(3));
}
{% endhighlight %}


# 5. Multimap

이제 Multimap을 살펴보겠습니다. 


다음 예제에서 Multimap으로 `각 key에 여러개의 value를 저장`할 수 있음을 확인할 수 있습니다. 

{% highlight java%}
@Test
public void whenCreateMultimap_thenCreated() {
    Multimap<String, String> multimap = ArrayListMultimap.create();
    multimap.put("fruit", "apple");
    multimap.put("fruit", "banana");
    multimap.put("pet", "cat");
    multimap.put("pet", "dog");
 
    assertThat(multimap.get("fruit"), containsInAnyOrder("apple", "banana"));
    assertThat(multimap.get("pet"), containsInAnyOrder("cat", "dog"));
}
{% endhighlight %}


# 5. Table

이제는 Guava Table을 살펴볼 차례입니다; value에 하나 이상의 key가 필요할 때 Table을 사용합니다. 
아래 예제는 두 도시간의 거리를 Table을 사용하여 저장하였습니다. 

{% highlight java%}
@Test
public void whenCreatingTable_thenCorrect() {
    Table<String,String,Integer> distance = HashBasedTable.create();
    distance.put("London", "Paris", 340);
    distance.put("New York", "Los Angeles", 3940);
    distance.put("London", "New York", 5576);
 
    assertEquals(3940, distance.get("New York", "Los Angeles").intValue());
    assertThat(distance.columnKeySet(), 
      containsInAnyOrder("Paris", "New York", "Los Angeles"));
    assertThat(distance.rowKeySet(), containsInAnyOrder("London", "New York"));
}
{% endhighlight %}


그리고 아래 예제와 같이 Tables.transpose() 를 통해 row와 column 키를 뒤집을 수 있습니다. 

{% highlight java%}
@Test
public void whenTransposingTable_thenCorrect() {
    Table<String,String,Integer> distance = HashBasedTable.create();
    distance.put("London", "Paris", 340);
    distance.put("New York", "Los Angeles", 3940);
    distance.put("London", "New York", 5576);
 
    Table<String, String, Integer> transposed = Tables.transpose(distance);
 
    assertThat(transposed.rowKeySet(), 
      containsInAnyOrder("Paris", "New York", "Los Angeles"));
    assertThat(transposed.columnKeySet(), containsInAnyOrder("London", "New York"));
}
{% endhighlight %}



# 6. ClassToInstanceMap

다음으로 ClassToInstanseMap을 살펴보겠습니다. object의 클래스를 key로 사용하고자 할 경우 ClassToInstanceMap을 사용할 수 있습니다. 


{% highlight java%}
@Test
public void whenCreatingClassToInstanceMap_thenCorrect() {
    ClassToInstanceMap<Number> numbers = MutableClassToInstanceMap.create();
    numbers.putInstance(Integer.class, 1);
    numbers.putInstance(Double.class, 1.5);
 
    assertEquals(1, numbers.get(Integer.class));
    assertEquals(1.5, numbers.get(Double.class));
}
{% endhighlight %}



# 7. Group List using Multimap

다음은 Multimap을 통해 List를 그룹화 하는것을 보겠습니다. 다음 예제에서 Multimaps.index() 를 사용해 리스트에 있는 이름(names)의 길이로 그룹화 시킬 수 있습니다. 


{% highlight java%}
@Test
public void whenGroupingListsUsingMultimap_thenGrouped() {
    List<String> names = Lists.newArrayList("John", "Adam", "Tom");
    Function<String,Integer> func = new Function<String,Integer>(){
        public Integer apply(String input) {
            return input.length();
        }
    };
    Multimap<Integer, String> groups = Multimaps.index(names, func);
 
    assertThat(groups.get(3), containsInAnyOrder("Tom"));
    assertThat(groups.get(4), containsInAnyOrder("John", "Adam"));
}
{% endhighlight %}



# 8. Conclusion

위 예제들을 통해 Guava 라이브러리를 통해 Map을 유용하게 사용할 수 있는 여러가지 예제를 살펴봤습니다. 
위 예제 코드들은 [Guava github 프로젝트](https://github.com/eugenp/tutorials/tree/master/guava#readme)에서 찾아볼 수 있습니다. - 이클립스 기반 프로젝트로 쉽게 import하여 사용할 수 있습니다. 













