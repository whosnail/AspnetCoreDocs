Choosing the Right .NET For You on the Server
=============================================

By `Daniel Roth`_

ASP.NET Core is based on the `.NET Core`_ project model, which supports building applications that can run cross-platform on Windows, Mac and Linux. When building a .NET Core project you also have a choice of which .NET flavor to target your application at: .NET Framework (CLR), .NET Core (CoreCLR) or `Mono <http://mono-project.com>`_. Which .NET flavor should you choose? Let's look at the pros and cons of each one.

여러분의 서버에 적합한 닷넷 선택하기
============================

By `Daniel Roth`_

ASP.NET Core는 `.NET Core`_ 프로젝트를 기반으로 한다. 닷넷 코어 프로젝트는 윈도우, 맥, 리눅스에서 크로스플랫폼으로 동작하는 애플리케이션 작성할 수 있도록 지원하기 위해 시작되었다. 닷넷 코어 프로젝트를 작성할 때, 특정 닷넷을 대상으로 애플리케이션을 작성할 수 있는데 그 선택의 대상으로는 닷넷 프레임워크 (CLR), 닷넷 코어 (CoreCLR) or `모노 <http://mono-project.com>`_ 가 있다.

.NET Framework
--------------

The .NET Framework is the most well known and mature of the three options. The .NET Framework is a mature and fully featured framework that ships with Windows. The .NET Framework ecosystem is well established and has been around for well over a decade. The .NET Framework is production ready today and provides the highest level of compatibility for your existing applications and libraries.

The .NET Framework runs on Windows only. It is also a monolithic component with a large API surface area and a slower release cycle. While the code for the .NET Framework is `available for reference <http://referencesource.microsoft.com/>`_ it is not an active open source project.

닷넷 프레임워크
-------------

닷넷 프레임워크는 위에 언급한 세가지 옵션중에 가장 잘 알려진 성숙한 닷넷이다. 닷넷 프레임워크는 윈도우와 함께 배포되는 성숙하면서도 전체 기능을 탑재한 프레임워크다. 닷넷 프레임워크의 생태계는 잘 조성되어 있고 벌써 10년이나 훨씬 지나왔다. 닷넷 프레임워크는 당장 제품 제작에 사용할 수 있고 여러분이 갖고 있는 애플리케이션들과 라이브러리들을 위한 가장 높은 수준의 호환성을 제공한다.

닷넷 프레임워크는 윈도우 운영체제에서만 동작한다. 또한, 엄청나게 많은 API를 갖고 있는 비대한 컴포넌트이고 공개 발표하는 주기가 길다. 닷넷 프레임워크 코드가 `참조용으로 공개 <http://referencesource.microsoft.com/>`_ 되어 있긴 하지만 활성화된 오픈 소스 프로젝트는 아니다.

.NET Core
---------

.NET Core is a modular runtime and library implementation that includes a subset of the .NET Framework. .NET Core is supported on Windows, Mac and Linux. .NET Core consists of a set of libraries, called "CoreFX", and a small, optimized runtime, called "CoreCLR". .NET Core is open-source, so you can follow progress on the project and contribute to it on `GitHub <https://github.com/dotnet>`_.

The CoreCLR runtime (Microsoft.CoreCLR) and CoreFX libraries are distributed via `NuGet`_. Because .NET Core has been built as a componentized set of libraries you can limit the API surface area your application uses to just the pieces you need. You can also run .NET Core based applications on much more constrained environments (ex. `Windows Server Nano <http://blogs.technet.com/b/windowsserver/archive/2015/04/08/microsoft-announces-nano-server-for-modern-apps-and-cloud.aspx>`_).

The API factoring in .NET Core was updated to enable better componentization. This means that existing libraries built for the .NET Framework generally need to be recompiled to run on .NET Core. The .NET Core ecosystem is relatively new, but it is rapidly growing with the support of popular .NET packages like JSON.NET, AutoFac, xUnit.net and many others.

Developing on .NET Core allows you to target a single consistent platform that can run on multiple platforms. 

닷넷 코어
--------

닷넷 코어는 모듈화된 런타임이면서 닷넷 프레임워크의 부분 집합을 구현하는 라이브러리다. 닷넷 코어는 윈도우, 맥, 리눅스에서 모두 지원된다. 닷넷 코어는 "CoreFX"라고 하는 라이브러리들의 집합과, "CoreCLR" 이라고 불리는 작고 최적화된 런타임으로 구성된다. 닷넷 코어는 오픈 소스이므로 `GitHub <https://github.com/dotnet>`_ 에서 프로젝트의 진행상황을 볼 수 있고 소스 기여도 할 수 있다.

CoreCLR 런타임과 CoreFX 라이브러리들은 `NuGet`_ 을 통해 배포된다. 닷넷 코어는 컴포넌트화되어 작성되었기 때문에 애플리케이션에서 사용하는 API의 범위를 여러분이 필요한 수준으로만 제한할 수 있다. 또한, 닷넷 코어 기반의 애플리케이션들을 `Windows Server Nano <http://blogs.technet.com/b/windowsserver/archive/2015/04/08/microsoft-announces-nano-server-for-modern-apps-and-cloud.aspx>`_ 처럼 굉장히 제약된 환경에서 운영할 수 있다.

닷넷 코어의 API는 닷넷 프레임워크에 비해 좀더 컴포넌트화가 가능하도록 업데이트되었다. 이 것은 닷넷 프레임워크를 위해 작성된 기존 라이브러리들은 닷넷 코어에서 동작하도록 재컴파일되어야 한다는 것을 의미한다. 닷넷 코어의 생태계는 상대적으로 새로 탄생했지만 빠르게 성장하고 있다. 그 성장의 이면에서 JSON.NET, AutoFac, xUnit.net과 같은 인기 패키지들의 지원이 있다.

Mono
----

`Mono <http://mono-project.com>`_ is a port of the .NET Framework built primarily for non-Windows platforms. Mono is open source and cross-platform. It also shares a similar API factoring to the .NET Framework, so many existing managed libraries work on Mono today. Mono is a good proving ground for cross-platform development while cross-platform support in .NET Core matures.

모노
----

`모노 <http://mono-project.com>`_ 는 비윈도우 플랫폼을 위해 작성된 닷넷 프레임워크의 포팅이다. 모노는 오픈 소스이고 크로스플랫폼이다. 모노는 닷넷 프레임워크가 제공하는 것과 유사한 API를 공유하기 때문에 기존의 많은 매니지드 라이브러리들이 오늘날 모노에서도 동작한다. 닷넷 코어에서 크로스플랫폼 지원이 성숙해질 동안 모노는 크로스플랫폼 개발을 위한 좋은 시험 무대이다.

Summary
-------

The .NET Core project model makes .NET development available for more scenarios than ever before. With .NET Core you have the option to target your application at existing available .NET platforms. Which .NET flavor you pick will depend on your specific scenarios, timelines, feature requirements and compatibility requirements.

요약
----

닷넷 코어 프로젝트 모델 덕분에 과거 어느 때보다도 보다 많은 시나리오에서 닷넷 개발이 가능해졌다. 기존에 사용 가능한 환경(예, 리눅스)에서 동작하는 애플리케이션을 개발하는 것이 닷넷 코어덕에 닷넷 개발자의 선택사항이 되었다. 어떤 닷넷을 선택하느냐 하는 것은 여러분의 시나리오, 개발 기간, 기능 요구사항, 호환성 요구사항에 달려있다.

