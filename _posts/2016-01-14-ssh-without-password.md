---
layout: post
title: ssh login without password
categories: [linux]
tags: [linux, ssh]
description: ssh login without password
---

### 비밀번호 없이 ssh 로그인
  

로컬(A로 지칭)에서 특정 서버(B로 지칭)로 ssh 원격접속하기 위해 매 번 비밀번호를 입력하는 것이 불편하여 비밀번호 없이 접속하기
  
  
* Hot to do it  
  


```
a@A:~> ssh-keygen -t rsa
Generating public/private rsa key pair.
Enter file in which to save the key (/home/a/.ssh/id_rsa):
Created directory '/home/a/.ssh'.
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
Your identification has been saved in /home/a/.ssh/id_rsa.
Your public key has been saved in /home/a/.ssh/id_rsa.pub.
The key fingerprint is:
3e:4f:05:79:3a:9f:96:7c:3b:ad:e9:58:37:bc:37:e4 a@A
```

* B 서버의 b유저로 ssh 접속하여  ~/.ssh 디렉토리 생성(이미 해당 디렉토리가 존재한다면 패스):
  


```
a@A:~> ssh b@B mkdir -p .ssh
b@B's password:
```
  

* A에서 생성한 id_rsa.pub(public_key)를 B의 .ssh/authorized_keys 파일에 입력.
  

```
a@A:~> cat .ssh/id_rsa.pub | ssh b@B 'cat >> .ssh/authorized_keys'
b@B's password:
```
  
* 이제부터 password 없이 A에서 B로 ssh 로그인 할 수 있습니다.
  

```
a@A:~> ssh b@B
```

##### 참고  

[http://www.linuxproblem.org/art_9.html](http://www.linuxproblem.org/art_9.html)