---
layout: post
title: object list subtract 
categories: [java]
tags: [java, list, subtract, removeAll, CollectionUtils]
description: object list subtract via CollectionUtils
---


### 상황 
* String이나 primitive 타입이 아닌 리스트 `List<A>` 간 subtract가 필요했습니다.  
* 리스트 A와 리스트 B는 서로 다른 객체이지만 필드들이 같은 값을 가지고 있고 `equals()` 메소드도 Override  
* 인터넷에 나와있는 대부분의 예시는 String이나 Primitive 타입(Integer, etc..)이라 `CollectionUtils.subtract(a, b)` 로 제거가 가능했지만 Object list는 제대로 차집합을 구하지 못해 여러차례 다른 방법 시도했습니다.

#### 해결

* `List c = CollectionUtils.removeAll(listA, listB)`  
* `removeAll()`의 경우 내부적으로 같은 아이템인지 확인하는 과정에 내부적으로 `equals()` 호출.
* removeAll 이란 이름에서 혹시 listA 에서 중복이 제거되지 않을까 싶지만 실제로는 `new ArrayList<>()` 로 새 리스트를 반환

{% highlight java %}
// package org.apache.commons.collections4.ListUtils
public static <E> List<E> removeAll(final Collection<E> collection, final Collection<?> remove) {
    final List<E> list = new ArrayList<E>();
    for (final E obj : collection) {
        if (!remove.contains(obj)) {
            list.add(obj);
        }
    }
    return list;
}

{% endhighlight %}
