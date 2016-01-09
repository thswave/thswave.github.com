---
layout: post
title: java detect encoding
categories: [java]
tags: [java, encoding, charset]
description: java detect encoding
---

프로그램을 짜면서 온갖 한글 깨짐 현상을 경험해 보셨을겁니다. 
한글이 깨지는 곳은 많지만 웹 개발을 하면서 자주 경험하는 곳은 `tomcat`, `html 페이지`, `DB` 정도 같습니다. 하지만 비교적 간단한 설정으로 해결할 수 있는것 같습니다.
(ex> tomcat은 Connector에서 URIEncoding, DB 스키마/테이블 인코딩 설정, DB connection url에서의 인코딩 명시 등등)
오늘 알아볼 인코딩 탐지는 조금 특수한 케이스라 할 수 있습니다.
  

   
#### *파일에서 읽어들인 byte[]의 인코딩을 탐지하는 방법*

* 사실 java 8 에서(이전 버전은 동일하지 않을 수 있습니다) 인코딩을 굳이 탐지 하지 않아도 자동으로 처리해 줍니다.

String 생성자로 charset을 명시적으로 넘겨주기도 하지만 `String(byte[] bytes, Charset charset)`, byte[]만 넘겨줘도 내부적으로 인코딩을 탐지 해 줍니다. `String(byte[] bytes)`
하지만 이렇게 생성된 String은 어떤 인코딩으로 처리되었는지 알수 없습니다.(알 수 있는 방법이 있다면 댓글 부탁드리겠습니다.) 

인코딩인지 탐지 라이브러리를 사용해서 알 수 있습니다.

* 참조: [http://mvnrepository.com/artifact/com.googlecode.juniversalchardet/juniversalchardet](http://mvnrepository.com/artifact/com.googlecode.juniversalchardet/juniversalchardet)

예제.

```
private String detectCharset(byte[] bytes) {
		UniversalDetector detector = new UniversalDetector(null);
		detector.handleData(bytes, 0, bytes.length);
		detector.dataEnd();
		String detectedCharset = detector.getDetectedCharset();
		if (detectedCharset != null && detector.isDone() && Charset.isSupported(detectedCharset)) {
           return detectedCharset;
		}
	}
```

아주 간단한 예제로 작성되었지만 쉽게 이해하실 수 있습니다. 
  


#### 참고

* encoding? charset? 무슨 차이인가?

> `charset` is the set of characters you can use  
> `encoding` is the way these characters are stored into memory

[http://stackoverflow.com/questions/2281646/whats-the-difference-between-encoding-and-charset](http://stackoverflow.com/questions/2281646/whats-the-difference-between-encoding-and-charset)

위 링크의 답변 중 특별한 사항이 있다면 javadoc에서 encoding을 charset로 잘못사용했다는 의견입니다.

```
Note that javadoc wrongly uses "charset" instead of "encoding", for example in InputStreamReader, we read "An InputStreamReader is a bridge from byte streams to character streams: It reads bytes and decodes them into characters using a specified charset. The charset that it uses may be specified by name or may be given explicitly, or the platform's default charset may be accepted.". However, what they mean is "encoding". –
```

Q. UTF-8은 encoding? charset?
A. Encoding입니다. unicode를 표현하는 여러 인코딩 중 하나입니다.(UTF-16, UTF-8 등등)

