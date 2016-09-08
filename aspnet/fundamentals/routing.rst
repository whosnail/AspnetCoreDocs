라우팅
=======
By `Ryan Nowak`_, `Steve Smith`_, and `Rick Anderson`_

라우팅은 요청을 경로 핸들러 (route handler) 에 연결할 때 사용합니다. 경로 핸들러는 어플리케이션이 시작할 때 설정됩니다. 경로 핸들러를 통해 URL 내에서 요청을 처리할 때 필요한 값을 추출할 수 있습니다. 또한 ASP.NET 어플리케이션 내에 정의한 경로 핸들러에서 사용할 링크도 라우팅 기능을 통해 생성할 수 있습니다.

이 문서에서는 저수준의 ASP.NET Core 라우팅 기능 까지 다루고 있습니다. ASP.NET Core MVC 라우팅을 확인하기 위해서는 :doc:`/mvc/controllers/routing` 를 확인하세요.

.. contents:: Sections
  :local:
  :depth: 1

`샘플 코드를 확인하거나 다운로드 받으세요. <https://github.com/aspnet/Docs/tree/master/aspnet/fundamentals/routing/sample>`__

라우팅의 기본
----------------

라우팅에서는 *경로 핸들러 (route handler)* (:dn:iface:`~Microsoft.AspNetCore.Routing.IRouter` 의 구현체) 를 다음과 같은 목적으로 사용합니다.:

- 인입되는 요청을 *경로 핸들러* 로 연결
- 응답에서 사용할 URL 생성

일반적으로 하나의 어플리케이션에는 경로들에 대한 목록이 하나 있습니다. 경로 목록은 순서대로 처리됩니다. 요청 측면에서 볼 때는 요청의 URL 에 일치하는 경로를 경로 목록에서 찾기 위해 :ref:`URL-Matching-ref` 을 사용합니다. 응답 측면에서 볼 때는 응답 내에서 사용할 URL 을 생성하기 위해 라우팅을 사용합니다.

라우팅을 :doc:`middleware <middleware>` 처리경로에 :dn:class:`~Microsoft.AspNetCore.Builder.RouterMiddleware` 클래스를 통해 연동할 수 있습니다. :doc:`ASP.NET MVC </mvc/overview>` 에서는 설정을 통해 미들웨어 처리경로에 라우팅을 추가할 수 있습니다. 독립적인 컴포넌트로서 라우팅을 사용하는 방법을 확인하기 위해서는 using-routing-middleware_ 를 확인하세요.

.. _URL-Matching-ref:

URL 일치 확인
^^^^^^^^^^^^
URL 일치 확인은 라우팅이 인입되는 요청을 어떤 *경로 핸들러 (route handler)* 에서 전달할지를 결정하는 절차입니다. 이 절차는 일반적인 URL 상의 데이터를 기반으로 합니다. 하지만 요청 내의 다른 종류의 데이터를 사용하여 확장할 수도 있습니다. 요청을 각각의 핸들러로 전달하는 기능이야 말로 어플리케이션의 크기와 복잡도를 가늠하는 척도입니다.

인입되는 요청이 :dn:cls:`~Microsoft.AspNetCore.Builder.RouterMiddleware` 으로 진입하면, 미들웨어는 순서대로 각 경로 핸들러의 :dn:method:`~Microsoft.AspNetCore.Routing.IRouter.RouteAsync` 를 호출합니다. :dn:iface:`~Microsoft.AspNetCore.Routing.IRouter` 의 구현체인 경로 핸들러 개체는 ``RounteAsync`` 의 매개변수로 전달된 :dn:cls:`~Microsoft.AspNetCore.Routing.RouteContext` 상의 :dn:prop:`~Microsoft.AspNetCore.Routing.RouteContext.Handler` 속성에 null 이 아닌 :dn:delegate:`~Microsoft.AspNetCore.Http.RequestDelegate` 를 할당하는가 마는가를 통해 요청을 *처리할지 말지*를 결정합니다. 경로 핸들러에서 처리하기로 결정했다면, 요청을 처리하도록 호출될 것이고 미들웨어 내에서 그 이상의 경로는 처리되지 않을 것입니다. 모든 경로를 처리하였고 요청에 대한 핸들러가 더 없다면, 미들웨어는 *next* 를 호출하여 요청 처리경로 상의 다음 미들웨어를 호출합니다.

``RouteAsync`` 메서드의 가장 중요한 입력값은 :dn:cls:`~Microsoft.AspNetCore.Routing.RouteContext` 의 :dn:prop:`~Microsoft.AspNetCore.Routing.RouteContext.HttpContext` 속성으로서, 현재 요청과 관련되어 있습니다. ``RouteContext.Handler`` 와 :dn:cls:`~Microsoft.AspNetCore.Routing.RouteContext` 의 :dn:prop:`~Microsoft.AspNetCore.Routing.RouteContext.RouteData` 속성은 URL 일치 확인에 성공한 후에 설정될 출력값입니다.

``RouteAsync`` 실행 중에 URL 일치 확인에 성공한 경우, ``RouteContext.RouteData`` 의 속성들에 요청 처리 과정의 결과를 바탕으로 적절한 값을 할당합니다. 즉, ``RouteContext.RouteData`` 에는 경로의 *결과* 에 대한 중요한 정보를 담고 있습니다.

:dn:cls:`~Microsoft.AspNetCore.Routing.RouteData` 의 :dn:prop:`~Microsoft.AspNetCore.Routing.RouteData.Values` 속성은 경로 핸들러에서 만든 *경로 값들*의 사전입니다. 이 값들은 보통 URL 을 토큰화하는 과정에서 결정되고, 사용자의 입력을 수용할 때 쓰이거나 혹은 어플리케이션 내에서 더 복잡한 어떤 선택을 할 때 쓰입니다.

:dn:cls:`~Microsoft.AspNetCore.Routing.RouteData` 의 :dn:prop:`~Microsoft.AspNetCore.Routing.RouteData.DataTokens` 속성은 일치한 경로와 관련된 추가적인 데이터들의 ``PropertyBag`` 입니다. ``DataTokens`` 는 각 경로의 상태 데이터와 관련되어 있어, 어플리케이션에서 어떤 경로에 일치하는가에 따라 선택을 해야 할 때 사용합니다. 이 값들은 개발자가 정의한 것으로서 라우팅의 행태에 *어떠한* 영향도 끼치지 않습니다. 게다가, ``DataTokens`` 속성 에 저장된 값들은 어떠한 형이라도 괜찮습니다. 이는 ``Values`` 속성 내의 경로 값들은 문자열로 쉽게 변환 가능해야 한다는 점과는 차이가 있습니다.

:dn:cls:`~Microsoft.AspNetCore.Routing.RouteData` 의 :dn:prop:`~Microsoft.AspNetCore.Routing.RouteData.Routers` 속성은 요청 일치 확인에 성공한 모든 경로 핸들러의 목록입니다. 경로 핸들러들은 서로 중첩될 수 있고, ``Routers`` 속성은 URL 일치 확인 결과에 따라 생성된 경로 핸들러들의 로직  트리를 반영합니다. 일반적으로 ``Routers`` 속성의 첫 번째 항목은 경로 핸들러 콜렉션 개체로서, URL 생성 시에 사용되어야 합니다. 마지만 항목은 일치에 성공한 경로 핸들러 입니다.

URL 생성
^^^^^^^^^^^^^^
URL 생성은 라우팅 미들웨어가 경로 값들을 기반으로 URL 을 만드는 절차입니다. 이를 통해 핸들러와 핸들러에 접근하는 URL 을 논리적으로 분리할 수 있습니다.

URL 생성은 다른 비슷한 반복적인 절차들과 마찬가지로서, 사용자나 프레임워크가 경로 핸들러 콜렉션에 대해 :dn:method:`~Microsoft.AspNetCore.Routing.IRouter.GetVirtualPath` 메서드를 호출합니다. 각각의 *경로* 의 ``GetVirtualPath`` 메서드를 null 이 아닌 :dn:cls:`~Microsoft.AspNetCore.Routing.VirtualPathData` 이 반환될 때까지 호출합니다.

``GetVirtualPath`` 에 대한 가장 중요한 입력값들은 다음과 같습니다.:

- :dn:cls:`~Microsoft.AspNetCore.Routing.VirtualPathContext` 의 :dn:prop:`~Microsoft.AspNetCore.Routing.VirtualPathContext.HttpContext` 속성
- :dn:cls:`~Microsoft.AspNetCore.Routing.VirtualPathContext` 의 :dn:prop:`~Microsoft.AspNetCore.Routing.VirtualPathContext.Values` 속성
- :dn:cls:`~Microsoft.AspNetCore.Routing.VirtualPathContext` 의 :dn:prop:`~Microsoft.AspNetCore.Routing.VirtualPathContext.AmbientValues` 속성

경로 핸들러는 우선 ``Values`` 와 ``AmbientValues`` 로 제공되는 경로 값을 사용하여, 어디서 URL 을 생성할 수 있고 어떤 값을 포함해야 할지를 결정합니다. ``AmbientValues`` 는 라우팅 시스템에서 현재 요청에 대한 일치 확인을 하는 과정에서 생성된 경로 값들의 모음입니다. 대조적으로 ``Values`` 는 현재 동작에 적합한 URL 을 어떻게 생성할지를 지정하는 경로 값들입니다. ``HttpContext`` 는 경로 핸들러에서 서비스나 현재의 컨텍스트와 관련 추가 데이터가 필요할 때 사용합니다. 

.. tip:: ``Values`` 를 ``AmbientValues`` 에 대한 
.. tip:: Think of ``Values`` as being a set of overrides for the ``AmbientValues``. URL generation tries to reuse route values from the current request to make it easy to generate URLs for links using the same route or route values.

The output of ``GetVirtualPath`` is a :dn:cls:`~Microsoft.AspNetCore.Routing.VirtualPathData`. ``VirtualPathData`` is a parallel of ``RouteData``; it contains the ``VirtualPath`` for the output URL as well as the some additional properties that should be set by the route.

The :dn:cls:`~Microsoft.AspNetCore.Routing.VirtualPathData` :dn:prop:`~Microsoft.AspNetCore.Routing.VirtualPathData.VirtualPath`
property contains the *virtual path* produced by the route. Depending on your needs you may need to process the path further. For instance, if you want to render the generated URL in HTML you need to prepend the base path of the application.

The :dn:cls:`~Microsoft.AspNetCore.Routing.VirtualPathData` :dn:prop:`~Microsoft.AspNetCore.Routing.VirtualPathData.Router` is a reference to the route that successfully generated the URL.

The :dn:cls:`~Microsoft.AspNetCore.Routing.VirtualPathData` :dn:prop:`~Microsoft.AspNetCore.Routing.VirtualPathData.DataTokens` properties is a dictionary of additional data related to the route that generated the URL. This is the parallel of ``RouteData.DataTokens``.

라우트 생성하기
^^^^^^^^^^^^^^^
Routing provides the :dn:cls:`~Microsoft.AspNetCore.Routing.Route` class as the standard implementation of ``IRouter``. ``Route`` uses the *route template* syntax to define patterns that will match against the URL path when :dn:method:`~Microsoft.AspNetCore.Routing.IRouter.RouteAsync` is called. ``Route`` will use the same route template to generate a URL when :dn:method:`~Microsoft.AspNetCore.Routing.IRouter.GetVirtualPath` is called.

Most applications will create routes by calling ``MapRoute`` or one of the similar extension methods defined on :dn:iface:`~Microsoft.AspNetCore.Routing.IRouteBuilder`. All of these methods will create an instance of ``Route`` and add it to the route collection.

.. note:: :dn:method:`~Microsoft.AspNetCore.Builder.MapRouteRouteBuilderExtensions.MapRoute` doesn't take a route handler parameter - it only adds routes that will be handled by the :dn:prop:`~Microsoft.AspNetCore.Routing.IRouteBuilder.DefaultHandler`. Since the default handler is an :dn:iface:`~Microsoft.AspNetCore.Routing.IRouter`, it may decide not to handle the request. For example, ASP.NET MVC is typically configured as a default handler that only handles requests that match an available controller and action. To learn more about routing to MVC, see :doc:`/mvc/controllers/routing`.

This is an example of a ``MapRoute`` call used by a typical ASP.NET MVC route definition:

.. code-block:: c#

    routes.MapRoute(
        name: "default",
        template: "{controller=Home}/{action=Index}/{id?}");

This template will match a URL path like ``/Products/Details/17`` and extract the route values ``{ controller = Products, action = Details, id = 17 }``. The route values are determined by splitting the URL path into segments, and matching each segment with the *route parameter* name in the route template. Route parameters are named. They are defined by enclosing the parameter name in braces ``{ }``.

The template above could also match the URL path ``/`` and would produce the values ``{ controller = Home, action = Index }``. This happens because the ``{controller}`` and ``{action}`` route parameters have default values, and the ``id`` route parameter is optional. An equals ``=`` sign followed by a value after the route parameter name defines a default value for the parameter. A question mark ``?`` after the route parameter name defines the parameter as optional. Route parameters with a default value *always* produce a route value when the route matches - optional parameters will not produce a route value if there was no corresponding URL path segment.

See route-template-reference_ for a thorough description of route template features and syntax.

This example includes a *route constraint*:

.. code-block:: c#

    routes.MapRoute(
        name: "default",
        template: "{controller=Home}/{action=Index}/{id:int}");

This template will match a URL path like ``/Products/Details/17``, but not ``/Products/Details/Apples``. The route parameter definition ``{id:int}`` defines a *route constraint* for the ``id`` route parameter. Route constraints implement ``IRouteConstraint`` and inspect route values to verify them. In this example the route value ``id`` must be convertable to an integer. See route-constraint-reference_ for a more detailed explaination of route constraints that are provided by the framework.

Additional overloads of ``MapRoute`` accept values for ``constraints``, ``dataTokens``, and ``defaults``. These additional parameters of ``MapRoute`` are defined as type ``object``. The typical usage of these parameters is to pass an anonymously typed object, where the property names of the anonymous type match route parameter names.

The following two examples create equivalent routes:

.. code-block:: c#

    routes.MapRoute(
        name: "default_route",
        template: "{controller}/{action}/{id?}",
        defaults: new { controller = "Home", action = "Index" });

    routes.MapRoute(
        name: "default_route",
        template: "{controller=Home}/{action=Index}/{id?}");

.. tip:: The inline syntax for defining constraints and defaults can be more convenient for simple routes. However, there are features such as data tokens which are not supported by inline syntax.

.. review-required: changed template and add MVC controller sample

This example demonstrates a few more features:

.. code-block:: c#

  routes.MapRoute(
    name: "blog",
    template: "Blog/{*article}",
    defaults: new { controller = "Blog", action = "ReadArticle" });

This template will match a URL path like ``/Blog/All-About-Routing/Introduction`` and will extract the values ``{ controller = Blog, action = ReadArticle, article = All-About-Routing/Introduction }``. The default route values for ``controller`` and ``action`` are produced by the route even though there are no corresponding route parameters in the template. Default values can be specified in the route template. The ``article`` route parameter is defined as a *catch-all* by the appearance of an asterix ``*`` before the route parameter name. Catch-all route parameters capture the remainder of the URL path, and can also match the empty string.

This example adds route constraints and data tokens:

.. code-block:: c#

  routes.MapRoute(
      name: "us_english_products",
      template: "en-US/Products/{id}",
      defaults: new { controller = "Products", action = "Details" },
      constraints: new { id = new IntRouteConstraint() },
      dataTokens: new { locale = "en-US" });

This template will match a URL path like ``/en-US/Products/5`` and will extract the values ``{ controller = Products, action = Details, id = 5 }`` and the data tokens ``{ locale = en-US }``.


.. image:: routing/_static/tokens.png

.. _url-generation:

URL 생성
^^^^^^^^^^^^^^^
The ``Route`` class can also perform URL generation by combining a set of route values with its route template. This is logically the reverse process of matching the URL path.

.. tip:: To better understand URL generation, imagine what URL you want to generate and then think about how a route template would match that URL. What values would be produced? This is the rough equivalent of how URL generation works in the ``Route`` class.

This example uses a basic ASP.NET MVC style route:

.. code-block:: c#

    routes.MapRoute(
        name: "default",
        template: "{controller=Home}/{action=Index}/{id?}");

With the route values ``{ controller = Products, action = List }``, this route will generate the URL ``/Products/List``. The route values are substituted for the corresponding route parameters to form the URL path. Since ``id`` is an optional route parameter, it's no problem that it doesn't have a value.

With the route values ``{ controller = Home, action = Index }``, this route will generate the URL ``/``. The route values that were provided match the default values so the segments corresponding to those values can be safely omitted. Note that both URLs generated would round-trip with this route definition and produce the same route values that were used to generate the URL.

.. tip:: An app using ASP.NET MVC should use :dn:cls:`~Microsoft.AspNetCore.Mvc.Routing.UrlHelper` to generate URLs instead of calling into routing directly.

For more details about the URL generation process, see url-generation-reference_.

.. _using-routing-middleware:

라우팅 미들웨어 사용하기
-------------------------
To use routing middleware, add it to the **dependencies** in *project.json*:

``"Microsoft.AspNetCore.Routing": <current version>``

Add routing to the service container in *Startup.cs*:

.. literalinclude:: routing/sample/RoutingSample/Startup.cs
  :dedent: 8
  :language: c#
  :lines: 11-14
  :emphasize-lines: 3

Routes must configured in the ``Configure`` method in the ``Startup`` class. The sample below uses these APIs:

- :dn:cls:`~Microsoft.AspNetCore.Routing.RouteBuilder`
- :dn:method:`~Microsoft.AspNetCore.Routing.RouteBuilder.Build`
- :dn:method:`~Microsoft.AspNetCore.Routing.RequestDelegateRouteBuilderExtensions.MapGet`  Matches only HTTP GET requests
- :dn:method:`~Microsoft.AspNetCore.Builder.RoutingBuilderExtensions.UseRouter`

.. literalinclude:: routing/sample/RoutingSample/Startup.cs
  :dedent: 8
  :start-after: // Routes must configured in Configure
  :end-before: // Show link generation when no routes match.

The table below shows the responses with the given URIs.

===================== ====================================================
URI                    응답
===================== ====================================================
/package/create/3     Hello! Route values: [operation, create], [id, 3]
/package/track/-3     Hello! Route values: [operation, track], [id, -3]
/package/track/-3/    Hello! Route values: [operation, track], [id, -3]
/package/track/       <Fall through, no match>
GET /hello/Joe        Hi, Joe!
POST /hello/Joe       <Fall through, matches HTTP GET only>
GET /hello/Joe/Smith  <Fall through, no match>
===================== ====================================================

If you are configuring a single route, call ``app.UseRouter`` passing in an ``IRouter`` instance. You won't need to call ``RouteBuilder``.

The framework provides a set of extension methods for creating routes such as:

- :dn:method:`~Microsoft.AspNetCore.Builder.MapRouteRouteBuilderExtensions.MapRoute`
- :dn:method:`~Microsoft.AspNetCore.Routing.RequestDelegateRouteBuilderExtensions.MapGet`
- :dn:method:`~Microsoft.AspNetCore.Routing.RequestDelegateRouteBuilderExtensions.MapPost`
- :dn:method:`~Microsoft.AspNetCore.Routing.RequestDelegateRouteBuilderExtensions.MapPut`
- :dn:method:`~Microsoft.AspNetCore.Routing.RequestDelegateRouteBuilderExtensions.MapDelete`
- :dn:method:`~Microsoft.AspNetCore.Routing.RequestDelegateRouteBuilderExtensions.MapVerb`

Some of these methods such as ``MapGet`` require a :dn:delegate:`~Microsoft.AspNetCore.Http.RequestDelegate` to be provided. The ``RequestDelegate`` will be used as the *route handler* when the route matches. Other methods in this family allow configuring a middleware pipeline which will be used as the route handler. If the *Map* method doesn't accept a handler, such as ``MapRoute``, then it will use the :dn:prop:`~Microsoft.AspNetCore.Routing.IRouteBuilder.DefaultHandler`.

The ``Map[Verb]`` methods use constraints to limit the route to the HTTP Verb in the method name. For example, see `MapGet <https://github.com/aspnet/Routing/blob/1.0.0/src/Microsoft.AspNetCore.Routing/RequestDelegateRouteBuilderExtensions.cs#L85-L88>`__ and `MapVerb <https://github.com/aspnet/Routing/blob/1.0.0/src/Microsoft.AspNetCore.Routing/RequestDelegateRouteBuilderExtensions.cs#L156-L180>`__.

.. _route-template-reference:

라우트 템플릿에 대한 참고사항
------------------------
Tokens within curly braces (``{ }``) define *route parameters* which will be bound if the route is matched. You can define more than one route parameter in a route segment, but they must be separated by a literal value. For example ``{controller=Home}{action=Index}`` would not be a valid route, since there is no literal value between ``{controller}`` and ``{action}``. These route parameters must have a name, and may have additional attributes specified.

Literal text other than route parameters (for example, ``{id}``) and the path separator ``/`` must match the text in the URL. Text matching is case-insensitive and based on the decoded representation of the URLs path. To match the literal route parameter delimiter ``{`` or  ``}``, escape it by repeating the character (``{{`` or ``}}``).

URL patterns that attempt to capture a filename with an optional file extension have additional considerations. For example, using the template ``files/{filename}.{ext?}`` -
When both ``filename`` and ``ext`` exist, both values will be populated. If only ``filename`` exists in the URL, the route matches because the trailing period ``.`` is  optional. The following URLs would match this route:

- ``/files/myFile.txt``
- ``/files/myFile.``
- ``/files/myFile``

You can use the ``*`` character as a prefix to a route parameter to bind to the rest of the URI - this is called a *catch-all* parameter. For example, ``blog/{*slug}`` would match any URI that started with ``/blog`` and had any value following it (which would be assigned to the ``slug`` route value). Catch-all parameters can also match the empty string.

Route parameters may have *default values*, designated by specifying the default after the parameter name, separated by an ``=``. For example, ``{controller=Home}`` would define ``Home`` as the default value for ``controller``. The default value is used if no value is present in the URL for the parameter. In addition to default values, route parameters may be optional (specified by appending a ``?`` to the end of the parameter name, as in ``id?``). The difference between optional and "has default" is that a route parameter with a default value always produces a value; an optional parameter has a value only when one is provided.

Route parameters may also have constraints, which must match the route value bound from the URL. Adding a colon ``:`` and constraint name after the route parameter name specifies an *inline constraint* on a route parameter. If the constraint requires arguments those are provided enclosed in parentheses ``( )`` after the constraint name. Multiple inline constraints can be specified by appending another colon ``:`` and constraint name. The constraint name is passed to the :dn:iface:`~Microsoft.AspNetCore.Routing.IInlineConstraintResolver` service to create an instance of :dn:iface:`~Microsoft.AspNetCore.Routing.IRouteConstraint` to use in URL processing. For example, the route template ``blog/{article:minlength(10)}`` specifies the ``minlength`` constraint with the argument ``10``. For more description route constraints, and a listing of the constraints provided by the framework, see route-constraint-reference_.

The following table demonstrates some route templates and their behavior.


+-----------------------------------+--------------------------------+------------------------------------------------+
| Route Template                    | URL 일치 확인 예시           | 비고                                          |
+===================================+================================+================================================+
| hello                             | | /hello                       | | Only matches the single path ‘/hello’        +
+-----------------------------------+--------------------------------+------------------------------------------------+
|{Page=Home}                        | | /                            | | Matches and sets ``Page`` to ``Home``        |
+-----------------------------------+--------------------------------+------------------------------------------------+
|{Page=Home}                        | | /Contact                     | | Matches and sets ``Page`` to ``Contact``     |
+-----------------------------------+--------------------------------+------------------------------------------------+
| {controller}/{action}/{id?}       | | /Products/List               | | Maps to ``Products`` controller and ``List`` |
|                                   | |                              | | action                                       |
+-----------------------------------+--------------------------------+------------------------------------------------+
| {controller}/{action}/{id?}       | | /Products/Details/123        | | Maps to ``Products`` controller and          |
|                                   | |                              | | ``Details`` action.  ``id`` set to 123       |
+-----------------------------------+--------------------------------+------------------------------------------------+
| {controller=Home}/                | |   /                          | | Maps to ``Home`` controller and ``Index``    |
|            {action=Index}/{id?}   | |                              | | method; ``id`` is ignored.                   |
+-----------------------------------+--------------------------------+------------------------------------------------+

Using a template is generally the simplest approach to routing. Constraints and defaults can also be specified outside the route template.

.. tip:: Enable :doc:`logging` to see how the built in routing implementations, such as ``Route``, match requests.

.. _route-constraint-reference:

라우트 제약사항에 대한 참고사항
--------------------------
Route constraints execute when a ``Route`` has matched the syntax of the incoming URL and tokenized the URL path into route values. Route constraints generally inspect the route value associated via the route template and make a simple yes/no decision about whether or not the value is acceptable. Some route constraints use data outside the route value to consider whether the request can be routed. For example, the `HttpMethodRouteConstraint <https://docs.asp.net/projects/api/en/latest/autoapi/Microsoft/AspNetCore/Routing/Constraints/HttpMethodRouteConstraint/index.html#httpmethodrouteconstraint-class>`_ can accept or reject a request based on its HTTP verb.

.. warning:: Avoid using constraints for **input validation**, because doing so means that invalid input will result in a 404 (Not Found) instead of a 400 with an appropriate error message. Route constraints should be used to **disambiguate** between similar routes, not to validate the inputs for a particular route.

The following table demonstrates some route constraints and their expected behavior.

.. TODO to-do when we migrate to MD, make sure this table doesn't require a scroll bar

.. list-table:: Inline Route Constraints
  :header-rows: 1

  * - constraint
    - Example
    - Example Match
    - Notes
  * - ``int``
    - {id:int}
    - 123
    - Matches any integer
  * - ``bool``
    - {active:bool}
    - true
    - Matches ``true`` or ``false``
  * - ``datetime``
    - {dob:datetime}
    - 2016-01-01
    - Matches a valid ``DateTime`` value (in the invariant culture - see `options <http://msdn.microsoft.com/en-us/library/aszyst2c(v=vs.110).aspx>`_)
  * - ``decimal``
    - {price:decimal}
    - 49.99
    - Matches a valid ``decimal`` value
  * - ``double``
    - {weight:double}
    - 4.234
    - Matches a valid ``double`` value
  * - ``float``
    - {weight:float}
    - 3.14
    - Matches a valid ``float`` value
  * - ``guid``
    - {id:guid}
    - 7342570B-<snip>
    - Matches a valid ``Guid`` value
  * - ``long``
    - {ticks:long}
    - 123456789
    - Matches a valid ``long`` value
  * - ``minlength(value)``
    - {username:minlength(5)}
    - steve
    - String must be at least 5 characters long.
  * - ``maxlength(value)``
    - {filename:maxlength(8)}
    - somefile
    - String must be no more than 8 characters long.
  * - ``length(min,max)``
    - {filename:length(4,16)}
    - Somefile.txt
    - String must be at least 8 and no more than 16 characters long.
  * - ``min(value)``
    - {age:min(18)}
    - 19
    - Value must be at least 18.
  * - ``max(value)``
    - {age:max(120)}
    - 91
    - Value must be no more than 120.
  * - ``range(min,max)``
    - {age:range(18,120)}
    - 91
    - Value must be at least 18 but no more than 120.
  * - ``alpha``
    - {name:alpha}
    - Steve
    - String must consist of alphabetical characters.
  * - ``regex(expression)``
    - {ssn:regex(^\d{3}-\d{2}-\d{4}$)}
    - 123-45-6789
    - String must match the provided regular expression.
  * - ``required``
    - {name:required}
    - Steve
    - Used to enforce that a non-parameter value is present during URL generation.

.. warning:: Route constraints that verify the URL can be converted to a CLR type (such as ``int`` or ``DateTime``) always use the invariant culture - they assume the URL is non-localizable. The framework-provided route constraints do not modify the values stored in route values. All route values parsed from the URL will be stored as strings. For example, the `Float route constraint <https://github.com/aspnet/Routing/blob/1.0.0/src/Microsoft.AspNetCore.Routing/Constraints/FloatRouteConstraint.cs#L44-L60>`__ will attempt to convert the route value to a float, but the converted value is used only to verify it can be converted to a float.

.. tip:: To constrain a parameter to a known set of possible values, you can use a regular expression ( for example ``{action:regex(list|get|create)}``. This would only match the ``action`` route value to ``list``, ``get``, or ``create``. If passed into the constraints dictionary, the string "list|get|create" would be equivalent. Constraints that are passed in the constraints dictionary (not inline within a template) that don't match one of the known constraints are also treated as regular expressions.

.. _url-generation-reference:

URL 생성에 대한 참고사항
------------------------
The example below shows how to generate a link to a route given a dictionary of route values and a ``RouteCollection``.

.. literalinclude:: routing/sample/RoutingSample/Startup.cs
  :start-after: // Show link generation when no routes match.
  :end-before: // End of app.Run
  :dedent: 12

The ``VirtualPath`` generated at the end of the sample above is ``/package/create/123``.

The second parameter to the :dn:cls:`~Microsoft.AspNetCore.Routing.VirtualPathContext` constructor is a collection of `ambient values`. Ambient values provide convenience by limiting the number of values a developer must specify within a certain request context. The current route values of the current request are considered ambient values for link generation. For example, in an ASP.NET MVC app if you are in the ``About`` action of the ``HomeController``, you don't need to specify the controller route value to link to the ``Index`` action (the ambient value of ``Home`` will be used).

Ambient values that don't match a parameter are ignored, and ambient values are also ignored when an explicitly-provided value overrides it, going from left to right in the URL.

Values that are explicitly provided but which don't match anything are added to the query string. The following table shows the result when using the route template ``{controller}/{action}/{id?}``.

.. list-table:: Generating links with ``{controller}/{action}/{id?}`` template
  :header-rows: 1


  * - Ambient Values
    - Explicit Values
    - Result

  * - controller="Home"
    - action="About"
    - ``/Home/About``
  * - controller="Home"
    - controller="Order",action="About"
    - ``/Order/About``
  * - controller="Home",color="Red"
    - action="About"
    - ``/Home/About``
  * - controller="Home"
    - action="About",color="Red"
    - ``/Home/About?color=Red``


If a route has a default value that doesn't correspond to a parameter and that value is explicitly provided, it must match the default value. For example:

.. code-block:: c#

  routes.MapRoute("blog_route", "blog/{*slug}",
    defaults: new { controller = "Blog", action = "ReadPost" });

Link generation would only generate a link for this route when the matching values for controller and action are provided.
