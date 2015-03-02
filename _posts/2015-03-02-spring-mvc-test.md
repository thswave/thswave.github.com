---
layout: post
title: How to Spring MVC Unit Test 스프링 MVC 단위 테스트
categories: [java]
tags: [java, unit test]
description: Spring MVC Unit Test! 스프링 MVC 단위 테스트
---

# 스프링 MVC 단위 테스트!

계속해서 들려오는 TDD를 해보고 싶었던차에 강제적으로(?) 단위테스트를 만들면서 개발해야되는 상황이 되었습니다.

지금까지 TDD 예제나 소개에서는 간단한 알고리즘 문제 풀이를 다루고 있었고 단순한 JUnit Test 케이스를 만들순 있었습니다.

하지만 Spring MVC의 Controller, Service, Model  성격이 다른 각 계층에 대한 테스트 케이스를 만드려고 하니 막막해졌습니다.
MVC를 구성하는 각 레이어 별 어떻게 테스트를 해야 할지 크게 세가지 정도가 감이 오지 않았습니다. 

- *1. Controller Method*

Controller에 request에 매핑되는 메서드는 일반 객체의 메서드를 호출하듯 마음대로 호출할 수 있는게 아니었기 때문 이를 어떻게 단위테스트에서 호출할 수 있는지 의문이었습니다. 

- *2. Controller의 테스트 방식*

Controller에서는 Service의 메서드를 호출하기만 하는데 이를 어떻게 단위 테스트 할 수 있는지 의문이었습니다. 

- *3. DAO의 테스트 방식*

단위테스트는 어느환경에서 실행하여도 성공을 보장해주어야 하는데 데이터베이스를 접근하는 DAO의 경우 데이터베이스 접근이 실패하거나 테스트 할때의 데이터베이스에 있는 테이블 데이터에 따라 성공 혹은 실패하기 때문에 이를 어떻게 처리해야 하는지 의문이었습니다.

이제부터 Spring MVC의 단위테스트를 위한 과정을 하나씩 살펴보겠습니다. 
`Controller`, `Service`, `DAO` 별로 테스트 하는 방식이 다르기 때문에 하나씩 살펴보겠습니다.

## Controller 단위 테스트

Spring MVC, JUnit, Maven을 사용중이라는 가정하에 maven의 pom.xml에서 스프링 테스트, Mockito, JUnit을 추가해 줍니다. 

```xml
<dependency>
    <groupId>junit</groupId>
    <artifactId>junit</artifactId>
    <version>4.11</version>
    <scope>test</scope>
</dependency>
<dependency>
    <groupId>org.mockito</groupId>
    <artifactId>mockito-all</artifactId>
    <version>1.9.5</version>
    <scope>test</scope>
</dependency>
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-test</artifactId>
    <version>3.2.3.RELEASE</version>
    <scope>test</scope>
</dependency>
```

그리고 특정 카테고리에 대한 Controller

{% highlight java %}
@RequestMapping(value = "/category")
public class CategoryController {
     @Autowired
     private CategoryService categoryService;

     @RequiredLogin
     @RequestMapping(value = "/list", method = RequestMethod.GET)
     public ModelAndView allCategoryList() {
          return categoryService.method1();
     }
}
{% endhighlight %}

CategoryController에서는 @Autowired로 CategoryService를 사용합니다.

본격적으로 Test 케이스를 작성해 봅시다. 

`CategoryControllerTest.java`

{% highlight java %}
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(locations = {"file:src/main/webapp/WEB-INF/spring/appServlet/test-servlet-context.xml",
     "file:src/main/webapp/WEB-INF/mybatis/test-mybatis-context.xml"})
@WebAppConfiguration
public class CategoryControllerTest {
     @Mock
     CategoryService categoryService;
     @InjectMocks
     private CategoryController categoryController;

     @Autowired
     private WebApplicationContext wac;

     private MockMvc mockMvc;

     @Before
     public void setUp() throws ExceptiMon {
          MockitoAnnotations.initMocks(this);
          mockMvc = MockMvcBuilders.standaloneSetup(categoryController).build();
     }

     @Test
     public void testCategoryController() throws Exception {
          when(categoryService.method1()).thenReturn(10);
          mockMvc.perform(get("/category/list")).andExpect(status().isOk());

          verify(categoryService).method1();
          verifyNoMoreInteractions(categoryService);
     }
}
{% endhighlight %}

테스트 클래스 상단에 아래와 같이 작성해 줍니다. 
{% highlight java %}
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(locations = {"file:src/main/webapp/WEB-INF/spring/appServlet/test-servlet-context.xml",
     "file:src/main/webapp/WEB-INF/mybatis/test-mybatis-context.xml"})
@WebAppConfiguration
public class ClassName {
	// 생략
}
{% endhighlight %}

테스트를 위한 유일한 방법이 아니고 여러가지 다른 방법이 있지만 저는 xml설정 파일을 읽어 테스트하도록 하였습니다.

`SpringJUnit4ClassRunner.class` 클래스는 spring-test 에서 제공하는 단위테스트를 위한 클래스 러너입니다. 

아래 `@ContextConfiguration` 어노테이션은 스프링 빈 설정, 데이터베이스 설정 등이 있는 `servlet-context.xml`과 같은 xml 설정을 읽어 스프링 테스트 시작 전에 빈을 초기화 하여 `@Autowired` 를 통해 빈 팩토리에서 미리 생성한 빈들을 읽어 들일 수 있고 그 밖의 spring 프레임워크 설정들을 읽어 들일 수 있습니다.

위와 같이 설정하고 실행할 경우 스프링을 실행할 때와 마찬가지로 설정 파일을 읽어들이여 Request Mapping 등의 작업을 하는걸 확인할 수 있습니다.

Controller에서 @Autowired로 된걸 단위테스트에서 동일하게 주입시키기 위해 Mockito의 @InjectMocks, @Mock을 설정합니다. 

{% highlight java %}
     @Mock
     CategoryService categoryService;
     @InjectMocks
     private CategoryController categoryController;
{% endhighlight %}

*여기서 헷갈리는 부분이 있다면 `@Mock`은 Mockito 단위 테스트 프레임워크라는 것이고 이것은 spring-test와는 무관하다 볼 수 있습니다.*

Controller를 테스트 할 것이므로 해당 컨트롤러에 @Autowired 한 CategoryService 빈을 주입해 줍니다.

그리고 반드시 `@Before`에서 아래와 같이 호출해줍니다.

{% highlight java %}
    @Before
     public void setUp() throws Exception {
          MockitoAnnotations.initMocks(this);
          mockMvc = MockMvcBuilders.standaloneSetup(categoryController).build();
     }
{% endhighlight %}

여기서 MockitoAnnotations.initMocks(this); 이부분이 @Autowired를 해주는 것이라 생각하시면 됩니다. 그리고 Controller 단위테스트의 핵심이라 볼수있는 MockMVC를 설정하게 됩니다.

{% highlight java %}
  mockMvc = MockMvcBuilders.standaloneSetup(categoryController).build();
{% endhighlight %}

standaloneSetup()말고 webAppContextSetUp 방식도 있으니 참고바랍니다.

* MockMvc : Controller에 request를 수행해줍니다.
* Mockito: 동작이나 값을 대신해주는 목 객체 생성하고 verify를 통해 정상적으로 호출되었는지 검증.

이제 *Mockito의 when().thenReturn()* 등을 활용해 컨트롤러에 로직을 테스트 해주면 됩니다. 

여기서 중요한건 `Controller의 수행 흐름`을 테스트 한다는 것이지 `CategoryService의 동작`을 테스트 하는게 아닙니다.
(지금은 매우 간단한 로직이지만 controller의 Controlling 해주는 부분이 늘어나면 이를 더 테스트 해야 합니다.)

`mockMvc.perform()`으로 실제 request를 테스트 해볼 수 있습니다.

{% highlight java %}
          mockMvc.perform(get("/category/list")).andExpect(status().isOk());
{% endhighlight %}

정상적으로 200 status코드를 받았는지 단순히 테스트하는 정도지만 여기서 다양한 테스트를 해볼 수 있습니다.(예를 들어, json 을 받았다면 해당 데이터가 몇 개의 요소들로 있는지 어떤 데이터로 구성되어 있는지, 또는 예외가 발생한 다면 어떤 예외가 발생할 수 있는지 등 아주 세세하게 테스트 해볼 수 있습니다.)

Controller에서 호출한 메서드의 호출이 되었는지 확인하기 위해서는 mockito의 verify를 사용해야 합니다.


* mockito의 기본적인 사용방법 참조 사이트 

[http://www.baeldung.com/mockito-behavior](http://www.baeldung.com/mockito-behavior)
[http://www.baeldung.com/mockito-verify](http://www.baeldung.com/mockito-verify)

## 결론

컨트롤러를 테스트 할 수 있는 방법은 여러가지 일 수 있습니다. MockHttpServlet 을 활용하여 테스트 한다거나 혹은, controller를 일반 함수라고 가정하고 각 파라미터들을 주입하고 이를 테스트 해볼 수 있습니다. 단 제 생각은 Controller에는 말 그대로 Request에 대한 Controlling을 하는 부분이 들어가 있어야 하고 Service 레이어에 로직이 들어가 있어야 할것입니다. 그렇다면 로직을 테스트하는 것인지 혹은 어디로 요청을 보내는지 를 테스트 하는지 구분 지어 테스트해야 할 것입니다. spring-test 를 활용하여 컨트롤러를 테스트한 이유는 spring의 AOP라던가 Interceptor 등 스프링이 제공해주는 기능을 포함하여 테트스 할 때 필요할 것입니다. 위 간단한 예제에서는 비록 복잡한 설정이 없었기 때문에 다른 방식으로도 충분히 테스트 할 수 있었지만 스프링 설정을 읽어들여 좀 더 정확한 테스트를 하고자 한다면 spring-test에서 제공하는 MockMvc를 활용하면 더욱 효율적일 것 같습니다.

Service layer 테스트는 다음 포스팅에 계속 하겠습니다. 

*포스트를 정리 한 후 예제 full source code를 다시 올리겠습니다.*

#### ps. request에 attribute를 넣어주는 방법.

{% highlight java %}
   MockHttpServletRequestBuilder request = get("category/list");
    request.with(new RequestPostProcessor() {

         @Override
         public MockHttpServletRequest postProcessRequest(MockHttpServletRequest request) {
              request.setAttribute("a", "a");
              return request;
         }
    });
    return request;
{% endhighlight %}

우선 구글링을 통해 단위테스트 작성에 도움되는 사이트들을 소개해 드립니다.

* 참조 사이트
[http://www.petrikainulainen.net/spring-mvc-test-tutorial/](http://www.petrikainulainen.net/spring-mvc-test-tutorial/)
[http://www.petrikainulainen.net/programming/spring-framework/integration-testing-of-spring-mvc-applications-configuration/](http://www.petrikainulainen.net/programming/spring-framework/integration-testing-of-spring-mvc-applications-configuration/)
[http://www.petrikainulainen.net/programming/spring-framework/unit-testing-of-spring-mvc-controllers-configuration/](http://www.petrikainulainen.net/programming/spring-framework/unit-testing-of-spring-mvc-controllers-configuration/)
[http://thysmichels.com/2014/01/07/junit-test-spring-mvc-web-services/](http://thysmichels.com/2014/01/07/junit-test-spring-mvc-web-services/)
