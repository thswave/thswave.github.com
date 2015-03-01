---
layout: post
title: 스프링 &lt;context:component-scan&gt; 분석
categories: [spring]
tags: [java, spring]
description: 스프링 &lt;context:component-scan&gt; 분석
---

### 스프링 `<context:component-scan>` 분석
asd

처음으로 스프링 MVC로 프로젝트를 개발하다보니 모르는것도 많고 어떻게 해야하는지 정확히 잘 몰라 구글링과 스프링 설정에 대한 블로그들을 보면서 이것저것 복사 붙여넣기 식으로 수행했습니다.
그 중에서 스프링 XML 설정 지옥에서 벗어나게 해준 어노테이션. 스프링이 제공하는 어노테이션 기반 설정은 많지만 가장 유용했던 단 한 줄로 설정할 수 있었습니다.

{% highlight xml%}
<context:component-scan base-package="com.nhnent.spring" />
{% endhighlight %}

* Component Scan이 어떻게 내부적으로 수행하는지 살펴보겠습니다.

### Component Scan

Component Scan은 XML에 일일이 빈등록을 하지않고 각 빈 클래스에 @Component를 통해 자동 빈 등록
@Component는 스프링이 어노테이션에 담긴 메타정보를 이용하기 시작했을 때 @Autowired와 함께 소개된 대표적인 어노테이션. @Component 어노테이션을 클래스에 작성하면 빈 스캐너를 통해 자동 빈 등록됩니다.
특정 패키지 아래에 위치한 빈들을 Component Scan하기 위한 방법은 다음과 같습니다. 

1. XML에서 `<context:component-scan base-package="com.nhnent.spring" />`
2. 자바 파일의 설정 클래스(@Configuration)에서 @ConponentScan 어노테이션 설정 

Context Scan을 처리하는 부분은 spring-context-버전.jar에 위치하고 있으며 본문에 나오는 테스트코드와 소스는 스프링 프로젝트에서 확인하실 수 있습니다.
[스프링 프로젝트 소스](https://github.com/spring-projects/spring-framework.git) 에서 구할 수 있습니다.

분석은 크게 3가지 방향으로 진행하였습니다.

1. Component scan를 수행하는 과정.
2. 컨테이너에서 getBean(beanName.class)을 호출하여 빈이 호출되는 과정. 
3. Spring MVC에서 Component Scan 

스프링 컨테이너가 실행되면서 수행하는 여러가지 초기화 과정 중에서 Component Scan부분만 딱 잘라서 설명할 경우 앞뒤 과정이 없어 생소할 수 있기 때문에 기본적인 빈 등록, 빈 검색을 같이 연관성있는 부분들도 같이 설명하도록 하겠습니다. 
실질적인 Component scan이 진행되기까지 절차가 길기 때문에 요약을 하자면 아래와 같은 순서로 진행됩니다.

1. @Configuration 어노테이션 클래스 파싱 (보통 @Configuration 설정 클래스에 @ComponentScan이 설정)
2. @ComponentScan 어노테이션 클래스 파싱
3. ComponentScanAnnotationParser 클래스
4. 실질적으로 Component scan 패키지 디렉토리에서 리소스를 찾는 ClassPathBeanDefinitionScanner 

#### ComponentScan Test Code

ComponentScan에 대한 테스트 케이스는 아래와 같습니다. ComponentScan수행시 기본적으로는 base-package를 지정하지만 이 외에도 IncludeFilter, ExcludeFilter, ScopeProxy 등 다양항 설정 가능합니다.
여기서는 가장 기본적인 base-package를 기반으로 빈 등록하는 과정만 집중적으로 살펴보겠습니다.

* org.springframework.context.annotation.ComponentScanAnnotationIntegrationTests.java
{% highlight java %}

public class ComponentScanAnnotationIntegrationTests {

	@Test
	public void viaContextRegistration_WithValueAttribute() {
		AnnotationConfigApplicationContext ctx = new AnnotationConfigApplicationContext();
		ctx.register(ComponentScanAnnotatedConfig_WithValueAttribute.class); // 1. 빈 등록
		ctx.refresh();														 // 2. Component scan 수행
		ctx.getBean(ComponentScanAnnotatedConfig_WithValueAttribute.class);
		ctx.getBean(TestBean.class);										 // 3. 빈 호출
		assertThat("config class bean not found", ctx.containsBeanDefinition("componentScanAnnotatedConfig_WithValueAttribute"), is(true));
		assertThat("@ComponentScan annotated @Configuration class registered directly against " +
				"AnnotationConfigApplicationContext did not trigger component scanning as expected",
				ctx.containsBean("fooServiceImpl"), is(true));
	}

	// 다른 테스트 케이스 생략 ...
}


@Configuration
@ComponentScan("example.scannable")
class ComponentScanAnnotatedConfig_WithValueAttribute {
	@Bean
	public TestBean testBean() {
		return new TestBean();
	}
}
{% endhighlight %}

5번째 줄의 AnnotationConfigApplicationContext 클래스는 스크링 컨테이너로 이름에서 유추할 수 있듯 어노테이션 설정을 처리할 수 있는 Context.

19~21번째 줄의 @Configuration(설정 어노테이션)과 @ComponentScan 어노테이션이 붙은 클래스(ComponentScanAnnotatedConfig_WithValueAttribute)를 등록(register)하고 이를 호출하는 테스트 코드입니다. "example.scannable" 패키지 위치에는 @Component 어노테이션을 가진 클래스 파일들이 위치해 있습니다.

Component scan을 수행하기 전에 register() 메서드로 검색할 패키지에 대한 정보를 가지는 @ComponentScan 어노테이션과 설정에 관련된 클래스임을 알려주는 @Configuration을 가진 ComponentScanAnnotatedConfig_WithValueAttribute.class를 빈 등록. (빈을 등록하는 절차 조금 복잡한 절차이지만 생략해도 Componentscan을 이해하는데 지장이 없으므로 분석을 건너뛰도록 하겠습니다.)

7번째 줄의 ctx.refresh() 메서드에 Component Scan 처리 부분이 담겨있습니다.

#### component scan 분석

* AnnotationConfigApplicationContext.refresh()
{% highlight java %}

@Override
public void refresh() throws BeansException, IllegalStateException {
	// 동기화 코드 생략..

	// Prepare this context for refreshing.
	prepareRefresh();

	// Tell the subclass to refresh the internal bean factory.
	ConfigurableListableBeanFactory beanFactory = obtainFreshBeanFactory();

	// Prepare the bean factory for use in this context.
	prepareBeanFactory(beanFactory);

	// Allows post-processing of the bean factory in context subclasses.
	postProcessBeanFactory(beanFactory);

	// Invoke factory processors registered as beans in the context.
	invokeBeanFactoryPostProcessors(beanFactory); // 여기서 Component Scan 작업 수행

	// Register bean processors that intercept bean creation.
	registerBeanPostProcessors(beanFactory);

	// Initialize message source for this context.
	initMessageSource();

	// Initialize event multicaster for this context.
	initApplicationEventMulticaster();

	// Initialize other special beans in specific context subclasses.
	onRefresh();

	// Check for listener beans and register them.
	registerListeners();

	// Instantiate all remaining (non-lazy-init) singletons.
	finishBeanFactoryInitialization(beanFactory);

	// Last step: publish corresponding event.
	finishRefresh();

	// 예외처리 생략.
}
{% endhighlight %}


AnnotationConfigApplicationContext의 빈 등록(register)을 마친 후 refresh() 메서드를 호출하는데 여기서 9번째 줄에서 빈 팩토리를 준비하고(prepareBeanFactory(beanFactory)) 이와 관련된 후처리(PostProcessor) 작업과 같은 몇몇 절차가 있으나 그 중 Component Scan은 19번째 줄의 `invokeBeanFactoryPostProcessors(beanFactory)` 메서드에서 이뤄집니다.
Component Scan 부분만 살펴보기 위해 나머지 메서드들에 대한 내용은 생략하겠습니다.
여기서 매개변수로 넘겨준 beanFactory는 DefaultListableBeanFactory 구체 클래스로 빈들을 등록/검색할 수 있는 BeanDefinitionRegistry 인터페이스를 상속하고 있습니다. 등록된 빈들은 DefaultListableBeanFactory 클래스의 ConcurrentHashMap타입의 beanDefinitionMap에 저장됩니다. 

{% highlight java %}
private final Map<String, BeanDefinition> beanDefinitionMap = new ConcurrentHashMap<String, BeanDefinition>(64);
{% endhighlight %}


DefaultListableBeanFactory 팩토리에서 상속하는 인터페이스의 타입 계층 구조를 살펴보면 이름을 통해 어떤 역활을 해주는지 유추해 볼 수 있습니다. 


DefaultListableBeanFactory 클래스는 빈을 싱글톤으로 관리해주는 SingletonRegistry 인터페이스 역시 상속하고 있습니다. 
타입 계층 구조의 각 클래스에는 의존성이나 빈의 등록과 세세한 설정에 대한 사항을 다루는 부분이 담겨있습니다.

invokeBeanFactoryPostProcessors(beanFactory)에서 빈 팩토리를 BeanDefinitionRegistry로 형 변환 하여 넘기게 됩니다.(단순 형변환하여 넘기는게 아니라 부가 작업들이 있지만 중요한 내용이 아니므로 생략하였습니다.)

#### @Configuration 어노테이션 클래스 파싱

* ConfigurationClassPostProcessor.processConfigBeanDefinitions(BeanDefinitionRegistry registry)

{% highlight java %}
public void processConfigBeanDefinitions(BeanDefinitionRegistry registry) {
	Set<BeanDefinitionHolder> configCandidates = new LinkedHashSet<BeanDefinitionHolder>();
	// 매개변수 registry로 부터 Configuration 정보를 담고있는 BeanDefinitionHolder를 찾아 configCandidates에 넣어주는 작업

	// 생략..

	// Parse each @Configuration class
	ConfigurationClassParser parser = new ConfigurationClassParser(
			this.metadataReaderFactory, this.problemReporter, this.environment,
			this.resourceLoader, this.componentScanBeanNameGenerator, registry);

	Set<ConfigurationClass> alreadyParsed = new HashSet<ConfigurationClass>(configCandidates.size());
	
	parser.parse(configCandidates);
	parser.validate();

	// 생략...
	// Component Scan 과정 후 빈을 찾을때 활용되는 코드 생략. 나중에 getBean()으로 빈을 찾을때 다시 언급
	//
{% endhighlight %}				

invokeBeanFactoryPostProcessors() 메서드 빈 등록을 처리하기 위해 여러 작업을 수행하지만 실질적으로 @Configuration 어노테이션 클래스를 파싱하는 것은  ConfigurationClassParser 클래스의 parse() 메서드입니다.

이 때 매개변수로 BeanDefinitionRegistry(beanFactory에 포함된 빈 registry)를 넘겨줍니다. 그리고 이 beanFactory에는 테스트 코드에서 컨텍스트에 등록해주었던 ComponentScanAnnotatedConfig_WithValueAttribute 클래스의 정보도 같이 포함되어 있습니다.

parse 메서드 안에서도 다시 몇 번의 메서드 호출 뒤 doProcessConfigurationClass 메서드로 넘어오며 여기서 실질적으로 Component Scan작업이 수행됩니다.

#### @ComponentScan 어노테이션 클래스 파싱

* ConfigurationClassParser.doProcessConfigurationClass()

{% highlight java %}

protected final SourceClass doProcessConfigurationClass(ConfigurationClass configClass, SourceClass sourceClass) throws IOException {
		
		// Process any @PropertySource annotations
		// 생략...

		// Process any @ComponentScan annotations
		AnnotationAttributes componentScan = AnnotationConfigUtils.attributesFor(sourceClass.getMetadata(), ComponentScan.class);
		if (componentScan != null && !this.conditionEvaluator.shouldSkip(sourceClass.getMetadata(), ConfigurationPhase.REGISTER_BEAN)) {
			// The config class is annotated with @ComponentScan -> perform the scan immediately
			Set<BeanDefinitionHolder> scannedBeanDefinitions =
					this.componentScanParser.parse(componentScan, sourceClass.getMetadata().getClassName());  // <- 여기서 Component scan 수행!
			// Check the set of scanned definitions for any further config classes and parse recursively if necessary
			for (BeanDefinitionHolder holder : scannedBeanDefinitions) {
				if (ConfigurationClassUtils.checkConfigurationClassCandidate(holder.getBeanDefinition(), this.metadataReaderFactory)) {
					parse(holder.getBeanDefinition().getBeanClassName(), holder.getBeanName());
				}
			}
		}

		// Process any @Import annotations
		// 생략...

		// Process any @ImportResource annotations
		// 생략...

		// Process individual @Bean methods
		// 생략...

		return null;
	}
{% endhighlight %}

생략된 코드의 주석을 보시면 @ComponentScan 어노테이션을 처리하는 부분 외에도 @PropertySource, @Import, @ImportResource, @Bean을 처리하는 부분도 같이 있습니다. 

여기서 나머지 가장 핵심은 11번째 줄인데 ConfigurationClassParser 클래스에는 내부적으로 ComponentScanAnnotationParser 클래스의 인스턴스를 componentScanParser 이름의 필드로 가지고 있습니다. 

#### ComponentScanAnnotationParser 클래스

* ComponentScanAnnotationParser.java

{% highlight java %}
public ComponentScanAnnotationParser(ResourceLoader resourceLoader, Environment environment,
			BeanNameGenerator beanNameGenerator, BeanDefinitionRegistry registry)
{% endhighlight %}

ComponentScanAnnotationParser 생성자로 넘어가는 매개변수의 이름을 살펴보면 
* ResourceLoader : 스프링은 file이나 어떠한 형태의 자원을 Resource로 관리
* Environment : @PropertySource나 XML에 명시된 properties관련 파일에 있는 property 정보가 들어있는 Environment
* BeanNameGenerator : 빈의 이름을 생성해주느는 BeanNameGenerator
* BeanDefinitionRegistry : BeanDefinition을 등록/관리하는 BeanDefinitionRegistry

componentScanParser.parse() 메서드로 넘어가는 매개변수 componentScan와 sourceClass.getMetadata().getClassName()를 하나씩 살펴보겠습니다. 
*componentScan* : AnnotationAttributes 클래스로 Component Scan시 옵션으로 설정할수 있는 필드들을 가지는 LinkedHashMap 상속 클래스. 
* value, basePackageClasses, nameGenerator, scopeResolver, scopedProxy, lazyInit, includeFilters, excludeFilters, resourcePattern, useDefaultFilters, basePackages
* 테스트 케이스에서 componentScan에 설정된 값은 아래와 같습니다. 

{% highlight java %}
	 value=[example.scannable],
	 basePackageClasses=[],
	 nameGenerator=interface org.springframework.beans.factory.support.BeanNameGenerator,
	 scopeResolver=class org.springframework.context.annotation.AnnotationScopeMetadataResolver,
	 scopedProxy=DEFAULT,
	 lazyInit=false,
	 includeFilters=[],
	 excludeFilters=[],
	 resourcePattern=**/*.class,
	 useDefaultFilters=true,
	 basePackages=[]
{% endhighlight %}

테스트 코드의 설정 클래스에서 @ComponentScan("example.scannable")로 설정했기 때문에 basePackages 값이 비어있고 기본값을 넣어주는 value에 "example.scannable" 값이 들어가 있는것을 확인할 수 있습니다. 하지만 내부에서 처리할 때에는 basePackages나 value에 넣어두나 동일한 과정을 거치게 됩니다. 

*sourceClass.getMetadata().getClassName()*

SourceClass 클래스는 간단히 설명하자면 Configuration 정보가 있는 클래스의 인스턴스와 이 클래스의 어노테이션과 같은 메타정보를 분석해주는 AnnotationMetadata을 가지도록 Wrapping한 것이라 보시면 됩니다. 그렇기 때문에 sourceClass.getMetadata().getClassName()는 @Configuration 어노테이션이 있는 "org.springframework.context.annotation.ComponentScanAnnotatedConfig_WithValueAttribute" 가 됩니다. 


* ComponentScanAnnotationParser.parse(AnnotationAttributes, String)

{% highlight java %}

public Set<BeanDefinitionHolder> parse(AnnotationAttributes componentScan, final String declaringClass) {
	ClassPathBeanDefinitionScanner scanner =
			new ClassPathBeanDefinitionScanner(this.registry, componentScan.getBoolean("useDefaultFilters"));
 
	// scanner.setBeanNameGenerator() 설정
	// scanner.setScopedProxyMode(scopedProxyMode) 설정
	// scanner.setResourcePattern(componentScan.getString("resourcePattern")) 설정
	// scanner.addIncludeFilter(typeFilter) 설정
	// scanner.addExcludeFilter(typeFilter) 설정
	// scanner.getBeanDefinitionDefaults().setLazyInit(true) 설정 

	List<String> basePackages = new ArrayList<String>();
	for (String pkg : componentScan.getStringArray("value")) {
		if (StringUtils.hasText(pkg)) {
			basePackages.add(pkg);
		}
	}
	for (String pkg : componentScan.getStringArray("basePackages")) {
		if (StringUtils.hasText(pkg)) {
			basePackages.add(pkg);
		}
	}
	for (Class<?> clazz : componentScan.getClassArray("basePackageClasses")) {
		basePackages.add(ClassUtils.getPackageName(clazz));
	}

	if (basePackages.isEmpty()) {
		basePackages.add(ClassUtils.getPackageName(declaringClass));
	}

	return scanner.doScan(StringUtils.toStringArray(basePackages));
}
{% endhighlight %}

위 메서드 두 번째 줄에 선언된 ClassPathBeanDefinitionScanner 클래스의 인스턴스는 이름에서 알 수 있듯 클래스패스에서 BeanDefinition을 찾는다. 

위 코드의 생략된 부분은 첫 번째 매개변수인 componentScan에 포함되어있는 @ComponentScan의 각 옵션들(excludeFilters, scopeProxy, nameGenerator 등)을 읽어 이를 처리할 수 있도록  ClassPathBeanDefinitionScanner의 인스턴스 scanner에 설정하는 코드. 

그리고 나머지 부분은 scan작업을 할 package, classes들을 설정하여 scanner.doScan() 메서드를 통해 스캔작업을 실행한다. 이 예제에서는 특정 패키지(example.scannable)를 스캔하게 된다.

#### 실질적으로 Component scan 패키지 디렉토리에서 리소스를 찾는 ClassPathBeanDefinitionScanner 

* ClassPathBeanDefinitionScanner.doScan(String...)

{% highlight java %}

protected Set<BeanDefinitionHolder> doScan(String... basePackages) {
	Assert.notEmpty(basePackages, "At least one base package must be specified");
	Set<BeanDefinitionHolder> beanDefinitions = new LinkedHashSet<BeanDefinitionHolder>();
	for (String basePackage : basePackages) {
		Set<BeanDefinition> candidates = findCandidateComponents(basePackage); // 특정 패키지에 리소스를 모두 읽어온다.
		for (BeanDefinition candidate : candidates) {  // 리소스를 검사하기 각 리소스를 순회
			ScopeMetadata scopeMetadata = this.scopeMetadataResolver.resolveScopeMetadata(candidate);
			candidate.setScope(scopeMetadata.getScopeName());
			String beanName = this.beanNameGenerator.generateBeanName(candidate, this.registry);
			if (candidate instanceof AbstractBeanDefinition) {  
				postProcessBeanDefinition((AbstractBeanDefinition) candidate, beanName); // 해당 빈의 Autowired가 있는지 검사
			}
			if (candidate instanceof AnnotatedBeanDefinition) {
				AnnotationConfigUtils.processCommonDefinitionAnnotations((AnnotatedBeanDefinition) candidate); // 다른 어노테이션들 검사(@Lazy, @Role 등..)
			}
			if (checkCandidate(beanName, candidate)) { // includeFilter, excludeFilter 검사로 빈 필터링
				BeanDefinitionHolder definitionHolder = new BeanDefinitionHolder(candidate, beanName);
				definitionHolder = AnnotationConfigUtils.applyScopedProxyMode(scopeMetadata, definitionHolder, this.registry);
				beanDefinitions.add(definitionHolder);
				registerBeanDefinition(definitionHolder, this.registry); // bean registry에 등록
			}
		}
	}
	return beanDefinitions;
}
{% endhighlight %}


여기서 5번째 줄의  Set<BeanDefinition> candidates = findCandidateComponents(basePackage); 에서 특정 패키지에 속한 리소스들(파일들)을 모두 읽어온다. 


* ClassPathBeanDefinitionScanner.findCandidateComponents(String...)

{% highlight java %}

public Set<BeanDefinition> findCandidateComponents(String basePackage) {
	Set<BeanDefinition> candidates = new LinkedHashSet<BeanDefinition>();
	
	// error loggin부분 생략,  예외처리 코드 생략...

	String packageSearchPath = ResourcePatternResolver.CLASSPATH_ALL_URL_PREFIX +
			resolveBasePackage(basePackage) + "/" + this.resourcePattern;

	Resource[] resources = this.resourcePatternResolver.getResources(packageSearchPath);

	for (Resource resource : resources) {
		
		MetadataReader metadataReader = this.metadataReaderFactory.getMetadataReader(resource);
		if (isCandidateComponent(metadataReader)) {
			ScannedGenericBeanDefinition sbd = new ScannedGenericBeanDefinition(metadataReader);
			sbd.setResource(resource);
			sbd.setSource(resource);
			if (isCandidateComponent(sbd)) {
				if (debugEnabled) {
					logger.debug("Identified candidate component class: " + resource);
				}
				candidates.add(sbd);
			}
			
		}
		
	}
	
	return candidates;
}
{% endhighlight %}

6번째 줄에서 packageSearchPath에는 "classpath*:example/scannable/**/*.class" 와 같은 문자열로 설정되어 특정 클래스 패스의 특정 패키지 밑에 소속한 모든 .class파일들이 스캔 대상이 된다. 

9 번째 줄의 this.resourcePatternResolver.getResources(packageSearchPath)을 통해 Resource들을(여기선 모두 파일이기 때문에 FileSystemReosource) 구할 수 있다. 즉, example.scannable 패키지에 위치한 모든 클래스 파일들을 구할 수 있다.(inner class포함)


메소드 실행 후 리소스들을 구했다면 다음에 실행되는 for문에서 각 리소스를 순회하는데 if문의 isCandidateComponent(metadataReader)를 통해 ComponentScan에 excludeFilter, includeFilter와 같은 필터를 설정해두었다면 이를 검사한다.(아래코드 참조) 

* ClassPathBeanDefinitionScanner.isCandidateComponent(MetadataReader metadataReader)

{% highlight java %}
protected boolean isCandidateComponent(MetadataReader metadataReader) throws IOException {
	for (TypeFilter tf : this.excludeFilters) {
		if (tf.match(metadataReader, this.metadataReaderFactory)) {
			return false;
		}
	}
	for (TypeFilter tf : this.includeFilters) {
		if (tf.match(metadataReader, this.metadataReaderFactory)) {
			return isConditionMatch(metadataReader);
		}
	}
	return false;
}
{% endhighlight %}

위 코드에서 별도의 excludeFilter를 설정해두지 않았기 때문에 생략되지만 includeFilter에는 기본적으로 3가지 필터가 있는데 그 중 하나가 클래스의 어노테이션 중 org.springframework.stereotype.Component 어노테이션을 가졌는지 검사하는 필터가 있다. 즉 여기서 @Component 어노테이션을 가진 클래스들은 필터를 통과해 빈 등록 과정을 거치게 된다. 

리소스들 중 필터를 거친 리소스들을 읽어와 그다음으로 수행하는 for문에서 빈이 싱글톤으로 관리될지 아니면 새로운 인스턴스를 만드는지를 판별하는 Scoped Proxy와 beanNameGenerator를 통해 bean의 이름을 생성한 뒤 이를 BeanDefinitionHolder로 만들어 beanDefinitions(LinkedHashSet)에 담아두고 이를 registry에서 등록하게 됩니다.

여기서 registry는 제일 처음 매개변수로 넘겨주었던 beanFactory로 DefaultListableBeanFactory 클래스의 인스턴스입니다. 
결론적으로 빈 팩토리의 beanResgitry에 등록된다. 


### 빈 호출과정

Component scan으로 빈이 등록되었고, 테스트 코드의 9번째 줄에서 ctx.getBean(TestBean.class);를 처리하는 과정을 살펴보겠습니다.
빈을 등록했을때 beanFactory를 활용했듯이 역시 beanFactory에서 getBean(Class<T>) 메서드로 찾고자 하는 빈의 클래스정보를 넘겨주면 beanFactory(DefaultListableBeanFactory)의 상위 타입인 SingletonBeanRegistry의 getSingleton(beanName)을 호출하게됩니다.  

* DefaultSingletonBeanRegistry.getSingleton(String, boolean)

{% highlight java %}
protected Object getSingleton(String beanName, boolean allowEarlyReference) {
	Object singletonObject = this.singletonObjects.get(beanName);  // singletonObjects에 singleton 빈들이 등록되어 있다.
	if (singletonObject == null && isSingletonCurrentlyInCreation(beanName)) {
		synchronized (this.singletonObjects) {
			singletonObject = this.earlySingletonObjects.get(beanName);
			if (singletonObject == null && allowEarlyReference) {
				ObjectFactory<?> singletonFactory = this.singletonFactories.get(beanName);
				if (singletonFactory != null) {
					singletonObject = singletonFactory.getObject();
					this.earlySingletonObjects.put(beanName, singletonObject);
					this.singletonFactories.remove(beanName);
				}
			}
		}
	}
	return (singletonObject != NULL_OBJECT ? singletonObject : null);
}
{% endhighlight %}

* 2번째 줄의 this.singletonObjects.get(beanName); 을통해 실질적인 TestBean 빈을 찾게되는데 singletonObjects는 이전 빈 등록과정에서 registry 처럼 ConcurrentHashMap<String, Object>(64) 타입이다. 
* Component Scan시 빈이 등록되는 곳은 DefaultListableBeanFactory클래스의 beanDefinitionMap 이라고 언급했는데 Component Scan을 진행하는 메서드 stack 중 ConfigurationClassPostProcessor.processConfigBeanDefinitions() 메서드(본 글의 3번째 코드를 참조) 진행 후 

* ConfigurationClassPostProcessor.processConfigBeanDefinitions()
{% highlight java %}
// 생략..
parser.parse(configCandidates); // Component Scan 수행 부분
// 생략...

if (singletonRegistry != null) {
	if (!singletonRegistry.containsSingleton(IMPORT_REGISTRY_BEAN_NAME)) {
		singletonRegistry.registerSingleton(IMPORT_REGISTRY_BEAN_NAME, parser.getImportRegistry());
	}
}
{% endhighlight %}

위 코드에서 singletonRegistry가 BeanFactory인 DefaultListableBeanFactory. 
singletonRegistry.registerSingleton(IMPORT_REGISTRY_BEAN_NAME, parser.getImportRegistry()); 는 singletonRegistry의 상위 클래스인 DefaultSingletonBeanRegistry에 자신의 registry에 등록된 빈들을 singleton일 경우 넘겨준다. 

DefaultListableBeanFactory.registerSingleton()

{% highlight java %}
@Override
public void registerSingleton(String beanName, Object singletonObject) throws IllegalStateException {
	super.registerSingleton(beanName, singletonObject); // 상위 클래스가 DefaultSingletonBeanRegistry
	clearByTypeCache();
}
{% endhighlight %}

지금까지가 Singleton 빈이 호출되는 과정이다. 
지금까지 설명한 빈 검색은 싱글톤이 아닐경우나 다른 설정들이 있을 경우 다른 방식으로 진행되지만 기본적인 싱글톤 빈의 경우의 검색의 경우에 해당한다.


### Spring MVC에서의 Component Scan

위 분석에서는 AnnotationConfigApplicationContext 클래스가 스프링 컨테이너로 사용되었지만 우리가 실질적으로 사용하는 Spring MVC에서 사용되는 Context는 org.springframework.web.context.support.XmlWebApplicationContext 다.
이는 spring-web-버전.jar에서 org.springframework.web.context 패키지에 ContextLoader.properties 파일을 열어보면 알 수 있다.

* ContextLoader.properties

{% highlight java %}
# Default WebApplicationContext implementation class for ContextLoader.
# Used as fallback when no explicit context implementation has been specified as context-param.
# Not meant to be customized by application developers.

org.springframework.web.context.WebApplicationContext=org.springframework.web.context.support.XmlWebApplicationContext

{% endhighlight %}

XmlWebAppplicationContext에서는 초기화 과정에서 상위 클래스인 AbstractRefreshableApplicationContext.refreshBeanFactory() 메서드를 통해  DefaultListableBeanFactory를 생성하여 사용한다. 

* AbstractRefreshableApplicationContext.refreshBeanFactory()

{% highlight java %}
protected final void refreshBeanFactory() throws BeansException {
	if (hasBeanFactory()) {
		destroyBeans();
		closeBeanFactory();
	}
	try {
		DefaultListableBeanFactory beanFactory = createBeanFactory(); // beanFactory 생성.
		beanFactory.setSerializationId(getId());
		customizeBeanFactory(beanFactory);
		loadBeanDefinitions(beanFactory);
		synchronized (this.beanFactoryMonitor) {
			this.beanFactory = beanFactory;
		}
	} catch (IOException ex) {
		throw new ApplicationContextException("I/O error parsing bean definition source for "
			+ getDisplayName(), ex);
	}
}
{% endhighlight %}

* 이후의 과정은 생략된 부분이 많이 있지만 Component Scan은 앞서 설명한 과정을 통해 빈을 등록하게 된다. 

### 부록 1. @Component

@Component는 스프링이 어노테이션에 담긴 메타정보를 이용하기 시작했을 때 @Autowired와 함께 소개된 대표적인 어노테이션입니다. @Component는 클래스에 부여되는데 이는 빈 스캐너를 통해 자동으로 빈으로 등록됩니다. 정확히는 @Component 또는 @Component를 메타 어노테이션으로 갖고 있는 어노테이션(@Controller, @Repository, @Service를 말함)이 붙은 클래스가 빈 등록 대상이 됩니다.

@Component의 @Controller, @Repository, @Service

보통 스프링 MVC 코드를 작성할 때 @Controller, @Repository, @Service를 각 클래스의 성격에 맞추어 붙여주곤 했습니다. 
가끔 @Component를 붙여도 별다른 차이 없이 동작하는걸 확인 할 수 있는데 이는 @Component 어노테이션이 앞서 언급한 3개의 어노테이션의 메타 어노테이션(어노테이션에 대한 어노테이션)으로 사용되었기 때문입니다. 
스프링 컨테이너가 초기화 될 때 설정에따라(XML이나 어노테이션 기반 설정) 빈 등록과정에서 특정 클래스의 어노테이션을 읽어(리플렉션 API인 getDeclaredAnnoations(), getAnnotations()를 활용)들입니다. 이 때 해당 특정 어노테이션을 만나면 해당 어노테이션의 메타 어노테이션을 재귀적으로 읽어들이기 때문에 @Controller, @Repository, @Service를 만나면 그 메타 어노테이션인 @Component 어노테이션을 읽어들이게 되고 하나의 빈으로 등록하게 됩니다.

이 어노테이션들의 패키지명이 org.springframework.stereotype인데 왜 org.springframework.annotation이라는 패키지가 아닌 stereotype인가 의문이긴 한데 streotype를 사전에서 찾아보면 '정형화 하다, 형식화 하다' 라는 의미가 있어 그 의미에 맞추어 패키지명을 정하지 않았는가 짐작해봅니다.
네 개의 어노테이션 말고도 @Component를 메타 어노테이션으로 가진 사용자 정의 어노테이션도 정의 가능합니다. 어노테이션의 용도가 AOP에서 포인트컷을 작성할 경우와 같이 다양한 용도로 활용할 수 있습니다.
일반적으로 @Component와 나머지 세 개의 어노테이션이 차이점이 없음에도 불구하고 각 클래스의 성격에 맞추어 어노테이션을 붙이길 권장하는데 이는 각 클래스의 성격에 맞게 작성하는게 가독성이나 적합한 의미부여에 좋습니다. 

@Controller의 코드를 확인하면 @Conponent 어노테이션을 어노테이션으로 가지고 있습니다.

* Controller.java
{% highlight java %}
@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Component
public @interface Controller {

	String value() default "";

}
{% endhighlight %}
