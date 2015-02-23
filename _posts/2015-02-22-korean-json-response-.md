---
layout: post
title: spring에서 json response 한글 깨짐 
categories: [spring]
tags: [spring, json]
description: spring에서 json response 한글 깨짐 
---


상황 : RESTful api 구현을 위해 @RestController에서 json객체를 produces를 application/json으로 설정 시 한글 깨짐

the default charset of @ResponseBody is iso-8859-1, how to change to `utf8`.

{% highlight java %}
@RequestMapping(value = "/path", produces="text/plain;charset=UTF-8")
public @ResponseBody String handlePath() {

    .....

}
{% endhighlight %}


produces="text/plain;charset=UTF-8"를 사용하여 응답 페이지에 대한 UTF-8 인코딩이 가능하여 한글 깨짐을 방지 할 수 있음.


