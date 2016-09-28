﻿.. _fundamentals-static-files:

Working with Static Files
=========================
By `Rick Anderson`_

Static files, such as HTML, CSS, image, and JavaScript, are assets that an ASP.NET Core app can serve directly to clients.
정적 파일은 ASP.NET Core 어플리케이션에서 클라이언트에게 바로 전달할 수 있는 자원으로서, HTML 파일과 CSS 파일, 이미지 파일, 자바스크립트 파일 등이 이에 해당합니다.

`View or download sample code <https://github.com/aspnet/Docs/tree/master/aspnet/fundamentals/static-files/sample>`__
`샘플 코드를 확인하거나 다운로드 받으세요. <https://github.com/aspnet/Docs/tree/master/aspnet/fundamentals/static-files/sample>`__

.. contents:: Sections
  :local:
  :depth: 1

Serving static files
--------------------

Static files are typically located in the ``web root`` (*<content-root>/wwwroot*) folder. See Content root and Web root in  :doc:`/intro` for more information. You generally set the content root to be the current directory so that your project's ``web root`` will be found while in development.
기본적으로 정적 파일은 ``web root`` (*<콘텐트 루트>/wwwroot*) 에 저장됩니다. 콘텐트 루트와 웹 루트는 :doc:`/intro` 에서 더 확인해보세요. 여러분은 보통 현재 디렉토리를 콘텐트 루트로 지정하므로, 여러분의 프로젝트의 ``web root`` 는 개발 중에 확인할 수 있을 것입니다.

.. literalinclude:: ../../common/samples/WebApplication1/src/WebApplication1/Program.cs
  :language: c#
  :lines: 12-22
  :emphasize-lines: 5
  :dedent: 8


Static files can be stored in any folder under the ``web root`` and accessed with a relative path to that root. For example, when you create a default Web application project using Visual Studio, there are several folders created within the *wwwroot*  folder - *css*, *images*, and *js*. The URI to access an image in the *images* subfolder:
정적 파일은 ``web root`` 디렉토리 하위의 어떠한 폴더에도 저장할 수 있고, 그 루트에 대한 상대 경로를 통해 접근할 수 있습니다. 예를 들어, Visual Studio 에서 기본 웹 어플리케이션 프로젝트를 생성해보면, *webroot* 폴더에 *css* 와 *images*, *js* 등의 몇 가지 폴더가 생성되어 있음을 확인할 수 있습니다. ``images`` 폴더에 있는 이미지 파일을 직접 접근하기 위한 URI 는 다음과 같습니다.: 

- \http://<app>/images/<imageFileName>
- \http://localhost:9189/images/banner3.svg


In order for static files to be served, you must configure the :doc:`middleware` to add static files to the pipeline. The static file middleware can be configured by adding a dependency on the *Microsoft.AspNetCore.StaticFiles* package to your project and then calling the :dn:method:`~Microsoft.AspNetCore.Builder.StaticFileExtensions.UseStaticFiles` extension method from ``Startup.Configure``:
여러분이 정적 파일을 제공하기 위해서는, 정적 파일에 대한 처리 과정을 요청 처리경로에 추가하기 위해 :doc:`middleware` 를 설정해야 합니다. 정적 파일 미들웨어를 설정하기 위해서는, *Microsoft.AspNetCore.StaticFiles* 패키지에 대한 의존성을 여러분의 프로젝트에 추가하고 ``Startup.Configure`` 에서 :dn:method:`~Microsoft.AspNetCore.Builder.StaticFileExtensions.UseStaticFiles` 확장 메서드를 호출합니다.:

.. literalinclude:: static-files/sample/StartupStaticFiles.cs
  :language: c#
  :start-after: >Configure
  :end-before: <Configure
  :emphasize-lines: 3
  :dedent: 8

``app.UseStaticFiles();`` makes the files in ``web root`` (*wwwroot* by default) servable. Later I'll show how to make other directory contents servable with ``UseStaticFiles``.
``app.UseStaticFiles();`` 를 통해 ``web root`` (기본적으로 *wwwroot* 입니다.) 에 있는 정적 파일을 제공 가능하도록 설정할 수 있습니다. 이후에 ``UseStaticFiles`` 을 통해 다른 디렉토리에 있는 콘텐트를 제공 가능하도록 설정하는 방법도 확인해보겠습니다. 

You must include "Microsoft.AspNetCore.StaticFiles" in the *project.json* file.
여러분은 "Microsoft.AspNetCore.StaticFiles" 를 *project.json* 파일에 포함해야 합니다.

.. note:: ``web root`` defaults to the *wwwroot* directory, but you can set the ``web root`` directory with :dn:method:`~Microsoft.AspNetCore.Hosting.HostingAbstractionsWebHostBuilderExtensions.UseWebRoot`. See :doc:`/intro` for more information.
.. note:: ``web root`` 의 기본값은 *wwwroot* 디렉토리이지만, 여러분은 :dn:method:`~Microsoft.AspNetCore.Hosting.HostingAbstractionsWebHostBuilderExtensions.UseWebRoot` 를 통해 ``web root`` 디렉토리를 지정할 수 있습니다. 더 자세한 정보는 :doc:`/intro` 에서 확인하세요.

Suppose you have a project hierarchy where the static files you wish to serve are outside the ``web root``. For example:
``web root`` 외부에 제공하고자 하는 정적 파일이 존재하는 프로젝트 구조가 있다고 가정해보겠습니다. 예를 들면 다음과 같습니다.:

  - wwwroot

    - css
    - images
    - ...

  - MyStaticFiles

    - test.png

For a request to access *test.png*, configure the static files middleware as follows:
*test.png* 파일에 접근하려는 요청을 처리하기 위해서, 다음과 같은 정적 파일 미들웨어를 설정하세요.

.. literalinclude:: static-files/sample/StartupTwoStaticFiles.cs
  :language: c#
  :start-after: >Configure
  :end-before: <Configure
  :emphasize-lines: 5-10
  :dedent: 8

A request to ``http://<app>/StaticFiles/test.png`` will serve the *test.png* file.
``http://<어플리케이션>/StaticFiles/test.png`` 에 대한 요청에 대해 *test.png* 파일을 제공할 것입니다.

Static file authorization
-------------------------

The static file module provides **no** authorization checks. Any files served by it, including those under *wwwroot* are publicly available. To serve files based on authorization:
정적 파일 모듈에서는 어떠한 인가 절차도 거치지 **않습니다**. 정적 파일 모듈을 통해 제공되는 모든 파일은, *wwwroot* 아래 있는 파일 까지 포함하여 모두 공개되어 있습니다. 인가를 통해 파일을 제공하기 위해서는 다음과 같은 처리를 해야 합니다.

- Store them outside of *wwwroot* and any directory accessible to the static file middleware **and**
- Serve them through a controller action, returning a :dn:class:`~Microsoft.AspNetCore.Mvc.FileResult` where authorization is applied
- 정적 파일 미들웨어를 통해 접근 가능한 *wwwroot* 포함하여 모든 디렉토리의 외부에 파일을 **저장하고**
- 인가를 적용한 :dn:class:`~Microsoft.AspNetCore.Mvc.FileResult` 를 컨트롤러의 동작에서 반환하도록 하여 파일을 **제공합니다**. 

Enabling directory browsing
---------------------------

Directory browsing allows the user of your web app to see a list of directories and files within a specified directory. Directory browsing is disabled by default for security reasons (see Considerations_). To enable directory browsing, call the :dn:method:`~Microsoft.AspNetCore.Builder.DirectoryBrowserExtensions.UseDirectoryBrowser` extension method from  ``Startup.Configure``:
디렉토리 브라우징을 통해 웹 어플리케이션의 사용자가 특정 디렉토리 내의 디렉토리들과 파일들의 목록을 확인 가능하도록 허용할 수 있습니다. 보안 때문에 디렉토리 브라우징은 기본적으로 꺼져있습니다. (고려사항_ 을 확인하세요.) 디렉토리 브라우징을 켜려면, ``Startup.Configure`` 에서 :dn:method:`~Microsoft.AspNetCore.Builder.DirectoryBrowserExtensions.UseDirectoryBrowser` 확장 메서드를 호출하세요.: 

.. literalinclude:: static-files/sample/StartupBrowse.cs
  :language: c#
  :start-after: >Configure
  :end-before: <Configure
  :dedent: 8

And add required services by calling :dn:method:`~Microsoft.Extensions.DependencyInjection.DirectoryBrowserServiceExtensions.AddDirectoryBrowser` extension method from  ``Startup.ConfigureServices``:
그리고 ``Startup.ConfigureServices`` 에서 :dn:method:`~Microsoft.Extensions.DependencyInjection.DirectoryBrowserServiceExtensions.AddDirectoryBrowser` 확장 메서드를 호출하여 필요한 서비스를 추가하세요. 

.. literalinclude:: static-files/sample/StartupBrowse.cs
  :language: c#
  :start-after: >Services
  :end-before: <Services
  :dedent: 8


The code above allows directory browsing of the *wwwroot/images* folder using the URL \http://<app>/MyImages, with links to each file and folder:
위 코드에서 \http://<어플리케이션>/MyImages URL 을 통해 *wwwroot/images* 폴더를 디렉토리 브라우징할 수 있도록 허용하였습니다. 각각의 파일과 폴더에 대한 링크는 다음과 같습니다.:

.. image:: static-files/_static/dir-browse.png


See Considerations_ on the security risks when enabling browsing.
디렉토리 브라우징을 허용했을 때의 보안 위험에 대해서는 고려사항_ 을 확인하세요.

Note the two ``app.UseStaticFiles`` calls. The first one is required to serve the CSS, images and JavaScript in the *wwwroot* folder, and the second call for directory browsing of the *wwwroot/images* folder using the URL \http://<app>/MyImages:
``app.UseStaticFiles`` 를 두 번 호출한 점에 주의하십시오. 첫 번째 호출은 *wwwroot* 폴더 내의 CSS 와 이미지, JavaScript 를 제공하기 위한 것입니다. 두 번째 호출은 *wwwroot/images* 폴더에 대한 디렉토리 브라운징을 \http://<어플리케이션>/MyImages 을 통해 하기 위한 것입니다.: 

.. literalinclude:: static-files/sample/StartupBrowse.cs
  :language: c#
  :start-after: >Configure
  :end-before: <Configure
  :dedent: 8
  :emphasize-lines: 3,5

Serving a default document
--------------------------

Setting a default home page gives site visitors a place to start when visiting your site. In order for your Web app to serve a default page without the user having to fully qualify the URI, call the ``UseDefaultFiles`` extension method from ``Startup.Configure`` as follows.
기본 홈 페이지를 설정하면 사이트 방문자들이 여러분의 사이트를 방문할 때 시작할 지점을 제공할 수 있습니다. 사용자가 전체 URI 주소를 입력하지 않고도 기본 페이지를 볼 수 있도록 하기 위해서, 다음과 같이 ``Startup.Configure`` 에서 ``UseDefaultFiles`` 확장 메서드를 호출하세요.  

.. literalinclude:: static-files/sample/StartupEmpty.cs
  :language: c#
  :start-after: >Configure
  :end-before: <Configure
  :emphasize-lines: 3
  :dedent: 8

.. note:: :dn:method:`~Microsoft.AspNetCore.Builder.DefaultFilesExtensions.UseDefaultFiles` must be called before ``UseStaticFiles`` to serve the default file. ``UseDefaultFiles`` is a URL re-writer that doesn't actually serve the file. You must enable the static file middleware (``UseStaticFiles``) to serve the file.
.. note:: :dn:method:`~Microsoft.AspNetCore.Builder.DefaultFilesExtensions.UseDefaultFiles` 메서드는 반드시 ``UseStaticFiles`` 메서드를 호출하기 전에 호출하십시오. ``UseDefaultFiles`` 메서드는 단지 URL 재생성기 일 뿐 파일을 실제로 제공하는 미들웨어는 아닙니다. 따라서 파일을 제공하기 위한 정적 파일 미들웨어 (``UseStaticFiles``) 를 사용하도록 설정해야 합니다.

With :dn:method:`~Microsoft.AspNetCore.Builder.DefaultFilesExtensions.UseDefaultFiles`, requests to a folder will search for:
:dn:method:`~Microsoft.AspNetCore.Builder.DefaultFilesExtensions.UseDefaultFiles` 를 통해, 어떤 폴더에 대한 요청을 처리하는 과정에서 다음 파일들을 찾아봅니다.

  - default.htm
  - default.html
  - index.htm
  - index.html

The first file found from the list will be served as if the request was the fully qualified URI (although the browser URL will continue to show the URI requested).
위 파일 중 첫 번째로 찾은 파일을 해당 요청이 그 파일에 대한 전체 URI 인 것처럼 제공합니다. (하지만 브라우저 상의 URL 은 요청한 URI 그대로 보여질 것입니다.)

The following code shows how to change the default file name to *mydefault.html*.
다음 코드에서 *mydefault.html* 에 대한 기본 파일 이름을 바꾸는 방법을 확인할 수 있습니다.

.. literalinclude:: static-files/sample/StartupDefault.cs
  :language: c#
  :start-after: >Configure
  :end-before: <Configure
  :dedent: 8

UseFileServer
------------------------------

:dn:method:`~Microsoft.AspNetCore.Builder.FileServerExtensions.UseFileServer` combines the functionality of :dn:method:`~Microsoft.AspNetCore.Builder.StaticFileExtensions.UseStaticFiles`, :dn:method:`~Microsoft.AspNetCore.Builder.DefaultFilesExtensions.UseDefaultFiles`, and :dn:method:`~Microsoft.AspNetCore.Builder.DirectoryBrowserExtensions.UseDirectoryBrowser`.
:dn:method:`~Microsoft.AspNetCore.Builder.FileServerExtensions.UseFileServer` 메서드는 :dn:method:`~Microsoft.AspNetCore.Builder.StaticFileExtensions.UseStaticFiles` 메서드와 :dn:method:`~Microsoft.AspNetCore.Builder.DefaultFilesExtensions.UseDefaultFiles` 메서드, :dn:method:`~Microsoft.AspNetCore.Builder.DirectoryBrowserExtensions.UseDirectoryBrowser` 메서드의 기능을 모두 포함합니다.

The following code enables static files and the default file to be served, but does not allow directory browsing:
다음 코드에서는 정적 파일과 기본 파일의 제공은 허용하고 있지만, 디렉토리 브라우징은 허용하지 않고 있습니다.

.. code-block:: c#

  app.UseFileServer();

The following code enables static files, default files and  directory browsing:
다음 코드에서는 정적 파일과 기본 파일의 제공 그리고 디렉토리 브라우징을 허용하고 있습니다.

.. code-block:: c#

  app.UseFileServer(enableDirectoryBrowsing: true);

See Considerations_ on the security risks when enabling browsing. As with ``UseStaticFiles``, ``UseDefaultFiles``, and ``UseDirectoryBrowser``, if you wish to serve files that exist outside the ``web root``, you instantiate and configure an :dn:class:`~Microsoft.AspNetCore.Builder.FileServerOptions` object that you pass as a parameter to ``UseFileServer``. For example, given the following directory hierarchy in your Web app:
디렉토리 브라우징을 허용했을 때의 보안 위험에 대해서는 고려사항_ 을 확인하세요. ``UseStaticFiles`` 과 ``UseDefaultFiles``, ``UseDirectoryBrowser`` 메서드와 마찬가지로 ``web root`` 외부의 파일을 제공하길 원한다면, :dn:class:`~Microsoft.AspNetCore.Builder.FileServerOptions` 개체를 생성하고 설정한 뒤에 ``UseFileServer`` 메서드에 매개변수로 전달하세요. 예를 들어, 여러분의 웹 어플리케이션에서 다음과 같은 디렉토리 구조를 사용한다고 해보겠습니다.

- wwwroot

  - css
  - images
  - ...

- MyStaticFiles

  - test.png
  - default.html

Using the hierarchy example above, you might want to enable static files, default files, and browsing for the ``MyStaticFiles`` directory. In the following code snippet, that is accomplished with a single call to :dn:class:`~Microsoft.AspNetCore.Builder.FileServerOptions`.
위 예시와 같은 구조를 사용한다면, 여러분은 ``MyStaticFiles`` 디렉토리 내의 정적 파일과 기본 파일의 제공 및 브라우징을 가능하도록 하길 원할 수 있습니다. 다음 코드 토막에서는 :dn:class:`~Microsoft.AspNetCore.Builder.FileServerOptions` 에 대한 한 번의 호출로 그 기능을 수행하고 있습니다.

.. literalinclude:: static-files/sample/StartupUseFileServer.cs
  :language: c#
  :start-after: >Configure
  :end-before: <Configure
  :dedent: 8
  :emphasize-lines: 5-11


If ``enableDirectoryBrowsing`` is set to ``true`` you are required to call :dn:method:`~Microsoft.Extensions.DependencyInjection.DirectoryBrowserServiceExtensions.AddDirectoryBrowser` extension method from  ``Startup.ConfigureServices``:
``enableDirectoryBrowsing`` 를 ``true`` 로 설정했다면, ``Startup.ConfigureServices`` 에서 :dn:method:`~Microsoft.Extensions.DependencyInjection.DirectoryBrowserServiceExtensions.AddDirectoryBrowser` 확장 메서드를 호출해야 합니다.

.. literalinclude:: static-files/sample/StartupUseFileServer.cs
  :language: c#
  :start-after: >Services
  :end-before: <Services
  :dedent: 8


Using the file hierarchy and code above:
위 예시에서의 디렉토리 구조와 코드 토막을 사용하면, 사용자가 여러 URI 를 브라우징할 때 다음과 같이 동작합니다.:

==========================================  ===================================
URI                                         Response
==========================================  ===================================
\http://<app>/StaticFiles/test.png          MyStaticFiles/test.png
\http://<app>/StaticFiles                   MyStaticFiles/default.html
==========================================  ===================================

If no default named files are in the *MyStaticFiles* directory, \http://<app>/StaticFiles returns the directory listing with clickable links:
*MyStaticFiles* 디렉토리에 기본 파일이 존재하지 않는다면, \http://<app>/StaticFiles URI 에 대한 요청에서 클릭 가능한 링크가 걸린 디렉토리 목록이 반환됩니다.

.. image:: static-files/_static/db2.PNG

.. note:: ``UseDefaultFiles`` and ``UseDirectoryBrowser`` will take the url \http://<app>/StaticFiles without the trailing slash and cause a client side redirect to \http://<app>/StaticFiles/ (adding the trailing slash). Without the trailing slash relative URLs within the documents would be incorrect.
.. note:: ``UseDefaultFiles`` 와 ``UseDirectoryBrowser`` 에서 끝에 슬래쉬 (/) 가 없는 URL \http://<app>/StaticFiles 을 받게 되면, 클라이언트에서 끝에 슬래쉬를 추가한 \http://<app>/StaticFiles/ 로 리다이렉트하도록 합니다. 끝에 슬래쉬가 없다면, 문서 내의 상대 경로가 맞지 않을 것이기 때문입니다.

FileExtensionContentTypeProvider
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The :dn:class:`~Microsoft.AspNetCore.StaticFiles.FileExtensionContentTypeProvider` class contains a  collection that maps file extensions to MIME content types. In the following sample, several file extensions are registered to known MIME types, the ".rtf" is replaced, and ".mp4" is removed.
:dn:class:`~Microsoft.AspNetCore.StaticFiles.FileExtensionContentTypeProvider` 클래스에는 파일 확장자와 MIME 콘텐트 타입을 연결하는 콜렉션 개체를 포함하고 있습니다. 다음 예시에서, 몇몇 파일 확장자를 알려진 MIME 타입에 연결하여, ".rtf" 에 대한 연결은 변경하고 ".mp4" 에 대한 연결은 제거하고 있습니다.

.. literalinclude:: static-files/sample/StartupFileExtensionContentTypeProvider.cs
  :language: c#
  :start-after: >Configure
  :end-before: <Configure
  :dedent: 8
  :emphasize-lines: 3-12,19

See   `MIME content types <http://www.iana.org/assignments/media-types/media-types.xhtml>`__.
`MIME 콘텐트 타입 <http://www.iana.org/assignments/media-types/media-types.xhtml>`__ 에 대해 확인해보세요.

Non-standard content types
--------------------------

The ASP.NET static file middleware understands almost 400 known file content types. If the user requests a file of an unknown file type, the static file middleware returns a HTTP 404 (Not found) response. If directory browsing is enabled, a link to the file will be displayed, but the URI will return an HTTP 404 error.
ASP.NET 정적 파일 미들웨어는 약 400개의 알려진 콘텐트 타입을 인식합니다. 사용자가 미들웨어에서 인식할 수 없는 파일 타입에 접근하려하면, 정적 파일 미들웨어는 HTTP 404 (존재하지 않음.) 응답을 반환합니다. 디렉토리 브라우징을 허용하면, 파일에 대한 링크를 보여주기는 하지만 링크 내의 URI 가 HTTP 404 오류를 반환할 것입니다.

The following code enables serving unknown types and will render the unknown file as an image.
다음 코드에서는 인식할 수 없는 타입을 제공하도록 허용하고, 알려지지 않은 파일을 이미지로 표시되도록 하고 있습니다.

.. literalinclude:: static-files/sample/StartupServeUnknownFileTypes.cs
  :language: c#
  :start-after: >Configure
  :end-before: <Configure
  :dedent: 8

With the code above, a request for a file with an unknown content type will be returned as an image.
위 코드를 사용하면, 인식할 수 없는 콘텐트 타입의 파일에 대한 요청을 이미지로서 반환할 것입니다. 

.. warning:: Enabling :dn:property:`~Microsoft.AspNetCore.Builder.StaticFileOptions.ServeUnknownFileTypes` is a security risk and using it is discouraged.  :dn:class:`~Microsoft.AspNetCore.StaticFiles.FileExtensionContentTypeProvider`  (explained below) provides a safer alternative to serving files with non-standard extensions.
.. warning:: :dn:property:`~Microsoft.AspNetCore.Builder.StaticFileOptions.ServeUnknownFileTypes` 허용하면, 보안 위협이 있어 사용하지 않도록 권고하고 있습니다. 아래에서 설명하는 바와 같이 :dn:class:`~Microsoft.AspNetCore.StaticFiles.FileExtensionContentTypeProvider` 에서 비표준 확장자를 가진 파일을 제공하는 더 안전한 대체수단을 제공합니다.

Considerations
^^^^^^^^^^^^^^^^

.. warning:: ``UseDirectoryBrowser`` and ``UseStaticFiles`` can leak secrets. We recommend that you **not** enable directory browsing in production. Be careful about which directories you enable with ``UseStaticFiles`` or ``UseDirectoryBrowser`` as the entire directory and all sub-directories will be accessible. We recommend keeping public content in its own directory such as *<content root>/wwwroot*, away from application views, configuration files, etc.
.. warning:: ``UseDirectoryBrowser`` 와 ``UseStaticFiles`` 는 중요 정보를 유출시킬 가능성이 있습니다. 상용 시에 디렉토리 브라우징을 **허용하지 않기를** 권장합니다. ``UseStaticFiles`` 나 ``UseDirectoryBrowser`` 에서 허용하는 디렉토리가 어떤 것인지에 주의를 기울여, 전체 디렉토리에 접근을 허용하지 않도록 하십시오. 공개할 콘텐트를 *<콘텐트 루트>/wwwroot* 와 같은 별도의 디렉토리에 두고, 어플리케이션 뷰나 설정 파일 등과는 격리시키기를 권장합니다.

- The URLs for content exposed with ``UseDirectoryBrowser`` and ``UseStaticFiles`` are subject to the case sensitivity and character restrictions of their underlying file system. For example, Windows is case insensitive, but Mac and Linux are not.
- ``UseDirectoryBrowser`` 와 ``UseStaticFiles`` 를 통해 노출되는 콘텐트의 URL 은 해당 콘텐트가 저장된 파일 시스템의 대소문자 구분과 문자 제한 정책을 따릅니다. 예를 들어, 윈도우에서는 대소문자가 구분되지 않으나 Mac 과 Linux 에서는 대소문자가 구분됩니다.

- ASP.NET Core applications hosted in IIS use the ASP.NET Core Module to forward all requests to the application including requests for static files. The IIS static file handler is not used because it doesn't get a chance to handle requests before they are handled by the ASP.NET Core Module.
- To remove the IIS static file handler (at the server or website level):
- IIS 에서 호스팅 되는 ASP.NET Core 어플리케이션은 ASP.NET Core 모듈을 사용하여 정적 파일에 대한 요청을 포함한 모든 요청을 어플리케이션으로 전달합니다. IIS 정적 파일 핸들러에게 ASP.NET Core 모듈에서 요청을 처리하기 전에 기회가 주어지지 않기 때문에 사용되지 않습니다.
- IIS 정적 파일 핸들러를 제거하려면 다음과 같이 하십시오. (서버 수준에서든 혹은 웹사이트 수준에서든):

    - Navigate to the **Modules** feature
    - Select **StaticFileModule** in the list
    - Tap **Remove** in the **Actions** sidebar
    - **Modules** 기능으로 이동하세요.
    - 목록에서 **StaticFileModule** 을 선택하세요.
    - **Actions** 사이드바에서 **Remove** 를 누르세요.
    
.. warning:: If the IIS static file handler is enabled **and** the ASP.NET Core Module (ANCM) is not correctly configured (for example if *web.config* was not deployed), static files will be served.
.. warning:: IIS 정적 파일 핸들러가 **허용되어 있고** ASP.NET Core 모듈 (ANCM : ASP.NET Core Module) 이 정확하게 **설정되어 있지 않다면** (예를 들어 *web.config* 가 적용되지 않은 경우를 들 수 있습니다.), 정적 파일을 IIS 정적 파일 핸들러가 제공할 것입니다.

- Code files (including c# and Razor) should be placed outside of the app project's ``web root`` (*wwwroot* by default). This creates a clean separation between your app's client side content and server side source code, which prevents server side code from being leaked.
- 코드 파일들 (C# 과 Razor 를 포함) 은 어플리케이션 프로젝트의 ``web root`` (기본적으로 *wwwroot*) 외부에 반드시 위치시켜야 합니다. 이를 통해 클라이언트 측의 콘텐트와 서버 측의 소스 코드를 명백히 분리할 수 있고, 이를 통해 서버 측 코드가 유출되지 않도록 방지할 수 있습니다.

Additional Resources
--------------------

- :doc:`middleware`
- :doc:`/intro`
