:version: 1.0.0-rc1

오류 처리하기
==============

By `Steve Smith`_

여러분의 ASP.NET 어플리케이션에서 오류가 발생할 경우, 여러가지 방법으로 처리할 수 있습니다.

.. contents:: Sections
	:local:
	:depth: 1

`샘플 코드를 확인하거나 다운로드 받으세요. <https://github.com/aspnet/Docs/tree/master/aspnet/fundamentals/error-handling/sample>`_

예외 처리 페이지 설정하기
--------------------------------------

여러분은 ``Startup`` 클래스의 ``Configure()`` 메서드에서 각각의 요청에 대한 처리경로를 설정할 수 있습니다. 설정 과정에서 매우 쉽게 개발 용도로만 사용할 간단한 예외 페이지를 추가할 수 있습니다. ``Microsoft.AspNetCore.Diagnostics`` 에 대한 의존성을 프로젝트에 추가하고, ``Startup.cs`` 내의 ``Configure()`` 메서드에 한 줄만 추가하면 됩니다.: 

.. literalinclude:: error-handling/sample/src/ErrorHandlingSample/Startup.cs
	:language: c#
	:lines: 21-29
	:dedent: 8
	:emphasize-lines: 6,8

위 코드에서는 ``UseDeveloperExceptionPage`` 메서드에 대한 호출 전에 개발 환경인지 확인하고 있습니다. 이는 적절한 습관으로, 일반적으로 상용 환경에서 공개적으로 예외에 대한 상세한 정보를 노출하지는 않기 때문입니다. :doc:`환경 설정에 대해서 더 확인해보세요. <environments>`.

다음 예제에서는 예외를 발생시켜보는 간단한 방법을 포함하고 있습니다.

.. literalinclude:: error-handling/sample/src/ErrorHandlingSample/Startup.cs
	:language: c#
	:lines: 58-77
	:dedent: 8
	:emphasize-lines: 5-8

값이 있는 ``throw`` 변수가 요청의 질의 문자열에 들어있다면, (예. ``/?throw=true``) 예외를 던질 것입니다. 개발 환경인 경우에는 개발자 예외 페이지가 보여질 것입니다.:

.. image:: error-handling/_static/developer-exception-page.png

개발 환경이 아닌 경우에는, ``UseExceptionHandler`` 미들웨어를 사용하여 예외 핸들러를 설정하는 것이 좋습니다.

.. code-block:: c#

  app.UseExceptionHandler("/Error");

For the action associated with the endpoint, don't explicitly decorate the ``IActionResult`` with HTTP method attributes, such as ``HttpGet``. Using explicit verbs could prevent some requests from reaching the method.
요청의 말단에 대한 동작을 정의할 때, ``IActionResult`` 에 ``HttpGet`` 과 같은 HTTP 메서드 특성을 명시적으로 지정하지는 마세요. HTTP 동사를 명시하는 경우 일부 요청에서 해당 메서드가 실행되지 않을 수도 있습니다.

.. code-block:: c#

  [Route("/Error")]
  public IActionResult Index()
  {
      // 여기서 오류 처리
  }

개발자 예외 페이지 사용하기
----------------------------------

개발자 예외 페이지에서는 요청 처리경로에서 처리하지 못한 예외가 발생한 경우 유용하게 사용할 수 있는 진단 정보를 노출합니다. 해당 페이지은 발생한 예외와 관련된 요청에 대한 정보를 포함하는 여러 개의 탭으로 구성되어 있습니다. 첫 번째 탭에서는 스택 트레이스 정보를 확인할 수 있습니다.:

.. image:: error-handling/_static/developer-exception-page.png

그 다음 탭에서는 쿼리 문자열에 매개변수가 있었다면 그에 대한 정보를 확인할 수 있습니다.:

.. image:: error-handling/_static/developer-exception-page-query.png

위 예제에서는 요청을 통해 전달된 ``throw`` 매개변수의 값을 확인할 수 있습니다. 이 요청에는 쿠키를 포함하지 않고 있으나, 쿠키를 포함하고 있었다면 Cookies 탭에서 쿠키 정보를 확인할 수 있었을 것입니다. 마지막 탭에서는 헤더 정보를 확인할 수 있습니다.:

.. image:: error-handling/_static/developer-exception-page-headers.png

.. _status-code-pages:

상태 코드 페이지 설정하기
-----------------------------

기본적으로 여러분의 앱에서는 500 (서버 내부 오류) 나 404 (찾을 수 없음) 과 같은 HTTP 상태 코드에 대한 상세한 정보를 제공하는 페이지를 제공하지는 않습니다. ``Configure`` 메서드에 아래와 같이 코드를 추가하여 ``StatusCodePagesMiddleware`` 를 설정할 수 있습니다.:

.. code-block:: c#

  app.UseStatusCodePages();

기본적으로 이 미들웨어는 매우 간단하고 문자열 만 사용하는 일반적 상태 코드에 대한 핸들러를 추가합니다. 예를 들어, 다음은 404 (찾을 수 없음) 상태 코드에 대한 결과입니다.:

.. image:: error-handling/_static/default-404-status-code.png

미들웨어에서는 여러가지 확장 메서드를 지원합니다. 사용자 정의된 람다 표현식을 전달할 수도 있습니다.

.. code-block:: c#

  app.UseStatusCodePages(context => 
    context.HttpContext.Response.SendAsync("Handler, status code: " +
    context.HttpContext.Response.StatusCode, "text/plain"));

혹은 간단하게 콘텐트 타입과 포맷 문자열을 전달할 수도 있습니다.

.. code-block:: c#

  app.UseStatusCodePages("text/plain", "Response, status code: {0}");

미들웨어에 상태 코드 페이지로의 리다이렉트 경로 (상대 URL 경로와 절대 URL 경로 모두 지원) 를 전달하여 처리할 수도 있습니다. 이 경우, URL 의 일부로서 상태 코드를 전달하면 됩니다.

.. code-block:: c#

  app.UseStatusCodePagesWithRedirects("~/errors/{0}");

위 경우에, 클라이언트 측인 브라우저에 ``302 / 찾음`` 상태를 노출한 후, 전달한 URL 로 리다이렉트할 것입니다.

혹은, 미들웨어에서 새로운 경로 문자열을 통한 요청을 재실행하게 할 수도 있습니다.

.. code-block:: c#

  app.UseStatusCodePagesWithReExecute("/errors/{0}");

``UseStatusCodePagesWithReExecute`` 메서드는 브라우저에 원래의 상태 코드를 반환하지만, 지정된 경로 상의 요청 핸들러도 실행합니다.

특정 요청에 대해 상태 코드 페이지를 보이지 않도록 하기 위해서는, 다음 코드를 사용하면 됩니다.

.. code-block:: c#

  var statusCodePagesFeature = context.Features.Get<IStatusCodePagesFeature>();
  if (statusCodePagesFeature != null)
  {
    statusCodePagesFeature.Enabled = false;
  }

클라이언트-서버 상호작용 중의 예외 처리의 한계점
------------------------------------------------------------------

웹 어플리케이션은 예외 처리 기능에 어느 정도 한계점을 가지고 있습니다. 이는 끊어진 HTTP 요청과 응답의 특성 때문입니다. 따라서, 예외 처리 방식을 설계할 때 이를 염두에 두어야 합니다.

#. 응답에 대한 헤더가 전송된 후에는, 응답 상의 상태 코드를 변경하거나 다른 예외 페이지 혹은 핸들러로 변경할 수 없습니다. 응답을 완료하거나 연결을 중지해야 합니다.
#. 클라이언트에서 응답을 전송받는 중간에 연결을 끊은 경우, 나머지 응답 콘텐츠를 전송할 수 없습니다.
#. 예외 처리 단계 직전에 예외가 발생할 가능성이 언제나 존재합니다.
#. 절대 잊지 말아야 할 부분은 예외 처리 페이지 자체에서 예외를 발생할 수도 있다는 점입니다. 상용 오류 페이지를 완전한 정적 콘텐츠로 구성하는 방법은 좋은 발상입니다.

위의 권장 사항을 따른다면, 여러분의 어플리케이션이 나은 반응성을 가지게 하고 발생 가능한 예외를 우아하게 처리할 수 있도록 보장할 수 있습니다.

서버 예외 처리하기
-------------------------

여러분의 어플리케이션 내의 예외 처리 로직에 더해, 어플리케이션을 호스팅하는 웹 서버에서 몇 가지 예외 처리를 수행할 수 있습니다. 응답 헤더를 보내기 전에 서버에서 예외를 검출하게 되면, 서버는 응답 본문 없이 500 내부 서버 오류 만 보낼 것입니다. 헤더를 보낸 후에 서버가 예외를 검출한다면, 서버는 연결을 끊어야 합니다. 여러분의 어플리케이션에서 처리하지 않은 요청의 경우 서버에서 처리하고, 예외가 발생하게 되면 서버에서 처리할 것입니다. 어플리케이션에서 설정한 어떠한 사용자 정의된 오류 페이지든 어떠한 예외 처리 미들웨어든 어떠한 필터라도 이 행태에 영향을 줄 수 없습니다.

.. _startup-error-handling:

시작점의 예외 처리하기
--------------------------

어플리케이션에서 예외를 처리하기 가장 까다로운 지점은 어플리케이션 시작점입니다. 호스팅 단계에서만 어플리케이션 시작점에서 발생하는 예외를 처리할 수 있습니다. 어플리케이션 시작점에서 발생하는 예외는 서버의 행태에 영향을 줄 수도 있습니다. 예를 들어, Kestrel 서버 내의 SSL 을 허용하기 위해서는, ``KestrelServerOptions.UseHttps()`` 메서드로 서버를 설정해야 합니다. 그런데 ``Startup`` 메서드에서 이 메서드가 실행되기 전에 예외가 발생한다면, 기본 호스팅에서 예외를 검출하고 서버를 시작한 뒤 SSL 이 아닌 일반 HTTP 포트를 통해 오류 페이지를 보일 것입니다. 하지만 해당 라인이 실행된 후에 예외가 발생한다면, 오류 페이지를 HTTPS 를 통해 보일 것입니다.

ASP.NET MVC 오류 처리하기
--------------------------

:doc:`MVC </mvc/index>` 어플리케이션에는 오류를 처리하기 위한 몇 가지 추가적인 선택지가 더 있습니다. 예외 필터나 모델 검증과 같은 것입니다.

예외 필터
^^^^^^^^^^^^^^^^^

예외 필터는 글로벌하게 혹은 컨트롤러 단위로 혹은 액션 단위로 :doc:`MVC </mvc/index>` 어플리케이션에 설정할 수 있습니다. 필터는 컨트롤러의 액션을 실행할 때 혹은 다른 필터를 실행할 때 발생하는 오류를 처리하고, 그 외의 경우에는 호출되지 않습니다. 예외 필터에 대한 자세한 내용은 :doc:`filters </mvc/controllers/filters>` 를 참고하세요.

.. tip:: 예외 필터는 MVC 액션 내에서 발생하는 예외를 검출하기에 적절합니다. 하지만 오류 처리 미들웨어 만큼 유연하지 않습니다. 대부분의 경우 미들웨어가 적합하고, MVC 액션에 따라 *다르게* 오류를 처리해야 하는 경우에만 필터를 사용하세요.

모델 상태 오류 처리하기
^^^^^^^^^^^^^^^^^^^^^^^^^^^

:doc:`모델 검증 </mvc/models/validation>` 은 각 컨트롤러 액션이 실행되기 전에 수행됩니다. ``ModelState.IsValid`` 를 호출하고 그 결과를 적절히 처리하는 것은 액션 메서드의 몫입니다. 많은 경우, 적절히 처리하는 방법은 모델 검증에 실패한 원인에 대해 상세히 알리는 오류 응답을 반환하는 것입니다.

Some apps will choose to follow a standard convention for dealing with model validation errors, in which case a :doc:`filter </mvc/controllers/filters>` may be an appropriate place to implement such a policy. You should test how your actions behave with valid and invalid model states (learn more about :doc:`testing controller logic </mvc/controllers/testing>`).
일부 어플리케이션의 경우 모델 검증 오류를 처리할 때 표준적인 절차를 따릅니다. 이 경우 :doc:`필터 </mvc/controllers/filters>` 가 정책을 구현하기에 적절한 위치입니다. 컨트롤러의 액션이 모델의 적절한 상태와 부적절한 상태에 따라 어떤 행태를 보이는지 테스트해야 합니다. (:doc:`컨트롤러 로직 테스트하기 </mvc/controllers/testing>` 에서 더 자세히 확인할 수 있습니다.)
