.. _application-startup:

어플리케이션 시작하기
===================

By `Steve Smith`_

ASP.NET Core 를 통해 여러분의 어플리케이션에서 처리해야 하는 각각의 요청을 완전하게 제어할 수 있습니다. ``Startup`` 클래스는 어플리케이션의 진입점으로서, 환경설정을 지정하고 어플리케이션에서 사용할 서비스에 연결하는 작업을 할 수 있는 클래스 입니다. 즉, 개발자들은 ``Startup`` 클래스 내에서 어플리케이션에 전달되는 모든 요청에 대한 처리경로를 설정할 수 있습니다.   

.. contents:: 단락:
  :local:
  :depth: 1

Startup 클래스
-----------------

ASP.NET Core 에서 ``Startup`` 클래스는 어플리케이션에 대한 진입점으로서, 모든 어플리케이션에 필수적입니다. 여러 환경에 맞춰 여러가지 시작용 클래스와 메서드를 사용할 수 있지만 (참고 :doc:`environments`), 어플리케이션의 진입점으로서는 하나의 ``Startup`` 클래스 만 사용할 수 있습니다. ASP.NET 은 (전체 네임스페이스에서) ``Startup`` 이라는 이름의 클래스를 포함하는 첫 번째 어셈블리를 찾아봅니다. `Hosting:Application` 이라는 설정 키로 특정한 어셈블리를 ``Startup`` 클래스를 찾아볼 어셈블리로 지정할 수도 있습니다. 그 클래스를 ``public`` 으로 정의했는지는 상관없습니다. ASP.NET 은 명명규칙에 합당한 이름이기만 하면 클래스를 읽어들일 수 있습니다. 여러 개의 ``Startup`` 클래스가 있다하더라도, 예외를 발생시키지 않습니다. ASP.NET 은 네임스페이스에서 유추하여 하나를 선택할 것입니다. (우선 프로젝트의 루트 네임스페이스와 비교해서 일치하는 것이 있다면 그 클래스를 선택합니다. 일치하는 것이 없다면, 네임스페이스를 알파벳순으로 나열하고 그 중 첫 번째 클래스를 선택합니다.)

``Startup`` 클래스의 생성자에서 :doc:`의존성 주입 <dependency-injection>` 을 사용하여 의존성을 선택적으로 지정할 수 있습니다. 일반적으로 어플리케이션을 설정하는 방식은 Startup 클래스의 생성자에서 정의합니다. (참고 :doc:`configuration`) 그 외에 Startup 클래스에서 ``Configure`` 메서드를 정의해야 하고, ``ConfigureServices`` 메서드를 정의할지는 선택할 수 있습니다. 이 두 가지 메서드는 어플리케이션이 시작될 때 호출됩니다. 

Configure 메서드
--------------------

``Configure`` 메서드를 통해 ASP.NET 어플리케이션이 각각의 HTTP 요청에 대해 어떻게 응답할지 지정할 수 있습니다. 가장 단순하게는, 모든 요청에 대해 동일한 응답을 하도록 설정할 수도 있습니다. 하지만, 대부분의 실제 어플리케이션에서는 이보다 많은 기능이 필요합니다. 더 복잡한 처리경로들에 대한 설정을 :doc:`미들웨어 <middleware>` 에 포함시킨 후, IApplicationBuilder_ 상의 확장 메서드를 통해 사용하도록 할 수 있습니다.

여러분의 ``Configure`` 메서드에서는 매개변수로 전달되는 IApplicationBuilder_ 서비스를 처리해야 합니다. 추가로 ``IHostingEnvironment`` 와 ``ILoggerFactory`` 와 같은 서비스도 처리해야 할 수 있습니다. 이런 서비스들을 사용할 수 있는 경우에 서버에서 전달합니다. 기본 웹 사이트 템플릿에서 가져온 다음의 예제에서는, 처리경로에서 `BrowserLink <http://www.asp.net/visual-studio/overview/2013/using-browser-link>`_와 에러 페이지들, 정적 파일들, ASP.NET MVC, Identity 를 사용하도록 하기 위해 몇몇 확장 메서드들을 사용하고 있습니다.

.. literalinclude:: /../common/samples/WebApplication1/src/WebApplication1/Startup.cs
  :language: c#
  :linenos:
  :lines: 58-86
  :dedent: 8
  :emphasize-lines: 8-10,14,17,19,23

각각의 ``Use`` 확장 메서드를 통해 요청 처리경로에 :doc:`미들웨어 <middleware>` 를 추가합니다. 예를 들어, ``UseMvc`` 확장 메서드를 통해 :doc:`라우팅 <routing>` 미들웨어를 요청 처리경로에 추가하고 :doc:`MVC </mvc/index>` 를 기본 처리자로 설정합니다.

여러분은 :doc:`middleware` 주제에서 미들웨어와 요청 처리경로를 정의하기 위해 IApplicationBuilder_ 를 사용하는 방법에 대해 알 수 있습니다.

ConfigureServices 메서드
----------------------------

여러분은 ``Startup`` 클래스에 어플리케이션에서 사용하는 서비스를 설정하기 위한 ``ConfigureServices`` 메서드를 선택적으로 포함시킬 수 있습니다. ``ConfigureServices`` 메서드는 ``Startup`` 클래스 상의 공개 메서드로서, IServiceCollection_ 인스턴스를 매개변수로 받고 ``IServiceProvider`` 를 선택적으로 반환합니다. ``ConfigureServices`` 는 ``Configure`` 보다 먼저 호출됩니다. 이는 중요한 부분으로서, ASP.NET MVC 같은 몇몇 기능은 ``ConfigureServices`` 메서드를 통해 특정 서비스를 추가한 뒤 요청 처리경로에 연결해야 합니다.

``Configure`` 와 마찬가지로, ``ConfigureServices`` 내의 우선적인 설정을 필요로 하는 기능들을 IServiceCollection_ 상의 확장 메서드로 분리하기를 권합니다. 기본 웹 사이트 템플릿에서 가져온 다음 예제에서는 몇몇 ``Add[Something]`` 확장 메서드를 통해 Entity 프레임워크와 Identity, MVC 의 서비스를 사용하도록 어플리케이션 설정하고 있습니다.

.. literalinclude:: /../common/samples/WebApplication1/src/WebApplication1/Startup.cs
  :language: c#
  :linenos:
  :lines: 40-55
  :dedent: 8
  :emphasize-lines: 4,7,11

:doc:`의존성 주입<dependency-injection>` 을 통해 서비스를 서비스 컨테이너에 추가하여 여러분의 어플리케이션에서 사용할 수 있습니다. ``Startup`` 클래스에서 특정 구현체를 하드코딩하기 보다 메서드의 매개변수를 사용하여 의존성을 지정하였듯이, 미들웨어나 MVC 컨트롤러, 혹은 다른 클래스에 대해서도 의존성을 지정할 수 있습니다.

또한 ``ConfigureServices`` 메서드는 위의 예제 상의 ``AppSettings`` 와 같은 설정용 클래스를 추가해야 위치입니다. 설정에 대해 더 많은 부분을 확인하려면 :doc:'configuration' 주제를 확인하세요.

Startup 클래스에서 사용할 수 있는 서비스들
-----------------------------

ASP.NET Core 에서는 여러분의 어플리케이션이 시작하는 동안 특정 어플리케이션 서비스와 객체를 제공합니다. ``Startup`` 클래스의 생성자나 ``Configure`` 메서드, ''ConfigureServices`` 메서드에 매개변수로서 적절한 인터페이스를 제공하여 이런 서비스들을 요청할 수 있습니다. ``Startup`` 클래스의 각 메서드에서 가능한 서비스는 다음과 같습니다. 

IApplicationBuilder
  어플리케이션의 요청 처리경로를 구축할 때 사용합니다. ``Startup`` 의 ``Configure`` 메서드에서만 사용할 수 있습니다. :doc:'request-features' 에서 더 확인할 수 있습니다.

IApplicationEnvironment
  어플리케이션의 속성에 대한 접근 방법을 제공합니다. 어플리케이션의 속성은 ``ApplicationName`` 과 ``ApplicationVersion``, ``ApplicationBasePath`` 와 같은 것입니다. ``Startup`` 생성자와 ``Configure`` 메서드에서 사용할 수 있습니다.

IHostingEnvironment
  현재의 ``EnvironmentName`` 과 ``WebRootPath``, 웹 루트 파일 제공자를 제공합니다. ``Startup`` 생성자와 ``Configure`` 메서드에서 사용할 수 있습니다.

ILoggerFactory
  로거를 생성하는 방법을 제공합니다. ``Startup`` 생성자와 ``Configure`` 메서드에서 사용할 수 있습니다. :doc:`logging` 에서 더 확인할 수 있습니다.

IServiceCollection
  현재 컨테이너에 설정된 서비스들의 집합입니다. ``ConfigureServices`` 메서드에서만 사용할 수 있습니다. 어플리케이션에서 사용하는 서비스들을 설정하기 위해 사용합니다.

호출되는 순서에 따라 ``Startup`` 클래스의 각각의 메서드를 살펴보면, 다음과 같은 서비스들을 매개변수로서 요청합니다. 

Startup 생성자
- ``IApplicationEnvironment``
- ``IHostingEnvironment``
- ``ILoggerFactory``

ConfigureServices
- ``IServiceCollection``

Configure
- ``IApplicationBuilder``
- ``IApplicationEnvironment``
- ``IHostingEnvironment``
- ``ILoggerFactory``

.. note:: ``ILoggerFactory`` 는 생성자에서 설정할 수도 있지만, 일반적으로 ``Configure`` 메서드에서 설정합니다. :doc:`logging` 에서 자세한 내용을 확인하십시오.

추가 자료
--------------------

- :doc:`environments`
- :doc:`middleware`
- :doc:`owin`

.. _IApplicationBuilder: https://docs.asp.net/projects/api/en/latest/autoapi/Microsoft/AspNet/Builder/IApplicationBuilder/index.html
.. _IServiceCollection: https://docs.asp.net/projects/api/en/latest/autoapi/Microsoft/Extensions/DependencyInjection/IServiceCollection/index.html
