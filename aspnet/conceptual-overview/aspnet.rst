Introduction to ASP.NET Core
============================

By `Daniel Roth`_

ASP.NET Core is a significant redesign of ASP.NET. This topic introduces the new concepts in ASP.NET Core and explains how they help you develop modern web apps.

.. contents:: Sections:
  :local:
  :depth: 1

ASP.NET Core 소개
============================

By `Daniel Roth`_

ASP.NET Core를 설계 관점에서 한 마디로 설명하자면 ASP.NET의 심도깊은 재설계(significant redesign)라고 할 수 있다. 이 글은 ASP.NET Core의 새로운 개념을 소개하고 그 것들이 어떻게 모던 웹앱을 개발하는데 도움이 되는지 설명한다.

What is ASP.NET Core?
---------------------

ASP.NET Core is a new open-source and cross-platform framework for building modern cloud-based Web applications using .NET. We built it from the ground up to provide an optimized development framework for apps that are either deployed to the cloud or run on-premises. It consists of modular components with minimal overhead, so you retain flexibility while constructing your solutions. You can develop and run your ASP.NET Core applications cross-platform on Windows, Mac and Linux. ASP.NET Core is fully open source on `GitHub <https://github.com/aspnet/home>`_.

ASP.NET Core란 무엇인가?
---------------------

ASP.NET Core는 닷넷을 사용하여 클라우드 기반의 모던 웹 애플리케이션을 만들기 위한 새로운 오픈소스이자 크로스플랫폼 프레임워크다. 우리는 클라우드에 배포되거나 자체 호스팅 서버(on-premise)에서 동작하는 앱을 위한 최적의 개발 프레임워크를 제공하기 위해 프레임워크를 바닥부터 다시 만들었다. ASP.NET Core는 최소한의 오버헤드만을 갖도록 모듈화된 컴포넌트들로 구성되어 있기 때문에 솔루션을 구성하면서 유연성을 지킬 수 있다. 또한, ASP.NET 5 애플리이션을 윈도우, 맥, 리눅스에서 운영할 수 있다. ASP.NET Core는 GitHub 의 오픈 소스이다.

Why build ASP.NET Core?
-----------------------

The first preview release of ASP.NET came out almost 15 years ago as part of the .NET Framework.  Since then millions of developers have used it to build and run great web applications, and over the years we have added and evolved many, many capabilities to it.

왜 ASP.NET Core를 만들었는가?
---------------------------

ASP.NET 1.O의 최초 프리뷰가 나온 것이 어느 덧 15년 전의 일이다. 그 이래로 정말 많은 개발자들이 ASP.NET을 사용하여 훌륭한 웹 애플리케이션을 만들어 왔고 우리는 그 동안 수 많은 기능들을 추가하고 발전시켜왔다.

With ASP.NET Core we are making a number of architectural changes that make the core web framework much leaner and more modular. ASP.NET Core is no longer based on System.Web.dll, but is instead based on a set of granular and well factored NuGet packages allowing you to optimize your app to have just what you need. You can reduce the surface area of your application to improve security, reduce your servicing burden and also to improve performance in a true pay-for-what-you-use model.

ASP.NET Core에서 우리는 상당히 많은 아키텍쳐 변경을 통해 군더더기 없이 모듈화된 코어(core) 웹 프레임워크를 만들고 있다. ASP.NET Core는 System.Web.dll 에 더이상 기반하지 않고, 잘게 분리된 NuGet 패키지들에 기반해서 여러분의 요구에 최적화된 앱을 만들 수 있게 해준다. 애플리케이션이 필요 이상의 모듈을 포함하지 않기 때문에 보안상 개선 효과가 있고, 서비스 하는 부담 또한 줄여준다. 결국, 사용한만큼 지불하는(pay-for-what-you-use) 모델을 채택함으로써 애플리케이션의 성능이 개선된다.

ASP.NET Core is built with the needs of modern Web applications in mind, including a unified story for building Web UI and Web APIs that integrate with today's modern client-side frameworks and development workflows. ASP.NET Core is also built to be cloud-ready by introducing environment-based configuration and by providing built-in dependency injection support.

ASP.NET Core는 모던 웹 애플리케이션에 대한 필요를 염두에 두고 만들어졌다. 웹 사용자 인터페이스를 만드는 것, 클라이언트 측 프레임워크와 통합되는 Web API들을 만드는 것, 이런 개발 작업의 흐름이 통일된 하나의 스토리가 된다. ASP.NET Core는 또한 클라우드 준비된(cloud-ready) 프레임워크라고 할 수 있다. 환경 기반의 구성(environment-based configuration)이 가능하고, 빌트-인 종속성 주입(dependency injection)을 제공하기 때문이다.

To appeal to a broader audience of developers, ASP.NET Core supports cross-platform development on Windows, Mac and Linux. The entire ASP.NET Core stack is open source and encourages community contributions and engagement. ASP.NET Core comes with a new, agile project system in Visual Studio while also providing a complete command-line interface so that you can develop using the tools of your choice.

보다 다양한 개발자들의 주목을 끌기 위해, ASP.NET Core는 윈도우, 맥, 리눅스에서 크로스플랫폼 개발을 지원한다. 전체 ASP.NET Core 스택은 오픈 소스이고 커뮤니티의 기여와 참여를 장려하고 있다. 비주얼 스튜디오는 완벽한 커맨드-라인 인터페이스를 지원하면서 새롭고 민첩한 프로젝트 시스템을 도입하여 여러분이 원하는 도구를 사용해서 개발할 수 있다. (역주: 비주얼 스튜디오에 대한 설명이 다소 추상적인데 bower, npm 등을 사용할 수 있다는 느낌으로 해석할 수 있겠다)

In summary, with ASP.NET Core you gain the following foundational improvements:

- New light-weight and modular HTTP request pipeline
- Ability to host on IIS or self-host in your own process
- Built on `.NET Core`_, which supports true side-by-side app versioning
- Ships entirely as NuGet packages
- Integrated support for creating and using NuGet packages
- Single aligned web stack for Web UI and Web APIs
- Cloud-ready environment-based configuration
- Built-in support for dependency injection
- New tooling that simplifies modern web development
- Build and run cross-platform ASP.NET apps on Windows, Mac and Linux
- Open source and community focused

정리하자면, ASP.NET Core에서는 다음과 같이 기초적으로 중대한 개선사항이 포함되었다.

* 새롭게 경량화되고 모듈화된 HTTP 요청 파이프라인
* IIS 또는 개발자 자신의 프로세스에서 셀프 호스트할 수 있는 능력
* 닷넷 코어에 기반한 진정한 sidy-by-side 앱 버전 관리(versioning)
* 모든 기능이 NuGet 패키지 형태로 추가
* NuGet 패키지들을 생성하고 사용하는 것에 대한 통합된 지원
* 웹 사용자 인터페이스와 웹 API를 위한 단일 웹 스택
* 클라우드를 위한 환경 기반 구성
* 내장된 종속성 주입 기능
* 모던 웹 개발을 단순화 시킨 새로운 도구들(tooling)
* 윈도우, 맥, 리눅스에서 개발하고 실행할 수 있는 크로스플랫폼
* 오픈 소스와 커뮤니티에 초점

Application anatomy
-------------------

ASP.NET Core applications are defined using a public ``Startup`` class:

애플리케이션 뜯어보기
------------------

ASP.NET 5 애플리케이션은 ``Startup`` 클래스를 사용하여 정의된다.

.. code-block:: c#

  public class Startup
  {
      public void ConfigureServices(IServiceCollection services)
      {
      }

      public void Configure(IApplicationBuilder app)
      {
      }
  }

The ``ConfigureServices`` method defines the services used by your application and the ``Configure`` method is used to define what middleware makes up your request pipeline. See :doc:`/fundamentals/startup` for more details.

``ConfigureServices`` 메서드는 애플리케이션에 사용될 서비스들을 정의하고 ``Configure`` 메서드는 요청 파이프라인을 구성할 미들웨어를 정의하는데 사용된다. 보다 자세한 사항은 :doc:`/fundamentals/startup` 아티클을 참조하자.

Services
--------

A service is a component that is intended for common consumption in an application. Services are made available through dependency injection. ASP.NET Core includes a simple built-in inversion of control (IoC) container that supports constructor injection by default, but can be easily replaced with your IoC container of choice. See :doc:`/fundamentals/dependency-injection` for more details.

서비스
-----

서비스는 애플리케이션에서 공통적으로 사용하고자 하는 목적의 컴포넌트다. 서비스들은 종속성 주입을 통해 사용이 가능하다. ASP.NET Core 는 간단한 내장(built-in) 제어 역전 (IoC) 컨테이너를 포함하고 있는데 생성자를 이용한 주입 방식을 기본으로 지원하고 있다. 그러나, 여러분이 사용하는 IoC 컨테이너로 쉽게 대체할 수도 있다. 자세한 내용은 :doc:`/fundamentals/dependency-injection` 아티클을 참조하자.

Services in ASP.NET Core come in three varieties: singleton, scoped and transient. Transient services are created each time they’re requested from the container. Scoped services are created only if they don’t already exist in the current scope. For Web applications, a container scope is created for each request, so you can think of scoped services as per request. Singleton services are only ever created once.

ASP.NET Core 에서 서비스는 세 가지 종류(singleton, scoped, transient)로 구분된다. Transient 서비스는 컨테이너로부터 요청될 때마다 생성되고 Scoped 서비스는 현재 scope에 서비스가 존재하지 않을 때만 서비스를 생성한다. 웹 애플리케이션에서 컨테이너의 범주는 매 요청에 해당하므로 scoped 서비스는 요청당 서비스로 생각할 수 있다. 반면, Singleton 서비스는 오직 한번만 생성된다.

Middleware
----------

In ASP.NET Core you compose your request pipeline using :doc:`/fundamentals/middleware`. ASP.NET Core middleware perform asynchronous logic on an ``HttpContext`` and then optionally  invoke the next middleware in the sequence or terminate the request directly. You generally "Use" middleware by invoking a corresponding extension method on the ``IApplicationBuilder`` in your ``Configure`` method.

미들웨어
-------

ASP.NET Core에서는 :doc:`/fundamentals/middleware` 를 사용하여 요청 파이프라인을 구성한다. ASP.NET Core 미들웨어는 ``HttpContext`` 에 대해 비동기 로직을 수행하고 선택적으로 다음 미들웨어를 호출하거나 요청 처리 작업을 중단한다. 일반적으로 미들웨어를 사용하는 방식은  ``Configure`` 메서드에서 ``IApplicationBuilder`` 의 "Use"로 시작하는 확장 메서드를 실행하는 것이다.

ASP.NET Core comes with a rich set of prebuilt middleware:

ASP.NET Core는 다음과 같이 사전 작성된 풍부한 미들웨어를 제공한다.

- :doc:`/fundamentals/static-files`
- :doc:`/fundamentals/routing`
- :doc:`/fundamentals/diagnostics`
- :doc:`/security/authentication/index`

You can also author your own :doc:`custom middleware </fundamentals/middleware>`.

여러분은 자신만의 :doc:`custom middleware </fundamentals/middleware>` 를 작성할 수도 있다.

You can use any `OWIN <http://owin.org>`_-based middleware with ASP.NET Core. See :doc:`/fundamentals/owin` for details.

또한 `OWIN <http://owin.org>`_ 기반의 미들웨어를 ASP.NET Core와 함께 사용할 수 있다. 자세한 내용은 :doc:`/fundamentals/owin` 아티클을 참조하자.


Servers
-------

The ASP.NET Core hosting model does not directly listen for requests, but instead relies on an HTTP :doc:`server </fundamentals/servers>` implementation to surface the request to the application as a set of feature interfaces that can be composed into an HttpContext. ASP.NET Core includes a managed cross-platform web server, called :ref:`Kestrel <kestrel>`, that you would typically run behind a production web server like `IIS <https://iis.net>`__ or `nginx <http://nginx.org>`__.

서버
----

ASP.NET Core 호스팅 모델은 요청을 직접 수신하지 않는 대신, HTTP :doc:`server </fundamentals/servers>` 구현에 의지하여 애플리케이션에 대한 요청을 (HttpContext 와 작용하는 기능적인 인터페이스들의 모음으로서) 표면화한다. ASP.NET Core는 :ref:`Kestrel <kestrel>` 라고 불리는 매니지드 크로스플랫폼 웹서버를 포함한다. Kestrel은 여러분이 통상적으로 운영환경의 웹서버로 사용하고자 하는 `IIS <https://iis.net>`__ 또는 `nginx <http://nginx.org>`__ 같은 것이다.

Web root
--------

The Web root of your application is the root location in your project from which HTTP requests are handled (ex. handling of static file requests). The Web root of an ASP.NET Core application is configured using the "webroot" property in your project.json file.

웹 루트
------

애플리케이션내의 웹 루트는 프로젝트에서 루트 위치이다. 이 곳으로부터 예를 들면, 정적 파일 요청 같은 HTTP 요청이 처리된다. ASP.NET Core 애플리케이션의 웹 루트는 project.json 파일의 "webroot" 속성을 사용하여 설정된다.

Configuration
-------------

ASP.NET Core uses a new configuration model for handling of simple name-value pairs that is not based on System.Configuration or web.config. This new configuration model pulls from an ordered set of configuration providers. The built-in configuration providers support a variety of file formats (XML, JSON, INI) and also environment variables to enable environment-based configuration. You can also write your own custom configuration providers. Environments, like Development and Production, are a first-class notion in ASP.NET Core and can also be set up using environment variables:

구성
----

ASP.NET 5는 간단한 이름-값 쌍을 다루는 새로운 구성모델을 사용하며 이 새 모델은 System.Configuration 또는 web.config 에 기반하지 않는다. 새 구성 모델은 구성 제공자(configuration provider)의 순차적인 집합에서 정보를 얻는다. 내장된 구성 제공자는 XML, JSON, INI 파일등 다양한 파일 형식을 지원하며 또한, 환경 기반의 구성을 가능케 하는 환경 변수도 지원한다. 여러분은 이와 더불어 사용자 정의 구성 제공자를 작성할 수도 있다. Development, Productions 처럼 환경 자체가 ASP.NET Core에서는 가장 우선시 되는 개념이고 환경 변수를 사용하여 설정된다.

.. literalinclude:: /../common/samples/WebApplication1/src/WebApplication1/Startup.cs
  :language: c#
  :lines: 22-34
  :dedent: 12

See :doc:`/fundamentals/configuration` for more details on the new configuration system and :doc:`/fundamentals/environments` for details on how to work with environments in ASP.NET Core.

새로운 구성모델에 대한 자세한 내용은 :doc:`/fundamentals/configuration` 아티클을 참조하고 ASP.NET Core 에서 환경과 관련된 작업에 대한 자세한 내용은 :doc:`/fundamentals/environments` 아티클을 참조하자.


Client-side development
-----------------------

ASP.NET Core is designed to integrate seamlessly with a variety of client-side frameworks, including :doc:`AngularJS </client-side/angular>`, :doc:`KnockoutJS </client-side/knockout>` and :doc:`Bootstrap </client-side/bootstrap>`. See :doc:`/client-side/index` for more details.

클라이언트단 개발
---------------

ASP.NET Core는 클라이언트 측 프레임워크와 매끄럽게 통합되도록 설계되었고 `AngularJS <https://angularjs.org/>`_ , `KnockoutJS <http://knockoutjs.com>`_ and `Bootstrap <http://getbootstrap.com/>`_ 같은 프레임워크를 포함한다. 이와 관련하여 자세한 내용은 :doc:`/client-side/index` 아티클을 참조하자.

