---
layout: post
title: jenkins에서 fabric을 활용한 원격 배포 remote deploy 
categories: [server]
tags: [jenkins, CI, deploy, fabric, make]
description: jenkins에서 fabric을 활용한 원격 배포 remote deploy 
---



* Makefile

```
all:

deploy: build
	fab -R $(PROFILE) deploy:$(PROFILE)

build: check-profile clean
	sbt clean stage

init: check-profile
	fab -R $(PROFILE) init

check-profile:
ifndef PROFILE
	$(error PROFILE is undefined)
endif

clean: clean-target

clean-target:
	rm -rf target

.PHONY: clean-deploy clean-target check-profile

# vim:noet
```



