---
layout: post
title: javascript 이벤트 핸들러 등록
categories: [javascript]
tags: [javascript, event handler]
description: javascript event handler 등록
---

{% highlight javascript %}
var test = document.getElementById("id");

test.onclick = functionName1;
test.onclick = functionName2;

// 기존에 이벤트 처리가 하나만 처리되는 방식이었지만 아래와 같이 하면 두개의 리스너를 달 수 있다.

test.addEventListener("click", functionName1, false); 
test.addEventListener("click", functionName2, false);

{% endhighlight %}


이벤트 리스너의 세번째 매개변수는 이벤트 처리단계가 캡처(true)단계인지 버블링 단계(false)인지 구분

### 이벤트 e

event.target.nodeName;
event.target.id;
event.type;

마우스 이벤트의 종류

```
click
dbclick
mousedown
mouseup
mouseover
mousemove
mouseout
``` 

### 마우스 이벤트 발생 위치 구하기

* clientX clientY : 웹브라우저에서 문서가 출력되는 좌표
* screenX screenY : 전체 화면에서의 좌표

마우스는 문서상의 동일한 위치에 있지만 이벤트가 발생할 경우 이벤트가 문서의 어떤 좌표에서 발생했는지 알아내려면 스크롤된 값을 반영해 주어야 한다.

{% highlight javascript %}
scrollLeft +=document.documentElement.scrollLeft
scrollTop += document.documentElement.scrollTop
{% endhighlight %}

마우스 클릭 버튼 판별

0 : 일반 버튼
1 : 추가 버튼 ( 휠 관련)
2 : 콘텍스트 버튼 (오른쪽)