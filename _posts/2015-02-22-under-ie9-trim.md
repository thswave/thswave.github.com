---
layout: post
title: javascript trim() method
categories: [javascript]
tags: [javascript]
description: javascript trim() method
---

문자열 앞 뒤 공백을 제거하기 위해 trim()을 적용해야 하는데 trim() 메서드는 ie9 이상 부터 적용 가능하다.
즉, `ie8` 이하에서 에러 발생한다. 

* solution

* regex 

```
// ltrim
str.replace(/\s+$/,"");
// rtrim
str.replace(/^\s+/,"");
// trim
str.replace(/^\s+|\s+$/g,"");
```