---
layout:            post
title:             "Spring WebFlux"
date:              2017-12-12 18:33:00 +0300
tags:              spring
category:          Java
author:            eceris
---

### 1. Spring WebFlux
## 1.1. Introduction

스프링 프레워크에서 오리지날 웹 프레임워크는 포함되었다. Spring Web MVC는 Servlet API와 Servlet 컨테이너 용으로 개발되었습니다. reactive 스택이며 웹 프레임 워크 인 Spring WebFlux는 나중에 버전 5.0에 추가되었습니다. 그것은 완전히 non-blocking이며 Reactive Streams back pressure를 지원하며 Netty, Undertow 및 Servlet 3.1+ 컨테이너와 같은 서버에서 실행됩니다.

두 웹 프레임 워크는 스프링 모듈의 소스 모듈 인 spring-webmvc와 spring-webflux의 이름을 미러링하고 Spring Framework에서 나란히 공존합니다. 각 모듈은 선택 사항입니다. 응용 프로그램은 하나 또는 다른 모듈을 사용할 수도 있고 어떤 경우에는 둘 다 사용할 수도 있습니다.(리액션 WebClient가있는 스프링 MVC 컨트롤러)


# 1.1.1. Why a new web framework?
이 대답의 일부는 비 블로킹 웹 스택이 소수의 스레드로 동시성을 처리하고 하드웨어 자원을 줄이면서 확장 할 필요가 있다는 것입니다. Servlet 3.1은 non-blocking I/O를위한 API를 제공합니다. 그러나 이를 사용하면 synchronous (필터, 서블릿) 또는 blocking (getParameter, getPart)하는 나머지 Servlet API에서 멀어집니다. 이는 새로운 공통 API가 모든 non-blocking runtime에 대한 기반 역할을 하도록하는 motivation이 되었습니다. 이는 async, non-blocking space에 잘 설정된 Netty와 같은 서버 때문에 중요합니다.


대답의 다른 부분은 함수형 프로그래밍입니다. Java 5에서 annotations을 추가 한 것과 마찬가지로 많은 기회가있었습니다 - e.g. annotated REST controllers과 유닛테스트, 자바8의 람다식은 자바의 functional API에 대한 많은 기회를 만들었습니다. 


이것은 non-blocking 어플리케이션과 연결된 API style을 위한 좋은 일입니다. CompletableFuture 및 ReactiveX에 의해 대중화 되었고, asynchronous 로직에 대한 선언적 구성이 가능합니다. 프로그래밍 모델 레벨에서 Java 8은 Spring WebFlux가 annotation이 달린 컨트롤러와 함께 functional web endpoints를 제공할 수 있게 하였습니다.

# 1.1.2. Reactive: what and why?
우리는 non-blocking과 functional이 무엇인지에 대해 잠시 알아봤습니다. 그러나 왜 reactive이며, 그것이 뭘 의미하는지 알아봅니다.

reactive라는 단어는 변화에 reacting하는 프로그래밍 모델을 의미합니다. 예를 들면, I/O 이벤트에 반응하는 네트워크 구성 요소, 마우스 이벤트에 반응하는 UI 컨트롤러 등입니다. 이러한 의미로 non-blocking은 reactive 입니다. 왜냐하면 blocked 대신에 작업이 완료되거나 데이터가 사용 가능할 때 알림에 반응하는 방식에 있기 때문입니다.

그리고 또다른 중요한 메커니즘은 스프링 팀에서 reactive라고 부르는 것으로 그것은 non-blocking backpressure 메커니즘입니다. 동기식 필수코드에서 blocking은 발신자를 기다리게 하는 자연스러운 형태입니다. 그러나 비동기식 필수코드에서는 빠른 생산자가 목적지를 압도하지 않도록 이벤트 속도를 제어하는 것이 중요합니다. 

Reactive Streams는 자바 9에 적용된 작은 스펙입니다.(그것은 비동기 컴포넌트가 backpressure처럼 상호작용하는 것을 의미합니다.) 예를 들면 데이터 레포지토리에서 Publisher는 데이터를 생산할 수 있고, Subscriber는 response를 응답할 수 있습니다. Reactive Streams의 메인 목표는 Publisher가 생산하는 데이터의 속도를 Subscriber가 조절할 수 있는 것 입니다. 

> 일반적인 질문 : publisher가 속도를 늦출 수없는 경우 어떻게해야합니까?
> Reactive Stream의 목적은 메커니즘과 경계를 설정하는 것입니다. publisher가 속도를 늦출 수없는 경우 buffer, drop 또는 fail 여부를 결정해야합니다.

# 1.1.3. Reactive API
Reactive Stream은 상호 운용성에 중요한 역할을합니다. 라이브러리 및 인프라 구성 요소는 흥미로운 대상이지만 application API는 너무 low level이므로 유용하지는 않습니다. 어플리케이션이 필요한 것은 비동기 로직을 구성하는 functional api와 같은 high level의 풍부한 기능입니다. 자바8의 Stream API와 비슷하지만 컬렉션만을 위한 것이 아닙니다. 이것은 reactive 라이브러리가 수행하는 역할입니다.

Reactor는 Spring WebFlux에서 선택한 reactive 라이브러리입니다. 이것은 ReactX와 연계 된 풍부한 연산자 집합을 통해 0..1 및 0..N의 데이터 시퀀스에서 작동하는 Mono 및 Flux API 유형을 제공합니다. Reactor는 Reactive Streams 라이브러리이므로 모든 연산자는 non-blocking을 지원합니다. Reactor는 서버 측 Java에 중점을두고 있으며, Spring과 긴밀히 협력하여 개발되었습니다.

WebFlux는 Reactor를 핵심 종속성으로 요구하지만 Reactive Streams를 통해 다른 대응 라이브러리와 상호 운용됩니다.
일반적으로 WebFlux API는 일반 Publisher를 입력으로 허용하며, 내부적으로 Reactor 유형에 적용하고 사용하여 Flux 또는 Mono를 출력으로 반환합니다. 따라서 모든 Publisher를 입력으로 전달할 수 있으며 출력에 연산을 적용 할 수 있지만 다른 반응 라이브러리에서 사용하기 위해 출력을 조정해야합니다.

# 1.1.4. Programming models
Spring-Web 모듈에는 Spring WebFlux의 밑바탕이되는 reactive foundation이(HTTP 추상화, Reactive Streams 서버 어댑터, Reactive Streams 코덱 및 Servlet API와 비슷하지만 Non-Blokcing semantic을 가진 핵심 웹 API) 포함되어 있습니다.

이 기반에서 Spring WebFlux는 두 가지 프로그래밍 모델 중에서 선택할 수 있습니다.

  - Annotation을 사용한 Controllers : Spring MVC와 일치하며, spring-web 모듈의 annotation과 동일한 방식의 모델.(주목할만한 차이점은 WebFlux가 reactive @RequestBody 인수를 지원한다는 것입니다.)
  - Functional Endpoints : 람다 based, 가벼운 functional 프로그래밍 모델(어플리케이션이 처음부터 끝까지 요청의 처리를 담당하는 anntation 모델과의 가장 큰 차이점은 annotation을 통해 intent를 선언하고 콜백으로 다시 호출된다는 점 입니다.)


# 1.1.5. Choosing a web framework
Spring MVC 또는 WebFlux를 사용해야합니까? 몇 가지 다른 관점을 살펴 보겠습니다.

잘 작동하는 Spring MVC 애플리케이션을 가지고 있다면, 굳이 변경할 필요가 없습니다. 명령형 프로그래밍은 코드를 작성, 이해 및 디버그하는 가장 쉬운 방법입니다. 역사적으로 대부분 blocking의 형태이므로 최대한 많은 라이브러리를 선택할 수 있습니다.


만약 non-blocking 웹 스택을 이미 생각하고 있다면 Spring WebFlux는이 공간에서 다른 실행 모델과 동일한 실행 모델 혜택을 제공하며 Netty, Tomcat, Jetty, Undertow, Servlet 3.1+ 컨테이너 등 다양한 서버를 선택할 수 있습니다. 프로그래밍 모델 - annotated controller 및 기능 웹 엔드 포인트, reactive 라이브러리, RxJava 또는 기타의 반응 라이브러리 선택.

Java 8 lambda 또는 Kotlin과 함께 사용하기위한 가볍고 기능적인 웹 프레임 워크에 관심이 있다면 Spring WebFlux 기능 웹 엔드 포인트를 사용하십시오. 이는 보다 작은 투명성과 제어의 혜택을 누릴 수 있는 덜 복잡한 요구 사항을 가진 소규모 응용 프로그램 또는 마이크로 서비스에도 적합합니다.

마이크로 서비스 아키텍처에서는 Spring MVC 또는 Spring WebFlux 컨트롤러 또는 Spring WebFlux 기능 엔드 포인트와 함께 다양한 애플리케이션을 사용할 수 있습니다. 두 프레임 워크에서 동일한 Annotation 기반 프로그래밍 모델을 지원하면 올바른 작업에 적합한 도구를 선택하는 동시에 지식을 쉽게 재사용 할 수 있습니다.

응용 프로그램을 평가하는 간단한 방법은 응용 프로그램의 종속성을 확인하는 것입니다. blocking persistence API (JPA, JDBC) 나, 네트워킹 API를 사용한다면, Spring MVC가 최고의 선택입니다. Reactor와 RxJava 모두 별도의 스레드에서 Blocking API를  호출하는 것이 기술적으로는 가능하지만 non-blocking 웹 스택을 최대한 활용하지는 않습니다.


원격 서비스 호출이있는 Spring MVC 애플리케이션을 사용하고 있다면 Reactive WebClient를 사용해보십시오. 반응 형 (Reactor, RxJava 또는 기타)을 Spring MVC 컨트롤러 메소드에서 직접 반환 할 수 있습니다. 호출 당 대기 시간 또는 호출 간 상호 의존성이 클수록 더 많은 이점이 있습니다. 스프링 MVC 컨트롤러는 다른 reacitive 컴포넌트도 호출 할 수있습니다. 

규모가 큰 팀의 경우 non-blcoking, 기능 및 선언적 프로그래밍으로 전환하는 과정에서 가파른 학습 곡선을 염두에 두십시오. 전체 스위치없이 시작하는 실질적인 방법은 사후 대응하는 WebClient를 사용하는 것입니다. 그리고 나서 작은 서비스에 적용하고 이점이 있는지 파악합니다. 우리는 광범위한 응용 분야에서 변화가 필요하지 않을 것으로 예상합니다.

찾아야 할 이점이 확실하지 않은 경우 Non-Blocking I/O가 작동하는 방식 (예 : 단일 스레드 Node.js의 동시성이 모순되지 않음) 및 그 효과에 대해 학습하면서 시작하십시오. 태그 라인은 "하드웨어가 적은 스케일"이지만, 그 효과는 느려지거나 예측할 수 없는 네트워크 I/O 없이는 보장되지 않습니다. 이 Netflix 블로그 게시물은 좋은 자료입니다.


# 1.1.6. Choosing a server

Spring WebFlux는 Netty, Undertow, Tomcat, Jetty 및 Servlet 3.1+ 컨테이너에서 지원됩니다. 각 서버는 공통 Reactive Streams API에 맞게 조정됩니다. 스프링 WebFlux 프로그래밍 모델은 공통 API를 기반으로합니다.

> 일반적인 질문 : 두 스택 모두에서 Tomcat과 Jetty를 어떻게 사용할 수 있습니까?
> Tomcat과 Jetty의 core 는 non-blocking 입니다. 이들은 blocking facade가 추가된 Servlet API 입니다. 버전 3.1부터 Servlet API는 non-blocking I/O에 대한 선택 사항을 추가합니다. 그러나 그것을 사용하기 위해 다른 동기 및 blocing parts를 피해서 사용해야 합니다. 이러한 이유로 Spring의 Reactive web stack에는 Reactive Stream에 연결하기위한 low-level Servlet 어댑터가 있지만 서블릿 API는 직접 사용하지 못하도록 노출하지 않습니다.

Spring Boot 2는 기본적으로 WebFlux와 함께 Netty를 사용합니다. Netty가 비동기식 non-blocking 공간에서 보다 널리 사용되며 리소스를 공유할 수 있는 클라이언트와 서버를 모두 제공하기 때문입니다. 비교하여 Servlet 3.1 non-blocking I/O는 사용 빈도가 너무 높기 때문에 많은 사용을 보지 못했습니다. Spring WebFlux는 채택에 대한 한 가지 실질적인 길을 열었습니다.

스프링 부트의 서버 선택은 기본적으로 기본 제공 환경에 관한 것입니다. 응용 프로그램은 성능을 위해 최적화되고 완전히 차단되지 않고 Reactive Stream backpressure에 맞게 조정 된 다른 지원되는 서버를 선택할 수 있습니다. Spring Boot에서는 스위치를 만드는 것이 쉽습니다.


# 1.1.7. Performance vs scale
성과에는 많은 특징과 의미가 있습니다. Reactive 및 Non-Blocking 형은 일반적으로 응용 프로그램을 더 빠르게 실행하지 않습니다. 경우에 따라 WebClient를 사용하여 원격 호출을 병렬로 실행할 수 있습니다. 전체적으로 Non-Blocking 방식을 수행하기 위해 더 많은 작업이 필요하며 이는 필요한 처리 시간을 약간 늘릴 수 있습니다.

Reactive 및 Non-Blocking의 주요 이점은 작은 고정 된 스레드 수와 적은 메모리로 확장 할 수 있다는 것입니다. 따라서 응용 프로그램이 예측 가능한 방식으로 확장되므로 응용 프로그램이 로드 하에서보다 탄력적입니다. 그러나 이러한 이점을 관찰하려면 느리고 예측할 수없는 네트워크 I/O를 포함하여 대기 시간이 필요합니다. 이것이 바로 반응성 스택이 강점을 보여주기 시작하고 그 차이가 극적 일 수있는 부분입니다.

## 1.2. Reactive Spring Web
Spring-web 모듈은 반응이 적은 웹 애플리케이션을 구축하기 위해 클라이언트 및 서버와 같은 낮은 수준의 인프라와 HTTP 추상화를 제공합니다. 모든 공개 API는 Reactor가 지원되는 Reactive Streams를 기반으로합니다.

서버 지원은 두 가지 계층으로 구성됩니다.
 - HttpHandler and server adapters : Reactive Streams 배압장치로써 HTTP 요청 처리를 위한 가장 기본적인 공통 API.
 - WebHandler API : 약간 더 높은 수준이지만 필터 체인 스타일 처리 기능을 갖춘 범용 서버 웹 API입니다.


# 1.2.1. HttpHandler

모든 HTTP 서버에는 HTTP 요청 처리를위한 API가 있습니다. HttpHandler는 요청 및 응답을 처리하는 한 가지 방법을 사용하는 간단한 약속입니다. 그것은 아주 minimal 하게 만들어 졌으며, 주요 목적은 서로 다른 서버에서 HTTP 요청 처리를 위한 공통 Reactive Stream 기반 API를 제공하는 것입니다.

spring-web 모듈에는 지원되는 모든 서버에 대한 어댑터가 있습니다. 아래 표는 서버 API가 사용되고 Reactive Streams 지원이 제공되는 위치를 보여줍니다.

| Server name | Server API used | Reactive Streams support |
|:--------|:--------|:--------|
| Netty | Netty API | Reactor Netty |
| Undertow | Undertow API | spring-web: Undertow to Reactive Streams bridge |
| Tomcat | Servlet 3.1 non-blockingI/O; byte[]가 아닌 ByteBuffers를 읽고 쓰는 Tomcat API | spring-web: Servlet 3.1 non-blocking I/O to Reactive Streams bridge |
| Jetty | Servlet 3.1 non-blockingI/O; byte[]가 아닌 ByteBuffers를 읽고 쓰는 Tomcat API | spring-web: Servlet 3.1 non-blocking I/O to Reactive Streams bridge |
| Servlet 3.1 container | Servlet 3.1 non-blocking I/O | spring-web: Servlet 3.1 non-blocking I/O to Reactive Streams bridge |

다음은 각 서버에 대한 필수 종속성, 지원되는 버전 및 코드조각 입니다. 

| Server name | Group id | Artifact name |
|:--------|:--------|:--------|
| Reactor Netty | io.projectreactor.ipc | reactor-netty |
| Undertow | io.undertow | undertow-core |
| Tomcat | org.apache.tomcat.embed | tomcat-embed-core |
| Jetty | org.eclipse.jetty | jetty-server, jetty-servlet |

Reactor Netty:
```java
HttpHandler handler = ...
ReactorHttpHandlerAdapter adapter = new ReactorHttpHandlerAdapter(handler);
HttpServer.create(host, port).newHandler(adapter).block();
```
Undertow:
```java
HttpHandler handler = ...
UndertowHttpHandlerAdapter adapter = new UndertowHttpHandlerAdapter(handler);
Undertow server = Undertow.builder().addHttpListener(port, host).setHandler(adapter).build();
server.start();
```
Tomcat:
```java
HttpHandler handler = ...
Servlet servlet = new TomcatHttpHandlerAdapter(handler);

Tomcat server = new Tomcat();
File base = new File(System.getProperty("java.io.tmpdir"));
Context rootContext = server.addContext("", base.getAbsolutePath());
Tomcat.addServlet(rootContext, "main", servlet);
rootContext.addServletMappingDecoded("/", "main");
server.setHost(host);
server.setPort(port);
server.start();
```
Jetty:
```java
HttpHandler handler = ...
Servlet servlet = new JettyHttpHandlerAdapter(handler);

Server server = new Server();
ServletContextHandler contextHandler = new ServletContextHandler(server, "");
contextHandler.addServlet(new ServletHolder(servlet), "/");
contextHandler.start();

ServerConnector connector = new ServerConnector(server);
connector.setHost(host);
connector.setPort(port);
server.addConnector(connector);
server.start();
```


> Servlet 3.1+ 컨테이너에 WAR로 배포하려면 HttpHandler를 ServletHttpHandlerAdapter로 감싸서 서블릿으로 등록하십시오.이것은 AbstractReactiveWebInitializer를 사용하여 자동화 할 수 있습니다.

# 1.2.2. WebHandler API
HttpHandler는 다른 서버에서 실행하기 위한 기본입니다. 이 기본에서 WebHandler API는 예외 핸들러 (WebExceptionHandler), 필터 (WebFilter) 및 대상 핸들러 (WebHandler)의 약간 더 높은 레벨 처리 체인을 제공합니다.

모든 컴포넌트는 ServerWebExchange에서 동작합니다- 요청 속성, 세션 속성, 양식 데이터, 멀티 파트 데이터에 대한 액세스 등을 추가하는 HTTP 요청 및 응답을위한 컨테이너.

processing chain은 WebHttpHandlerBuilder와 함께 사용할 수 있습니다. WebHttpHandlerBuilder는  server adapter로 실행될 수있는 HttpHandler를 만듭니다. 빌더를 사용하려면 컴포넌트를 개별적으로 추가하거나 ApplicationContext를 가리켜 다음을 탐지해야합니다.

| Bean name | Bean type  | Count | Description |
|:--------|:--------|:--------|:--------|
| "webHandler" | WebHandler | 1 | 필터 후 Target handler |
| <any> | WebFilter | 0..N | 필터 |
| <any> | WebExceptionHandler | 0..N | 필터체인 이후 Exception handlers |
| "webSessionManager" | WebSessionManager | 0..1 | Custom session manager; default로 DefaultWebSessionManager |
| "serverCodecConfigurer" | ServerCodecConfigurer | 0..1 | Custom form and multipart data decoders; default로 ServerCodecConfigurer.create() |
| "localeContextResolver" | LocaleContextResolver | 0..1 | Custom resolver for LocaleContext; default로 AcceptHeaderLocaleContextResolver |


# 1.2.3. Codecs
spring-web 모듈은 HTTP 요청 및 응답 본문을 리 액티브 스트림으로 인코딩 및 디코딩하기 위해 HttpMessageReader 및 HttpMessageWriter를 제공합니다. 제공 되는 모듈은 spring-core 모듈의 lower level을 기반으로 합니다.
 - DataBuffer : 바이트 버퍼 추상화(예 : Netty의 ByteBuf, java.nio.ByteBuffer, 데이터 버퍼와 코덱을 참조하십시오.)
 - Encoder : 오브젝트의 스트림을 데이터 버퍼의 스트림에 serialize 한다
 - Decoder : 데이터 버퍼의 스트림을 객체의 스트림에 deserialize 한다

기본 인코더 및 디코더 구현은 스프링 코어에 있지만 스프링 웹은 JSON, XML 및 기타 형식에 대한 추가 기능을 추가합니다. Encoder와 Detoder는 EncoderHttpMessageWriter 및 DecoderHttpMessageReader를 사용하여 reader 또는 writer로 래핑 할 수 있습니다. 서버 전송 이벤트, 양식 데이터 등에 대한 웹 전용 리더 및 작성기 구현이 추가로 있습니다.

마지막으로, ClientCodecConfigurer와 ServerCodecConfigurer를 사용하여 readers와 writers 목록을 초기화 할 수 있습니다. 여기에는 classpath detection  및 기본 설정을 override or replace 할 수있는 기능을 제공합니다.

## 1.3. DispatcherHandler
스프링 MVC와 마찬가지로 Spring WebFlux는 중앙 WebHandler 인 DispatcherHandler가 요청 처리를 위한 공유 알고리즘을 제공하는 반면 실제 작업은 구성 가능한 위임 구성 요소로 수행되는 front controller pattern 을 중심으로 설계되었습니다. 이 모델은 유연하고 다양한 워크 플로를 지원합니다.

DispatcherHandler는 Spring 구성에서 필요한 위임 구성 요소를 검색합니다. 또한 Spring 빈 자체로 설계되었으며, 실행되는 컨텍스트에 액세스하기 위해 ApplicationContextAware를 구현합니다. DispatcherHandler가 bean 이름이 "webHandler"로 선언 된 경우 WebHandler API에서 설명한대로 요청 처리 체인을 결합하는 WebHttpHandlerBuilder에 의해 차례로 발견됩니다.

WebFlux 애플리케이션의 Spring 구성은 일반적으로 다음을 포함합니다.

 - bean 이름이 "webHandler"인 DispatcherHandler
 - WebFilter 및 WebExceptionHandler beans
 - DispatcherHandler special beans
 - 그 외..

WebHttpHandlerBuilder에 처리 체인을 만들기위한 구성이 제공됩니다.
```java
ApplicationContext context = ...
HttpHandler handler = WebHttpHandlerBuilder.applicationContext(context);
```

그 결과로 HttpHandler는 서버 어댑터와 함께 사용할 준비가 되었습니다.

# 1.3.1. Special bean types
DispatcherHandler는 요청을 처리하고 적절한 응답을 렌더링하기 위해 special beans에 위임합니다. "special beans"이란 아래 표에 나열된 프레임 워크 약속 중 하나를 구현하는 Spring 관리 객체 인스턴스를 의미합니다. Spring WebFlux는 이러한 계약의 기본 구현을 제공하지만 커스터마이징, 확장 또는 대체 할 수도 있습니다.

| Bean type | Explanation |
|:--------|:--------|
| HandlerMapping | Request를 handler에 매핑합니다. 매핑은 HandlerMapping 구현에 따라 세부사항이 달라지는 몇가지 기준을 기반으로 합니다(ex: annotated controllers, simple URL pattern mappings). |
| HandlerAdapter | DispatcherHandler가 핸들러가 실제로 호출되는 방법에 관계없이 요청에 매핑 된 핸들러를 호출 할 수있게 도와줍니다. 예를 들어 annotated controllers를 호출하려면 Annotation을 해석해야합니다. HandlerAdapter의 주요 목적은 DispatcherHandler를 그러한 세부 사항으로부터 보호하는 것입니다.|
| HandlerResultHandler | Handler호출의 결과를 처리하여 response를 마무리합니다. 내장 HandlerResultHandler 구현은 ResponseEntity 리턴 값을 지원하는 ResponseEntityResultHandler, @ResponseBody 메소드를 지원하는 ResponseBodyResultHandler, functional 엔드 포인트에서 리턴 된 ServerResponse를 지원하는 ServerResponseResultHandler 그리고 뷰 및 모델로 렌더링을 지원하는 ViewResolutionResultHandler입니다. |

# 1.3.2. Framework Config
DispatcherHandler는 ApplicationContext에서 필요한 특수 bean을 감지합니다. 어플리케이션은 원하는 special beans을 선언 할 수 있습니다. 그러나 대부분의 애플리케이션은 더 높은 수준의 구성 API를 제공하는 WebFlux Java 구성에서 더 나은 출발점을 찾아 필요한 Bean 선언을 만듭니다.

# 1.3.3. Processing
DispatcherHandler는 다음과 같이 요청을 처리합니다.
 - 각 HandlerMapping은 일치하는 핸들러를 찾고 첫 번째 일치가 사용됩니다.
 - 핸들러가 발견되면 HandlerResult로 실행 된 리턴 값을 표시하는 적절한 HandlerAdapter를 통해 핸들러가 실행됩니다.
 - HandlerResult는 적절한 HandlerResultHandler에 주어져 응답에 직접 쓰거나 렌더링 할 뷰를 사용하여 처리를 완료합니다.

## 1.4. Annotated Controllers
Spring WebFlux는 @Controller 및 @RestController 구성 요소가 Annotation을 사용하여 요청 매핑, 요청 입력, 예외 처리 등을 표현하는 Annotation 기반 프로그래밍 모델을 제공합니다. Annotation 컨트롤러에는 유연한 메서드 서명이 있으며 기본 클래스를 확장하거나 특정 인터페이스를 구현할 필요가 없습니다.
```java
@RestController
public class HelloController {

    @GetMapping("/hello")
    public String handle() {
        return "Hello WebFlux";
    }
}
```
이 예제에서 메서드는 응답 본문에 쓸 String을 반환합니다.

# 1.4.1. @Controller
표준 스프링 빈 정의를 사용하여 컨트롤러 빈을 정의 할 수 있습니다. @Controller 스테레오 타입은 클래스 패스에서 @Component 클래스를 감지하고 bean 정의를 자동 등록하는 Spring의 일반적인 지원과 일치하는 자동 Detection 을 허용합니다. 또한 annotation이 달린 클래스의 스테레오 타입 역할을 하여 웹 컴포넌트 역할을 나타냅니다.

이러한 @Controller 빈을 자동 Detection 하려면 Java 구성에 구성 요소 검색을 추가 할 수 있습니다.
```java
@Configuration
@ComponentScan("org.example.web")
public class WebConfig {

    // ...
}
```

@RestController는 @Controller와 @ResponseBody가 합쳐진 Annotation으로, 모든 메소드가 @ResponseBody Annotation을 type-level에서 상속하므로 response body (vs model-and-view rendering)에 기록합니다.

# 1.4.2. Request Mapping
@RequestMapping Annotation은 요청을 매핑하는 데 사용됩니다. URL, HTTP 메소드, 요청 매개 변수, 헤더 및 미디어 유형에 따라 일치시킬 다양한 속성이 있습니다. 클래스 레벨에서 공유 맵핑을 표현하거나 메소드 레벨에서 특정 엔드 포인트 맵핑으로 범위를 좁히기 위해 사용할 수 있습니다.

@RequestMapping의 HTTP 메소드 관련 단축키 변형도 있습니다.
 - @GetMapping

 - @PostMapping

 - @PutMapping

 - @DeleteMapping

 - @PatchMapping
단축키 변형은 구성된 Annotation (@RequestMapping으로 Annotation이 포함된)입니다. 이들은 일반적으로 메소드 레벨에서 사용됩니다. 클래스 수준에서 @RequestMapping은 공유 매핑을 표현하는 데 더 유용합니다.
```java
@RestController
@RequestMapping("/persons")
class PersonController {

    @GetMapping("/{id}")
    public Person getPerson(@PathVariable Long id) {
        // ...
    }

    @PostMapping
    @ResponseStatus(HttpStatus.CREATED)
    public void add(@RequestBody Person person) {
        // ...
    }
```

# URI Patterns
glob 패턴 및 와일드 카드를 사용하여 요청을 매핑 할 수 있습니다.
 - ? : 한문자 일치
 - * : 경로 세그먼트 내의 0 개 이상의 문자와 일치합니다.
 - ** : 0 개 이상의 경로 세그먼트와 일치

또한 URI 변수를 선언하고 @PathVariable을 사용하여 값에 액세스 할 수 있습니다.
```java
@GetMapping("/owners/{ownerId}/pets/{petId}")
public Pet findPet(@PathVariable Long ownerId, @PathVariable Long petId) {
    // ...
}
```
URI 변수는 클래스 및 메서드 수준에서 선언 할 수 있습니다.
```java
@Controller
@RequestMapping("/owners/{ownerId}")
public class OwnerController {

    @GetMapping("/pets/{petId}")
    public Pet findPet(@PathVariable Long ownerId, @PathVariable Long petId) {
        // ...
    }
}
```

URI 변수는 자동으로 적절한 유형으로 변환되거나 'TypeMismatchException'이 발생합니다. 단순 유형 - int, long, Date는 기본적으로 지원되며 다른 모든 데이터 유형에 대한 지원을 등록 할 수 있습니다.

URI 변수는 명시 적으로 이름을 지정할 수 있습니다. 예 : @PathVariable ( "customId"), 이름이 동일하고 디버깅 정보 또는 Java 8의 -parameters 컴파일러 플래그로 코드가 컴파일 된 경우 해당 세부 정보를 남길 수 있습니다.

구문 {\*varName}은 0 개 이상의 나머지 경로 세그먼트와 일치하는 URI 변수를 선언합니다. 예를 들어, /resources/{\*path}는 모든 파일 /resources/와 일치하고 "path"변수는 완전한 상대 경로를 캡처합니다.
```java
@GetMapping("/{name:[a-z-]+}-{version:\\d\\.\\d\\.\\d}{ext:\\.[a-z]+}")
public void handle(@PathVariable String version, @PathVariable String ext) {
    // ...
}
```

URI 경로 패턴에는 로컬, 시스템, 환경 및 기타 속성 소스에 대해 PropertyPlaceHolderConfigurer를 통해 시작시 해결되는 ${...} 위치 표시자가 포함될 수도 있습니다. 예를 들어 외부 구성에 따라 기본 URL을 매개 변수화하는 데 사용할 수 있습니다.

> Spring WebFlux는 Spring-web에 위치하고 URI 경로 패턴의 수가 많은 런타임에 일치하는 웹 응용 프로그램의 HTTP URL 경로와 함께 사용하도록 명시된 URI 경로 일치 지원을 위해 PathPattern 및 PathPatternParser를 사용합니다.

스프링 WebFlux는 /person과 같은 매핑이 /person.* 와 일치하는 Spring MVC와 달리 접미사 패턴 일치를 지원하지 않습니다. URL 기반 콘텐츠 협상의 경우 필요한 경우 URL 경로 기반 공격에 대해보다 간단하고 명시 적이며 취약하지 않은 쿼리 매개 변수를 사용하는 것이 좋습니다.


# Pattern Comparison
URL과 일치하는 패턴이 여러 개인 경우 가장 적합한 패턴을 찾으려면 비교해야합니다. 이것은 더 구체적인 패턴을 찾는 PathPattern.SPECIFICITY_COMPARATOR로 수행됩니다.

모든 패턴에 대해 점수는 URI 변수의 수와 와일드 카드보다 낮은 URI 변수가있는 와일드 카드를 기반으로 계산됩니다. 총 점수가 낮은 패턴이 승리합니다. 두 패턴의 점수가 같으면 더 긴 패턴이 선택됩니다. 

패턴을 모두 잡아라. \*\*, {\*varName}은 채점에서 제외되며 항상 마지막으로 정렬됩니다.

# Consumable Media Types
요청의 Content-Type을 기반으로 요청 매핑 범위를 좁힐 수 있습니다. 
```java
@PostMapping(path = "/pets", consumes = "application/json")
public void addPet(@RequestBody Pet pet) {
    // ...
}
```
consumes 속성은 부정화 표현식도 지원합니다. ! text / plain은 "text / plain"이외의 모든 내용 유형을 의미합니다.

클래스 수준에서 공유 consumes attribute을 선언 할 수 있습니다. 그러나 클래스 레벨에서 사용될 때 대부분의 다른 요청 맵핑 속성과는 달리 메소드 레벨의 consumes 속성은 클래스 레벨 선언을 확장하는 것이 아니라 오버라이드합니다.

> MediaType은 일반적으로 사용되는 미디어 유형에 대한 상수를 제공합니다. APPLICATION_JSON_VALUE, APPLICATION_JSON_UTF8_VALUE.


# Producible Media Types
Accept request header와 컨트롤러 메서드가 생성하는 content type 목록을 기반으로 요청 매핑 범위를 좁힐 수 있습니다.
```java
@GetMapping(path = "/pets/{petId}", produces = "application/json;charset=UTF-8")
@ResponseBody
public Pet getPet(@PathVariable String petId) {
    // ...
}
```
Media type은 character set 를 지정할 수 있습니다. 제외 된 표현식이 지원됩니다. ! text / plain은 "text / plain"이외의 모든 내용 유형을 의미합니다.

클래스 수준에서 공유 consumes attribute을 선언 할 수 있습니다. 그러나 클래스 레벨에서 사용될 때 대부분의 다른 요청 맵핑 속성과는 달리 메소드 레벨 생성 속성은 클래스 레벨 선언을 확장하지 않고 대체합니다.

> MediaType은 일반적으로 사용되는 미디어 유형에 대한 상수를 제공합니다. APPLICATION_JSON_VALUE, APPLICATION_JSON_UTF8_VALUE.

#Parameters and Headers
쿼리 매개 변수 조건을 기반으로 요청 매핑 범위를 좁힐 수 있습니다. 부재 ( "! myParam") 또는 특정 값 ( "myParam = myValue")에 대한 쿼리 매개 변수 ( "myParam")의 존재 여부를 테스트 할 수 있습니다.
```java
@GetMapping(path = "/pets/{petId}", params = "myParam=myValue")
public void findPet(@PathVariable String petId) {
    // ...
}
```


또한 요청 헤더 조건과 동일하게 사용할 수 있습니다.
```java
@GetMapping(path = "/pets", headers = "myHeader=myValue")
public void findPet(@PathVariable String petId) {
    // ...
}
```

# HTTP HEAD, OPTIONS
@GetMapping - @RequestMapping (method = HttpMethod.GET)은 요청 매핑 목적으로 HTTP HEAD를 투명하게 지원합니다. 컨트롤러 메소드는 변경할 필요가 없습니다. HttpHandler 서버 어댑터에 적용된 응답 래퍼는 "Content-Length"헤더가 기록 된 바이트 수로 설정되고 실제로 응답에 쓰지 않도록합니다.

기본적으로 HTTP OPTIONS는 URL 패턴이 일치하는 모든 @RequestMapping 메소드에 나열된 HTTP 메소드 목록에 
허락되는 응답 헤더를 설정하여 처리됩니다.

HTTP 메소드 선언이없는 @RequestMapping의 경우 허용된 헤더가 "GET, HEAD, POST, PUT, PATCH, DELETE, OPTIONS"로 설정됩니다. 컨트롤러 메소드는 @GetMapping, @PostMapping 등과 같은 HTTP 메소드 특정 변형을 사용하여 지원되는 HTTP 메소드를 항상 선언해야합니다.

@RequestMapping 메서드는 HTTP HEAD 및 HTTP OPTIONS에 명시 적으로 매핑 될 수 있지만 일반적인 경우에는 필요하지 않습니다.


# 1.4.3. Handler methods
@RequestMapping handler 메서드는 유연한 서명을 가지며 지원되는 컨트롤러 메서드 인수 및 반환 값 범위에서 선택할 수 있습니다.

아래 표는 지원되는 컨트롤러 메소드 인수를 보여줍니다.

Reactive Type (Reactor, RxJava 또는 기타)은 blocking I/O가 필요한 인수에서 지원됩니다. 해결할 요청 본문을 읽습니다. 설명 열에 표시됩니다. Blocking을 필요로하지 않는 인수에는 Reactive types이 필요하지 않습니다.

JDK 1.8의 java.util.Optional은 require attribute를 가진 annotation들을 메소드 argument로써 지원합니다.(ex: @RequestParam, @RequestHeader, etc, and is equivalent to required=false. )

| Controller method argument | Description |
|:--------|:--------|
| ServerWebExchange | 전체 ServerWebExchange에 대한 접근 - HTTP request 와 response, request 와 session attribute, checkNotModified 메소드 등을 위한 컨테이너. |
| ServerHttpRequest, ServerHttpResponse | HTTP request or response 에 대한 접근 |
| WebSession | session으로의 접근; attribute가 새로 생성되지 않는 한, 새로운 세션이 시작되지 않습니다.(Reactive type 지원) |
| java.security.Principal | 현재 인증된 사용자; 아마도 특정 Principal의 구현체(Reactive type 지원) |
| org.springframework.http.HttpMethod | request의 HTTP method |
| java.util.Locale | request의 현재 locale, 사용가능한 LocaleResolver에 의해 결정됨.|
| Java 6+: java.util.TimeZone
Java 8+: java.time.ZoneId | request와 관련된 timezone, LocaleContextResolver에 의해 결정 |
| @PathVariable | URI 템플릿 변수에 접근하기 위한 인수 |
| @MatrixVariable | URI path segments의 name-value pair에 접근하기 위한 인수 |
| @RequestParam | Servlet request parameters에 접근하기 위한 인수 |
| @RequestHeader | request의 header에 접근하기 위한 인수 |
| @RequestBody | requset의 body에 접근하기 위한 인수  |
| HttpEntity<B> | request의 header와 body에 접근하기 위한 인수(body는 HttpMessageReader에 의해 변환됨) |
| @RequestPart | multipart/form-data request에 접근하기 위한 인수 |
| java.util.Map, org.springframework.ui.Model, org.springframework.ui.ModelMap | web view로 노출되는 암시적 모델에 업데이트 혹은 접근하기 위한 인수 |
| Command or form object (with optional @ModelAttribute) | request parameter에 바인드할 속성이 있는 Command 객체|
| Errors, BindingResult | command/form 객체 데이터 바인딩에 대한 Validation 검사 결과 |
| SessionStatus | class-level 의 @SessionAttributes annotation을 통해 session attribute의 cleanup을 트리거하는 form 처리의 완료를 마킹 |
| UriComponentsBuilder | 현재 request의 host, prot, scheme, context path 와 servlet 매핑을 준비하기 위한 인수. Forwarded 및 X-Forwarded- * 헤더도 고려합니다. |
| @SessionAttribute | 모든 session attribute 에 접근하기 위한 인수;  |
| @RequestAttribute | request attributes에 접근하기 위한 인수 |




# Return values
아래 표는 지원되는 컨트롤러 메서드 반환 값을 보여줍니다. 반응 형 - 모든 반환 값에 대해 Reactor, RxJava 또는 기타가 지원됩니다.

| Controller method return value | Description |
|:--------|:--------|
| @ResponseBody | 반환 값은 HttpMessageWriters를 통해 인코딩되고 응답에 기록됩니다. |
| HttpEntity<B>, ResponseEntity<B> | 반환 값은 HTTP 헤더 및 본문을 포함하는 전체 응답을 지정하며 HttpMessageWriters를 통해 인코딩되고 응답에 기록됩니다. |
| HttpHeaders | header는 포함되고 body는 없는 응답 |
| String | ViewResolver로 resolve되며 암시적 모델과 함께 사용되는 뷰의 이름 |
| View | 암시 적 모델과 함께 렌더링 할 View 인스턴스 |
| java.util.Map, org.springframework.ui.Model | view name과 함께 암시적 모델로 추가되는 attributes(암시 적으로 요청 경로에서 결정) |
| Rendering | 모델 및 view rendering 시나리오를 위한 API |
| void | void를 가지는 메소드, 비동기 가능, return type (또는 반환 값 null)은 ServerHttpResponse 또는 ServerWebExchange 인수 또는 @ResponseStatus annotation이 있는 경우 응답을 완전히 처리 한 것으로 간주됩니다. |
| Flux<ServerSentEvent>, Observable<ServerSentEvent>, or other reactive type | 서버가 보낸 이벤트 방출 |
| Any other return type | RequestToViewNameTranslator를 통해 암시 적으로 결정된 뷰 이름을 사용하여 암시 적 모델에 추가 할 단일 모델 속성 |

## 1.5. Functional Endpoints
Spring WebFlux는 기능을 사용하여 요청을 라우팅하고 처리하고 약속이 변경되지 않도록 설계된 경량의 기능 프로그래밍 모델을 제공합니다. Annotation 기반 프로그래밍 모델의 대안이지만 동일한 Reactive Spring Web 기반에서 실행됩니다.

# 1.5.1. HandlerFunction
들어오는 HTTP 요청은 본질적으로 ServerRequest를 사용하고 Mono <ServerResponse>를 반환하는 함수 인 HandlerFunction에 의해 처리됩니다. 핸들러 함수에 대응되는 Annotation은 @RequestMapping 메소드입니다. 

ServerRequest 및 ServerResponse는, reactive Stream의 Non-Blocking backpressure으로, 기본이되는 HTTP 메세지의 친숙한 액세스를 제공하는 불변의 JDK-8 인터페이스입니다. 요청은 Reactor Flux  또는 Mono  유형으로 body 를 노출합니다. 응답은 본문으로 모든 Reactive Stream Publisher를 허용합니다 (Reactive Libraries 참고).

ServerRequest 는 다양한 HTTP request 요소를 제공합니다 : 메소드, URI, query params... 그리고 헤더들...
예를 들어 Request body를 Mono<String>으로 추출하는 방법은 다음과 같습니다.
```java
Mono<String> string = request.bodyToMono(String.class);
```

Body를 Flux로 추출하는 방법입니다. Person은 본문의 내용에서 deserialised 할 수있는 클래스입니다. 즉, 본문에 JSON이 있으면 Jackson이 지원하고 XML이면 JAXB를 지원합니다.
```java
Flux<Person> people = request.bodyToFlux(Person.class);
```

위의 bodyToMono 및 bodyToFlux는 사실상 일반적인 ServerRequest.body (BodyExtractor) 메서드를 사용하는 convenience 메서드입니다. BodyExtractor는 독자적인 추출 논리를 작성할 수있는 기능적 전략 인터페이스이지만 일반적인 BodyExtractor 인스턴스는 BodyExtractors 유틸리티 클래스에서 찾을 수 있습니다. 따라서 위의 예는 다음으로 대체 될 수 있습니다.
```java
Mono<String> string = request.body(BodyExtractors.toMono(String.class);
Flux<Person> people = request.body(BodyExtractors.toFlux(Person.class);
```

마찬가지로 ServerResponse는 HTTP 응답에 대한 액세스를 제공합니다. 불변이므로, Builder로 ServerResponse를 작성합니다. 빌더를 사용하면 응답 상태를 설정하고 응답 헤더를 추가하고 본문을 제공 할 수 있습니다. 예를 들어, 200 OK 상태, JSON 콘텐츠 유형 및 본문을 사용하여 응답을 만드는 방법입니다.
```java
Mono<Person> person = ...
ServerResponse.ok().contentType(MediaType.APPLICATION_JSON).body(person);
```

다음은 201 CREATED 상태, "Location"헤더 및 빈 본문을 사용하여 응답을 작성하는 방법입니다.
```java
URI location = ...
ServerResponse.created(location).build();
```

이 둘을 함께 사용하면 HandlerFunction을 만들 수 있습니다. 예를 들어, 200 상태의 응답과 String을 기반으로 한 본문을 반환하는 간단한 "Hello World"핸들러 lambda의 예제가 있습니다.
```java
HandlerFunction<ServerResponse> helloWorld =
  request -> ServerResponse.ok().body(fromObject("Hello World"));
```

위와 같이 람다처럼 핸들러 함수를 작성하는 것은 편리하지만, 여러 기능을 처리 할 때 가독성이 떨어지고 관리하기가 어려워 질 수 있습니다. 따라서 관련 핸들러 함수를 핸들러 또는 컨트롤러 클래스로 그룹화하는 것이 좋습니다. 예를 들면, 다음은 Reactive Person Repository 를 노출하는 클래스입니다.
```java
import static org.springframework.http.MediaType.APPLICATION_JSON;
import static org.springframework.web.reactive.function.BodyInserters.fromObject;

public class PersonHandler {

    private final PersonRepository repository;

    public PersonHandler(PersonRepository repository) {
        this.repository = repository;
    }

    public Mono<ServerResponse> listPeople(ServerRequest request) { 
        Flux<Person> people = repository.allPeople();
        return ServerResponse.ok().contentType(APPLICATION_JSON).body(people, Person.class);
    }

    public Mono<ServerResponse> createPerson(ServerRequest request) { 
        Mono<Person> person = request.bodyToMono(Person.class);
        return ServerResponse.ok().build(repository.savePerson(person));
    }

    public Mono<ServerResponse> getPerson(ServerRequest request) { 
        int personId = Integer.valueOf(request.pathVariable("id"));
        Mono<ServerResponse> notFound = ServerResponse.notFound().build();
        Mono<Person> personMono = this.repository.getPerson(personId);
        return personMono
                .flatMap(person -> ServerResponse.ok().contentType(APPLICATION_JSON).body(fromObject(person)))
                .switchIfEmpty(notFound);
    }
}
```
 - listPeople 함수는 저장소에있는 모든 Person 객체를 JSON으로 반환하는 핸들러 함수입니다.
 - createPerson 함수는 요청 본문에 포함 된 새 Person을 저장하는 핸들러 함수입니다. PersonRepository.savePerson(Person)은 Mono<Void>를 반환합니다. 빈 Mono는 사람이 요청에서 읽혀 저장 될 때 완료 신호를 내 보냅니다. 따라서 완성 신호가 수신되면, 즉 Person이 저장되었을 때 응답을 보내려면 build (Publisher<Void>) 메서드를 사용합니다.
  - getPerson 함수는 경로 변수 id를 통해 식별되는 단일 사용자를 리턴하는 핸들러 함수입니다. 저장소를 통해 Person을 검색하고 JSON 응답이 발견되면 작성합니다. 발견되지 않으면 switchIfEmpty(Mono <T>)를 사용하여 404 Not Found 응답을 반환합니다.


# 1.5.2. RouterFunction

들어오는 요청은 ServerRequest를 사용하고 Mono <HandlerFunction>를 반환하는 함수 인 RouterFunction을 사용하여 handler 함수로 라우팅됩니다. 요청이 특정 경로와 일치하면 핸들러 함수가 반환됩니다. 그렇지 않으면 빈 Mono를 반환합니다. RouterFunction은 @Controller 클래스의 @RequestMapping Annotation과 비슷한 목적을 가지고 있습니다.

일반적으로 라우터 함수는 직접 작성하지 않고 RouterFunctions.route (RequestPredicate, HandlerFunction)를 사용하여 요청 predicate 및 handler 함수를 사용하여 라우터 함수를 만듭니다. predicate가 적용되면 요청은 주어진 handler 함수로 라우트됩니다. 그렇지 않으면 라우팅이 수행되지 않으므로 404 Not Found 응답이 발생합니다. RequestPredicates 유틸리티 클래스는 경로, HTTP 메소드, 컨텐츠 유형 등을 기반으로하는 일치 검색과 같이 일반적으로 사용되는 조건자를 제공합니다. 경로를 사용하여 "Hello World"경로로 라우팅 할 수 있습니다. 
```java
RouterFunction<ServerResponse> helloWorldRoute =
    RouterFunctions.route(RequestPredicates.path("/hello-world"),
    request -> Response.ok().body(fromObject("Hello World")));
```

두개의 router function 은 새로운 하나의 라우터로 구성되어 각 기능에 라우팅 됩니다 : 만약 첫번째 route의 predicate가 매칭되지 않으면 두번째가 평가됩니다. 구성된 라우터 기능은 순서대로 평가되므로, 특별한 기능을 일반적인 것들보다 앞에 넣는것이 좋습니다. RouterFunction.and(RouterFunction), 나 RouterFunction.andRoute(RequestPredicate, HandlerFunction) 호출하면 되는데 이것들의 convenient combination 은  RouterFunction.and() with RouterFunctions.route() 입니다.

우리가 위에서 보여준 PersonHandler를 감안할 때, 이제는 각각의 핸들러 함수로 라우팅하는 라우터 함수를 정의 할 수 있습니다. 우리는 메소드 참조를 사용하여 핸들러 함수를 참조합니다.
```java
mport static org.springframework.http.MediaType.APPLICATION_JSON;
import static org.springframework.web.reactive.function.server.RequestPredicates.*;

PersonRepository repository = ...
PersonHandler handler = new PersonHandler(repository);

RouterFunction<ServerResponse> personRoute =
    route(GET("/person/{id}").and(accept(APPLICATION_JSON)), handler::getPerson)
        .andRoute(GET("/person").and(accept(APPLICATION_JSON)), handler::listPeople)
        .andRoute(POST("/person").and(contentType(APPLICATION_JSON)), handler::createPerson);
```

라우터 기능 외에도 RequestPredicate.and(RequestPredicate) 또는 RequestPredicate.or(RequestPredicate)를 호출하여 요청 Predicate를 구성할 수 있습니다. 두 Predicate가가 일치하면 결과 Predicate가 and 에 대해 예상대로 작동합니다 or는 Predicate가 하나라도 일치하면 일치합니다. RequestPredicates에있는 대부분의 Predicate는 합성입니다. 예를 들어, RequestPredicates.GET(String)은 RequestPredicates.method(HttpMethod)와 RequestPredicates.path(String)의 조합입니다.

# 1.5.3. Running a server
HTTP 서버에서 라우터 기능을 어떻게 실행합니까? 간단한 옵션은 RouterFunctions.toHttpHandler(RouterFunction)를 통해 라우터 함수를 HttpHandler로 변환하는 것입니다. 그런 다음 HttpHandler를 여러 서버 어댑터와 함께 사용할 수 있습니다.

DispatcherHandler 설정을 Annotation 컨트롤러와 나란히 실행할 수도 있습니다. 가장 쉬운 방법은 라우터 및 handler 기능을 사용하여 요청을 처리하는 데 필요한 구성을 만드는 WebFlux Java Config를 사용하는 것입니다.

# 1.5.4. HandlerFilterFunction
라우터 함수에 의해 매핑 된 경로는 RouterFunction.filter(HandlerFilterFunction)를 호출하여 필터링 할 수 있습니다. HandlerFilterFunction은 본질적으로 ServerRequest 및 HandlerFunction을 사용하는 함수이며 ServerResponse를 반환합니다. 핸들러 함수 매개 변수는 체인의 다음 요소를 나타내며 일반적으로 라우팅되는 HandlerFunction이지만 여러 필터가 적용될 경우 다른 FilterFunction이 될 수도 있습니다. Annotation을 사용하면 @ControllerAdvice 및 / 또는 ServletFilter를 사용하여 유사한 기능을 구현할 수 있습니다. 우리의 경로에 간단한 보안 필터를 추가해 봅시다. 특정 경로가 허용되는지 여부를 결정할 수있는 SecurityManager가 있다고 가정합니다.
```java
import static org.springframework.http.HttpStatus.UNAUTHORIZED;

SecurityManager securityManager = ...
RouterFunction<ServerResponse> route = ...

RouterFunction<ServerResponse> filteredRoute =
    route.filter(request, next) -> {
        if (securityManager.allowAccessTo(request.path())) {
            return next.handle(request);
        }
        else {
            return ServerResponse.status(UNAUTHORIZED).build();
        }
  });
```

이 예제에서 next.handle(ServerRequest) 호출은 선택적입니다. 액세스가 허용 될 때만 핸들러 함수가 실행되도록 허용합니다.

> functional 엔드 포인트에 대한 CORS 지원은 전용 CorsWebFilter를 통해 제공됩니다.

## 1.6. WebFlux Java Config
WebFlux Java 구성은 대부분의 응용 프로그램에 적합한 기본 구성을 제공하며 이를 구성하는 구성 API를 제공합니다. 구성 API에서는 사용할 수 없는 고급 사용자 정의를 보려면 고급 구성 모드를 참조하십시오.

Java 구성에 의해 만들어진 기본 bean을 이해할 필요는 없지만 WebFluxConfigurationSupport에서 쉽게 볼 수 있습니다. 자세한 내용은 특수 bean 유형을 참조하십시오.


# 1.6.1. Enable WebFlux config
Java 구성에서 @EnableWebFlux 주석을 사용하십시오.
```java
@Configuration
@EnableWebFlux
public class WebConfig {
}
```

위의 코드는 JSON, XML 등의 클래스 패스에서 사용할 수있는 의존성에 적응하는 여러 Spring WebFlux 인프라 bean을 등록합니다.


# 1.6.2. WebFlux config API
Java 구성에서 WebFluxConfigurer 인터페이스를 구현하십시오.
```java
@Configuration
@EnableWebFlux
public class WebConfig implements WebFluxConfigurer {

    // Implement configuration methods...

}
```

# 1.6.3. Conversion, formatting
@NumberFormat 및 @DateTimeFormat 주석에 대한 지원을 포함하여 기본적으로 Number 및 Date 유형의 포맷터가 설치됩니다. Joda Time이 클래스 경로에 있는 경우 Joda Time 형식 라이브러리에 대한 완벽한 지원도 설치됩니다.

사용자 정의 converters 및 formatters 를 등록하려면 다음을 수행하십시오.
```java
@Configuration
@EnableWebFlux
public class WebConfig implements WebFluxConfigurer {

    @Override
    public void addFormatters(FormatterRegistry registry) {
        // ...
    }

}
```

> FormatterRegistrars의 사용시기에 대한 자세한 내용은 FormatterRegistrar SPI 및 FormattingConversionServiceFactoryBean을 참조하십시오.

# 1.6.4. Validation
Bean 유효성 검사(ex : Hibernate Validator)가 클래스 경로에 있는 경우 기본적으로 LocalValidatorFactoryBean은 @Valid와 함께 사용하기위한 전역 Validator로 등록되고 @Controller 메소드 인수에서 유효성 검사를받습니다.

Java 구성에서 전역 Validator instance를 커스터마이징 할 수 있습니다.
```java
@Configuration
@EnableWebFlux
public class WebConfig implements WebFluxConfigurer {

    @Override
    public Validator getValidator(); {
        // ...
    }

}
```

Validator를 로컬에 등록 할 수도 있습니다.
```java
@Controller
public class MyController {

    @InitBinder
    protected void initBinder(WebDataBinder binder) {
        binder.addValidators(new FooValidator());
    }

}
```

> 어딘가에 LocalValidatorFactoryBean을 삽입해야한다면, MVC 설정에서 선언 된 것과 충돌을 피하기 위해 빈을 만들어 @Primary로 표시하십시오.

# 1.6.5. Content type resolvers
Spring WebFlux가 요청에서 @Controller에 대해 요청 된 미디어 유형을 결정하는 방법을 구성 할 수 있습니다. 기본적으로 "Accept"헤더만 선택되어 있지만 query parameter 기반 전략을 사용하도록 설정할 수도 있습니다.
```java
@Configuration
@EnableWebFlux
public class WebConfig implements WebFluxConfigurer {

    @Override
    public void configureContentTypeResolver(RequestedContentTypeResolverBuilder builder) {
        // ...
    }
}
```

# 1.6.6. HTTP message codecs
요청 및 응답 본문을 읽고 쓰는 방법을 사용자 정의하려면 다음을 수행하십시오.
```java
@Configuration
@EnableWebFlux
public class WebConfig implements WebFluxConfigurer {

    @Override
    public void configureHttpMessageCodecs(ServerCodecConfigurer configurer) {
        // ...
    }
}
```

ServerCodecConfigurer는 일련의 readers 와 writers 를 제공합니다. 이 도구를 사용하여 더 많은 readers 와 writers를 추가하거나, default값을 커스터마이징 하거나 교체할 수 있습니다.

Jackson JSON 및 XML의 경우 Jackson의 기본 속성을 다음과 같이 사용자 지정하는 Jackson2ObjectMapperBuilder를 사용하는 것이 좋습니다.
 - DeserializationFeature.FAIL_ON_UNKNOWN_PROPERTIES 를 사용할 수 없습니다.
 - MapperFeature.DEFAULT_VIEW_INCLUSION 를 사용할 수 없습니다.

또한 클래스 패스에서 다음과 같은 잘 알려진 모듈이 감지되면 자동으로 등록됩니다.
 - jackson-datatype-jdk7: java.nio.file.Path와 같은 Java 7 유형 지원.
 - jackson-datatype-joda: Joda-Time 유형 지원.
 - jackson-datatype-jsr310: Java 8 Date & Time API 유형 지원.
 - jackson-datatype-jdk8: 선택 사항과 같은 다른 Java 8 유형 지원.

# 1.6.7. View resolvers
View resolution 를 구성하려면 다음과 같이하십시오. 
```java
@Configuration
@EnableWebFlux
public class WebConfig implements WebFluxConfigurer {

    @Override
    public void configureViewResolvers(ViewResolverRegistry registry) {
        // ...
    }
}
```
FreeMarker는 또한 기본 뷰 기술을 구성해야합니다.
```java
@Configuration
@EnableWebFlux
public class WebConfig implements WebFluxConfigurer {

    // ...

    @Bean
    public FreeMarkerConfigurer freeMarkerConfigurer() {
        FreeMarkerConfigurer configurer = new FreeMarkerConfigurer();
        configurer.setTemplateLoaderPath("classpath:/templates");
        return configurer;
    }

}
```

# 1.6.8. Static resources
이 옵션은  Resource-based locations에서 static resources를 제공하는 편리한 방법을 제공합니다.

아래 예제에서 "/resources"로 시작하는 요청이 있으면 클래스 경로의 "/static"을 기준으로 정적 리소스를 찾고 제공하는 데 상대 경로가 사용됩니다. 브라우저 캐시를 최대로 사용하고 브라우저에서 HTTP 요청을 줄이기 위해 리소스는 1 년 후 만료됩니다. 브라우저 캐시를 최대로 사용하고 브라우저에서 HTTP 요청을 줄이기 위해 리소스는 1 년 후 만료됩니다.
```java
@Configuration
@EnableWebFlux
public class WebConfig implements WebFluxConfigurer {

    @Override
    public void addResourceHandlers(ResourceHandlerRegistry registry) {
        registry.addResourceHandler("/resources/**")
            .addResourceLocations("/public", "classpath:/static/")
            .setCachePeriod(31556926);
    }

}
```

또한 리소스 핸들러는 최적화 된 리소스로 작업하기위한 툴 체인을 만드는 데 사용할 수있는 ResourceResolvers 및 ResourceTransformers 체인을 지원합니다.

VersionResourceResolver는 고정된 어플리케이션 버전이나, MD5 해시 기반의 버저닝된 resource Urls 등등에 사용할 수 있습니다. ContentVersionStrategy (MD5 해시)는 모듈 로더와 함께 사용되는 JavaScript 같은 주목할 만한 예외가 있는 좋은 선택입니다. 

예를 들어 자바 설정에서;
```java
@Configuration
@EnableWebFlux
public class WebConfig implements WebFluxConfigurer {

    @Override
    public void addResourceHandlers(ResourceHandlerRegistry registry) {
        registry.addResourceHandler("/resources/**")
                .addResourceLocations("/public/")
                .resourceChain(true)
                .addResolver(new VersionResourceResolver().addContentVersionStrategy("/**"));
    }

}
```
ResourceUrlProvider를 사용하여 URL을 다시 작성하고 resolvers and transformers의 전체 체인을 적용 할 수 있습니다. (예를 들면 버전을 삽입하기 위해) WebFlux config는 ResourceUrlProvider를 제공하여 다른 컴포넌트에 주입 할 수 있습니다.

Spring MVC와는 달리 현재 WebFlux의 정적 리소스 URL을 투명하게 재 작성 할 수있는 방법은 없습니다. 왜냐하면 non-blocking 체인의 resolvers and transformers 를 사용할 수있는 뷰 기술이 없기 때문입니다. 로컬 리소스 만 제공하는 경우 해결 방법은 ResourceUrlProvider를 직접 (예 : 사용자 정의 태그를 통해) 사용하고 0 초 동안 차단하는 것입니다.

WebJars는 WebJarsResourceResolver를 통해 지원되며 "org.webjars : webjars-locator"가 클래스 경로에 있을 때 자동으로 등록됩니다. resolver는 jar 버전을 포함하도록 URL을 다시 쓸 수 있으며 버전이 없는 수신 URL에도 일치시킬 수 있습니다 (예 : "/jquery/jquery.min.js"를 "/jquery/1.2.0/jquery.min.js"로 변경하십시오.


# 1.6.9. Path Matching
Spring WebFlux는 경로 패턴, 즉 PathPattern과 들어오는 RequestPath의 구문 분석 된 표현을 사용합니다.

Spring WebFlux는 접미사 패턴 매칭도 지원하지 않으므로 경로 매칭과 관련하여 커스터마이징 할 수있는 두 가지 사소한 옵션만 있습니다. (trailing slashes(true by default)와 case-sensitive(false))

해당 옵션을 커스터마이징하려면 다음을 수행하십시오.
```java
@Configuration
@EnableWebFlux
public class WebConfig implements WebFluxConfigurer {

    @Override
    public void configurePathMatch(PathMatchConfigurer configurer) {
        // ...
    }

}
```

# 1.6.10. Advanced config mode
@EnableWebFlux는 (1) WebFlux 응용 프로그램에 대한 기본 Spring 구성을 제공하고 (2) WebFluxConfigurer를 감지하고 위임하여 해당 구성을 사용자 지정하는 DelegatingWebFluxConfiguration을 가져옵니다.

고급 모드의 경우 @EnableWebFlux를 제거하고 WebFluxConfigurer를 구현하는 대신 DelegatingWebFluxConfiguration에서 직접 확장합니다.
```java
@Configuration
public class WebConfig extends DelegatingWebFluxConfiguration {

    // ...

}
```

WebConfig에서 기존 메소드를 유지할 수 있지만 이제는 기본 클래스에서 bean 선언을 겹쳐 쓸 수 있으며 여전히 클래스 경로에 다른 WebMvcConfigurer를 얼마든지 가질 수 있습니다.

## 1.7. CORS
SKIP

### 2. WebClient
spring-webflux 모듈에는 Reactive Streams backpressure가 있는 HTTP 요청에 대한 non-blocking, reactive 클라이언트가 포함되어 있습니다. 이것들은 HTTP 코덱 및 기타 인프라를 서버 기능 웹 프레임 워크와 공유합니다.

WebClient는 HTTP 클라이언트 라이브러리보다 높은 수준의 API를 제공합니다. 기본적으로 Reactor Netty를 사용하지만 다른 ClientHttpConnector를 사용하여 플러그 할 수 있습니다. WebClient API는 출력용 Reactor Flux 또는 Mono를 반환하고 Reactive Stream Publisher를 입력으로 허용합니다 (Reactive Libraries 참고).

> RestTemplate과 비교하여 WebClient는 Java 8 람다를 최대한 활용하는 보다 기능적이고 풍부한 API를 제공합니다. 스트리밍을 비롯한 동기화 및 비동기 시나리오를 모두 지원하고 non-blocking I/O의 효율성을 제공합니다.

## 2.1. Retrieve
retrieve() 메소드는 응답 본문을 가져 와서 디코드하는 가장 쉬운 방법입니다.
```java
    WebClient client = WebClient.create("http://example.org");

    Mono<Person> result = client.get()
            .uri("/persons/{id}", id).accept(MediaType.APPLICATION_JSON)
            .retrieve()
            .bodyToMono(Person.class);
```
응답에서 디코딩 된 객체 스트림을 얻을 수도 있습니다.
```java
    Flux<Quote> result = client.get()
            .uri("/quotes").accept(MediaType.TEXT_EVENT_STREAM)
            .retrieve()
            .bodyToFlux(Quote.class);
```

기본적으로 4xx 또는 5xx 상태 코드로 응답하면 WebClientResponseException 유형의 오류가 발생하지만 사용자가 커스터마이징 할 수 있습니다.
```java
    Mono<Person> result = client.get()
            .uri("/persons/{id}", id).accept(MediaType.APPLICATION_JSON)
            .retrieve()
            .onStatus(HttpStatus::is4xxServerError, response -> ...)
            .onStatus(HttpStatus::is5xxServerError, response -> ...)
            .bodyToMono(Person.class);
```

## 2.2. Exchange
exchange() 메소드는보다 많은 제어를 제공합니다. 아래 예제는 retrieve()와 동일하지만 ClientResponse에 대한 액세스도 제공합니다.
```java
    Mono<Person> result = client.get()
            .uri("/persons/{id}", id).accept(MediaType.APPLICATION_JSON)
            .exchange()
            .flatMap(response -> response.bodyToMono(Person.class));
```
이 레벨에서 전체 ResponseEntity를 만들 수도 있습니다.
```java
    Mono<ResponseEntity<Person>> result = client.get()
            .uri("/persons/{id}", id).accept(MediaType.APPLICATION_JSON)
            .exchange()
            .flatMap(response -> response.toEntity(Person.class));
```

retrieve()와는 달리, exchange()에는 4xx 및 5xx 응답에 대한 자동 오류 신호가 없습니다. 상태 코드를 확인하고 진행 방법을 결정해야합니다.


> exchange()를 사용할 때는 리소스가 해제되고 HTTP 연결 풀링으로 인한 잠재적 문제를 피하기 위해 항상 ClientResponse의 body 또는 toEntity 메서드를 사용해야합니다. 응답 내용이 없으면 bodyToMono (Void.class)를 사용할 수 있습니다. 그러나 응답에 콘텐츠가있는 경우 연결이 닫히고 다시 풀에 배치되지 않습니다.

## 2.3. Request body
요청 본문은 Object에서 인코딩 할 수 있습니다.
```java
    Mono<Person> personMono = ... ;

    Mono<Void> result = client.post()
            .uri("/persons/{id}", id)
            .contentType(MediaType.APPLICATION_JSON)
            .body(personMono, Person.class)
            .retrieve()
            .bodyToMono(Void.class);
```
인코딩 된 객체 스트림을 가질 수도 있습니다.
```java
    Flux<Person> personFlux = ... ;

    Mono<Void> result = client.post()
            .uri("/persons/{id}", id)
            .contentType(MediaType.APPLICATION_STREAM_JSON)
            .body(personFlux, Person.class)
            .retrieve()
            .bodyToMono(Void.class);
```
또는 실제 값이 있으면 syncBody 바로 가기 메서드를 사용하십시오.
```java
    Person person = ... ;

    Mono<Void> result = client.post()
            .uri("/persons/{id}", id)
            .contentType(MediaType.APPLICATION_JSON)
            .syncBody(person)
            .retrieve()
            .bodyToMono(Void.class);
```

# 2.3.1. Form data
Form 데이터를 보내려면 body로 MultiValueMap <String, String>을 제공해야합니다. 내용은 FormHttpMessageWriter에 의해 자동으로 "application/x-www-form-urlencoded"로 설정됩니다.
```java
    MultiValueMap<String, String> formData = ... ;

    Mono<Void> result = client.post()
            .uri("/path", id)
            .syncBody(formData)
            .retrieve()
            .bodyToMono(Void.class);
```

BodyInserters를 통해 양식 데이터를 인라인으로 제공 할 수도 있습니다.
```java
    import static org.springframework.web.reactive.function.BodyInserters.*;

    Mono<Void> result = client.post()
            .uri("/path", id)
            .body(fromFormData("k1", "v1").with("k2", "v2"))
            .retrieve()
            .bodyToMono(Void.class);

```

# 2.3.2. Multipart data
multipart 데이터를 보내려면 MultiValueMap <String,?>을 제공해야 합니다. 여기서 value는 part body를 나타내는 Object 또는 part body와 header를 나타내는 HttpEntity입니다. MultipartBodyBuilder를 사용하여 part를 빌드 할 수 있습니다.
```java
    MultipartBodyBuilder builder = new MultipartBodyBuilder();
    builder.part("fieldPart", "fieldValue");
    builder.part("filePart", new FileSystemResource("...logo.png"));
    builder.part("jsonPart", new Person("Jason"));

    MultiValueMap<String, HttpEntity<?>> parts = builder.build();

    Mono<Void> result = client.post()
            .uri("/path", id)
            .syncBody(parts)
            .retrieve()
            .bodyToMono(Void.class);
```

각 part의 content type은 기록되는 파일의 확장자 또는 객체의 유형에 따라 자동으로 설정됩니다. 만약 원한다면 보다 명확하게 명시하고 각 파트의 내용 유형을 지정할 수도 있습니다.

BodyInserters를 통해 multipart 데이터를 인라인으로 제공 할 수도 있습니다.
```java
    import static org.springframework.web.reactive.function.BodyInserters.*;

    Mono<Void> result = client.post()
            .uri("/path", id)
            .body(fromMultipartData("fieldPart", "value").with("filePart", resource))
            .retrieve()
            .bodyToMono(Void.class);
```

## 2.4. Builder options
WebClient를 작성하는 간단한 방법은 모든 요청에 ​​대한 기본 URL로 정적 팩토리 메소드 인 create () 및 create (String)를 사용하는 것입니다. WebClient.builder()를 사용하여 더 많은 옵션에 액세스 할 수도 있습니다.
```java
    SslContext sslContext = ...

    ClientHttpConnector connector = new ReactorClientHttpConnector(
            builder -> builder.sslContext(sslContext));

    WebClient webClient = WebClient.builder()
            .clientConnector(connector)
            .build();
```
HTTP codecs를 커스터마이징하기위해 HTTP messages 를 encoding혹은 decoding하여 사용합니다.
```java
    ExchangeStrategies strategies = ExchangeStrategies.builder()
            .codecs(configurer -> {
                // ...
            })
            .build();

    WebClient webClient = WebClient.builder()
            .exchangeStrategies(strategies)
            .build();
```

Builder는 Filters를 넣기 위해 사용되기도 합니다.

IDE에서 WebClient.Builder를 탐색하여 URI 작성, 기본 헤더 (및 쿠키) 등과 관련된 기타 옵션을 확인하십시오.

WebClient가 빌드 된 후에는 현재 인스턴스에 영향을주지 않고 새로운 WebClient를 빌드하기 위해 항상 새 빌더를 얻을 수 있습니다.
```java
    WebClient modifiedClient = client.mutate()
            // user builder methods...
            .build();
```

## 2.5. Filters
WebClient는 interception 스타일의 request filtering을 제공합니다.
```java
    WebClient client = WebClient.builder()
            .filter((request, next) -> {
                ClientRequest filtered = ClientRequest.from(request)
                        .header("foo", "bar")
                        .build();
                return next.exchange(filtered);
            })
            .build();
```

ExchangeFilterFunctions는 기본 인증을위한 필터를 제공합니다.
```java
    // static import of ExchangeFilterFunctions.basicAuthentication

    WebClient client = WebClient.builder()
            .filter(basicAuthentication("user", "pwd"))
            .build();
```

또한, 원본에 영향을주지 않고 기존 WebClient 인스턴스를 변형 할 수도 있습니다.
```java
    WebClient filteredClient = client.mutate()
            .filter(basicAuthentication("user", "pwd")
            .build();
```


### 3. WebSockets
### 4. Testing
### 5. Reactive Libraries


https://docs.spring.io/spring/docs/current/spring-framework-reference/web-reactive.html#webflux-config-enable