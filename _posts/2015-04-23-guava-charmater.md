---
layout: post
title: Guava CharMatcher
categories: [guava, java]
tags: [guava, java]
description: Guava CharMatcher
---

*본 포스트의 원문은 [http://www.baeldung.com](http://www.baeldung.com/guava-string-charmatcher)에서 확인할 수 있으며 원 저작자의 번역 포스팅 허락을 받았음을 알려드립니다.*

이번 예제에서는 Guava에서의 utility 클래스 *CharMatcher* 를 살펴보겠습니다. 
<br>

### 1. Remove Special Characters from a String
<br>
String에서 모든 특수문자를 제거 해 보겠습니다. 
<br>

아래 예제에서 retainFrom() 으로 문자나 숫자가 아닌 character들을 제거합니다: 
<br>
{% highlight java %}
@Test
public void whenRemoveSpecialCharacters_thenRemoved(){
    String input = "H*el.lo,}12";
    CharMatcher matcher = CharMatcher.JAVA_LETTER_OR_DIGIT;
    String result = matcher.retainFrom(input);
 
    assertEquals("Hello12", result);
}
{% endhighlight %}

### 2. Remove non ASCII characters from String
<br>
그리고 CharMatcher를 통해 String에서 ASCII가 아닌 문자를 제거할 수 있습니다. 
<br>

{% highlight java %}
@Test
public void whenRemoveNonASCIIChars_thenRemoved() {
    String input = "あhello₤";
 
    String result = CharMatcher.ASCII.retainFrom(input);
    assertEquals("hello", result);
 
    result = CharMatcher.inRange('0', 'z').retainFrom(input);
    assertEquals("hello", result);
}
{% endhighlight %}


### 3. Remove Characters not in the Charset

이제 특정 Charset에 속하지 않는 문자를 어떻게 제거하는지 살펴보겠습니다. 아래 예제는 "cp437" Charset에 속하지 않는 문자를 제거합니다. 

<br>
{% highlight java %}
@Test
public void whenRemoveCharsNotInCharset_thenRemoved() {
    Charset charset = Charset.forName("cp437");
    CharsetEncoder encoder = charset.newEncoder();
 
    Predicate<Character> inRange = new Predicate<Character>() {
        @Override
        public boolean apply(Character c) {
            return encoder.canEncode(c);
        }
    };
 
    String result = CharMatcher.forPredicate(inRange)
                               .retainFrom("helloは");
    assertEquals("hello", result);
}
{% endhighlight %}

<br>
주의: CharsetEncoder는 주어진 Character를 특정 Charset으로 인코딩하는 Predicate를 만듭니다. 
<br>
### 4. Validate String
<br>
다음은 CharMatcher로 String을 validate하는 방법입니다.
<br>
We can use matchesAllOf() to check if all characters match a condition. And we can make use of matchesNoneOf() to check if a condition doesn’t apply on any of the String characters.
matchesAllOf() 메서드로 모든 character들이 조건에 매칭되는지 검사하고 matchesNonOf() 메서드는 주어진 character들이 어떠한 조건에 매칭되지 않는 지 검사합니다. 
<br>
아래 예제는 String이 lowercase인지, , 'e' 문자를 포함하고 있는지, 숫자를 포함하지 않는지 검사합니다:

<br>
{% highlight java %}
@Test
public void whenValidateString_thenValid(){
    String input = "hello";
 
    boolean result = CharMatcher.JAVA_LOWER_CASE.matchesAllOf(input);
    assertTrue(result);
 
    result = CharMatcher.is('e').matchesAnyOf(input);
    assertTrue(result);
 
    result = CharMatcher.JAVA_DIGIT.matchesNoneOf(input);
    assertTrue(result);
}
{% endhighlight %}
<br>

### 5. Trim String
<br>
CharMatcher로 trim 하는 방법을 살펴보겠습니다.  
<br>

trimLeading(), trimTrailing(), trimFrom() 메서드로 String trim을 합니다.
<br>

{% highlight java %}
@Test
public void whenTrimString_thenTrimmed() {
    String input = "---hello,,,";
 
    String result = CharMatcher.is('-').trimLeadingFrom(input);
    assertEquals("hello,,,", result);
 
    result = CharMatcher.is(',').trimTrailingFrom(input);
    assertEquals("---hello", result);
 
    result = CharMatcher.anyOf("-,").trimFrom(input);
    assertEquals("hello", result);
}
{% endhighlight %}


### 6. Collapse a String

다음으로 CharMatcher로 String을 collapse 하는 방법을 살펴보겠습니다. 
  
collapseFrom() 메서드로 연속적인 빈 칸(space)을 '-' 문자로 대치하는 방법입니다:
  
{% highlight java %}
@Test
public void whenCollapseFromString_thenCollapsed() {
    String input = "       hel    lo      ";
 
    String result = CharMatcher.is(' ').collapseFrom(input, '-');
    assertEquals("-hel-lo-", result);
 
    result = CharMatcher.is(' ').trimAndCollapseFrom(input, '-');
    assertEquals("hel-lo", result);
}
{% endhighlight %}

### 7. Replace from String

CharMatcher를 통해 String에서 특정 문자를 대치하는 방법입니다:
<br>
{% highlight java %}
@Test
public void whenReplaceFromString_thenReplaced() {
    String input = "apple-banana.";
 
    String result = CharMatcher.anyOf("-.").replaceFrom(input, '!');
    assertEquals("apple!banana!", result);
 
    result = CharMatcher.is('-').replaceFrom(input, " and ");
    assertEquals("apple and banana.", result);
}
{% endhighlight %}

### 8. Count Character Occurrences

마지막으로 Charset으로 문자의 빈도(occurrences)를 카운팅 하는 방법을 살펴보겠습니다. 
  
<br>
콤마와 'a'에서 'h' 까지 문자가 몇 개인지 카운팅하는 예제 입니다:
  
<br>
{% highlight java %}
@Test
public void whenCountCharInString_thenCorrect() {
    String input = "a, c, z, 1, 2";
 
    int result = CharMatcher.is(',').countIn(input);
    assertEquals(4, result);
 
    result = CharMatcher.inRange('a', 'h').countIn(input);
    assertEquals(2, result);
}
{% endhighlight %}

### 9. Conclusion

본 포스트에서는 몇 몇 유용한 API를 통해 Guava를 통해 실용적으로 String을 조작하는 방법을 살펴봤습니다.