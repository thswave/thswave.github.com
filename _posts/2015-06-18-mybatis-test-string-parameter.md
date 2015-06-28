---
layout: post
title: mybatis string parameter를 if문(test)에서 사용하는 방법
categories: [spring]
tags: [spring, json]
description: mybatis string parameter를 if문(test)문에서 사용하는 방법
---


* mybatis로 개발하면서 알게 된 팁을 공유하고자 합니다. *주의: ibatis와 동작이 다를 수 있습니다.*

* select 문 파라미터로 String을 매개변수로 넘기는 경우가 있습니다. 


```
return selectList("A.selectXXX", "StringParame");
```

* 이 때 쿼리에서 이 String 파라미터를 지칭/레펀런스 할 경우 임의의 이름으로 붙여도 정상동작 합니다.

```
<select id="selectXXX" parameterType="String" resultType="resultType">
	SELECT a, b, c
	FROM table
	WHERE condition = #{anyName}
</select>
```

<br>

* 이 외에 추가적인 if 문(test)이 필요할 경우가 있는데 이 때는 넘어온 파라미터를 임의의 이름으로 지칭할 경우 예외가 발생합니다. 

```
There is no getter for property named 'abc' in 'class java.lang.String'
```

* 해결책: `_parameter` 로 파라미터를 지칭할 수 있습니다.

```
<select id="selectXXX" parameterType="String" resultType="resultType">
	SELECT a, b, c 
	FROM table
	<where>
		<if test = “ _parameter != null and _parameter.equals('aaa') ">
		AND d = #{_parameter}
		</if>
	</where>
</select>
```

* 여기저기 찾아보니 `_parameter` is undocumented. MyBatis is a nice framework but, sadly, lacks on the documentation part 설명처럼 문서화가 부족한 부분이 있어 쉽게 해법을 찾지 못하고 헤멧던 것 같네요.
