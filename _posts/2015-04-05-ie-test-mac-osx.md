---
layout: post
title: 맥 OSX에서 IE 테스트
categories: [mac]
tags: [test, mac, osx]
description: 맥 OSX에서 IE 테스트
---



`본 포스트는 (아직까지) IE 11(최신 버전)에서 테스트 하는 방법을 소개해 드리고자 합니다. 테스트 이외의 용도(ex> 쇼핑, 결재)는 부적합 한 방식임을 미리 알려드립니다.`



### 맥에서 IE 테스트하기


맥에서 웹 서비스를 개발하면서 IE(Internet Explorer)를 테스트하는 건 정말 고역입니다. 개발자라면 한번쯤 IE를 없어졌으면 좋겠다는 생각을 한 번 쯤....을 넘어 매번 매순간 느끼실 겁니다.


만약 IE에서의 테스트를 해야한다면 몇가지 대안이 있습니다. 

#### 1. [VirtualBox](http://www.browserstack.com/), [VMWare](http://www.vmware.com/), [Hyper-V](http://www.microsoft.com/en-us/server-cloud/solutions/virtualization.aspx) 와 같은 VM(Virtual Machine)에 IE 버전 별로 각각 설치후 테스트
<br>
장점: 윈도우와 똑같은 환경. 가장 일반적인 방법
단점: 가상머신을 띄우는데 많은 메모리 소모. 여러대의 VM을 띄울경우 느려질 수 있음.
기존 [mordern.ie](https://www.modern.ie/virtualization-tools) 에서 vm에 각 ie 버전 별로 설치된 이미지를 제공했으나 사용 기간이 정해져 있어 만기되면 다시 설치해야되는 불편함
<br>
<br>
#### 2. [browserstack](http://www.browserstack.com/)과 같은 브라우저 테스트 클라우드 서비스 이용
<br>
장점: IE뿐 아니라 OS별 브라우저 별 다양한 지원  
단점: 영어 + `유료 (무료이용은 30분 시간제한)`, 로컬 테스트의 번거로움
<br>
<br>
#### 3. 윈도우가 설치된 노트북 대여 후 테스트
<br>
장점: 설치가 필요없으므로 노트북을 대여 후 원격 접속  
단점: 노트북을 추가로 구할 수 있는 환경(회사 차원 지원)적 제약사항
<br>
<br>
#### 4. *[mordern.ie](http://remote.mordern.ie) + [ngrok](https://ngrok.com/)* 조합으로 `IE 11` 테스트
<br>
지금부터 소개시켜 드릴 방법은 모든 IE 버전이 테스트 되는것도 아니고 만능은 아님을 미리 알려드립니다.  
테스트 환경은 IE 11입니다.  
현재는 최신 버전만 지원하지만 추후 다양한 버전을 테스트 환경으로 사용될 수 있을 것 같습니다.  

##### 1) [mordern.ie](http://remote.mordern.ie) 접속 및 sign in  
<br>
![0405_1.png](/assets/media/0405_1.png)  
<br>
아이디가 없다면 새로 가입해서 만드셔야 합니다.  
가입 했다면 인증 메일이 몇 분후에 도착해서 활성화 시켜줍니다.  
<br>
![0405_2.png](/assets/media/0405_2.png)  
<br>
로그인 하시면 remote desktop을 할 호스트 OS에 따른 설치 프로그램이 있습니다. 본 포스트는 mac을 기준으로 테스트 하지만 iphone/android 등 다양한 환경에서도 접속이 가능합니다.  
<br>
![0405_3.png](/assets/media/0405_3.png)  
<br>
`Microsoft Remote Desktop for Mac` 을 선택하면 [Microsoft Remote Desktop](https://itunes.apple.com/us/app/microsoft-remote-desktop/id715768417) 을 App Store에서 다운받아 설치합니다.  
<br>
![0405_4.png](/assets/media/0405_4.png)
<br>
<br>
앱 설치 후 실행 시키면 `Azure RemoteApp` 을 선택하고 다시 한번 Microsoft 계정으로 로그인 합니다.  
<br>
![0405_5.png](/assets/media/0405_5.png)  
<br>
<br>
그 후 Internet Exploer 체크 박스를 체크 해주고 닫아주면 목록에 `IE Technical Preview` 앱이 생성 됩니다.  
<br>
![0405_6.png](/assets/media/0405_6.png)
![0405_7.png](/assets/media/0405_7.png)  
<br>
<br>
접속하면 Internet Explorer 창이 보이고 `F12` 키를 누르면 *개발자 도구*를 열어볼 수 있습니다.  
<br>
![0405_8.png](/assets/media/0405_8.png)
![0405_9.png](/assets/media/0405_9.png)
<br>
<br>
IE에서 이제 테스트할 서비스 주소로 접속하면 됩니다. 하지만 remote 앱은 클라우드 환경에서 접속하기 때문에 local 테스트가 불가능 합니다. 그럼 local 앱을 외부에서도 접속할 수 있도록 프록시를 설정하거나 HTTP 터널을 설정해 주면 되는데 [ngrok](https://ngrok.com/)이 HTTP Tunnel을 대신해 주는 서비스 입니다.
<br>
##### 2) [ngrok](https://ngrok.com/) 을 통한 local 서비스 HTTP 터널링
<br>
ngrok 사이트에가서 다운로드/설치 후   
<br>

```
ngrok 80  # 80 대신 서비스가 떠있는 port 번호를 입력해도 됩니다.

``` 
<br>

실행하면 ngrok에서 http://4ec80ab0.ngrok.com 와 같은 주소를 넘겨줍니다. 이는 local이외 외부에서도 접속 가능합니다.  
<br>
![0405_10.png](/assets/media/0405_10.png)
<br>
<br>
ngrok에서 생성된 주소를 토대로 IE remote app에서 접속하여 테스트 하면 됩니다.

<br>
![0405_11.png](/assets/media/0405_11.png)
<br>
<br>
<br>
### 결론
<br>
#### 장점
* 별도의 VM 설치 없이 가볍게 IE 최신버전에서 테스트해볼 수 있습니다.
* 아직 개발이 진행중이므로 추후 IE 최신버전과 더불어 다양한 IE 버전을 테스트 해 볼 수 있을 것 같습니다.  

#### 단점
* *웹서핑이나 결재는 불가능 합니다. acticeX 및 기타 플러그인 설치 불가능* (결재를 위해 VM으로 IE 접속하고 있다보니 개인적으로 이부분이 가장 아쉽네요.)
* 아직까지 불안정합니다.(장시간 멈춤 현상)
* 로컬 테스트를 위해선 [ngrok](https://ngrok.com/)과 같은 별도의 HTTP가 필요합니다. 















