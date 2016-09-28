:version: 1.0.0-rc1

오류 처리하기
==============

By `Steve Smith`_

When errors occur in your ASP.NET app, you can handle them in a variety of ways, as described in this article.
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

The developer exception page displays useful diagnostics information when an unhandled exception occurs within the web processing pipeline. The page includes several tabs with information about the exception that was triggered and the request that was made. The first tab includes a stack trace:
개발자 예외 페이지에서는

.. image:: error-handling/_static/developer-exception-page.png

The next tab shows the query string parameters, if any:

.. image:: error-handling/_static/developer-exception-page-query.png

In this case, you can see the value of the ``throw`` parameter that was passed to this request. This request didn't have any cookies, but if it did, they would appear on the Cookies tab. You can see the headers that were passed in the last tab:

.. image:: error-handling/_static/developer-exception-page-headers.png

.. _status-code-pages:

상태 코드 페이지 설정하기
-----------------------------

By default, your app will not provide a rich status code page for HTTP status codes such as 500 (Internal Server Error) or 404 (Not Found). You can configure the ``StatusCodePagesMiddleware`` adding this line to the ``Configure`` method:

.. code-block:: c#

  app.UseStatusCodePages();

By default, this middleware adds very simple, text-only handlers for common status codes. For example, the following is the result of a 404 Not Found status code:

.. image:: error-handling/_static/default-404-status-code.png

The middleware supports several different extension methods. You can pass it a custom lamdba expression:

.. code-block:: c#

  app.UseStatusCodePages(context => 
    context.HttpContext.Response.SendAsync("Handler, status code: " +
    context.HttpContext.Response.StatusCode, "text/plain"));

Alternately, you can simply pass it a content type and a format string:

.. code-block:: c#

  app.UseStatusCodePages("text/plain", "Response, status code: {0}");

The middleware can handle redirects (with either relative or absolute URL paths), passing the status code as part of the URL:

.. code-block:: c#

  app.UseStatusCodePagesWithRedirects("~/errors/{0}");

In the above case, the client browser will see a ``302 / Found`` status and will redirect to the URL provided.

Alternately, the middleware can re-execute the request from a new path format string:

.. code-block:: c#

  app.UseStatusCodePagesWithReExecute("/errors/{0}");

The ``UseStatusCodePagesWithReExecute`` method will still return the original status code to the browser, but will also execute the handler given at the path specified.

If you need to disable status code pages for certain requests, you can do so using the following code:

.. code-block:: c#

  var statusCodePagesFeature = context.Features.Get<IStatusCodePagesFeature>();
  if (statusCodePagesFeature != null)
  {
    statusCodePagesFeature.Enabled = false;
  }

클라이언트-서버 상호작용 중의 예외 처리의 한계점
------------------------------------------------------------------

Web apps have certain limitations to their exception handling capabilities because of the nature of disconnected HTTP requests and responses. Keep these in mind as you design your app's exception handling behavior.

#. Once the headers for a response have been sent, you cannot change the response's status code, nor can any exception pages or handlers run. The response must be completed or the connection aborted.
#. If the client disconnects mid-response, you cannot send them the rest of the content of that response.
#. There is always the possibility of an exception occuring one layer below your exception handling layer.
#. Don't forget, exception handling pages can have exceptions, too. It's often a good idea for production error pages to consist of purely static content.

Following the above recommendations will help ensure your app remains responsive and is able to gracefully handle exceptions that may occur.

서버 예외 처리하기
-------------------------

In addition to the exception handling logic in your app, the server hosting your app will perform some exception handling. If the server catches an exception before the headers have been sent it will send a 500 Internal Server Error response with no body. If it catches an exception after the headers have been sent it must close the connection. Requests that are not handled by your app will be handled by the server, and any exception that occurs will be handled by the server's exception handling. Any custom error pages or exception handling middleware or filters you have configured for your app will not affect this behavior.

.. _startup-error-handling:

시작점의 예외 처리하기
--------------------------

One of the trickiest places to handle exceptions in your app is during its startup. Only the hosting layer can handle exceptions that take place during app startup. Exceptions that occur in your app's startup can also impact server behavior. For example, to enable SSL in Kestrel, one must configure the server with ``KestrelServerOptions.UseHttps()``. If an exception happens before this line in ``Startup``, then by default hosting will catch the exception, start the server, and display an error page on the non-SSL port. If an exception happens after that line executes, then the error page will be served over HTTPS instead.

ASP.NET MVC 오류 처리하기
--------------------------

:doc:`MVC </mvc/index>` apps have some additional options when it comes to handling errors, such as configuring exception filters and performing model validation.

예외 필터
^^^^^^^^^^^^^^^^^

Exception filters can be configured globally or on a per-controller or per-action basis in an :doc:`MVC </mvc/index>` app. These filters handle any unhandled exception that occurs during the execution of a controller action or another filter, and are not called otherwise. Exception filters are detailed in :doc:`filters </mvc/controllers/filters>`.

.. tip:: Exception filters are good for trapping exceptions that occur within MVC actions, but they're not as flexible as error handling middleware. Prefer middleware for the general case, and use filters only where you need to do error handling *differently* based on which MVC action was chosen.

모델 상태 오류 처리하기
^^^^^^^^^^^^^^^^^^^^^^^^^^^

:doc:`Model validation </mvc/models/validation>` occurs prior to each controller action being invoked, and it is the action method’s responsibility to inspect ``ModelState.IsValid`` and react appropriately. In many cases, the appropriate reaction is to return some kind of error response, ideally detailing the reason why model validation failed. 

Some apps will choose to follow a standard convention for dealing with model validation errors, in which case a :doc:`filter </mvc/controllers/filters>` may be an appropriate place to implement such a policy. You should test how your actions behave with valid and invalid model states (learn more about :doc:`testing controller logic </mvc/controllers/testing>`).
