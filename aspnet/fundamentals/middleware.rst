.. _fundamentals-middleware:

미들웨어
==========
By `Steve Smith`_ and `Rick Anderson`_

.. contents:: 단락:
  :local:
  :depth: 1

`샘플 코드 확인하거나 다운로드 받기 <https://github.com/aspnet/Docs/tree/master/aspnet/fundamentals/middleware/sample>`__

미들웨어란 무엇인가
------------------

미들웨어는 소프트웨어 컴포넌트들의 모음으로서, 여러분은 각각의 컴포넌트를 조합하여 미들웨어를 구성하여 HTTP 요청 (request) 과 응답 (response) 을 처리하는 어플리케이션 처리경로 (pipeline) 를 구성할 수 있습니다. 각 컴포넌트에서 처리경로 상의 다음 컴포넌트로 요청을 전달할지 결정할 수 있습니다. 또한 처리경로 상의 다음 컴포넌트를 호출하기 전에 혹은 호출한 후에 특정 동작을 수행하도록 할 수 있습니다. 요청 처리경로를 조립할 때는 요청 대리자 (request delegate) 를 사용합니다. 요청 대리자에서 각각의 HTTP 요청을 처리합니다.

``Startup`` 클래스의 ``Configure`` 메서드에 전달되는 `IApplicationBuilder <https://docs.asp.net/projects/api/en/latest/autoapi/Microsoft/AspNet/Builder/IApplicationBuilder/index.html>`_ 형에 대한 `Run <https://docs.asp.net/projects/api/en/latest/autoapi/Microsoft/AspNet/Builder/RunExtensions/index.html>`__ 과 `Map <https://docs.asp.net/projects/api/en/latest/autoapi/Microsoft/AspNet/Builder/MapExtensions/index.html?highlight=microsoft.aspnet.builder.mapextensions#Microsoft.AspNet.Builder.MapExtensions.Map>`__, `Use <https://docs.asp.net/projects/api/en/latest/autoapi/Microsoft/AspNet/Builder/UseExtensions/index.html?highlight=microsoft.aspnet.builder.useextensions#Microsoft.AspNet.Builder.UseExtensions.Use>`__ 확장 메서드를 사용하여 요청 대리자를 설정합니다. 각각의 요청 대리자는 익명 매서드를 사용하여 인라인 형태로 지정할 수도 있고, 재사용 가능하도록 클래스로 정의하여 지정할 수도 있습니다. 이런 재사용 가능한 클래스가 바로 `미들웨어` 혹은 `미들웨어 컴포넌트` 입니다. 요청 처리경로 상의 각각의 미들웨어 컴포넌트는 처리경로 상의 다음 컴포넌트를 호출하거나 처리경로를 적절하게 끝내야 합니다.

:doc:`/migration/http-modules` 에서 ASP.NET Core 의 요청 처리경로와 이전 버전에서의 요청 처리경로 간의 차이를 확인할 수 있습니다. 또한 미들웨어 에 대한 더 많은 예제도 확인할 수 있습니다.

IApplicationBuilder 를 통해 미들웨어로 이루어진 처리경로 만들기
-------------------------------------------------------

ASP.NET 의 요청 처리경로는 일련의 요청 대리자들로 구성됩니다. 각 요청 대리자는 다음 도표와 같이 차례대로 호출됩니다. (실행되는 순서는 검정 화살표의 흐름과 같습니다.) 

.. image:: middleware/_static/request-delegate-pipeline.png

각각의 대리자는 다음 대리자를 호출하기 전과 호출한 후에 작업을 수행할 수 있습니다. 어떤 대리자도 다음 대리자에게 요청을 전달하지 않고 요청을 직접 처리할 수 있다. 이를 요청 처리경로의 단락 (short-circuit) 이라고 부릅니다. 요청 처리경로를 단락함으로써 다음 대리자를 실행하지 않으므로 불필요한 작업을 하지 않을 수 있습니다. 예를 들어, 인가 (authorization) 미들웨어에서 요청이 인증되었을 때만 다음 대리자를 호출하도록 할 수 있습니다. 즉, 요청이 인증되지 않았다면 처리경로를 단락시키고 다음 대리자를 호출하지 않고 "인가되지 않음" 이라는 응답을 반환할 수 있습니다. 예외 처리 대리자들의 경우에 처리경로 초기에 호출되어야만, 처리경로 상의 후기에 발생하는 예외들을 처리할 수 있습니다.

여러분은 Visual Studio 2015 에 포함된 기본 웹사이트 템플릿에서 요청 처리경로를 설정하는 방법을 확인할 수 있습니다. ``Configure`` 메서드에서 다음과 같은 미들웨어 컴포넌트를 추가합니다.

#. 오류 처리 (개발 환경과 비개발 환경 모두에 대응)
#. 정적 파일 서버
#. 인증
#. MVC

.. literalinclude:: /../common/samples/WebApplication1/src/WebApplication1/Startup.cs
  :linenos:
  :language: c#
  :lines: 58-86
  :dedent: 8
  :emphasize-lines: 8-10,14,17,19,23

위 코드 중 비개발 환경에 대한 경로에서는 `UseExceptionHandler <https://docs.asp.net/projects/api/en/latest/autoapi/Microsoft/AspNet/Builder/ExceptionHandlerExtensions/index.html>`__ 를 첫 번째 미들웨어로서 처리경로에 추가하므로, 이후의 다른 대리자들에 대한 호출에서 발생하는 모든 예외를 처리할 수 있습니다. 

`정적 파일 모듈 <https://docs.asp.net/projects/api/en/latest/autoapi/Microsoft/AspNet/Builder/StaticFileExtensions/index.html>`__ 에서는 인가 (authorization) 확인을 하지 않습니다. 해당 모듈이 서비스하는 모든 파일들은 (*wwwroot* 에 있는 파일들까지도 포함합니다.) 공개적으로 접근 가능합니다. 인가 절차를 거친 후 파일에 접근할 수 있도록 하기 위해서는 다음과 같이 하세요.:

#. 정적 파일 모듈에서 접근 가능한 *wwwroot* 이나 어떠한 디렉토리가 아닌 다른 위치에 파일들을 저장하세요.   
#. 파일들을 컨트롤러 동작을 통해 전달하세요. 인가 절차를 적용한 `FileResult <https://docs.asp.net/projects/api/en/latest/autoapi/Microsoft/AspNet/Mvc/FileResult/index.html>`__ 을 반환하는 동작을 통해 가능합니다.

요청을 정적 파일 모듈로 처리하는 경우, 정적 파일 모듈에서 처리경로를 단락시킵니다. (참고 :doc:`static-files`) 요청을 정적 파일 모듈로 처리하지 않는 경우에는 `식별 모듈 (Identity module) <https://docs.asp.net/projects/api/en/latest/autoapi/Microsoft/AspNet/Builder/BuilderExtensions/index.html#methods>`__ 에 요청을 전달하여 인증 절차를 수행합니다. 요청이 인증에 실패하는 경우, 식별 모듈에서 처리경로를 단락시킵니다. 요청이 인증에 성공하는 경우에는 처리경로의 마지막 단계인 MVC 프레임워크를 호출합니다.

.. note:: 일반적으로 미들웨어 컴포넌트를 추가하는 순서가 요청을 처리하는 순서이고, 그 역순이 응답을 처리하는 순서입니다. 이는 앱의 보안과 성능, 기능에 중대한 영향을 끼칠 수 있습니다. 위의 코드에서 보면, `정적 파일 미들웨어 <https://docs.asp.net/projects/api/en/latest/autoapi/Microsoft/AspNet/Builder/StaticFileExtensions/index.html>`__ 를 처리경로 상의 앞 부분에서 호출하여 불필요한 컴포넌트를 거치지 않고 요청을 처리한 뒤 처리경로를 단락시킬 수 있었습니다. 인증 미들웨어의 경우, 인증 절차를 거쳐야 하는 요청들을 처리하는 미들웨어들보다 먼저 처리경로에 추가하였습니다. 또한 예외 처리 미들웨어의 경우, 다른 미들웨어 컴포넌트에서 발생하는 예외를 모두 처리하기 위해 먼저 처리경로에 추가해야 합니다.

ASP.NET 어플리케이션을 가장 간단하게 구성하는 경우, 모든 요청을 처리하는 하나의 요청 대리자 만 설정하면 됩니다. 이 경우에, 요청 "처리경로"가 실질적으로 존재하지 않고, 모든 HTTP 요청을 처리하는 하나의 익명 함수 만 존재합니다.

.. literalinclude:: middleware/sample/src/MiddlewareSample/Startup.cs
	:language: c#
	:lines: 23-26
	:dedent: 12

첫 번째 ``App.Run`` 대리자에서 처리경로를 종료합니다. 다음 예시에서는, 첫 번째 대리자 만 ("Hello, World!") 실행됩니다.

.. literalinclude:: middleware/sample/src/MiddlewareSample/Startup.cs
	:language: c#
	:lines: 20-31
	:emphasize-lines: 5
	:dedent: 8

여러 개의 요청 대리자를 엮을 수 있습니다. 이를 위해 ``next`` 매개변수에 처리경로 상의 다음 대리자가 전달됩니다. 처리경로를 종료, 즉 단락시키기 위해서는 이 `next` 매개변수에 대해 호출하지 않으면 됩니다. 또한 다음 대리자를 호출하기 전과 호출한 후에 원하는 동작들을 수행할 수도 있습니다. 다음 예시에서 확인할 수 있습니다.:

.. literalinclude:: middleware/sample/src/MiddlewareSample/Startup.cs
	:language: c#
	:lines: 34-49
	:emphasize-lines: 5,8,14
	:dedent: 8

.. warning:: 처리경로 상의 이후에 설정된 컴포넌트에서 응답을 변경하여 클라이언트에게 전달하고자 할 수 있으므로, 다음 컴포넌트를 호출한 후에 ``HttpResponse`` 를 수정하지 않도록 하십시오.  

.. note:: 이 ``ConfigureLogInline`` 메서드는 ``LogInline`` 환경설정 항목을 설정한 어플리케이션을 실행할 경우 호출됩니다. :doc:`environments` 에서 자세한 사항을 확인할 수 있습니다. 이후의 내용에서 다양한 옵션을 확인해보기 위해 여러가지 ``Configure[환경설정]`` 를 사용할 것입니다. Visual Studio 에서 예제들을 실행하는 가장 쉬운 방법은 ``web`` 명령어를 사용하는 것으로써, *project.json* 에서 설정할 수 있습니다. :doc:`startup` 을 확인하세요.

위의 예시에서 ``await next.Invoke()`` 를 호출하여 다음 대리자인 ``await context.Response.WriteAsync("Hello from " + _environment);`` 를 호출하였습니다. 클라이언트에게는 예상하는 바와 같이 ("Hello from LogInline") 응답을 전달할 것이고, 서버의 콘솔 출력에는 호출 직전의 메시지와 호출 직후의 메시지가 나타날 것입니다.

.. image:: middleware/_static/console-loginline.png

..  _middleware-run-map-use:

Run 과 Map 그리고 Use
^^^^^^^^^^^^^^^^^

여러분은 `Run <https://docs.asp.net/projects/api/en/latest/autoapi/Microsoft/AspNet/Builder/RunExtensions/index.html>`__ 과 `Map <https://docs.asp.net/projects/api/en/latest/autoapi/Microsoft/AspNet/Builder/MapExtensions/index.html>`__ 그리고 `Use <https://docs.asp.net/projects/api/en/latest/autoapi/Microsoft/AspNet/Builder/UseExtensions/index.html>`__ 를 사용하여 HTTP 처리경로를 설정할 수 있습니다. 우선, `Run` 메서드는 처리경로를 단락시킵니다. (즉, ``next`` 매개변수로 전달되는 요청 대리자를 호출하지 않습니다.) 따라서, ``Run`` 메서드는 처리경로의 마지막에 호출해야 합니다. ``Run`` 메서드는 모든 미들웨어에 공통적인 관례입니다. 그런 이유로 어떤 미들웨어 컴포넌트들의 경우에는 자신만의 Run[미들웨어의 이름] 메서드를 노출하고, 해당 메서드를 처리경로의 마지막에 호출해야 하도록 할 수 있습니다. 다음의 2가지 미들웨어는 동일한 응답을 반환합니다. ``Use`` 메서드를 사용하는 미들웨어에서 ``next`` 매개변수를 사용하지 않기 때문입니다.

.. literalinclude:: middleware/sample/src/MiddlewareSample/Startup.cs
	:language: c#
	:lines: 65-79
	:emphasize-lines: 3,11
	:dedent: 8

.. note:: `IApplicationBuilder  <https://docs.asp.net/projects/api/en/latest/autoapi/Microsoft/AspNet/Builder/IApplicationBuilder/index.html>`__ 인터페이스는 ``Use`` 메서드 하나만을 노출하고 있습니다. 따라서 기술적으로 위 3가지 메서드가 모두 *확장* 메서드는 아닙니다.

여러분은 이미 ``Use`` 메서드를 사용하여 요청 처리경로를 구성하는 방법에 대한 몇 가지 예시를 확인하였습니다. ``Map*`` 확장 메서드들의 경우 처리경로를 분기처리하기 위해 관례적으로 사용합니다. 현재의 ASP.NET 구현에서는 요청의 URL 경로나 *predicate* 을 통한 분기를 지원합니다. ``Map`` 확장 메서드는 요청의 경로에 따라 요청 대리자를 선택하기 위해 사용합니다. ``Map`` 메서드는 경로와 그 경로를 위한 별도의 미들웨어 처리경로를 설정하는 함수를 매개변수로 받습니다. 다음의 예시에서는 ``/maptest`` 로 시작하는 경로에 해당하는 모든 요청을 ``HandleMapTest`` 메서드에서 설정하는 처리경로에서 처리합니다.

.. literalinclude:: middleware/sample/src/MiddlewareSample/Startup.cs
	:language: c#
	:lines: 81-93
	:emphasize-lines: 11
	:dedent: 8

.. note:: ``Map`` 메서드를 사용하게 되면, 요청의 경로에서 찾은 일치하는 부분을 각 요청의 ``HttpRequest.Path`` 속성에서 잘라내어 ``HttpRequest.PathBase`` 속성에 붙입니다.

경로 기반의 매핑과는 별도로, ``MapWhen`` 메서드를 통해 predicate 기반의 미들웨어 분기처리를 지원합니다. 이를 통해 매우 유연한 형태로 각각의 처리경로를 구성할 수 있습니다. ``Func<HttpContext, bool>`` 형인 predicate 으로 처리경로 상의 새로운 분기를 요청에 매핑할 수 있습니다. 다음 예시에서는 ``branch`` 라는 문자열이 질의 문자열 (query string) 에 존재하는지 확인하는 간단한 predicate 을 사용하고 있습니다.

.. literalinclude:: middleware/sample/src/MiddlewareSample/Startup.cs
	:language: c#
	:lines: 95-113
	:emphasize-lines: 5,11-13
	:dedent: 8

위와 같이 설정하였으므로 ``branch`` 를 포함하는 질의 문자열을 포함하는 요청은 모두 ``HandleBranch`` 메서드에서 정의한 처리경로를 사용할 것입니다. (즉 "Branch used." 라는 응답을 반환합니다.) 다른 모든 요청의 경우 (즉 ``branch`` 를 질의 문자열에 포함하지 않는 경우) 17줄에 정의된 대리자를 통해 처리할 것입니다.

여러분은 다음과 같이 Map 을 중첩하여 사용할 수도 있습니다.:  

.. code-block:: javascript  

   app.Map("/level1", level1App => {
       level1App.Map("/level2a", level2AApp => {
           // "/level1/level2a"
           //...
       });
       level1App.Map("/level2b", level2BApp => {
           // "/level1/level2b"
           //...
       });
   });
   
내장된 미들웨어
-------------------

ASP.NET 은 다음과 같은 미들웨어 컴포넌트를 포함하고 있습니다.:

.. list-table:: 미들웨어
  :header-rows: 1

  *  - 미들웨어
     - 설명
  *  - :doc:`인증 </security/authentication/index>`
     - 인증 기능을 제공합니다.
  *  - :doc:`CORS </security/cors>`
     - 크로스 출처 자원 공유 (Cross-Origin Resource Sharing) 를 설정할 수 있습니다.
  *  - :doc:`라우팅 <routing>`
     - 요청 경로의 정의와 제한 방법을 제공합니다.
  *  - :ref:`세션 <session>`
     - 사용자 세션를 관리하기 위한 기능을 제공합니다.
  *  - :doc:`정적 파일 <static-files>`
     - 정적 파일을 노출과 디렉토리 브라우징에 대한 기능을 제공합니다.

.. _middleware-writing-middleware:

미들웨어 작성하기
------------------

`CodeLabs 미들웨어 튜터리얼 <https://github.com/Microsoft-Build-2016/CodeLabs-WebDev/tree/master/Module2-AspNetCore>`__ 에서 미들웨어를 작성하는 방법에 대한 설명을 확인할 수 있습니다. 

더 복잡한 요청을 처리하기 위한 기능을 확인하고자 할 경우, ASP.NET 팀에서는 별도의 클래스로 미들웨어를 구현하고 ``Configure`` 메서드에서 호출될 수 있는 ``IApplicationBuilder`` 확장 메서드를 노출하는 방법을 권장합니다. 이전 예시에서 확인했던 간단한 로깅 미들웨어를 다음과 같이 변경할 수 있습니다. 생성자에서 ``RequestDelegate`` 형인 next 매개변수를 받고 있고, ``Invoke`` 메서드를 지원하고 있습니다.: 

.. literalinclude:: middleware/sample/src/MiddlewareSample/RequestLoggerMiddleware.cs
	:language: c#
	:caption: RequestLoggerMiddleware.cs
	:emphasize-lines: 12, 18

미들웨어는 `명시적 의존성 원칙 (Explicit Dependencies Principle) <http://deviq.com/explicit-dependencies-principle/>`_ 을 따릅니다. 이 원칙에 따라 미들웨어의 생성자에서 의존성을 명시하고 있습니다. 아래의 예시에서 보는 바와 같이 미들웨어는 `UseMiddleware<T>`_ 확장 메서드를 사용하여, 자신의 생성자에 서비스들에 대한 의존성을 직접 주입할 수 있습니다. `UseMiddleware<T>`_ 확장 메서드의 매개변수에 의존성으로서 주입된 서비스들은 자동으로 삽입되고, 그 외에 의존성으로서 주입되지 않은 매개변수의 경우에는 ``params`` 인자 배열을 사용합니다.

.. literalinclude:: middleware/sample/src/MiddlewareSample/RequestLoggerExtensions.cs
	:language: c#
	:caption: RequestLoggerExtensions.cs
	:lines: 5-11
	:emphasize-lines: 5
	:dedent: 4

확장 메서드와 그와 관련된 미들웨어 클래스를 사용하면 ``Configure`` 메서드가 매우 간결해지고 가독성이 높아집니다.

.. literalinclude:: middleware/sample/src/MiddlewareSample/Startup.cs
	:language: c#
	:lines: 51-62
	:emphasize-lines: 6
	:dedent: 8

``RequestLoggerMiddleware`` 는 생성자에서 ``ILoggerFactory`` 를 매개변수로서 필요로 하지만, ``Startup`` 클래스나 ``UseRequestLogger`` 확장 메서드의 경우에는 명시적으로 매개변수로서 받을 필요는 없습니다. 대신 ``UseMiddleware<T>`` 에 의존성을 주입하여 자동으로 사용할 수 있습니다.

미들웨어를 테스트해보면 다음과 같이 출력되어야 합니다. (``Hostring:Environment`` 환경 변수에 ``LogMiddleware`` 를 설정하고 WebListener 를 사용합니다.):

.. image:: middleware/_static/console-logmiddleware.png

.. note:: `UseStaticFiles <https://docs.asp.net/projects/api/en/latest/autoapi/Microsoft/AspNet/Builder/StaticFileExtensions/index.html#meth-Microsoft.AspNet.Builder.StaticFileExtensions.UseStaticFiles>`_ 확장 메서드 (`StaticFileMiddleware <https://docs.asp.net/projects/api/en/latest/autoapi/Microsoft/AspNet/StaticFiles/StaticFileMiddleware/index.html>`_ 를 생성합니다.) 에서도 ``UseMiddleware<T>`` 를 사용합니다. 이 경우 ``StaticFileOptions`` 매개변수는 전달되지만, 생성자의 다른 매개변수들은 ``UseMiddleware<T>`` 에 대한 의존성 주입을 통해 전달됩니다.

추가 자료
--------------------

- `CodeLabs 미들웨어 튜터리얼 <https://github.com/Microsoft-Build-2016/CodeLabs-WebDev/tree/master/Module2-AspNetCore>`__
- `이 문서에서 사용된 샘플 코드 <https://github.com/aspnet/Docs/tree/master/aspnet/fundamentals/middleware/sample>`_
- :doc:`/migration/http-modules`
- :doc:`startup`
- :doc:`request-features`

.. _UseMiddleware<T>: https://docs.asp.net/projects/api/en/latest/autoapi/Microsoft/AspNetCore/Builder/UseMiddlewareExtensions/index.html#meth-Microsoft.AspNetCore.Builder.UseMiddlewareExtensions.UseMiddleware<TMiddleware>
