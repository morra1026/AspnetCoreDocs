Introduction to ASP.NET Core
============================

By `Daniel Roth`_, `Rick Anderson`_ and `Shaun Luttin <https://twitter.com/dicshaunary>`__

ASP.NET Core is a significant redesign of ASP.NET. This topic introduces the new concepts in ASP.NET Core and explains how they help you develop modern web apps.
ASP.NET Core를 설계 관점에서 한 마디로 설명하자면 ASP.NET의 심도깊은 재설계(significant redesign)라고 할 수 있다. 이 글은 ASP.NET Core의 새로운 개념을 소개하고 그 것들이 어떻게 모던 웹앱을 개발하는데 도움이 되는지 설명한다.

.. contents:: Sections:
  :local:
  :depth: 1

What is ASP.NET Core?
---------------------

ASP.NET Core is a new open-source and cross-platform framework for building modern cloud based internet connected applications, such as web apps, IoT apps and mobile backends. ASP.NET Core apps can run on `.NET Core <https://www.microsoft.com/net/core/platform>`__ or on the full .NET Framework. It was architected to provide an optimized development framework for apps that are deployed to the cloud or run on-premises. It consists of modular components with minimal overhead, so you retain flexibility while constructing your solutions. You can develop and run your ASP.NET Core apps cross-platform on Windows, Mac and Linux. ASP.NET Core is open source at `GitHub <https://github.com/aspnet/home>`_.
ASP.NET Core는 닷넷을 사용하여 클라우드 기반의 모던 웹 애플리케이션을 만들기 위한 새로운 오픈소스이자 크로스플랫폼 프레임워크다. 우리는 클라우드에 배포되거나 자체 호스팅 서버(on-premise)에서 동작하는 앱을 위한 최적의 개발 프레임워크를 제공하기 위해 프레임워크를 바닥부터 다시 만들었다. ASP.NET Core는 최소한의 오버헤드만을 갖도록 모듈화된 컴포넌트들로 구성되어 있기 때문에 솔루션을 구성하면서 유연성을 지킬 수 있다. 또한, ASP.NET 5 애플리이션을 윈도우, 맥, 리눅스에서 운영할 수 있다. ASP.NET Core는 GitHub 의 오픈 소스이다.

Why build ASP.NET Core?
-----------------------

The first preview release of ASP.NET came out almost 15 years ago as part of the .NET Framework.  Since then millions of developers have used it to build and run great web apps, and over the years we have added and evolved many capabilities to it.
ASP.NET 1.O의 최초 프리뷰가 나온 것이 어느 덧 15년 전의 일이다. 그 이래로 정말 많은 개발자들이 ASP.NET을 사용하여 훌륭한 웹 애플리케이션을 만들어 왔고 우리는 그 동안 수 많은 기능들을 추가하고 발전시켜왔다.

ASP.NET Core has a number of architectural changes that result in a much leaner and modular framework.  ASP.NET Core is no longer based on *System.Web.dll*. It is based on a set of granular and well factored `NuGet <http://www.nuget.org/>`__ packages. This allows you to optimize your app to include just the NuGet packages you need. The benefits of a smaller app surface area include tighter security, reduced servicing, improved performance, and decreased costs in a pay-for-what-you-use model.
ASP.NET Core에서 우리는 상당히 많은 아키텍쳐 변경을 통해 군더더기 없이 모듈화된 코어(core) 웹 프레임워크를 만들고 있다. ASP.NET Core는 System.Web.dll 에 더이상 기반하지 않고, 잘게 분리된 NuGet 패키지들에 기반해서 여러분의 요구에 최적화된 앱을 만들 수 있게 해준다. 애플리케이션이 필요 이상의 모듈을 포함하지 않기 때문에 보안상 개선 효과가 있고, 서비스 하는 부담 또한 줄여준다. 결국, 사용한만큼 지불하는(pay-for-what-you-use) 모델을 채택함으로써 애플리케이션의 성능이 개선된다.

With ASP.NET Core you gain the following foundational improvements:
정리하자면, ASP.NET Core에서는 다음과 같이 기초적으로 중대한 개선사항이 포함되었다 :

- A unified story for building web UI and web APIs
- Integration of :doc:`modern client-side frameworks </client-side/index>` and development workflows
- A cloud-ready environment-based :doc:`configuration system </fundamentals/configuration>`
- 클라우드를 위한 환경 기반 구성
- Built-in :doc:`dependency injection </fundamentals/dependency-injection>`
- 내장된 종속성 주입 기능
- New light-weight and modular HTTP request pipeline
- 새롭게 경량화되고 모듈화된 HTTP 요청 파이프라인
- Ability to host on IIS or self-host in your own process
- IIS 또는 개발자 자신의 프로세스에서 셀프 호스트할 수 있는 능력
- Built on `.NET Core`_, which supports true side-by-side app versioning
- 닷넷 코어에 기반한 진정한 sidy-by-side 앱 버전 관리(versioning)
- Ships entirely as `NuGet`_  packages
- 모든 기능이 NuGet 패키지 형태로 추가
- New tooling that simplifies modern web development
- 모던 웹 개발을 단순화 시킨 새로운 도구들(tooling)
- Build and run cross-platform ASP.NET apps on Windows, Mac and Linux
- 윈도우, 맥, 리눅스에서 개발하고 실행할 수 있는 크로스플랫폼
- Open source and community focused
- 오픈 소스와 커뮤니티에 초점

Application anatomy
-------------------

.. comment In RC1, The work of the WebHostBuilder was hidden in dnx.exe

An ASP.NET Core app is simply a console app that creates a web server in its ``Main`` method:
ASP.NET Core app은 ``Main`` method에서 web server를 만드는 간단한 console app이다 :

.. literalinclude:: /getting-started/sample/aspnetcoreapp/Program.cs
    :language: c#

``Main`` uses :dn:cls:`~Microsoft.AspNetCore.Hosting.WebHostBuilder`, which follows the builder pattern, to create a web application host. The builder has methods that define the web server (for example ``UseKestrel``) and the startup class (``UseStartup``). In the example above, the Kestrel web server is used, but other web servers can be specified. We'll show more about ``UseStartup`` in the next section. ``WebHostBuilder`` provides many optional methods including ``UseIISIntegration`` for hosting in IIS and IIS Express and ``UseContentRoot`` for specifying the root content directory. The ``Build`` and ``Run`` methods build the ``IWebHost`` that will host the app and start it listening for incoming HTTP requests.


Startup
---------------------------
The ``UseStartup`` method on ``WebHostBuilder`` specifies the ``Startup`` class for your app.

.. literalinclude:: /getting-started/sample/aspnetcoreapp/Program.cs
    :language: c#
    :lines: 6-17
    :dedent: 4
    :emphasize-lines: 7

The ``Startup`` class is where you define the request handling pipeline and where any services needed by the app are configured. The ``Startup`` class must be public and contain the following methods:

.. code-block:: c#

  public class Startup
  {
      public void ConfigureServices(IServiceCollection services)
      {
      }

      public void Configure(IApplicationBuilder app)
      {
      }
  }

- ``ConfigureServices`` defines the services (see Services_ below) used by your app (such as the ASP.NET MVC Core framework, Entity Framework Core, Identity, etc.)
- ``ConfigureServices`` 는 app에서 사용될 service(see Services below)들을 정의한다. (ASP.NET MVC Core framework, Entity Framework Core, Identity, 기타 service들)
- ``Configure`` defines the :doc:`middleware </fundamentals/middleware>` in the request pipeline
- ``Configure`` 는  request pipeline에서 :doc:`middleware </fundamentals/middleware>` 를 정의한다.
- See :doc:`/fundamentals/startup` for more details
- 보다 자세한 사항은 :doc:`/fundamentals/startup` 아티클을 참조하자.

Services
--------

A service is a component that is intended for common consumption in an application. Services are made available through dependency injection. ASP.NET Core includes a simple built-in inversion of control (IoC) container that supports constructor injection by default, but can be easily replaced with your IoC container of choice. In addition to its loose coupling benefit, DI makes services available throughout your app. For example, :doc:`Logging </fundamentals/logging>` is available throughout your app. See :doc:`/fundamentals/dependency-injection` for more details.
서비스는 애플리케이션에서 공통적으로 사용하고자 하는 목적의 컴포넌트다. 서비스들은 종속성 주입을 통해 사용이 가능하다. ASP.NET Core 는 간단한 내장(built-in) 제어 역전 (IoC) 컨테이너를 포함하고 있는데 생성자를 이용한 주입 방식을 기본으로 지원하고 있다. 그러나, 여러분이 사용하는 IoC 컨테이너로 쉽게 대체할 수도 있다. 자세한 내용은 :doc:`/fundamentals/dependency-injection` 아티클을 참조하자.

Middleware
----------

In ASP.NET Core you compose your request pipeline using :doc:`/fundamentals/middleware`. ASP.NET Core middleware performs asynchronous logic on an ``HttpContext`` and then either invokes the next middleware in the sequence or terminates the request directly. You generally "Use" middleware by taking a dependency on a NuGet package and invoking a corresponding ``UseXYZ`` extension method on the ``IApplicationBuilder`` in the ``Configure`` method.
ASP.NET Core에서는 :doc:`/fundamentals/middleware` 를 사용하여 요청 파이프라인을 구성한다. ASP.NET Core 미들웨어는 ``HttpContext`` 에 대해 비동기 로직을 수행하고 선택적으로 다음 미들웨어를 호출하거나 요청 처리 작업을 중단한다. 일반적으로 미들웨어를 사용하는 방식은  ``Configure`` 메서드에서 ``IApplicationBuilder`` 의 "Use"로 시작하는 확장 메서드를 실행하는 것이다.

ASP.NET Core comes with a rich set of prebuilt middleware:
ASP.NET Core는 다음과 같이 사전 작성된 풍부한 미들웨어를 제공한다 :

- :doc:`Static files </fundamentals/static-files>`
- :doc:`/fundamentals/routing`
- :doc:`/security/authentication/index`

You can also author your own :doc:`custom middleware </fundamentals/middleware>`.
여러분은 자신만의 :doc:`custom middleware </fundamentals/middleware>` 를 작성할 수도 있다.

You can use any `OWIN <http://owin.org>`_-based middleware with ASP.NET Core. See :doc:`/fundamentals/owin` for details. 
또한 `OWIN <http://owin.org>`_ 기반의 미들웨어를 ASP.NET Core와 함께 사용할 수 있다. 자세한 내용은 :doc:`/fundamentals/owin` 아티클을 참조하자.

Servers
-------

The ASP.NET Core hosting model does not directly listen for requests; rather it relies on an HTTP :doc:`server </fundamentals/servers>` implementation to forward the request to the application. The forwarded request is wrapped as a set of feature interfaces that the application then composes into an ``HttpContext``.  ASP.NET Core includes a managed cross-platform web server, called :ref:`Kestrel <kestrel>`, that you would typically run behind a production web server like `IIS <https://iis.net>`__ or `nginx <http://nginx.org>`__.
ASP.NET Core 호스팅 모델은 요청을 직접 수신하지 않는 대신, HTTP :doc:`server </fundamentals/servers>` 구현에 의지하여 애플리케이션에 대한 요청을 (HttpContext 와 작용하는 기능적인 인터페이스들의 모음으로서) 표면화한다. ASP.NET Core는 :ref:`Kestrel <kestrel>` 라고 불리는 매니지드 크로스플랫폼 웹서버를 포함한다. Kestrel은 여러분이 통상적으로 운영환경의 웹서버로 사용하고자 하는 `IIS <https://iis.net>`__ 또는 `nginx <http://nginx.org>`__ 같은 것이다.

.. _content-root-lbl:

Content root
------------

The content root is the base path to any content used by the app, such as its views and web content. By default the content root is the same as application base path for the executable hosting the app; an alternative location can be specified with `WebHostBuilder`.

.. _web-root-lbl:

Web root
--------

The web root of your app is the directory in your project for public, static resources like css, js, and image files. The static files middleware will only serve files from the web root directory (and sub-directories) by default. The web root path defaults to `<content root>/wwwroot`, but you can specify a different location using the `WebHostBuilder`.

Configuration
-------------

ASP.NET Core uses a new configuration model for handling simple name-value pairs. The new configuration model is not based on ``System.Configuration`` or *web.config*; rather, it pulls from an ordered set of configuration providers. The built-in configuration providers support a variety of file formats (XML, JSON, INI) and environment variables to enable environment-based configuration. You can also write your own custom configuration providers.

See :doc:`/fundamentals/configuration` for more information.

Environments
---------------------

Environments, like "Development" and "Production", are a first-class notion in ASP.NET Core and can  be set using environment variables. See :doc:`/fundamentals/environments` for more information.

Build web UI and web APIs using ASP.NET Core MVC
------------------------------------------------

- You can create well-factored and testable web apps that follow the Model-View-Controller (MVC) pattern. See :doc:`/mvc/index` and :doc:`/testing/index`.
- You can build HTTP services that support multiple formats and have full support for content negotiation. See :doc:`/mvc/models/formatting`
- `Razor <http://www.asp.net/web-pages/overview/getting-started/introducing-razor-syntax-c>`__ provides a productive language to create :doc:`Views </mvc/views/index>`
- :doc:`Tag Helpers </mvc/views/tag-helpers/intro>` enable server-side code to participate in creating and rendering HTML elements in Razor files
- You can create HTTP services with full support for content negotiation using custom or built-in formatters (JSON, XML)
- :doc:`/mvc/models/model-binding` automatically maps data from HTTP requests to action method parameters
- :doc:`/mvc/models/validation` automatically performs client and server side validation

Client-side development
-----------------------

ASP.NET Core is designed to integrate seamlessly with a variety of client-side frameworks, including :doc:`AngularJS </client-side/angular>`, :doc:`KnockoutJS </client-side/knockout>` and :doc:`Bootstrap </client-side/bootstrap>`. See :doc:`/client-side/index` for more details.
ASP.NET Core는 클라이언트 측 프레임워크와 매끄럽게 통합되도록 설계되었고 `AngularJS <https://angularjs.org/>`_ , `KnockoutJS <http://knockoutjs.com>`_ and `Bootstrap <http://getbootstrap.com/>`_ 같은 프레임워크를 포함한다. 이와 관련하여 자세한 내용은 :doc:`/client-side/index` 아티클을 참조하자.

Next steps
----------

- :doc:`/tutorials/first-mvc-app/index`
- :doc:`/tutorials/your-first-mac-aspnet`
- :doc:`/tutorials/first-web-api`
- :doc:`/fundamentals/index`
