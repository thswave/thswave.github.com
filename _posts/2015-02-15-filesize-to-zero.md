---
layout: post
title: linux file size 0으로 만들기 
categories: [linux]
tags: [linux, shell]
description: linux file size 0으로 만들기 cat /dev/null > dest
---

톰캣/아파치 로그가 쌓이고 있는데 log rotation을 하지 않는 상태에서 파일 사이즈만이라도 줄이고 싶을 수 있다.
(만약 파일을 지울경우 로그가 더이상 쌓이지 않기 때문에 지우지 않는 상태에서 파일 내용만 지워야 하는 경우)


```
cat /dev/null > dest
```