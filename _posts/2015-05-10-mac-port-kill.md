---
layout: post
title: 특정 포트를 점유한 프로세스 아이디 확인
categories: [mac]
tags: [mac]
description: 특정 포트를 점유한 프로세스 아이디 확인
---



Mac에서 작업을 하다 보면 특정 포트 번호를 사용중인 프로세스 아이디를 알아야 할 일이 종종 발생합니다.
  
예를 들어 intelliJ로 서버를 띄우려고 하는데 해당 포트가 이미 사용중이라는 에러와 함께 서버가 올라가지 않곤 합니다. 

java라던가 혹은 특정 프로세스 이름이라도 알면 `ps` 명령어로 PID를 알아낼 수 있는데 어떤 프로세스인지 모르는 상황에 특정 포트가 점유된 상황일 경우 특정 포트번호를 사용중인 PID를 확인 후 이를 kill 시켜주어야 합니다. 


```
# lsof -n -i4TCP:포트번호 | grep LISTEN
lsof -n -i4TCP:1099 | grep LISTEN
```

`lsof` : list open files 확인 명령어

-n : inhibits the conversion of  network  numbers  to  host  names  for  network  files.  
  네트워크 관련 호스트네임 변환 방지로 경우에 따라 넣어주지 않아도 상관 없습니다.

-i : selects  the  listing  of  files  any of whose Internet address matches the address specified in i  
뒤에 따라오는 4 혹은 6은 ipv4/ipv6를 의미하며 그 뒤에는 프로토콜이 온다. -i 뒤에 올 수 있는 옵션들은 다음과 같습니다.

```
[46][protocol][@hostname|hostaddr][:service|port]
```