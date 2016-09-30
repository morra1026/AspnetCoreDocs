:version: 1.0.0

Working with Multiple Environments
==================================

By `Steve Smith`_

ASP.NET Core introduces improved support for controlling application behavior across multiple environments, such as development, staging, and production. Environment variables are used to indicate which environment the application is running in, allowing the app to be configured appropriately.

ASP.NET Core는 개발, 스테이징, 운영서버등이 존재하는 다중 환경에서 애플리케이션의 행동을 환경에 맞게 조정하기 위해 개선된 방안을 소개하고 있다. 환경 변수(environment variables)에 정보를 기록한 후, 앱이 환경 변수를 참조하여 스스로 환경에 맞는 앱을 구성할 수 있도록 한 것이 그 개념이다.

.. contents:: Sections:
  :local:
  :depth: 1

`View or download sample code <https://github.com/aspnet/Docs/tree/master/aspnet/fundamentals/environments/sample>`__

`예제 다운로드   <https://github.com/aspnet/Docs/tree/master/aspnet/fundamentals/environments/sample>`__


Development, Staging, Production 
--------------------------------

ASP.NET Core references a particular `environment variable <https://github.com/aspnet/Home/wiki/Environment-Variables>`_, ``ASPNETCORE_ENVIRONMENT`` to describe the environment the application is currently running in. This variable can be set to any value you like, but three values are used by convention: ``Development``, ``Staging``, and ``Production``. You will find these values used in the samples and templates provided with ASP.NET Core.

ASP.NET Core는 특정 `환경 변수 <https://github.com/aspnet/Home/wiki/Environment-Variables>`_, ``ASPNETCORE_ENVIRONMENT`` 를 참조하여 애플리케이션이 현재 운영되고 있는 환경을 알아낸다. 환경 변수를 통해 운영환경을 애플리케이션에게 설명해 준다고 할 수 있다. 이 변수는 여러분이 원하는 값을 설정할 수 있지만 컨벤션에 의해 ``Development``, ``Staging``, ``Production`` 라는 세 가지 값을 사용한다. 이 값들은 ASP.NET Core와 함께 제공되는 예제와 템플릿 등에 사용되므로 쉽게 찾아볼 수 있다.

The current environment setting can be detected programmatically from within your application. In addition, you can use the Environment :doc:`tag helper </mvc/views/tag-helpers/index>` to include certain sections in your :doc:`view </mvc/views/index>` based on the current application environment.

애플리케이션에서는 프로그램적으로 현재 환경 설정을 탐지할 수 있다. 게다가 Envrionment :doc:`태그 헬퍼 </mvc/views/tag-helpers/index>` 를 사용하면 현재 애플리케이션 환경에만 사용할 컨텐츠를 :doc:`뷰 </mvc/views/index>` 에 포함시킬 수도 있다.

.. note:: The specified environment name is case insensitive. Whether you set the variable to ``Development`` or ``development`` or ``DEVELOPMENT`` the results will be the same.

.. note:: 환경 변수의 값으로 사용되는 환경 이름은 대소문자를 가리지 않는다. 변수를 어떻게 설정하든 - ``Development`` or ``development`` or ``DEVELOPMENT`` - 결과는 같다.

Development  
^^^^^^^^^^^

This should be the environment used when developing an application. When using Visual Studio, this setting can be specified in your project's debug profiles, such as for IIS Express, shown here:

Development는 애플리케이션을 개발할 때 사용되는 환경이 되어야 한다. 비주얼 스튜디오를 사용할 때, 이 설정은 프로젝트의 디버그 프로파일에 명시된다. 아래 그림과 같이 이 설정은 개발 머신의 IIS Express를 위한 것이다.

.. image:: environments/_static/project-properties-debug.png

When you modify the default settings created with the project, your changes are persisted in *launchSettings.json* in the ``Properties`` folder. This file holds settings specific to each profile Visual Studio is configured to use to launch the application, including any environment variables that should be used. (Debug profiles are discussed in more detail in :doc:`servers`). For example, after adding another profile configured to use IIS Express, but using an ``ASPNETCORE_ENVIRONMENT`` value of ``Staging``, the ``launchSettings.json`` file in our sample project is shown below:

프로젝트의 기본 설정을 수정하면 그 내용은 ``Properties`` 폴더의 *launchSettings.json* 파일에 저장된다. 이 파일은 비주얼 스튜디오가 애플리케이션을 구동하도록 설정되는 것과 관련하여, 어떤 환경 변수가 사용되어야 하는지를 포함한 모든 프로파일 설정을 갖고 있다. (디버그 프로파일은 :doc:`servers` 라는 문서에서 보다 자세한 내용을 다룬다). 예를 들어, IIS Express 가 다른 프로파일을 사용하도록 프로파일을 추가하고자 하는데, ``Staging`` 이라는 값을 ``ASPNETCORE_ENVIRONMENT`` 환경 변수로 사용하고 싶다면 아래의 예제 파일 ``launchSettings.json`` 과 같이 설정한다.

.. literalinclude:: environments/sample/src/Environments/Properties/launchSettings.json
  :language: javascript
  :caption: launchSettings.json
  :emphasize-lines: 15,22

.. note:: Changes made to project profiles or to *launchSettings.json* directly may not take effect until the web server used is restarted (in particular, Kestrel must be restarted before it will detect changes made to its environment).

.. note:: 프로젝트 프로파일 또는 *launchSettings.json* 파일에서 변경된 내용은 웹 서버가 재시작되기 전까지 반영되지 않을 수 있다. (특히 Kestrel 서버는 반드시 재시작되어야만 환경의 변화를 감지할 수 있다).

You can create multiple different launch profiles for various different configurations of your application, including those that require other environment variables.

필요에 따라 여러 개의 구동 프로파일(launch profile)을 만들어 애플리케이션에서 다양한 구성을 할 수 있다. 다른 환경 변수들을  필요로 하는 프로파일을 작성하는 것도 포함된다.

.. warning:: Environment variables stored in *launchSettings.json* are not secured in any way and will be part of the source code repository for your project, if you use one. **Never store credentials or other secret data in this file.** If you need a place to store such data, use the *Secret Manager* tool described in :ref:`security-app-secrets`.

.. warning:: *launchSettings.json* 파일에 저장되는 환경 변수들은 사실 안전하지 않다. 그리고, 프로젝트 소스 코드 저장소의 일부가 될 것이다. **신원정보 또는 보안정보를 절대로 이 파일에 저장해서는 안된다.** 그런 데이터를 저장하려는 장소가 필요하다면  :ref:`security-app-secrets` 문서에 설명된 *Secret Manager* 도구를 사용하도록 한다.

..  

Staging
^^^^^^^

By convention, a ``Staging`` environment is a pre-production environment used for final testing before deployment to production. Ideally, its physical characteristics should mirror that of production, so that any issues that may arise in production occur first in the staging environment, where they can be addressed without impact to users.

컨벤션에 따르면 ``Staging`` 환경은 운영에 가기 바로전 최종 테스트를 진행하는 사전 운영환경이다. 이 환경은 운영 환경의 물리적 특성을 동일하게 갖추는 것이 이상적이다. 그래서, 운영 환경에서 혹여나 발생할 만한 이슈를 ``Staging`` 환경에서 미리 포착하여 사용자에게 어떤 영향도 미치지 않고 해결할 수 있다.

Production
^^^^^^^^^^

The ``Production`` environment is the environment in which the application runs when it is live and being used by end users. This environment should be configured to maximize security, performance, and application robustness. Some common settings that a production environment might have that would differ from development include:

- Turn on caching
- Ensure all client-side resources are bundled, minified, and potentially served from a CDN
- Turn off diagnostic ErrorPages
- Turn on friendly error pages
- Enable production logging and monitoring (for example, `Application Insights <https://azure.microsoft.com/en-us/documentation/articles/app-insights-asp-net-five/>`_)

``Production`` 환경은 애플리케이션이 라이브 상태가 된 후 최종 사용자에 의해 사용될 때, 애플리케이션이 동작하는 환경이다. 이 환경은 보안, 성능, 견고함을 최대로 구성해야 한다. 개발 환경과 달리 운영 환경이 가져야 할 일반적인 설정은 다음과 같다.

- 캐싱 사용하기
- 모든 클라이언트 단 리소스를 같이 묶고(bundle), 최소화(minify)한다. CDN을 통해 클라이언트 단 파일을 서비스하는 것도 잠재적인 옵션이다
- 오류를 진단하기 위해 사용하는 오류 페이지를 사용하지 않는다
- 사용자 친화적인 오류 페이지를 사용한다
- 로깅, 모니터링을 활성화한다. (예, `Application Insights <https://azure.microsoft.com/en-us/documentation/articles/app-insights-asp-net-five/>`_)

This is by no means meant to be a complete list. It's best to avoid scattering environment checks in many parts of your application. Instead, the recommended approach is to perform such checks within the application's ``Startup`` class(es) wherever possible

이 리스트는 단지 예제일뿐 운영을 위한 완벽한 리스트가 아니다. 
애플리케이션에서 환경 관련한 확인이 필요하다면 여기 저기에서 비일관적으로 하는 것을 피하고 ``Startup`` 클래스와 같이 한 곳의 제한된 곳에서 확인하도록 하자.



Determining the environment at runtime
--------------------------------------

The :dn:iface:`~Microsoft.AspNetCore.Hosting.IHostingEnvironment` service provides the core abstraction for working with environments. This service is provided by the ASP.NET hosting layer, and can be injected into your startup logic via :doc:`dependency-injection`. The ASP.NET Core web site template in Visual Studio uses this approach to load environment-specific configuration files (if present) and to customize the app's error handling settings. In both cases, this behavior is achieved by referring to the currently specified environment by calling :dn:prop:`~Microsoft.AspNetCore.Hosting.IHostingEnvironment.EnvironmentName` or :dn:method:`~Microsoft.AspNetCore.Hosting.HostingEnvironmentExtensions.IsEnvironment` on the instance of :dn:iface:`~Microsoft.AspNetCore.Hosting.IHostingEnvironment` passed into the appropriate method.


:dn:iface:`~Microsoft.AspNetCore.Hosting.IHostingEnvironment` 서비스는 운영 환경과 관련된 핵심적인 추상화를 제공한다. 이 서비스는 ASP.NET 호스팅 단에서 제공되는데 :doc:`dependency-injection` 을 사용해 startup 로직에서 주입될 수도 있다. 비주얼 스튜디오의 ASP.NET Core 웹 사이트 템플릿이 이 접근법을 사용하는데, 특정 환경에 대한 구성 파일이 있다면 이 파일을 로드하고 오류 처리를 환경에 따라 다르게 하도록 구성한다. ``IHostingEnvironment`` 가 주입되던 그렇지 않던 환경에 따라 이런 식으로 처리할 수 있는데  :dn:iface:`~Microsoft.AspNetCore.Hosting.IHostingEnvironment` 인스턴스의 :dn:prop:`~Microsoft.AspNetCore.Hosting.IHostingEnvironment.EnvironmentName` 속성 또는 :dn:method:`~Microsoft.AspNetCore.Hosting.HostingEnvironmentExtensions.IsEnvironment` 메서드를 사용해서 가능하다.

.. note:: If you need to check whether the application is running in a particular environment, use ``env.IsEnvironment("environmentname")`` since it will correctly ignore case (instead of checking if ``env.EnvironmentName == "Development"`` for example).

.. note:: 애플리케이션이 특정 환경에서 동작하는지 확인할 필요가 있다면 ``env.IsEnvironment("environmentname")`` 을 사용하자. 이 문장은 ``env.EnvironmentName == "Development"`` 문장과 달리 대소문자를 구분하지 않는다. 

For example, you can use the following code in your Configure method to setup environment specific error handling:

예를 들어, 특정 환경에서의 오류 처리를 설정하려면 다음의 코드를 Configure 메서드에서 사용할 수 있다.

.. literalinclude:: environments/sample/src/Environments/Startup.cs
  :language: c#
  :dedent: 8
  :start-after: STARTSNIP:error-handling
  :end-before: ENDSNIP:error-handling

If the app is running in a ``Development`` environment, then it enables the runtime support necessary to use the "BrowserLink" feature in Visual Studio, development-specific error pages (which typically should not be run in production) and special database error pages (which provide a way to apply migrations and should therefore only be used in development). Otherwise, if the app is not running in a development environment, a standard error handling page is configured to be displayed in response to any unhandled exceptions.

``Development`` 환경에서 앱을 실행한다면, 비주얼 스튜디오에서 "BrowserLink" 기능을 사용하기 위해 런타임 지원 필요(runtime support necessary)가 활성화되는데, 이 기능은 개발용 오류 화면과 특별한 데이터베이스 오류 화면도 같이 지원한다. 개발용 오류 화면은 통상적으로 운영 환경에서 사용해서는 안되고, 데이터베이스 특정 오류 화면은 마이그레이션을 적용하는 방법을 제공하기 때문에 개발 환경에서만 사용되어야 한다. 반면, 앱이 개발 환경에서 실행되지 않을 경우에는, 표준 오류 화면을 통해 처리되지 않은 예외를 보여준다.

You may need to determine which content to send to the client at runtime, depending on the current environment. For example, in a development environment you generally serve non-minimized scripts and style sheets, which makes debugging easier. Production and test environments should serve the minified versions and generally from a CDN. You can do this using the Environment :doc:`tag helper </mvc/views/tag-helpers/intro>`. The Environment tag helper will only render its contents if the current environment matches one of the environments specified using the ``names`` attribute.

실행환경에 따라 어떤 컨텐트를 클라이언트에 보내야 할 지 결정해야 할 상황이 있을 수 있다. 예를 들면, 개발 환경에서는 디버깅 편의를 위해서 압축되지 않은 스크립트와 스타일 시트를 사용지만, 운영 환경과 테스트 환경에서는 응답 성능 향상을 위해 압축된 버전을 사용한다. 이 최소화된 버전은 일반적으로 CDN을 통해 받는다. :doc:`태그 헬퍼 </mvc/views/tag-helpers/intro>` 를 사용하여 이런 작업을 할 수 있는데, Environment 태그 헬퍼를 사용하면 현재 환경이 태그의 `name` 속성에 기재된 환경과 일치할 때만 해당 컨텐츠를 출력한다.

.. literalinclude:: environments/sample/src/Environments/Views/Shared/_Layout.cshtml
  :language: html
  :dedent: 4
  :start-after: STARTSNIP:env-tag-helper
  :end-before: ENDSNIP:env-tag-helper

To get started with using tag helpers in your application see :doc:`/mvc/views/tag-helpers/intro`.

여러분의 애플리케이션에서 태그 헬퍼 사용을 고려한다면 :doc:`/mvc/views/tag-helpers/intro` 문서를 참조하자.

Startup conventions
-------------------  

ASP.NET Core supports a convention-based approach to configuring an application's startup based on the current environment. You can also programmatically control how your application behaves according to which environment it is in, allowing you to create and manage your own conventions.

ASP.NET Core는 현재 환경에 따라 애플리케이션 구동방식을 구성할 수 있는 컨벤션 기반의 접근법을 지원한다. 여러분은 또한 프로그램적으로 애플리케이션의 행동 방식을 환경에 따라 제어할 수 있기 때문에 자신만의 컨벤션을 만들고 관리할 수도 있다.

When an ASP.NET Core application starts, the ``Startup`` class is used to bootstrap the application, load its configuration settings, etc. (:doc:`learn more about ASP.NET startup <startup>`). However, if a class exists named ``Startup{EnvironmentName}`` (for example ``StartupDevelopment``), and the ``ASPNETCORE_ENVIRONMENT`` environment variable matches that name, then that ``Startup`` class is used instead. Thus, you could configure ``Startup`` for development, but have a separate ``StartupProduction`` that would be used when the app is run in production. Or vice versa.

ASP.NET Core 애플리케이션이 시작될 때, ``Startup`` 클래스가 애플리케이션을 구동하고, 구성 설정등을 불러오는 등의 일을 한다 (:doc:`learn more about ASP.NET startup <startup>`). 그러나, ``Startup{EnvironmentName}`` 처럼, 환경 이름을 포함한 Startup 클래스를 정의하고 (예, ``StartupDevelopment``), ASPNETCORE_ENVIRONMENT 변수 값이 환경 이름과 일치한다면, 환경 특화된 클래스가 사용될 것이다. 따라서, 개발환경에 사용될 ``Startup`` 만을 구성할 수도 있겠지만 운영환경을 고려하여 ``StartupProduction`` 클래스를 별도로 가져갈 수도 있다. 혹은 그 반대로, ``Startup`` 을 운영환경에 사용하고 개발환경을 위한 ``StartupDevelopment`` 를 별도로 정의할 수도 있다.

In addition to using an entirely separate ``Startup`` class based on the current environment, you can also make adjustments to how the application is configured within a ``Startup`` class. The ``Configure()`` and ``ConfigureServices()`` methods support environment-specific versions similar to the ``Startup`` class itself, of the form ``Configure{EnvironmentName}()`` and ``Configure{EnvironmentName}Services()``. If you define a method ``ConfigureDevelopment()`` it will be called instead of ``Configure()`` when the environment is set to development. Likewise, ``ConfigureDevelopmentServices()`` would be called instead of ``ConfigureServices()`` in the same environment.

환경에 기초해 완벽하게 분리된 ``Startup`` 클래스를 운영하는 것과 더불어, ``Startup`` 클래스 내부에서 애플리케이션을 어떻게 구성할지 조정하는 것도 가능하다. ``Startup`` 클래스 자체가 환경에 특화된 별도 클래스를 가질 수 있는 것처럼 ``Configure()`` 와 ``ConfigureService()`` 메서드도  ``Configure{EnvironmentName}()`` 와 ``Configure{EnvironmentName}Services()`` 의 형태로 환경 특화된 메서드를 지원한다. 환경이 development로 지정되었을 때, ``ConfigureDevelopment()`` 메서드를 정의했다면 이 것이 ``Configure()`` 메서드 대신 호출될 것이다. 같은 환경이라면 앞서 방식과 동일하게 ``ConfigureServices()`` 메서드 대신  ``ConfigureDevelopmentServices()`` 메서드가 호출될 것이다.

Summary
-------

ASP.NET Core provides a number of features and conventions that allow developers to easily control how their applications behave in different environments. When publishing an application from development to staging to production, environment variables set appropriately for the environment allow for optimization of the application for debugging, testing, or production use, as appropriate.

ASP.NET Core 는 개발자들이 애플리케이션 배포 환경에 맞추어 관리하려고 할 때 필요한 많은 기능과 컨벤션을 제공한다. 개발환경에서부터 애플리케이션을 스테이징, 운영으로 배포할 때, 환경에 맞추어 설정된 환경 변수들은 애플리케이션을 디버깅, 테스팅 또는 운영 모드로 최적화시키는 역할을 한다. 

Additional Resources
--------------------

- :doc:`configuration`
- :doc:`/mvc/views/tag-helpers/intro`
