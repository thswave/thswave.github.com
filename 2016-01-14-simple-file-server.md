---
layout: post
title: ruby simple httpd (similar python SimpleHTTPServer)
categories: [ruby]
tags: [ruby, httpd]
description: ruby simple httpd (similar python SimpleHTTPServer)
---


#### 특정 디렉토리의 파일들을 다른 서버/컴퓨터 등에서 다운로드/접근 하고 싶을 경우
  
[sinatra를 활용한 간단한 서버 구축](http://thswave.github.io/ruby/sinatra/2015/05/26/introduce-sinatra.html) 과는 별도로 파일을 다른 위치(다른 서버)에서 접근하고 싶은 경우 보통은 다운로드 받아 해당 서버로 `scp`를 통해 넣어주곤 했는데 빠르게 전달하기 불편한 상황이 있을 수 있습니다.(예를 들어 각 서버에 접근하기 위해 여러 단계의 인증을 거쳐야만 접근할 수 있는 경우)  
  
이럴 때 ftp를 열어준다거나 파일 전송을 위해 아파치를 띄워 특정 경로 파일을 지정해주는 등 여러가지 방법이 있을 수 있지만 빠르고 쉽게 하는 방법을 찾았습니다.(혹시 더 간단한 방법이 있다면 제보 부탁드립니다.)
  
공유할 파일이 위치한 디렉토리로 이동 한 뒤 ruby/python으로 서버를 띄웁니다.
  
* ruby

```
ruby -run -e httpd . -p <port번호>
```

* python  
  
```
python -m SimpleHTTPServer <port번호>
```
  

브라우저로 접근할 경우 해당 디렉토리의 파일 리스트를 확인할 수 있습니다. 
  
```
curl http://<serverIp>:<port>/<filename>
```  
  

* 참고
** [http://stackoverflow.com/questions/3108395/serve-current-directory-from-command-line](http://stackoverflow.com/questions/3108395/serve-current-directory-from-command-line)