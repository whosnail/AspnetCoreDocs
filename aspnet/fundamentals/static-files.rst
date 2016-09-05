.. _fundamentals-static-files:

정적 파일 다루기
=========================
By `Tom Archer`_

정적 파일은 ASP.NET Core 어플리케이션에서 클라이언트에게 바로 전달할 수 있는 자원으로서, HTML 파일과 CSS 파일, 이미지 파일, 자바스크립트 파일 등이 이에 해당합니다.

`샘플 코드를 확인하거나 다운로드 받으세요. <https://github.com/aspnet/Docs/tree/master/aspnet/fundamentals/static-files/sample>`

.. contents:: Sections
  :local:
  :depth: 1

정적 파일 제공하기
--------------------

기본적으로 정적 파일은 ``web root`` (*<콘텐트의 루트>/wwwroot*) 에 저장됩니다. 콘텐트의 루트와 웹 루트는 :doc:`/intro` 에서 더 확인해보세요. 여러분은 보통 현재 디렉토리를 콘텐트 루트로 지정하므로, 여러분의 프로젝트의 ``web root`` 는 개발 중에 확인할 수 있을 것입니다.

.. literalinclude:: ../../common/samples/WebApplication1/src/WebApplication1/Program.cs
  :language: c#
  :lines: 12-22
  :emphasize-lines: 5
  :dedent: 8


정적 파일은 ``web root`` 디렉토리 하위의 어떠한 폴더에도 저장할 수 있고, 그 루트에 대한 상대 경로를 통해 접근할 수 있습니다. 예를 들어, Visual Studio 에서 기본 웹 어플리케이션 프로젝트를 생성해보면, *webroot* 폴더에 ``css`` 와 ``images``, ``js`` 등의 몇 가지 폴더가 생성되어 있음을 확인할 수 있습니다. ``images`` 폴더에 있는 이미지 파일을 직접 접근하기 위한 URI 는 다음과 같습니다.: 

- \http://<어플리케이션>/images/<이미지 파일 이름>
- \http://localhost:9189/images/banner3.svg

여러분이 정적 파일을 제공하기 위해서는, 정적 파일에 대한 처리 과정을 요청 처리경로에 추가하기 위해 :doc:`middleware` 를 설정해야 합니다. 정적 파일 미들웨어를 설정하기 위해서는, Microsoft.AspNetCore.StaticFiles 패키지에 대한 의존성을 여러분의 프로젝트에 추가하고 ``Startup.Configure`` 에서 :dn:method:`~Microsoft.AspNetCore.Builder.StaticFileExtensions.UseStaticFiles` 확장 메서드를 호출합니다.:

.. literalinclude:: static-files/sample/StartupStaticFiles.cs
  :language: c#
  :start-after: >Configure
  :end-before: <Configure
  :emphasize-lines: 3
  :dedent: 8

``app.UseStaticFiles();`` 를 통해 ``web root`` (기본적으로 *wwwroot* 입니다.) 에 있는 정적 파일을 제공 가능하도록 설정할 수 있습니다. 이후에 ``UseStaticFiles`` 을 통해 다른 디렉토리에 있는 콘텐트를 제공 가능하도록 설정하는 방법도 확인해보겠습니다. 

여러분은 "Microsoft.AspNetCore.StaticFiles" 를 *project.json* 파일에 포함해야 합니다.

.. note:: ``web root`` 의 기본값은 *wwwroot* 디렉토리이지만, 여러분은 :dn:method:`~Microsoft.AspNetCore.Hosting.HostingAbstractionsWebHostBuilderExtensions.UseWebRoot` 를 통해 ``web root`` 디렉토리를 지정할 수 있습니다. 더 자세한 정보는 :doc:`/intro` 에서 확인하세요.

``web root`` 외부에 제공하고자 하는 정적 파일이 존재하는 프로젝트 구조가 있다고 가정해보겠습니다. 예를 들면 다음과 같습니다.:

  - wwwroot

    - css
    - images
    - ...

  - MyStaticFiles

    - test.png

*test.png* 파일에 접근하려는 요청을 처리하기 위해서, 다음과 같은 정적 파일 미들웨어를 설정하세요.

.. literalinclude:: static-files/sample/StartupTwoStaticFiles.cs
  :language: c#
  :start-after: >Configure
  :end-before: <Configure
  :emphasize-lines: 5-10
  :dedent: 8

``http://<어플리케이션>/StaticFiles/test.png`` 에 대한 요청에 대해 *test.png* 파일을 제공할 것입니다.

정적 파일 인증
---------------------------

정적 파일 모듈에서는 어떠한 인가 절차도 거치지 **않습니다**. 정적 파일 모듈을 통해 제공되는 모든 파일은, *wwwroot** 아래 있는 파일 까지 포함하여 모두 공개되어 있습니다. 인가를 통해 파일을 제공하기 위해서는 다음과 같은 처리를 해야 합니다.

- 정적 파일 미들웨어를 통해 접근 가능한 모든 디렉토리 (*wwwroot* 포함) 외부에 파일을 **저장하고**
- 컨트롤러의 동작에서 인가를 적용한 :dn:class:`~Microsoft.AspNetCore.Mvc.FileResult` 를 반환하도록 하여 파일을 제공합니다. 

디렉토리 브라우징 허용하기
---------------------------

디렉토리 브라우징을 통해 웹 어플리케이션의 사용자가 특정 디렉토리 내의 디렉토리들과 파일들의 목록을 확인 가능하도록 허용할 수 있습니다. 보안 때문에 디렉토리 브라우징은 기본적으로 꺼져있습니다. (고려사항_ 을 확인하세요.) 디렉토리 브라우징을 켜려면, ``Startup.Configure`` 에서 :dn:method:`~Microsoft.AspNetCore.Builder.DirectoryBrowserExtensions.UseDirectoryBrowser` 확장 메서드를 호출하세요.: 

.. literalinclude:: static-files/sample/StartupBrowse.cs
  :language: c#
  :start-after: >Configure
  :end-before: <Configure
  :dedent: 8

그리고 ``Startup.ConfigureServices`` 에서 :dn:method:`~Microsoft.Extensions.DependencyInjection.DirectoryBrowserServiceExtensions.AddDirectoryBrowser` 확장 메서드를 호출하여 필요한 서비스를 추가하세요. 

.. literalinclude:: static-files/sample/StartupBrowse.cs
  :language: c#
  :start-after: >Services
  :end-before: <Services
  :dedent: 8


위 코드에서 \http://<app>/MyImages URL 을 통해 *wwwroot/images* 폴더를 디렉토리 브라우징할 수 있도록 허용하였습니다. 각각의 파일과 폴더에 대한 링크는 다음과 같습니다.:

.. image:: static-files/_static/dir-browse.png

디렉토리 브라우징을 허용했을 때의 보안 위험에 대해서는 고려사항_ 을 확인하세요.

``app.UseStaticFiles`` 를 두 번 호출한 점에 주의하십시오. 첫 번째 호출은 *wwwroot* 폴더 내의 CSS 와 이미지, JavaScript 를 제공하기 위한 것입니다. 두 번째 호출은 *wwwroot/images* 폴더에 대한 디렉토리 브라운징을 \http://<app>/MyImages 을 통해 하기 위한 것입니다.: 

.. literalinclude:: static-files/sample/StartupBrowse.cs
  :language: c#
  :start-after: >Configure
  :end-before: <Configure
  :dedent: 8
  :emphasize-lines: 3,5

기본 문서 제공하기
--------------------------

기본 홈 페이지를 설정하면 사이트 방문자들이 여러분의 사이트를 방문할 때 시작할 지점을 제공할 수 있습니다. 사용자가 전체 URI 주소를 입력하지 않고도 기본 페이지를 볼 수 있도록 하기 위해서, 다음과 같이 ``Startup.Configure`` 에서 ``UseDefaultFiles`` 확장 메서드를 호출하세요.  

.. literalinclude:: static-files/sample/StartupEmpty.cs
  :language: c#
  :start-after: >Configure
  :end-before: <Configure
  :emphasize-lines: 3
  :dedent: 8

.. note:: :dn:method:`~Microsoft.AspNetCore.Builder.DefaultFilesExtensions.UseDefaultFiles` 메서드는 반드시 ``UseStaticFiles`` 메서드를 호출하기 전에 호출하십시오. ``UseDefaultFiles`` 메서드는 단지 URL 재생성기 일 뿐 파일을 실제로 제공하는 미들웨어는 아닙니다. 따라서 파일을 제공하기 위한 정적 파일 미들웨어 (``UseStaticFiles``) 를 사용하도록 설정해야 합니다.

:dn:method:`~Microsoft.AspNetCore.Builder.DefaultFilesExtensions.UseDefaultFiles` 를 통해, 어떤 폴더에 대한 요청을 처리하는 과정에서 다음 파일들을 찾아봅니다.

  - default.htm
  - default.html
  - index.htm
  - index.html

위 파일 중 첫 번째로 찾은 파일을 해당 요청이 그 파일에 대한 전체 URI 인 것처럼 제공합니다. (하지만 브라우저 상의 URL 은 요청한 URI 그대로 보여질 것입니다.)

다음 코드에서 *mydefault.html* 에 대한 기본 파일 이름을 바꾸는 방법을 확인할 수 있습니다.

.. literalinclude:: static-files/sample/StartupDefault.cs
  :language: c#
  :start-after: >Configure
  :end-before: <Configure
  :dedent: 8

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

주의사항
^^^^^^^^^^^^^^^^

요약
-------

여러분은 ASP.NET Core 에서 정적 파일 미들웨어 컴포넌트를 통해 어떻게 정적 파일을 제공하고 디렉토리 브라우징을 허용하며 기본 파일을 제공하는지 확인하였습니다. 또한 ASP.NET 에서 인식하지 못하는 콘텐츠 타입을 다루는 방법도 확인하였습니다. 그리고 몇몇 IIS 관련 고려사항에 대해 알아보았고, 정적파일을 다루는 몇 가지 모범 사례도 확인하였습니다. 

추가 자료
--------------------

- :doc:`middleware`
