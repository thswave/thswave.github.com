---
layout: post
title: logrotate
categories: [linux]
tags: [logrotate]
description: 로그 로테이트, 아파치 로그 파일을 주기적으로(하루 단위로 rotation) 압축하여 일 별 관리하고 싶을 경우
---

### 상황 : 아파치 로그 파일을 주기적으로(하루 단위로 rotation) 압축하여 일 별 관리하고 싶을 경우

기존 아파치 설정  httpd.conf

```
CustomLog    "|/program/httpd-2.2.20/bin/rotatelogs -l /program/httpd-2.2.20/logs/%y%m%d.sample.activity_log 86400" combined env=!nolog

ErrorLog     "|/program/httpd-2.2.20/bin/rotatelogs -l /program/httpd-2.2.20/logs/%y%m%d.sample.error_log 86400"
```

* 하루 단위로 rotation 하고 있었지만 압축 저장하고 있지 않음.

### 해결방법 : 리눅스 기본 logrotate 활용!

* *logrotate란?* 

로그 파일(시스템 로그)을 rotates, compresses, and mails 을 할 수 있다.
설정 파일을 변경해도 관련 프로세스를 새로 시작할 필요 없이 cron 데몬이 주기적으로 실행 시켜준다.

```
$ vi /etc/cron.daily 
```

위 cron을 확인해보면 logrotate 설정 부분을 확인할 수 있다.

logrotate 관련 파일

 - /usr/sbin/logrotate : 데몬의 위치 및 데몬프로그램
 - /etc/logrotate.conf : 설정 파일.
 - /etc/logrotate.d : logrotate를 적용할 로그 파일 보관 디렉토리.
 - /var/lib/logrotate.status : logrotate가 작업 내역 보관 파일.
 - /etc/cron.daily/logrotate : logrotate : cron 에 의해 일 단위로 실행한다.



동작 순서를 살펴보면 
1. cron.daily 에서 /usr/sbin/logrotate 호출 
2. /usr/sbin/logrotate 에서 /etc/logrotate.conf 설정파일 참조 
3. /etc/logrotate.conf 설정 파일에서 /etc/logrotate.d 참조 ( logrotate.conf 파일 안에 "include /etc/logrotate.d")


logrotate가 정상 동작 하는지 최소한의 설정으로 확인해 보자

```
$ sudo vi /etc/logrotate.d/apache

/program/apache/logs/access_log {
  size +1k
  missingok
  notifempty
  create 0600 root root
  compress
  dateext
  postrotate
    /usr/bin/killall -HUP httpd
  endscript
}
```

각 옵션들은 잠시 뒤에 상세 설명하고 우선 당장 실행 시켜 보자.

루트 권한으로 아래 명령 실행.

```
$ /usr/sbin/logrotate -f /etc/logrotate.conf
```

-f 옵션은 강제 실행 옵션이다 (Tells logrotate to force the rotation, even if it doesn't think this is necessary)

```
$ /usr/sbin/logrotate -d /etc/logrotate.conf
```

-d 옵션 : 디버그 모드 (Turns on debug mode and implies -v. In debug mode, no changes will be made to the logs or to the logrotate state file.)

* 주의사항 : f 옵션이나 d 옵션 하나만 넣고 실행해야 한다. (두 옵션을 동시에 넣고 실행시 정상 동작안되서 한동안 삽질....)

* logrotate 옵션
copytruncate : Truncate the original log file to zero size in place after creating a copy, instead of moving the old log file and optionally creating a new one 
copytruncate옵션을 활용하면 postrotate를 통한 httpd 재시작 없이 무중단 로깅이 가능하다.

```
/path/to/log {
  daily
  copytruncate
  create 0700 root root
  compress
  notifempty
  missingok
  dateext
}
```

* rotate 30(숫자)  : log파일 30개 이상 되면 삭제
* maxage 30(숫자) : 30일 이산된 로그 파일 삭제
* size : 지정한 용량이 되면 로그로테이트를 실행한다. 10k, 10M 이런식으로 지정한다.
* create : [권한 유저 그룹] 으로 rotation된 로그파일 생성
* notifempty : log 내용이 없으면 rotation 하지 않는다.
* ifempty : 로그파일이 비어있는 경우에도 로테이트한다.
* monthly : 월 단위로 로테이트 한다.
* daily : 월 단위로 로테이트 한다.
* weekly : 월 단위로 로테이트 한다.
* compress : rotate 된 로그 gzip 압축
* nocompress : 압축을 원치 않는다.
* mail admin@mail : 로테이트 설정에 의해 보관주기가 끝난 파일을 메일로 발송한다.
* mailfirst admin@mail : 로테이트시 신규파일 이전의 로그를 메일로 발송한다.
* nomail : 메일로 통보받지 않음.
* errors admin@mail : 로테이트 실행시 에러가 발생하면 이메일로 통보한다.
* prerotate-endscript : 사이의 명령어를 로그파일 처리전에 실행한다.
* postrotate-endscript : 사이의 명령어를 로그파일 처리후에 실행한다.
* extension : 로테이트 후 생성되는 파일의 확정자를 지정한다.
* copytruncate : 이옵션을 넣지 않으면 현재 사용중인 로그를 다른이름으로 move하고 새로운 파일을 생성한다.

이 외의 옵션은 하단 참조에 link를 참조하거나 man logrotate를 확인하면 된다. 

보통은 rotation 후 아래 postrotate를 통해 httpd 를 재시작 해준다. 

```
postrotate
    /usr/bin/killall -HUP httpd
endscript
```

이 때 두가지 대안이 있다. 

1. killall -HUP 프로세스이름  (예> /usr/bin/killall -HUP httpd)
2. kill -HUP 프로세스번호 (예> /usr/bin/kill -HUP `cat /daum/program/apache/logs/httpd.pid 2> /dev/null` 2> /dev/null || true)

### 참조 링크 
* [logrotate 정의](http://manpages.ubuntu.com/manpages/precise/man8/logrotate.8.html)
* [Understanding logrotate - part 1](http://www.rackspace.com/knowledge_center/article/understanding-logrotate-* part-1)
* [How to Rotate Apache Log Files in Linux](http://www.thegeekstuff.com/2011/07/rotate-apache-logs/ )
* [logrotate example](http://www.thegeekstuff.com/2010/07/logrotate-examples/)
* [stackexchange](http://unix.stackexchange.com/questions/47688/how-to-avoid-apache-reload-when-rotating-logs)