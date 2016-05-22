Getting Started
시작하기
===============

1. Install `.NET Core`_
1. `.NET Core`_ 설치해주세요.

2. Create a new .NET Core project:
2. 신규 .NET Core 프로젝트를 생성해주세요.

  .. code-block:: console
    
    mkdir aspnetcoreapp
    cd aspnetcoreapp
    dotnet new

3. Update the *project.json* file to add the Kestrel HTTP server package as a dependency:
3. *project.json* 파일에서 Kestrel HTTP 서버 패키지를 종속성 (dependencies) 항목에 추가하여 저장해주세요.

  .. literalinclude:: getting-started/sample/aspnetcoreapp/project.json
    :language: c#
    :emphasize-lines: 11

4. Restore the packages:
4. 종속된 패키지들을 다시 불러와주세요.

  .. code-block:: console
    
    dotnet restore

5. Add a *Startup.cs* file that defines the request handling logic:
5. 요청을 처리할 로직을 정의하는 *Startup.cs* 파일을 추가해주세요.

  .. literalinclude:: getting-started/sample/aspnetcoreapp/Startup.cs
    :language: c#

6. Update the code in *Program.cs* to setup and start the Web host:
6. 웹 호스트를 설정하고 시작하는 코드를 *Program.cs* 파일에 추가해주세요.

  .. literalinclude:: getting-started/sample/aspnetcoreapp/Program.cs
    :language: c#
    :emphasize-lines: 2,10-15

7. Run the app  (the ``dotnet run`` command will build the app when it's out of date):
7. 앱을 실행해주세요. (``dotnet run`` 명령을 실행할 때 소스에 변경 사항이 있다면, 앱을 다시 빌드할 것입니다.)

  .. code-block:: console
  
    dotnet run

8. Browse to \http://localhost:5000:
8. \http://localhost:5000 을 브라우저에서 열어주세요.

  .. image:: getting-started/_static/running-output.png

Next steps
다음 단계
----------

- :doc:`/tutorials/first-mvc-app/index`
- :doc:`/tutorials/your-first-mac-aspnet`
- :doc:`/tutorials/first-web-api`
- :doc:`/fundamentals/index`
