---
layout: post
title: Spring RequestMapping url with digit, string 구분 방법. 
categories: [spring, java]
tags: [java, spring, requestmapping, 정규표현식, regex, regular expression]
description: Spring apply RequestMapping with difference digit, string 
---


### 상황 
* rest api 로 `@RequestMapping` uri를 설계 중 동일한 형태지만 숫자와 문자를 서로 다른 매핑을 적용하고 싶을 경우.

```
GET /abc/123
GET /abc/hi
```

* 위와 같은 형태의 uri를 `@RequestMapping("/abc/{id}")` 로 매핑할 경우 id가 숫자인지 문자인지 구분할 수 없어 적합한 매퍼로 연결할 수 없습니다.

{% highlight java 

@RequestMapping(value = "/abc/{id}", method = RequestMethod.GET)
public void a(@PathVariable long id) {}

@RequestMapping(value = "/abc/{name}", method = RequestMethod.GET)
public void b(@PathVariable String name) {}

{% endhighlight %}



#### 해결

* `정규식`!

{% highlight java 

@RequestMapping(value = "/abc/{id:[\\d]+}", method = RequestMethod.GET)
public void a(@PathVariable long id) {}

@RequestMapping(value = "/abc/{name:[\\w]+}", method = RequestMethod.GET)
public void b(@PathVariable String name) {}

{% endhighlight %}


* `id:[\\d]+` : 숫자.
* `name:[\\w]+` : 문자.
