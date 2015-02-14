---
layout: post
title: truncate 와 delete 차이점.
categories: [database]
tags: [database]
description: truncate 와 delete 차이점.
---

Truncate 와 Delete, Drop은 모두 데이터를 삭제하는 쿼리이지만 차이가 존재합니다.

- Drop [Tablename] 

#### 테이블 스키마 + 데이터를 삭제 *(Rollback불가능)*


- Delete From [Tablename] 

#### 데이터만 삭제. Commit하기전에는 Rollback이 가능함

- Truncate table[Tablename] 

역시 데이터만 삭제하지만 delete와의 차이점은 테이블을 최초 생성한 초기상태로 만듬 
( Rollback 불가능 )