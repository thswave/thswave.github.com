---
layout: post
title: sinatra를 활용한 간단한 서버 구축
categories: [ruby, sinatra]
tags: [ruby, sinatra]
description: sinatra를 활용한 간단한 서버 구축
---

### Sinatra

* [Sinatra 한글 홈페이지](http://www.sinatrarb.com/intro-ko.html)

* 외부 서버와 통신이 필요한데 어떤 이유(로컬에서 연결이 안된다거나, 외부 서버가 아직 미구현 단계이거나)에 의해 이를 대신할 서버가 필요한 경우가 있습니다.
* 만약 여러분이 간단하면서도 빠르게 이 외부서버를 대체할 임시 목(mock) 서버를 구축해야 하다면 어떻게 만드시겠습니까?
* 로컬 환경에서 개발하면서 외부 서버를 대체할 목 서버를 위해 스프링을 쓴다거나 아니면 톰켓에 띄우기 위해 Servlet을 구현하고 이를 서버로 띄우는 건 개인적으로 번거롭다는 생각이 듭니다.  단지 원하는건 `특정 요청에 대한 특정 응답`만 필요하기 때문에 무언가 부가 작업이 많이 들고 번거로움이 앞서기 때문입니다.
* 다양한 경량 웹서버가 많이 소개되어 있지만 개인적으로 [Sinatra](http://www.sinatrarb.com/) 활용하여 목 서버 역활을 하도록 구현했습니다. 
* [java의 spark](http://sparkjava.com/)나 , [python flask](http://flask.pocoo.org/), node 등을 활용할 수 있으니 언어나 개인적 취향에 맞춰 선택할 수 있습니다. 
<br><br>
#### Sinatra?
<br>
* `quickly` creating web applications in Ruby with `minimal effort`
* Sinatra는 위 설명에서 알 수 있듯 빠르고 쉽게 만들 수 있음이 특징입니다.
<br>
* ruby gem이 설치되어 있다고 가정하겠습니다. sinatra 설치는 다음과 같습니다. 만약 gem이 설치되어 있지 않다면 `gem install` 검색 하시면 됩니다.
<br>
```
gem install siantra
```
<br>
* 파일을 하나 만들고 (ex> mock.rb) 다음과 같이 작성합니다.
<br>
{% highlight ruby %}
# mock.rb file
require 'sinatra'
get '/' do
  'Hello world!'
end
{% endhighlight%}

<br>
* 콘솔에서 다음과 같이 실행하면 기본 4567 포트로 웹 서버가 뜹니다.

```
ruby mock.rb
```

* 저 같은 경우 특정 url 응답으로 json 필요해 아래와 같이 사용했습니다. 

{% highlight ruby %}
# mock.rb file
require 'sinatra'
require 'json'

get '/url' do
  content_type :json
  { :key1 => 'value1', :key2 => 'value2' }.to_json
end
{% endhighlight%}

* get뿐만 아니라 post, patch, delete 등 다양한 method를 사용할 수 있습니다. 공식 홈페이지에 다양한 사용법이 설명되어있으니 참고하시면 됩니다.   

* 사실 sinatra는 간단한 웹서버 뿐 아니라 정말 다양한 방법으로 응용할 수 있습니다. 경량 웹서버로 쓰거나 RESTful 방식으로 쓸 수도 있고 활용법은 정말 다양합니다. 
