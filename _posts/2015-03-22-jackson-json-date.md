---
layout: post
title: Java Jackson Date
categories: [java]
tags: [java, jackson]
description: Java Jackson Date
---

*본 포스트의 원문은 [http://www.baeldung.com](http://www.baeldung.com/jackson-serialize-dates)에서 확인할 수 있으며 원 저작자의 번역 포스팅 허락을 받았음을 알려드립니다.*

# 1. Overview

이번 예제는 jackson(library) 를 통해 날짜를 직렬화(serialize) 하는 방법에 대해 살펴보겠습니다. java.util.Date 를 serializing 하고 Joda Time과 Java 8 의 DateTim도 같이 살펴보겠습니다. 

# 2. Serialize Date with Jackson

First – let’s see how to serialize a simple java.util.Date with Jackson.

아래 예제는 "eventDate"라는 Date 타입 필드를 가진 Event 인스턴스를 serialize 하는 예제입니다 :

{% highlight java %}
@Test
public void whenSerializingDateWithJackson_thenSerializedToTimestamp()
  throws JsonProcessingException, ParseException {
    SimpleDateFormat df = new SimpleDateFormat("dd-MM-yyyy hh:mm");
    df.setTimeZone(TimeZone.getTimeZone("UTC"));
 
    Date date = df.parse("01-01-1970 01:00");
    Event event = new Event("party", date);
 
    ObjectMapper mapper = new ObjectMapper();
    mapper.writeValueAsString(event);
}
{% endhighlight %}


여기서 중요한 점은 Jackson은 Date를 기본적으로 timestamp 형태(milliseconds 숫자는 1970년 1월 1일 UTC)으로 serialize 합니다.

"event"의 실제 serialization 결과는 :

```
{
   "name":"party",
   "eventDate":3600000
}
```


3. Serialize Date to ISO-8601

우리가 원한것은 timestamp 형태로의 serializing이 최선의 결과가 아닐 수 있습니다. 이제 Date를 ISO-8601 포멧으로 serialize하는 방법을 살펴 보겠습니다. 

timestamp를 사용하지 않도록 할 경우 아래와 같이 자동으로 ISO-8601 형태로 출력합니다.

{% highlight java%}
@Test
public void whenSerializingDateToISO8601_thenSerializedToText()
  throws JsonProcessingException, ParseException {
    SimpleDateFormat df = new SimpleDateFormat("dd-MM-yyyy hh:mm");
    df.setTimeZone(TimeZone.getTimeZone("UTC"));
 
    String toParse = "01-01-1970 02:30";
    Date date = df.parse(toParse);
    Event event = new Event("party", date);
 
    ObjectMapper mapper = new ObjectMapper();
    mapper.disable(SerializationFeature.WRITE_DATES_AS_TIMESTAMPS);
 
    String result = mapper.writeValueAsString(event);
    assertThat(result, containsString("1970-01-01T02:30:00.000+0000"));
}
{% endhighlight %}


이제 위와 같은 형태의 date 출력이 훨씬 더 가독성이 좋습니다. - 그럼에도 아직까진 깔끔하다고 생각되지 않을 수 있습니다.


# 4. Configure ObjectMapper DateFormat

이전 방법은 java.util.Date 인스턴스를 딱 맞는 형태로 표현하도록 선택할 수 있는 유연성이 부족한 것 같습니다.  


이제 date를 우리가 원하는 형태로 설정하고 표현하는 방법에 대해 살펴보겠습니다.  

{% highlight java%}
@Test
public void whenSettingObjectMapperDateFormat_thenCorrect()
  throws JsonProcessingException, ParseException {
    SimpleDateFormat df = new SimpleDateFormat("dd-MM-yyyy hh:mm");
 
    String toParse = "20-12-2014 02:30";
    Date date = df.parse(toParse);
    Event event = new Event("party", date);
 
    ObjectMapper mapper = new ObjectMapper();
    mapper.setDateFormat(df);
 
    String result = mapper.writeValueAsString(event);
    assertThat(result, containsString(toParse));
}
{% endhighlight %}

이제 date 포멧을 좀 더 유연한 방법으로 표현했지만  전체 ObjectMapper를 통해 전역 설정으로 사용했습니다.


# 5. Use @JsonFormat to format Date

이제 @JsonFormat 어노테이션으로 어플리케이션 전역적으로 설정하는것이 아닌 각 개별적으로 date 형태를 지정하는 방법에 대해 살펴보겠습니다. 


{% highlight java%}
public class Event {
    public String name;
 
    @JsonFormat
      (shape = JsonFormat.Shape.STRING, pattern = "dd-MM-yyyy hh:mm:ss")
    public Date eventDate;
}
{% endhighlight %}

이제 테스트 해 보겠습니다:


{% highlight java%}
@Test
public void whenUsingJsonFormatAnnotationToFormatDate_thenCorrect()
  throws JsonProcessingException, ParseException {
    SimpleDateFormat df = new SimpleDateFormat("dd-MM-yyyy hh:mm:ss");
    df.setTimeZone(TimeZone.getTimeZone("UTC"));
 
    String toParse = "20-12-2014 02:30:00";
    Date date = df.parse(toParse);
    Event event = new Event("party", date);
 
    ObjectMapper mapper = new ObjectMapper();
    String result = mapper.writeValueAsString(event);
    assertThat(result, containsString(toParse));
}
{% endhighlight %}


# 6. Custom Date Serializer

다음으로 Date를 위한 custom serializer 를 통해 모든 출력을 변경해 보겠습니다. 


{% highlight java%}
public class CustomDateSerializer extends JsonSerializer<Date> {
    private static SimpleDateFormat formatter = 
      new SimpleDateFormat("dd-MM-yyyy hh:mm:ss");
 
    @Override
    public void serialize (Date value, JsonGenerator gen, SerializerProvider arg2)
      throws IOException, JsonProcessingException {
        gen.writeString(formatter.format(value));
    }
}
{% endhighlight %}


다음은 "eventDate" 필드를 serializer 를 사용하도록 설정하였습니다 :


{% highlight java%}
public class Event {
    public String name;
 
    @JsonSerialize(using = CustomDateSerializer.class)
    public Date eventDate;
}
{% endhighlight %}



마지막으로 테스트 해보겠습니다:



{% highlight java%}
@Test
public void whenUsingCustomDateSerializer_thenCorrect()
  throws JsonProcessingException, ParseException {
    SimpleDateFormat df = new SimpleDateFormat("dd-MM-yyyy hh:mm:ss");
 
    String toParse = "20-12-2014 02:30:00";
    Date date = df.parse(toParse);
    Event event = new Event("party", date);
 
    ObjectMapper mapper = new ObjectMapper();
    String result = mapper.writeValueAsString(event);
    assertThat(result, containsString(toParse));
}
{% endhighlight %}



# 7. Serialize Joda-Time with Jackson


Date들이 항상 java.util.Date의 인스턴스일 필요는 없습니다. 이를 표현할 수 있는 다른 클래스들이 많이 있습니다. - 그리고 일반적인 방법 중 하나로 Joda-Time 라이브러리에서 구현한 DateTime 이 있습니다.

이제 Jackson으로 DateTime을 serialize하는 방법에 대해 살펴보겠습니다. 

Joda-Time 출력을 지원하기 위해 jackson-datatype-joda 모듈을 사용하도록 하겠습니다. 


```
<dependency>
  <groupId>com.fasterxml.jackson.datatype</groupId>
  <artifactId>jackson-datatype-joda</artifactId>
  <version>2.4.0</version>
</dependency>
```

이제 간단하게 JodaModule을 등록하면 됩니다.



{% highlight java%}
@Test
public void whenSerializingJodaTime_thenCorrect() 
  throws JsonProcessingException {
    DateTime date = new DateTime(2014, 12, 20, 2, 30, 
      DateTimeZone.forID("Europe/London"));
 
    ObjectMapper mapper = new ObjectMapper();
    mapper.registerModule(new JodaModule());
    mapper.disable(SerializationFeature.WRITE_DATES_AS_TIMESTAMPS);
 
    String result = mapper.writeValueAsString(date);
    assertThat(result, containsString("2014-12-20T02:30:00.000Z"));
}
{% endhighlight %}


# 8. Serialize Joda DateTime with Custom Serializer

만약 추가된 Joda-Time과 Jackson 의존성을 원하지 않을 수 있습니다. - 그러면 custom serializer(이전 예제에서 살펴본 것과 비슷한) 를 통해 DateTime 인스턴스를 깔끔하게 serialize할 수 있습니다:


{% highlight java%}
public class CustomDateTimeSerializer extends JsonSerializer<DateTime> {
 
    private static DateTimeFormatter formatter = 
      DateTimeFormat.forPattern("yyyy-MM-dd HH:mm");
 
    @Override
    public void serialize
      (DateTime value, JsonGenerator gen, SerializerProvider arg2)
      throws IOException, JsonProcessingException {
        gen.writeString(formatter.print(value));
    }
}
{% endhighlight %}


다음은 "eventDate" 속성을 serializer를 사용하겠습니다. 


{% highlight java%}
public class Event {
    public String name;
 
    @JsonSerialize(using = CustomDateTimeSerializer.class)
    public DateTime eventDate;
}
{% endhighlight %}


마지막으로 다함께 테스트 해보겠습니다. 


{% highlight java%}
@Test
public void whenSerializingJodaTimeWithJackson_thenCorrect() 
  throws JsonProcessingException {
    DateTime date = new DateTime(2014, 12, 20, 2, 30);
    Event event = new Event("party", date);
 
    ObjectMapper mapper = new ObjectMapper();
    String result = mapper.writeValueAsString(event);
    assertThat(result, containsString("2014-12-20 02:30"));
}
{% endhighlight %}



# 9. Serialize Java 8 Date with Jackson

다음으로  Java 8 DateTime을 어떻게 serialize 하는지 살펴보겠습니다. 이번 예제에서는 jackson-datatype-jsr310 module을 통해 Jackson과 LocalDateTime 을 사용하겠습니다. 

```
<dependency>
    <groupId>com.fasterxml.jackson.datatype</groupId>
    <artifactId>jackson-datatype-jsr310</artifactId>
    <version>2.4.0</version>
</dependency>
```

이제 JSR310Module 을 등록하면 jackson이 나머지를 처리하게 됩니다. 


{% highlight java%}
@Test
public void whenSerializingJava8Date_thenCorrect()
  throws JsonProcessingException {
    LocalDateTime date = LocalDateTime.of(2014, 12, 20, 2, 30);
 
    ObjectMapper mapper = new ObjectMapper();
    mapper.registerModule(new JSR310Module());
    mapper.disable(SerializationFeature.WRITE_DATES_AS_TIMESTAMPS);
 
    String result = mapper.writeValueAsString(date);
    assertThat(result, containsString("2014-12-20T02:30"));
}
{% endhighlight %}


# 10. Serialize Java 8 Date with Jackson

만약 의존성이 추가되는 것을 원치 않는다면 항상 custom serializer를 통해 Java 8 DateTime을 JSON으로 출력할 수 있습니다. 


{% highlight java%}
public class CustomLocalDateTimeSerializer 
  extends JsonSerializer<LocalDateTime> {
 
    private static DateTimeFormatter formatter = 
      DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm");
 
    @Override
    public void serialize(LocalDateTime value, JsonGenerator gen, SerializerProvider arg2)
      throws IOException, JsonProcessingException {
        gen.writeString(formatter.format(value));
    }
}
{% endhighlight %}

이제 "eventDate" 필드에 serializer를 사용하겠습니다. :


{% highlight java%}
public class Event {
    public String name;
 
    @JsonSerialize(using = CustomLocalDateTimeSerializer.class)
    public LocalDateTime eventDate;
}
{% endhighlight %}

{% highlight java%}
@Test
public void whenSerializingJava8DateWithCustomSerializer_thenCorrect()
  throws JsonProcessingException {
    LocalDateTime date = LocalDateTime.of(2014, 12, 20, 2, 30);
    Event event = new Event("party", date);
 
    ObjectMapper mapper = new ObjectMapper();
    String result = mapper.writeValueAsString(event);
    assertThat(result, containsString("2014-12-20 02:30"));
}
{% endhighlight %}

# 11. Deserialize Date

다음으로는 Jackson을 통해 어떻게 Date를 deserialize 하는지 살펴보겠습니다. 다음 예제는 "Event" 인스턴스에 포함된 date를 deserialize합니다. 


{% highlight java%}
@Test
public void whenDeserializingDateWithJackson_thenCorrect()
  throws JsonProcessingException, IOException {
    String json = "{"name":"party","eventDate":"20-12-2014 02:30:00"}";
 
    SimpleDateFormat df = new SimpleDateFormat("dd-MM-yyyy hh:mm:ss");
    ObjectMapper mapper = new ObjectMapper();
    mapper.setDateFormat(df);
 
    Event event = mapper.reader(Event.class).readValue(json);
    assertEquals("20-12-2014 02:30:00", df.format(event.eventDate));
}
{% endhighlight %}

# 12. Custom Date deserializer

이제 어떻게 custom Date deserializer를 사용하는지 살펴보겠습니다. "eventDate" 속성을 위해 custom deserializer를 만들겠습니다. :


{% highlight java%}
public class CustomDateDeserializer extends JsonDeserializer<Date> {
 
    private static SimpleDateFormat formatter = 
      new SimpleDateFormat("dd-MM-yyyy hh:mm:ss");
 
    @Override
    public Date deserialize(JsonParser jsonparser, DeserializationContext context)
      throws IOException, JsonProcessingException {
        String date = jsonparser.getText();
        try {
            return formatter.parse(date);
        } catch (ParseException e) {
            throw new RuntimeException(e);
        }
    }
}
{% endhighlight %}


다음으로 "eventDate"에  deserializer를 사용합니다. 


{% highlight java%}
public class Event {
    public String name;
 
    @JsonDeserialize(using = CustomDateDeserializer.class)
    public Date eventDate;
}
{% endhighlight %}

마지막으로 테스트 해 보겠습니다 :


{% highlight java%}
@Test
public void whenDeserializingDateUsingCustomDeserializer_thenCorrect()
  throws JsonProcessingException, IOException {
    String json = "{"name":"party","eventDate":"20-12-2014 02:30:00"}";
 
    SimpleDateFormat df = new SimpleDateFormat("dd-MM-yyyy hh:mm:ss");
    ObjectMapper mapper = new ObjectMapper();
 
    Event event = mapper.reader(Event.class).readValue(json);
    assertEquals("20-12-2014 02:30:00", df.format(event.eventDate));
}
{% endhighlight %}

# 13. Conclusion

이번 Date 블로그에서 date를 JSON 원하는 형태로 marshalling과 unmarshall을 도와주는 jackson 을 사용하는 방법에 대해 살펴보았습니다. 



