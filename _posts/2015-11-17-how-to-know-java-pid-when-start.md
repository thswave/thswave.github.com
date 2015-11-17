---
layout: post
title: linux 에서 java 프로그램 시작 시 pid 알아내기 
categories: [java, linux]
tags: [java, linux, pid]
description: linux 에서 java 프로그램 시작 시 pid 알아내기 
---


### 상황 
* 웹 서비스가 아닌 일반 자바 프로그램을 CI 서버에 연동하는 과정에서 tomcat이나 play framework 처럼 start, stop으로 프로세스를 정지, 실행 시켜야 했습니다.
* `grep` 으로 pid를 찾을 수 도 있지만 이는 100% 정확하다고 보장할 수 없기 때문에 프로그램이 시작되고 할당 받는 pid를 알아낼 방법이 필요했습니다. 

#### 해결

* 자바 프로그램 실행 직후 `echo "$!" > test.pid` 를 통해 pid를 알아낼 수 있습니다. 
* start, stop이 되도록 shell script 작성.

```
#!/bin/bash

PID_FILE="$(pwd)/test.pid"

case "$1" in
start)
  nohup java -jar test.jar &
  echo "$!" > $PID_FILE
  echo STARTED
  ;;
stop)
  if [ -f $PID_FILE ]; then
    kill `cat $PID_FILE`
    echo "STOPPED"
    rm $PID_FILE
  else
    echo "It seems that the process isn't running."
  fi
  ;;
esac
```