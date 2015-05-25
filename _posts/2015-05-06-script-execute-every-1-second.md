---
layout: post
title: run script every 1 second via watch(not cron)
categories: [shell script, cron]
tags: [shell script, cron]
description: run script every 1 second via watch
---

### 1초(혹은 x초) 마다 스크립트 실행

#### 한 줄 요약

```
$ watch -n 1 execute command
```


서비스의 정상 동작 여부를 확인하기 위해 매 초 마다 서비스에 요청을 보내는 스크립트를 실행해야 했습니다.
처음에는 스케쥴링을 위해 crontab을 설정하려고 보니 최소 단위가 `분 단위 스케쥴링`을 지원 해 주기 때문에 요구사항을 만족 시킬 수 없었습니다.
  
두 번째로 생각한 방법은 쉘 스크립트로 while sleep을 실행하는 것인데 실행 작업 수행 시간 + 1초 간격으로 실행되기 때문에 반드시 1초마다 실행 시키려면 작업 수행 시간을 계산해 1초에서 작업시간 만큼 빼고 sleep 시키는 부가 작업이 필요했습니다.  


```
while [true];
do
  something;
  sleep 1000;
done 
```

좀 더 간단한 방법을 찾던 중 `watch` 명령어를 발견.

```
$ watch -n 1 execute command
```


`watch: execute a program periodically, showing output fullscreen`
  
* -n [seconds] : 특정 초마다 실행시킬 수 있는 옵션 ex> watch -n 10


