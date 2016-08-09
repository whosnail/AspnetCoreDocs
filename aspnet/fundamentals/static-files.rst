.. _fundamentals-static-files:

Working with Static Files
정적 파일 다루기
=========================
정적 파일 처리하기
=========================
By `Tom Archer`_

Static files, which include HTML files, CSS files, image files, and JavaScript files, are assets that the app will serve directly to clients. In this article, we'll cover the following topics as they relate to ASP.NET Core and static files.

정적 파일은 앱에서 클라이언트에게 바로 전달하는 자원으로서, HTML 파일과 CSS 파일, 이미지 파일, 자바스크립트 파일 등이 이에 해당합니다. 이번 장에서는 ASP.NET Core 와 정적 파일에 대한 다음과 같은 주제들을 살펴볼 것입니다.

.. contents:: Sections
  :local:
  :depth: 1

Serving static files
정적 파일 제공하기
--------------------
정적 파일 제공하기
--------------------

By default, static files are stored in the `webroot` of your project. The location of the webroot is defined in the project's ``hosting.json`` file where the default is `wwwroot`.

기본적으로 정적 파일은 여러분의 프로젝트 상의 `webroot` 에 저장합니다. webroot 의 위치는 프로젝트의 ``hosting.json`` 에서 정의합니다. 위치의 기본값은 `wwwroot` 입니다.

.. code-block:: json 

  "webroot": "wwwroot"

Static files can be stored in any folder under the webroot and accessed with a relative path to that root. For example, when you create a default Web application project using Visual Studio, there are several folders created within the webroot folder - ``css``, ``images``, and ``js``. In order to directly access an image in the ``images`` subfolder, the URL would look like the following:

정적 파일은 webroot 디렉토리 하위의 어떠한 폴더에도 저장할 수 있고, webroot 에 대한 상대 경로를 통해 접근할 수 있습니다. 예를 들어, Visual Studio 에서 기본 웹 어플리케이션 프로젝트를 생성해보면, webroot 폴더에 ``css``와 ``images``, ``js`` 폴더가 생성되어 있음을 확인할 수 있습니다. 이 중 ``images`` 폴더에 있는 이미지 파일을 직접 접근하기 위해서는 다음과 같은 URL 을 사용하면 됩니다. 

  \http://<yourApp>/images/<imageFileName>

  \http://<여러분의 앱>/images/<이미지 파일의 이름>

In order for static files to be served, you must configure the :doc:`middleware` to add static files to the pipeline. This specific middleware can be configured by adding a dependency on the Microsoft.AspNetCore.StaticFiles package to your project and then calling the ``UseStaticFiles`` extension method from ``Startup.Configure`` as follows:

여러분이 정적 파일을 제공하기 위해서는, 정적 파일에 대한 처리 과정을 요청 처리경로에 추가하기 위해 :doc:`middleware` 를 설정해야 합니다. Microsoft.AspNetCore.StaticFiles 패키지에 대한 의존성을 프로젝트에 추가하고, 다음과 같이 ``Startup.Configure`` 에서 ``UseStaticFiles`` 확장 메서드를 호출합니다.:

.. code-block:: c#
  :emphasize-lines: 5

  public void Configure(IApplicationBuilder app, IHostingEnvironment env, ILoggerFactory loggerFactory)
  {
    ...
    // Add static files to the request pipeline.
    // 요청 처리경로에 정적 파일을 추가
    app.UseStaticFiles();
    ...

Now, let's say that you have a project hierarchy where the static files you wish to serve are outside the webroot. For example,let's take a simple layout like the following:

이제 프로젝트의 webroot 하위가 아닌 다른 위치에 정적 파일이 존재하는 경우에 대해 알아보겠습니다. 예를 들어, 다음과 같이 간단한 디렉토리 구조인 경우에 대해 설정해보겠습니다.

  - wwwroot

    - css
    - images
    - ...

  - MyStaticFiles

    - test.png

In order for the user to access test.png, you can configure the static files middleware as follows:

사용자가 test.png 에 접근할 수 있도록 하기 위해서, 여러분은 다음과 같은 정적 파일 미들웨어를 설정하면 됩니다.

.. code-block:: c#
  :emphasize-lines: 5-9

  public void Configure(IApplicationBuilder app, IHostingEnvironment env, ILoggerFactory loggerFactory)
  {
    ...
    // Add MyStaticFiles static files to the request pipeline.
    // 요청 처리경로에 MyStaticFiles 디렉토리 상의 정적 파일을 추가
    app.UseStaticFiles(new StaticFileOptions()
    {
        FileProvider = new PhysicalFileProvider(@"D:\Source\WebApplication1\src\WebApplication1\MyStaticFiles"),
        RequestPath = new PathString("/StaticFiles")
    });
    ...

At this point, if the user enters an address of ``http://<yourApp>/StaticFiles/test.png``, the ``test.png`` image will be served.

이제 사용자가 ``http://<여러분의 앱>/StaticFiles/test.png`` 라는 주소를 브라우저에서 입력하면, ``test.png`` 이미지 파일이 제공될 것입니다.

Enabling directory browsing
디렉토리 브라우징 사용하기
---------------------------
디렉토리 브라우징 사용하기
---------------------------

Directory browsing allows the user of your Web app to see a list of directories and files within a specified directory (including the root). By default, this functionality is not available such that if the user attempts to display a directory within an ASP.NET Web app, the browser displays an error. To enable directory browsing for your Web app, call the ``UseDirectoryBrowser`` extension method from  ``Startup.Configure`` as follows:

<<<<<<< HEAD
디렉토리 브라우징을 통해 웹 어플리케이션의 사용자가 특정 디렉토리 (루트도 포함합니다.) 내의 디렉토리와 파일 목록을 확인할 수 있도록 할 수 있습니다. 기본적으로 이 기능은 꺼져있으므로, 사용자가 ASP.NET 웹 어플리케이션 내의 디렉토리를 확인하려하면 브라우저에서 오류를 보인다. 여러분의 웹 어플리케이션에서 디렉토리 브라우징을 켜려면, ``Startup.Configure`` 에서 다음과 같이 ``UseDirectoryBrowser`` 확장 메서드를 호출하세요.: 
=======
여러분은 디렉토리 브라우징 기능을 통해 웹 어플리케이션 사용자가 특정 디렉토리 내의 디렉토리들과 파일들의 목록을 확인할 수 있도록 허용할 수 있습니다. 디렉토리 브라우징은 '사용할 수 없음'이 기본 설정이기 때문에, 사용자가 ASP.NET 웹 어플리케이션의 디렉토리 내의 내용을 확인하려하면 브라우저는 오류를 표시합니다. 디렉토리 브라우징을 사용 가능하도록 설정하기 위해, ``Startup.Configure`` 에서 ``UseDirectoryBrowser`` 확장 메서드를 다음과 같이 호출하세요.:
>>>>>>> 1ae185ccc1ac283e8b63c776fa0fba71844370f6

.. code-block:: c#
  :emphasize-lines: 5

  public void Configure(IApplicationBuilder app, IHostingEnvironment env, ILoggerFactory loggerFactory)
  {
    ...
    // Turn on directory browsing for the current directory.
    // 현재 디렉토리에 대한 디렉토리 브라우징을 켬
    app.UseDirectoryBrowser();
    ...

The following figure illustrates the results of browsing to the Web app's ``images`` folder with directory browsing turned on:

<<<<<<< HEAD
다음 그림에서 디렉토리 브라우징을 켰을 때의 웹 어플리케이션의 ``images`` 폴더에 대한 브라우징 결과를 확인할 수 있습니다.:
=======
디렉토리 브라우징을 켰을 때 웹 어플리케이션의 ``images`` 폴더를 브라우징하면 확인할 수 있는 결과는 다음 그림과 같습니다.:
>>>>>>> 1ae185ccc1ac283e8b63c776fa0fba71844370f6

.. image:: static-files/_static/dir-browse.png

Now, let's say that you have a project hierarchy where you want the user to be able to browse a directory that is not in the webroot. For example, let's take a simple layout like the following:

<<<<<<< HEAD
이제 여러분이 webroot 하위에 있지 않은 디렉토리에 대한 브라우징을 사용자에게 제공하고자 한다고 해보겠습니다. 예를 들면, 다음과 같은 간단한 구조라고 해보겠습니다.:
=======
이제 여러분이 프로젝트 구조 상 사용자에게 webroot 외부에 있는 디렉토리에 대한 브라우징을 허용해야 한다고 가정해보겠습니다. 예를 들어, 다음과 같이 간단한 디렉토리 구조라고 하겠습니다.
>>>>>>> 1ae185ccc1ac283e8b63c776fa0fba71844370f6

  - wwwroot

    - css
    - images
    - ...

  - MyStaticFiles

In order for the user to browse the ``MyStaticFiles`` directory, you can configure the static files middleware as follows:

<<<<<<< HEAD
사용자가 ``MyStaticFiles`` 디렉토리를 브라우징할 수 있도록 하기 위해서, 다음과 같이 정적 파일 미들웨어를 설정할 수 있습니다.
=======
사용자가 ``MyStaticFiles`` 디렉토리를 브라우징하기 위해서 여러분은 다음과 같이 정적 파일 미들웨어를 설정할 수 있습니다.
>>>>>>> 1ae185ccc1ac283e8b63c776fa0fba71844370f6

.. code-block:: c#
  :emphasize-lines: 5-9

  public void Configure(IApplicationBuilder app, IHostingEnvironment env, ILoggerFactory loggerFactory)
  {
    ...
    // Add the ability for the user to browse the MyStaticFiles directory.
    // 사용자가 MyStaticFiles 디렉토리를 브라우징할 수 있도록 하는 기능을 추가
    app.UseDirectoryBrowser(new DirectoryBrowserOptions()
    {
        FileProvider = new PhysicalFileProvider(@"D:\Source\WebApplication1\src\WebApplication1\MyStaticFiles"),
        RequestPath = new PathString("/StaticFiles")
    });
    ...

At this point, if the user enters an address of ``http://<yourApp>/StaticFiles``, the browser will display the files in the ``MyStaticFiles`` directory.

<<<<<<< HEAD
이제 사용자가 ``http://<여러분의 앱>/StaticFiles`` 주소를 입력하면 브라우저에서 ``MyStaticFiles`` 디렉토리 내의 파일을 보여줄 것입니다.
=======
이제 사용자가 주소 ``http://<여러분의 앱>/StaticFiles`` 를 브라우저 주소창에 입력하게 되면, 브라우저에서 ``MyStaticFiles`` 디렉토리 내의 파일들을 보여주게 될 것입니다.
>>>>>>> 1ae185ccc1ac283e8b63c776fa0fba71844370f6

Serving a default document
기본 문서 제공하기
--------------------------
기본 문서 제공하기
--------------------------

Setting a default home page gives site visitors a place to start when visiting your site. Without a default site users will see a blank page unless they enter a fully qualified URI to a document.  In order for your Web app to serve a default page without the user having to fully qualify the URI, call the ``UseDefaultFiles`` extension method from ``Startup.Configure`` as follows.

<<<<<<< HEAD
기본 홈 페이지를 설정하면 사이트 방문자들이 여러분의 사이트를 방문할 때 시작할 지점을 제공할 수 있습니다. 기본 문서를 지정하지 않으면 사이트 방문자가 어떤 문서에 대한 전체 URI 주소를 입력하지 않았을 때 빈 페이지를 보게 될 것입니다. 사용자가 전체 URI 주소를 입력하지 않고록 기본 페이지를 볼 수 있도록 하기 위해서, 다음과 같이 ``Startup.Configure`` 에서 ``UseDefaultFiles`` 확장 메서드를 호출하세요.  
=======
기본 홈 페이지를 지정하면 여러분의 사이트에 대한 방문자에게 시작 지점을 제공할 수 있습니다. 기본 페이지가 없다면, 사이트 사용자는 어떤 페이지에 대한 완전한 URI 를 입력하지 않았을 때 공백 페이지를 보게 될 것입니다. 여러분의 웹 어플리케이션에서 사용자가 완전한 URI 를 입력하지 않았을 때도 기본 페이지를 제공하기 위해서는, ``Startup.Configure`` 에서 ``UseDefaultFiles`` 확장 메서드를 다음과 같이 호출하세요.
>>>>>>> 1ae185ccc1ac283e8b63c776fa0fba71844370f6

.. code-block:: c#
  :emphasize-lines: 5-6

  public void Configure(IApplicationBuilder app, IHostingEnvironment env, ILoggerFactory loggerFactory)
  {
    ...
    // Serve the default file, if present.
    // 존재한다면 기본 파일 제공
    app.UseDefaultFiles();
    app.UseStaticFiles();
    ...

.. note:: ``UseDefaultFiles`` must be called before ``UseStaticFiles`` or it will not serve up the default home page. You must still call ``UseStaticFiles``. ``UseDefaultFiles`` is a URL re-writer that doesn't actually serve the file. You must still specify middleware (UseStaticFiles, in this case) to serve the file.

<<<<<<< HEAD
.. note:: 
=======
.. note:: ``UseDefaultFiles`` 메서드를 ``UseStaticFiles`` 메서드보다 먼저 호출해야 합니다. 그렇지 않으면 기본 홈 페이지를 제공할 수 없습니다. 또한 ``UseDefaultFiles`` 메서드를 통해 설정하는 미들웨어는 파일을 직접 제공하지 않고 URL 을 재작성하기만 합니다. 따라서 실제로 파일을 제공하는 미들웨어를 설정하도록 ``UseStaticFiles`` 메서드를 반드시 호출해야 합니다.
>>>>>>> 1ae185ccc1ac283e8b63c776fa0fba71844370f6

If you call the ``UseDefaultFiles`` extension method and the user enters a URI of a folder, the middleware will search (in order) for one of the following files. If one of these files is found, that file will be used as if the user had entered the fully qualified URI (although the browser URL will continue to show the URI entered by the user).

여러분은 ``UseDefaultFiles`` 확장 메서드를 호출하였고 사용자는 어떤 폴더에 대한 URI 를 브라우저 주소창에 입력하였다면, 미들웨어에서 다음 파일들 중 하나를 (차례대로) 찾아봅니다. 이 파일들 중 하나를 찾는다면, 사용자가 전체 URI 경로를 입력한 것처럼 파일을 제공할 것입니다. (하지만 브라우저에는 사용자가 입력한 그대로의 URI 가 표시될 것입니다.)

  - default.htm
  - default.html
  - index.htm
  - index.html

To specify a different default file from the ones listed above, instantiate a ``DefaultFilesOptions`` object and set its ``DefaultFileNames`` string list to a list of names appropriate for your app. Then, call one of the overloaded ``UseDefaultFiles`` methods passing it the ``DefaultFilesOptions`` object. The following example code removes all of the default files from the ``DefaultFileNames`` list and adds  ``mydefault.html`` as the only default file for which to search.

위에서 나열한 파일들 외에 다른 파일을 기본 페이지로 지정하기 위해서는, ``DefaultFilesOptions`` 개체를 생성하고 해당 개체의 ``DefaultFileNames`` 문자열 목록에 적절한 파일 이름을 추가하세요. 그런 뒤, ``UseDefaultFiles`` 메서드 중 ``DefaultFilesOptions`` 개체를 매개변수로 받는 것을 호출하세요. 다음의 예제에서는 ``DefaultFileNames`` 목록에서 기본적으로 지정된 파일들을 모두 제거하고 ``mydefault.html`` 을 유일한 기본 파일로서 지정하고 있습니다.

.. code-block:: c#
  :emphasize-lines: 5-9

  public void Configure(IApplicationBuilder app, IHostingEnvironment env, ILoggerFactory loggerFactory)
  {
    ...
    // Serve my app-specific default file, if present.
    DefaultFilesOptions options = new DefaultFilesOptions();
    options.DefaultFileNames.Clear();
    options.DefaultFileNames.Add("mydefault.html");
    app.UseDefaultFiles(options);
    app.UseStaticFiles();
    ...

Now, if the user browses to a directory in the webroot with a file named ``mydefault.html``, that file will be served as though the user typed in the fully qualified URI.

이제, 사용자가 webroot 하위의 어떤 디렉토리에서 ``mydefault.html`` 이라는 파일로 브라우징을 하게 되면, 사용자가 해당 파일에 대한 전체 URI 를 입력한 것 처럼 제공할 것입니다.

But, what if you want to serve a default page from a directory that is outside the webroot directory? You could call both the ``UseStaticFiles`` and ``UseDefaultFiles`` methods passing in identical values for each method's parameters. However, it's much more convenient and recommended to call the ``UseFileServer`` method, which is covered in the next section.

하지만, 여러분이 webroot 디렉토리 외부의 디렉토리에 있는 어떤 기본 페이지를 제공하고 싶은 경우에는 어떻게 해야할까요? ``UseStaticFiles`` 메서드와 ``UseDefaultFiles`` 메서드에 동일한 파일 옵션 개체를 매개변수로서 전달하면 될 것입니다. 하지만, 이보다 훨씬 간단하게 ``UseFileServer`` 메서드를 사용하여 처리하는 방법을 다음 단락에서 알아보겠습니다.

Using the UseFileServer method
UseFileServer 메서드 사용하기
------------------------------
UseFileServer 메서드 사용하기
------------------------------

In addition to the ``UseStaticFiles``, ``UseDefaultFiles``, and ``UseDirectoryBrowser`` extensions methods, there is also a single method - ``UseFileServer`` - that combines the functionality of all three methods. The following example code shows some common ways to use this method:

``UseStaticFiles`` 와 ``UseDefaultFiles``, ``UseDirectoryBrowser`` 확장 메서드 외에 다른 메서드도 있습니다. ``UseFileServer`` 메서드로서 위 3가지 메서드의 기능을 통합한 메서드입니다. 다음 예제에서 이 메서드를 사용하는 몇 가지 일반적인 방법들을 확인할 수 있습니다.

.. code-block:: c#

  // Enable all static file middleware (serving of static files and default files) EXCEPT directory browsing.
  // 디렉토리 브라우징을 제외한 모든 정적 파일 관련 미들웨어 사용 (정적 파일 및 기본 파일 제공)
  app.UseFileServer();

.. code-block:: c#

  // Enables all static file middleware (serving of static files, default files, and directory browsing).
  // 모든 정적 파일 관련 미들웨어 사용 (정적 파일 및 기본 파일, 디렉토리 브라우징 제공)
  app.UseFileServer(enableDirectoryBrowsing: true);

As with the ``UseStaticFiles``, ``UseDefaultFiles``, and ``UseDirectoryBrowser`` methods, if you wish to serve files that exist outside the webroot, you instantiate and configure an "options" object that you pass as a parameter to ``UseFileServer``. For example, let's say you have the following directory hierarchy in your Web app:

``UseStaticFiles`` 과 ``UseDefaultFiles``, ``UseDirectoryBrowser`` 메서드와 마찬가지로 webroot 외부의 파일을 제공하길 원한다면, "options" 개체를 생성하고 설정한 뒤에 ``UseFileServer`` 메서드에 매개변수로 전달하세요. 예를 들어, 여러분의 웹 어플리케이션에서 다음과 같은 디렉토리 구조를 사용한다고 가정해보겠습니다.

- wwwroot

  - css
  - images
  - ...

- MyStaticFiles

  - test.png
  - default.html

Using the hierarchy example above, you might want to enable static files, default files, and browsing for the ``MyStaticFiles`` directory. In the following code snippet, that is accomplished with a single call to ``UseFileServer``.

위 예시와 같은 구조를 사용한다면, 여러분은 ``MyStaticFiles`` 디렉토리 내의 정적 파일과 기본 파일의 제공 및 브라우징을 가능하도록 하길 원할 수 있습니다. 다음 코드 토막에서는 ``UseFileServer`` 에 대한 한 번의 호출로 그러한 외부 디렉토리에 대한 기능을 수행하고 있습니다.

.. code-block:: c#

  // Enable all static file middleware (serving of static files, default files,
  // and directory browsing) for the MyStaticFiles directory.
  // MyStaticFiles 디렉토리에 대한 모든 정적 파일 미들웨어 사용 (정적 파일 및 기본 파일, 디렉토리 브라우징 제공)
  app.UseFileServer(new FileServerOptions()
  {
      FileProvider = new PhysicalFileProvider(@"D:\Source\WebApplication1\src\WebApplication1\MyStaticFiles"),
      RequestPath = new PathString("/StaticFiles"),
      EnableDirectoryBrowsing = true
  });

Using the example hierarchy and code snippet from above, here's what happens if the user browses to various URIs:

위 예시에서의 디렉토리 구조와 코드 토막을 사용하면, 사용자가 여러 URI 를 브라우징할 때 다음과 같은 일이 벌어집니다.

  - ``http://<yourApp>/StaticFiles/test.png`` - The ``MyStaticFiles/test.png`` file will be served to and presented by the browser.
  - ``http://<여러분의 앱>/StaticFiles/test.png``
  - ``http://<yourApp>/StaticFiles`` - Since a default file is present (``MyStaticFiles/default.html``), that file will be served. If that file didn't exist, the browser would present a list of files in the ``MyStaticFiles`` directory (because the ``FileServerOptions.EnableDirectoryBrowsing`` property is set to ``true``).
  - ``http://<여러분의 앱>/StaticFiles``

Working with content types
콘텐츠 타입 다루기
--------------------------
콘텐츠 타입 처리하기
--------------------------

The ASP.NET static files middleware understands almost 400 known file content types. If the user attempts to reach a file of an unknown file type, the static file middleware will not attempt to serve the file.

Let's take the following directory/file hierarchy example to illustrate:

- wwwroot

  - css
  - images

    - test.image

  - ...

Using this hierarchy, you could enable static file serving and directory browsing with the following:

.. code-block:: c#
  :emphasize-lines: 5-6

  public void Configure(IApplicationBuilder app, IHostingEnvironment env, ILoggerFactory loggerFactory)
  {
    ...
    // Serve static files and allow directory browsing.
    app.UseDirectoryBrowser();
    app.UseStaticFiles();

If the user browses to ``http://<yourApp>/images``, a directory listing will be displayed by the browser that includes the ``test.image`` file. However, if the user clicks on that file, they will see a 404 error - even though the file obviously exists. In order to allow the serving of unknown file types, you could set the ``StaticFileOptions.ServeUnknownFileTypes`` property to ``true`` and specify a default content type via ``StaticFileOptions.DefaultContentType``. (Refer to this `list of common MIME content types <http://www.freeformatter.com/mime-types-list.html>`_.)

.. code-block:: c#
  :emphasize-lines: 5-10

  public void Configure(IApplicationBuilder app, IHostingEnvironment env, ILoggerFactory loggerFactory)
  {
    ...
    // Serve static files and allow directory browsing.
    app.UseDirectoryBrowser();
    app.UseStaticFiles(new StaticFileOptions
    {
      ServeUnknownFileTypes = true,
      DefaultContentType = "image/png"
    });

At this point, if the user browses to a file whose content type is unknown, the browser will treat it as an image and render it accordingly.

So far, you've seen how to specify a default content type for any file type that ASP.NET doesn't recognize. However, what if you have multiple file types that are unknown to ASP.NET? That's where the ``FileExtensionContentTypeProvider`` class comes in.

The ``FileExtensionContentTypeProvider`` class contains an internal collection that maps file extensions to MIME content types. To specify custom content types, simply instantiate a ``FileExtensionContentTypeProvider`` object and add a mapping to the ``FileExtensionContentTypeProvider.Mappings`` dictionary for each needed file extension/content type. In the following example, the code adds a mapping of the file extension ``.myapp`` to the MIME content type ``application/x-msdownload``.

.. code-block:: c#
  :emphasize-lines: 5-13

  public void Configure(IApplicationBuilder app, IHostingEnvironment env, ILoggerFactory loggerFactory)
  {
    ...

    // Allow directory browsing.
    app.UseDirectoryBrowser();

    // Set up custom content types - associating file extension to MIME type
    var provider = new FileExtensionContentTypeProvider();
    provider.Mappings.Add(".myapp", "application/x-msdownload");

    // Serve static files.
    app.UseStaticFiles(new StaticFileOptions { ContentTypeProvider = provider });

    ...

Now, if the user attempts to browse to any file with an extension of ``.myapp``, the user will be prompted to download the file (or it will happen automatically depending on the browser).

IIS Considerations
IIS 고려사항
------------------
IIS 관련 고려사항
------------------

ASP.NET Core applications hosted in IIS use the HTTP platform handler to forward all requests to the application including requests for static files. The IIS static file handler is not used because it won’t get a chance to handle the request before it is handled by the HTTP platform handler.

IIS 를 통해 호스팅하는 ASP.NET Core 어플리케이션에서는 정적 파일을 포함한 모든 요청을 HTTP 플랫폼 핸들러를 통해 전달합니다. HTTP 플랫폼 핸들러를 통해 처리되므로 IIS 정적 파일 핸들러는 사용되지 않습니다. 

Best practices
모범 사례
--------------
모범 사례
--------------

This section includes a list of best practices for working with static files:

이번 단락에서는 정적 파일을 다루는 모범 사례들을 확인해보겠습니다.

  - Code files (including C# and Razor files) should be placed outside of the app project's webroot. This creates a clean separation between your app's static (non-compilable) content and source code.
  - 코드 파일들 (C# 이나 Razor 파일들) 은 어플리케이션 프로젝트의 webroot 외부에 저장해야 합니다. 이를 통해 어플리케이션의 정적 콘텐트 (컴파일 할 수 없는 파일들) 과 소스 코드를 명확히 분리할 수 있습니다.

Summary
요약
-------
요약
-------
In this article, you learned how the static files middleware component in ASP.NET Core allows you to serve static files, enable directory browsing, and serve default files. You also saw how to work with content types that ASP.NET doesn't recognize. Finally, the article explained some IIS considerations and presented some best practices for working with static files.

여러분은 ASP.NET Core 에서 정적 파일 미들웨어 컴포넌트를 통해 어떻게 정적 파일을 제공하고 디렉토리 브라우징을 가능하게 하며 기본 파일을 제공하는지 확인하였습니다. 또한 ASP.NET 에서 인식하지 못하는 콘텐츠 타입을 다루는 방법도 확인하였습니다. 그리고 몇몇 IIS 관련 고려사항에 대해 알아보았고, 정적파일을 다루는 몇 가지 모범 사례도 확인하였습니다. 

Additional Resources
추가 자료
--------------------
추가 자료
--------------------

- :doc:`middleware`
