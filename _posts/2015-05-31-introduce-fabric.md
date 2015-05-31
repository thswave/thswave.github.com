---
layout: post
title: fabric을 활용한 배포
categories: [python, fabric]
tags: [python, fabric]
description: fabric을 활용한 배포
---


### fabric ?

* [http://www.fabfile.org/](http://www.fabfile.org/) 
* *Fabric is a Python (2.5-2.7) library and command-line tool for streamlining the `use of SSH for application deployment or systems administration tasks`*
* 빌드 서버에서 간단한 명령으로 배포 및 스크립트 실행과 같은 task 들을 실행 시켜주는 tool
* 언제 유용할까? 빌드 서버에서 jenkins로 빌드한 후 빌드 결과물을 원격 서버로 배포(deploy)하고 원격 서버에서 서버의 재시작이나 기본적인 작업(task)를 수행하기 할 때 사용할 수 있다. 물론 빌드 서버에서 원격 서버로 ssh + scp 를 하는 스크립트를 만들어 할 수 있지만 python의 기능과 더불어 사용할 수 있기 때문에 좀 더 쉽게 배포 및 task 처리를 할 수 있습니다.
<br>
* command-line tool 이므로 jenkins 빌드 후 `execute shell`을 통해 fabric을 실행 시켜주면 됩니다.

###### 설치 

* 설치가이드 : [http://www.fabfile.org/installing.html](http://www.fabfile.org/installing.html)

<br>

```
pip install fabric
# 또는 
sudo apt-get install fabric
```

<br>
* Fabric requires Python version 2.5 - 2.7
<br>

* 간단한 튜토리얼 : [http://docs.fabfile.org/en/latest/tutorial.html](http://docs.fabfile.org/en/latest/tutorial.html) 
* 간단하게 설명드리면 `fabfile.py` 파일을 프로젝트 working directory에 만들어 둡니다.

<br>
{% highlight python%}
#!/usr/bin/env python
# -*- conding: utf-8 -*-

from __future__ import with_statement
from fabric.api import *
from fabric.contrib.console import confirm
from StringIO import StringIO
import pprint
import time

env.use_ssh_config = True

env.roledefs = {
    'dev': {
        'hosts': ['id@host'],
    }
}


def hello():
    print("Hello world!")

{% endhighlight%}


* *실행: fab + method 명*

```
$ fab hello
Hello world!

Done.

```

* *argument 받기*

{% highlight python%}
# 생략..

def hello(name="world"):
    print("Hello %s!" % name)

{% endhighlight%}


```
$ fab hello:name=Changwon
Hello Changwon!

Done.

```

* 프로젝트에 활용한 몇 가지 활용 사례를 예제와 함께 설명하겠습니다.
<br>
<br>
{% highlight python%}
def stop_tomcat():
    with cd('/path/to/tomcat'):
        run('./shutdown.sh')
        time.sleep(5)

def upload_war():
    put('./target/WAR_FILE_NAME', 'path/to/upload');

def execute_cmd():
	run('ln -s a b')
	run('mkdir -p /path/new/dir/test')

{% endhighlight%}

* puth(a,b) : 빌드 서버의 a 파일을 원격 서버의 b 로 복사합니다. 디렉토리/파일 모두 가능합니다. 
* run() : 메서드의 인자로 원격 서버에서 수행할 명령어를 넣어주면 됩니다.
* 복잡하지 않고 간단하게 원하는 작업만 수행할 수 있는 점이 편리한 것 같습니다.



* ruby의 빌드/배포 툴인 [capistrano](http://capistranorb.com/)를 활용할 수 있었지만 기존에 python으로 세팅되어 있어 굳이 capistrano를 설치하지 않고 python을 활용하게 되었습니다.
