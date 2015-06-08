---
layout: post
title: linux postfix를 활용한 mail 전송
categories: [linux, postfix]
tags: [linux, postfix]
description: linux postfix를 활용한 mail 전송
---

### postfix

* [postfix](http://www.postfix.org)
* 정의를 찾아보자면 `mail server` `alternative to the widely-used Sendmail program` 로 축약해 볼 수 있을것 같습니다. 메일 서버이며 sendmail 의 대안? 후발주자인 것 같습니다.
* 메일 서버를 직접 만들어 사용하는 사람은 극히 드물것입니다. rfc 문서를 읽어가며 프로토콜을 언제 다 이해하고 이를 구현체로 만드는 작업이 쉽지많은 않을것이기 때문입니다.
* 리눅스 기반에서는 *yum*이나 *apt-get* 으로 `posftfix`를 쉽게 설치할 수 있고 사용법 또한 생각보다 간단한 것 같습니다. 물론 보안이나 세부 설정으로 들어간다면 간단하지 않습니다.


##### Install postfix 

* CentOs를 사용하고 있어 yum 을 통해 설치합니다.(root권한 필요)
<br>
```
yum -y install postfix
```
<br>

* postfix를 실행 전에 /etc/postfix/main.cf 설정 파일을 수정해 주면 됩니다. 기본으로 주석 처리 되어있는 부분을 주석 해제하고 각 서버에 맞게 수정해주면 됩니다.

```
vi /etc/postfix/main.cf
```


```
# 기본적인 동작을 위해 필요한 부분만 보고 나머지는 생략

# 호스트 명 
myhostname = smtp.myhostname.com

# 도메인 명
mydomain = mydomain.com

# MAIL From 설정
myorigin = $mydomain

# 메일 수신 네트워크 범위 지정 
# 전체 수신을 원하면 all, 혹은 127.0.0.0/8 특정 대역만 수신하도록 지정할 수 있습니다.
net_interfaces = all
#inet_interfaces = $myhostname
#inet_interfaces = $myhostname, localhost
#inet_interfaces = localhost

# 메일 서버가 최종 수신처가 되는 메일수신 도메인 설정
mydestination = $myhostname, localhost.$mydomain, localhost, $mydomain


# 릴레이할 도메인 지정. 
# 외부로 relay 시킬 경우 다른(ex> gmail) smtp로 바로 밀어넣을 수 있습니다.
relay_domains = $mydestination
```
<br>
<br>
<br>
* CentOs 버전에 따라 sendmail이 설치 되어 있다면 이를 `chkconfig` 로 확인 후 `postfix`로 수정 해 주면 됩니다. 

<br>

```
# chkconfig postfix on
# chkconfig --list postfix
postfix         0:off   1:off   2:on    3:on    4:on    5:on    6:off
```

<br><br>

* 이제 postfix 메일서버를 가동해 보고 25번 포트에서 리스닝 하는지 확인하자!!
<br><br>

```
# /etc/init.d/postfix start
Starting postfix:                                          [  OK  ]

# netstat -nap|grep :25
tcp        0      0 127.0.0.1:25                0.0.0.0:*                   LISTEN
```

```
# telnet localhost 25
Trying 127.0.0.1...
Connected to localhost.localdomain (127.0.0.1).
Escape character is '^]'.
220 smtp.myhostname.com ESMTP Postfix
MAIL FROM: thswave@naver.com
250 2.1.0 Ok
RCPT TO: thswave@gmail.com
250 2.1.5 Ok
DATA
354 End data with <CR><LF>.<CR><LF>
Subject: title
mail content body
.
250 2.0.0 Ok: queued as B64BC48DDA0
QUIT
221 2.0.0 Bye
```


참조:

* [http://postfix.state-of-mind.de/patrick.koetter/smtpauth/postfix_configuration.html](http://postfix.state-of-mind.de/patrick.koetter/smtpauth/postfix_configuration.html)
* [http://fsteam.tistory.com/67](http://fsteam.tistory.com/67)