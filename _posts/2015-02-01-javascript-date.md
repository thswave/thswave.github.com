---
layout: post
title: javascript 날짜 계산 Date 사용법
categories: [javascript]
tags: [javascript, date]
description: javascript 날짜 계산 Date 사용법
---


`퍼온 글이나 원 출처를 분실하였습니다. 문제시 삭제 하겠습니다.`

{% highlight javascript %}
var loadDt = new Date(); //현재 날짜 및 시간   //현재시간 기준 계산
alert(new Date(Date.parse(loadDt) - 30 * 1000 * 60 * 60 * 24)); //30일전
alert(new Date(Date.parse(loadDt) - 15 * 1000 * 60 * 60 * 24)); //보름전
alert(new Date(Date.parse(loadDt) - 7 * 1000 * 60 * 60 * 24)); //일주일전
alert(new Date(Date.parse(loadDt) - 1 * 1000 * 60 * 60 * 24)); //하루전
alert(new Date(Date.parse(loadDt) + 1 * 1000 * 60 * 60 * 24)); //하루후
alert(new Date(Date.parse(loadDt) + 7 * 1000 * 60 * 60 * 24)); //일주일후
alert(new Date(Date.parse(loadDt) + 15 * 1000 * 60 * 60 * 24)); //보름후
alert(new Date(Date.parse(loadDt) + 30 * 1000 * 60 * 60 * 24)); //한달후
alert(new Date(Date.parse(loadDt) + 1000 * 60 * 60)); //한시간후
alert(new Date(Date.parse(loadDt) + 1000 * 60)); //1분후
alert(new Date(Date.parse(loadDt) + 1000)); //1초후

// 날짜를 입력 하면 오늘 날짜로부터 숫자만큼 전날의 날짜를 mm/dd/yyyy 형식으로 돌려 준다.
function caldate(day){
 
 var caledmonth, caledday, caledYear;
 var loadDt = new Date();
 var v = new Date(Date.parse(loadDt) - day*1000*60*60*24);
 
 caledYear = v.getFullYear();
 
 if( v.getMonth() < 9 ){
  caledmonth = '0'+(v.getMonth()+1);
 }else{
  caledmonth = v.getMonth()+1;
 }
 if( v.getDate() < 9 ){
  caledday = '0'+v.getDate();
 }else{
  caledday = v.getDate();
 }
 return caledmonth+'/'+caledday+'/'+caledYear;
}
{% endhighlight %}