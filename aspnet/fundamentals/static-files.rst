.. _fundamentals-static-files:

정적 파일 다루기
=========================
By `Tom Archer`_

정적 파일은 앱에서 클라이언트에게 바로 전달하는 자원으로서, HTML 파일과 CSS 파일, 이미지 파일, 자바스크립트 파일 등이 이에 해당합니다. 이번 장에서는 ASP.NET Core 와 정적 파일에 대한 다음과 같은 주제들을 살펴볼 것입니다.

.. contents:: Sections
  :local:
  :depth: 1

정적 파일 제공하기
--------------------

기본적으로 정적 파일은 여러분의 프로젝트 상의 `webroot` 에 저장합니다. webroot 의 위치는 프로젝트의 ``hosting.json`` 에서 정의합니다. 위치의 기본값은 `wwwroot` 입니다.

.. code-block:: json 

  "webroot": "wwwroot"

정적 파일은 webroot 디렉토리 하위의 어떠한 폴더에도 저장할 수 있고, webroot 에 대한 상대 경로를 통해 접근할 수 있습니다. 예를 들어, Visual Studio 에서 기본 웹 어플리케이션 프로젝트를 생성해보면, webroot 폴더에 ``css`` 와 ``images``, ``js`` 폴더가 생성되어 있음을 확인할 수 있습니다. 이 중 ``images`` 폴더에 있는 이미지 파일을 직접 접근하기 위해서는 다음과 같은 URL 을 사용하면 됩니다. 

  \http://<여러분의 앱>/images/<이미지 파일의 이름>

여러분이 정적 파일을 제공하기 위해서는, 정적 파일에 대한 처리 과정을 요청 처리경로에 추가하기 위해 :doc:`middleware` 를 설정해야 합니다. Microsoft.AspNetCore.StaticFiles 패키지에 대한 의존성을 프로젝트에 추가하고, 다음과 같이 ``Startup.Configure`` 에서 ``UseStaticFiles`` 확장 메서드를 호출합니다.:

.. code-block:: c#
  :emphasize-lines: 5

  public void Configure(IApplicationBuilder app, IHostingEnvironment env, ILoggerFactory loggerFactory)
  {
    ...
    // 요청 처리경로에 정적 파일을 추가
    app.UseStaticFiles();
    ...

이제 프로젝트의 webroot 하위가 아닌 다른 위치에 정적 파일이 존재하는 경우에 대해 알아보겠습니다. 예를 들어, 다음과 같이 간단한 디렉토리 구조의 경우에 대해 설정해보겠습니다.

  - wwwroot

    - css
    - images
    - ...

  - MyStaticFiles

    - test.png

사용자가 test.png 에 접근할 수 있도록 하기 위해서, 여러분은 다음과 같은 정적 파일 미들웨어를 설정하면 됩니다.

.. code-block:: c#
  :emphasize-lines: 5-9

  public void Configure(IApplicationBuilder app, IHostingEnvironment env, ILoggerFactory loggerFactory)
  {
    ...
    // 요청 처리경로에 MyStaticFiles 디렉토리 상의 정적 파일을 추가
    app.UseStaticFiles(new StaticFileOptions()
    {
        FileProvider = new PhysicalFileProvider(@"D:\Source\WebApplication1\src\WebApplication1\MyStaticFiles"),
        RequestPath = new PathString("/StaticFiles")
    });
    ...

이제 사용자가 ``http://<여러분의 앱>/StaticFiles/test.png`` 라는 주소를 브라우저에서 입력하면, ``test.png`` 이미지 파일이 제공될 것입니다.

디렉토리 브라우징 허용하기
---------------------------

디렉토리 브라우징을 통해 웹 어플리케이션의 사용자가 특정 디렉토리 (루트도 포함합니다.) 내의 디렉토리와 파일 목록을 확인할 수 있도록 허용할 수 있습니다. 기본적으로 이 기능은 허용되어 있지 않으므로, 사용자가 ASP.NET 웹 어플리케이션 내의 디렉토리를 확인하려하면 브라우저에서 오류를 보입니다. 여러분의 웹 어플리케이션에서 디렉토리 브라우징을 켜려면, ``Startup.Configure`` 에서 다음과 같이 ``UseDirectoryBrowser`` 확장 메서드를 호출하세요.: 

.. code-block:: c#
  :emphasize-lines: 5

  public void Configure(IApplicationBuilder app, IHostingEnvironment env, ILoggerFactory loggerFactory)
  {
    ...
    // 현재 디렉토리에 대한 디렉토리 브라우징을 허용
    app.UseDirectoryBrowser();
    ...

다음 그림에서 디렉토리 브라우징을 허용했을 때 웹 어플리케이션의 ``images`` 폴더에 대한 브라우징 결과를 확인할 수 있습니다.:

.. image:: static-files/_static/dir-browse.png

이제 여러분이 webroot 하위에 있지 않은 디렉토리에 대한 브라우징을 사용자에게 허용하고자 한다고 해보겠습니다. 다음과 같은 간단한 구조에 대한 예를 확인해보겠습니다.:

  - wwwroot

    - css
    - images
    - ...

  - MyStaticFiles

사용자가 ``MyStaticFiles`` 디렉토리를 브라우징할 수 있도록 하기 위해, 여러분은 다음과 같이 정적 파일 미들웨어를 설정할 수 있습니다.

.. code-block:: c#
  :emphasize-lines: 5-9

  public void Configure(IApplicationBuilder app, IHostingEnvironment env, ILoggerFactory loggerFactory)
  {
    ...
    // 사용자가 MyStaticFiles 디렉토리를 브라우징할 수 있도록 하는 기능을 추가
    app.UseDirectoryBrowser(new DirectoryBrowserOptions()
    {
        FileProvider = new PhysicalFileProvider(@"D:\Source\WebApplication1\src\WebApplication1\MyStaticFiles"),
        RequestPath = new PathString("/StaticFiles")
    });
    ...

이제 사용자가 주소 ``http://<여러분의 앱>/StaticFiles`` 를 브라우저 주소창에 입력하면, 브라우저에서 ``MyStaticFiles`` 디렉토리 내의 파일들을 볼 수 있게 될 것입니다.

기본 문서 제공하기
--------------------------

기본 홈 페이지를 설정하면 사이트 방문자들이 여러분의 사이트를 방문할 때 시작할 지점을 제공할 수 있습니다. 기본 문서를 지정하지 않으면 사이트 방문자가 어떤 문서에 대한 전체 URI 주소를 입력하지 않았을 때 빈 페이지를 보게 될 것입니다. 사용자가 전체 URI 주소를 입력하지 않고도 기본 페이지를 볼 수 있도록 하기 위해서, 다음과 같이 ``Startup.Configure`` 에서 ``UseDefaultFiles`` 확장 메서드를 호출하세요.  

.. code-block:: c#
  :emphasize-lines: 5-6

  public void Configure(IApplicationBuilder app, IHostingEnvironment env, ILoggerFactory loggerFactory)
  {
    ...
    // 파일이 존재한다면, 기본 파일 제공
    app.UseDefaultFiles();
    app.UseStaticFiles();
    ...

.. note:: ``UseDefaultFiles`` 메서드는 반드시 ``UseStaticFiles`` 메서드를 호출하기 전에 호출하십시오. 그렇지 않으면 기본 홈 페이지를 제공할 수 없습니다. 여러분은 ``UseStaticFiles`` 메서드를 반드시 호출해야 합니다. ``UseDefaultFiles`` 메서드는 단지 URL 재생성기 일 뿐 파일을 실제로 제공하는 미들웨어는 아닙니다. 따라서 파일을 제공하기 위한 미들웨어를 지정해야 합니다. (이번 경우에는 UseStaticFiles 입니다.)

여러분은 ``UseDefaultFiles`` 확장 메서드를 호출하였고 사용자는 어떤 폴더에 대한 URI 를 브라우저 주소창에 입력하였다면, 미들웨어에서 다음 파일들 중 하나를 (차례대로) 찾아봅니다. 이 파일들 중 하나를 찾는다면, 사용자가 전체 URI 경로를 입력한 것처럼 파일을 제공할 것입니다. (하지만 브라우저에는 사용자가 입력한 그대로의 URI 가 표시될 것입니다.)

  - default.htm
  - default.html
  - index.htm
  - index.html

위에서 나열한 파일들 외에 다른 파일을 기본 페이지로 지정하기 위해서는, ``DefaultFilesOptions`` 개체를 생성하고 해당 개체의 ``DefaultFileNames`` 문자열 목록에 적절한 파일 이름을 추가하세요. 그런 뒤, ``UseDefaultFiles`` 메서드 중 ``DefaultFilesOptions`` 개체를 매개변수로 받는 것을 호출하세요. 다음의 예제에서는 ``DefaultFileNames`` 목록에서 기본적으로 지정된 파일들을 모두 제거하고 ``mydefault.html`` 을 유일한 기본 파일로서 지정하고 있습니다.

.. code-block:: c#
  :emphasize-lines: 5-9

  public void Configure(IApplicationBuilder app, IHostingEnvironment env, ILoggerFactory loggerFactory)
  {
    ...
    // 파일이 존재한다면, 특정 기본 파일 제공
    DefaultFilesOptions options = new DefaultFilesOptions();
    options.DefaultFileNames.Clear();
    options.DefaultFileNames.Add("mydefault.html");
    app.UseDefaultFiles(options);
    app.UseStaticFiles();
    ...

이제, 사용자가 webroot 하위의 어떤 디렉토리에서 ``mydefault.html`` 이라는 파일로 브라우징을 하게 되면, 사용자가 해당 파일에 대한 전체 URI 를 입력한 것 처럼 제공할 것입니다.

하지만, 여러분이 webroot 디렉토리 외부의 디렉토리에 있는 어떤 기본 페이지를 제공하고 싶은 경우에는 어떻게 해야할까요? ``UseStaticFiles`` 메서드와 ``UseDefaultFiles`` 메서드에 동일한 파일 옵션 개체를 매개변수로서 전달하면 될 것입니다. 하지만, 이보다 훨씬 간단하게 ``UseFileServer`` 메서드를 사용하여 처리하는 방법을 다음 단락에서 알아보겠습니다.

UseFileServer 메서드 사용하기
------------------------------

``UseStaticFiles`` 와 ``UseDefaultFiles``, ``UseDirectoryBrowser`` 확장 메서드 외에 다른 메서드도 있습니다. ``UseFileServer`` 메서드로서 위 3가지 메서드의 기능을 통합한 메서드입니다. 다음 예제에서 이 메서드를 사용하는 몇 가지 일반적인 방법들을 확인할 수 있습니다.

.. code-block:: c#

  // 디렉토리 브라우징을 제외한 모든 정적 파일 관련 미들웨어 허용 (정적 파일 및 기본 파일 제공)
  app.UseFileServer();

.. code-block:: c#

  // 모든 정적 파일 관련 미들웨어 허용 (정적 파일 및 기본 파일, 디렉토리 브라우징 제공)
  app.UseFileServer(enableDirectoryBrowsing: true);

``UseStaticFiles`` 과 ``UseDefaultFiles``, ``UseDirectoryBrowser`` 메서드와 마찬가지로 webroot 외부의 파일을 제공하길 원한다면, "options" 개체를 생성하고 설정한 뒤에 ``UseFileServer`` 메서드에 매개변수로 전달하세요. 예를 들어, 여러분의 웹 어플리케이션에서 다음과 같은 디렉토리 구조를 사용한다고 가정해보겠습니다.

- wwwroot

  - css
  - images
  - ...

- MyStaticFiles

  - test.png
  - default.html

위 예시와 같은 구조를 사용한다면, 여러분은 ``MyStaticFiles`` 디렉토리 내의 정적 파일과 기본 파일의 제공 및 브라우징을 가능하도록 하길 원할 수 있습니다. 다음 코드 토막에서는 ``UseFileServer`` 에 대한 한 번의 호출로 그러한 외부 디렉토리에 대한 기능을 수행하고 있습니다.

.. code-block:: c#

  // MyStaticFiles 디렉토리에 대한 모든 정적 파일 미들웨어 허용 (정적 파일 및 기본 파일, 디렉토리 브라우징 제공)
  app.UseFileServer(new FileServerOptions()
  {
      FileProvider = new PhysicalFileProvider(@"D:\Source\WebApplication1\src\WebApplication1\MyStaticFiles"),
      RequestPath = new PathString("/StaticFiles"),
      EnableDirectoryBrowsing = true
  });

위 예시에서의 디렉토리 구조와 코드 토막을 사용하면, 사용자가 여러 URI 를 브라우징할 때 다음과 같은 일이 벌어집니다.

  - ``http://<여러분의 앱>/StaticFiles/test.png`` - ``MyStaticFiles/test.png`` 파일이 제공되고 브라우저에 표시될 것입니다.
  - ``http://<여러분의 앱>/StaticFiles`` - 기본 파일이 존재하기 때문에 (``MyStaticFiles/default.html``), 해당 파일이 제공될 것입니다. 파일이 존재하지 않는다면, 브라우저에 ``MyStaticFiles`` 디렉토리 내의 파일 목록이 표시될 것입니다. (``FileServerOptions.EnableDirectoryBrowsing`` 속성을 ``true`` 로 설정했기 때문입니다.)

콘텐트 타입 다루기
--------------------------

ASP.NET 정적 파일 미들웨어는 약 400개의 알려진 콘텐츠 타입을 인식합니다. 사용자가 미들웨어에서 인식할 수 없는 파일 타입에 접근하려면, 정적 파일 미들웨어는 파일을 제공하지 않을 것입니다.

다음 디렉토리/파일 구조의 예시에서 확인해보겠습니다.

- wwwroot

  - css
  - images

    - test.image

  - ...

이 구조에 대해 정적 파일 미들웨어와 디렉토리 브라우징 미들웨어를 다음과 같이 허용하도록 설정하겠습니다.

.. code-block:: c#
  :emphasize-lines: 5-6

  public void Configure(IApplicationBuilder app, IHostingEnvironment env, ILoggerFactory loggerFactory)
  {
    ...
    // 정적 파일 제공과 디렉토리 브라우징을 허용
    app.UseDirectoryBrowser();
    app.UseStaticFiles();

사용자가 ``http://<여러분의 앱>/images`` 를 브라우징한다면, 브라우저에는 ``test.image`` 를 포함하는 디렉토리 목록이 보일 것입니다. 하지만, 사용자가 ``test.image`` 파일을 클릭하면, 파일이 존재한다 하더라도 사용자는 404 오류를 보게 될 것입니다. 인식할 수 없는 파일 타입을 제공하려면, ``StaticFileOptions.ServeUnknownFileTypes`` 속성을 ``true`` 로 설정하고 ``StaticFileOptions.DefaultContentType`` 속성에 기본 콘텐트 타입을 지정하세요. (`MIME 콘텐트 타입 목록 <http://www.freeformatter.com/mime-types-list.html>`_ 을 참고하세요.)

.. code-block:: c#
  :emphasize-lines: 5-10

  public void Configure(IApplicationBuilder app, IHostingEnvironment env, ILoggerFactory loggerFactory)
  {
    ...
    // 정적 파일 제공과 디렉토리 브라우징을 허용
    app.UseDirectoryBrowser();
    app.UseStaticFiles(new StaticFileOptions
    {
      ServeUnknownFileTypes = true,
      DefaultContentType = "image/png"
    });

이제, 사용자가 인식할 수 없는 콘텐트 타입인 파일로 브라우징하면, 브라우저는 이미지로 인식하고 이미지로서 화면에 표시하려 할 것입니다.

지금까지 여러분은 ASP.NET 이 인식할 수 없는 파일 타입에 기본 콘텐트 타입을 지정하는 방법을 확인하였습니다. 하지만, ASP.NET 에 여러 개의 파일 타입을 지정하려고 할 경우에는 어떻게 해야 할까요? 그럴 경우 ``FileExtensionContentTypeProvider`` 클래스를 사용하십시오.

``FileExtensionContentTypeProvider`` 클래스 내부에는 파일 확장자에 MIME 콘텐트 타입을 지정하는 콜렉션 개체가 있습니다. 개발자가 정의한 콘텐트 타입 (custom content type) 을 지정하기 위해서는, ``FileExtensionContentTypeProvider`` 개체를 생성하고 ``FileExtensionContentTypeProvider.Mappings`` 사전 (dictionary) 에 각각의 파일 확장자와 콘텐트 타입 간의 지정을 추가하면 됩니다. 다음 예제에서 ``.myapp`` 파일 확장과 ``application/x-msdownload`` MIME 콘텐트 타입 간의 연결을 추가하는 코드를 확인할 수 있습니다.

.. code-block:: c#
  :emphasize-lines: 5-13

  public void Configure(IApplicationBuilder app, IHostingEnvironment env, ILoggerFactory loggerFactory)
  {
    ...

    // 디렉토리 브라우징 허용
    app.UseDirectoryBrowser();

    // 개발자가 정의한 콘텐트 타입 설정하기 - 파일 확장자와 MIME 타입 간의 연결
    var provider = new FileExtensionContentTypeProvider();
    provider.Mappings.Add(".myapp", "application/x-msdownload");

    // 정적 파일 제공하기
    app.UseStaticFiles(new StaticFileOptions { ContentTypeProvider = provider });

    ...

이제 사용자가 ``.myapp`` 확장자인 파일을 브라우징하려하면, 사용자는 파일을 다운로드하려 한다는 알림을 확인하게 될 것입니다. (혹은 브라우저에 따라 자동으로 다운로드할 수도 있습니다.)

IIS 관련 고려사항
------------------

IIS 를 통해 호스팅하는 ASP.NET Core 어플리케이션에서는 정적 파일을 포함한 모든 요청을 HTTP 플랫폼 핸들러를 통해 전달합니다. HTTP 플랫폼 핸들러를 통해 처리되므로 IIS 정적 파일 핸들러는 사용되지 않습니다. 

모범 사례
--------------

이번 단락에서는 정적 파일을 다루는 모범 사례들을 확인해보겠습니다.

  - 코드 파일들 (C# 이나 Razor 파일들) 은 어플리케이션 프로젝트의 webroot 외부에 저장해야 합니다. 이를 통해 어플리케이션의 정적 콘텐트 (컴파일 할 수 없는 파일들) 과 소스 코드를 명확히 분리할 수 있습니다.

요약
-------

여러분은 ASP.NET Core 에서 정적 파일 미들웨어 컴포넌트를 통해 어떻게 정적 파일을 제공하고 디렉토리 브라우징을 허용하며 기본 파일을 제공하는지 확인하였습니다. 또한 ASP.NET 에서 인식하지 못하는 콘텐츠 타입을 다루는 방법도 확인하였습니다. 그리고 몇몇 IIS 관련 고려사항에 대해 알아보았고, 정적파일을 다루는 몇 가지 모범 사례도 확인하였습니다. 

추가 자료
--------------------

- :doc:`middleware`
