﻿Routing
=======
By `Ryan Nowak`_, `Steve Smith`_, and `Rick Anderson`_

Routing is used to map requests to route handlers. Routes are configured when the application starts up, and can extract values from the URL that will be used for request processing. Routing functionality is also responsible for generating links using the defined routes in ASP.NET apps.
라우팅은 요청을 라우트 핸들러 (route handler) 에 지정할 때 사용합니다. 라우트 (route) 는 어플리케이션이 시작할 때 설정합니다. 요청을 처리할 때 필요한 값을 URL 에서 라우트를 통해 추출할 수 있습니다. 또한 ASP.NET 어플리케이션 내에서 정의한 라우트를 사용하는 링크도 라우팅 기능을 통해 생성할 수 있습니다.
Routing은 request들을 route handler들에 매핑하는데 사용된다. Route들은 application이 시작될 때 설정되고, URL에서 request를 처리하는데 사용될 value들을 추출 할 수 있다. Routing 기능은 또한 ASP.NET app에서 정의된 route들을 사용하여 link 생성을 책임진다.

This document covers the low level ASP.NET Core routing. For ASP.NET Core MVC routing, see :doc:`/mvc/controllers/routing`
이 문서에서는 저수준의 ASP.NET Core 라우팅 기능을 다루고 있습니다. ASP.NET Core MVC 라우팅을 확인하기 위해서는 :doc:`/mvc/controllers/routing` 를 확인하세요.
이 문서는 low level ASP.NET Core routing을 설명한다. ASP.NET Core MVC routing에 대해서는 :doc:`/mvc/controllers/routing` 을 참조하라.

.. contents:: Sections
  :local:
  :depth: 1

`View or download sample code <https://github.com/aspnet/Docs/tree/master/aspnet/fundamentals/routing/sample>`__
`샘플 코드를 확인하거나 다운로드 받으세요. <https://github.com/aspnet/Docs/tree/master/aspnet/fundamentals/routing/sample>`_

Routing basics
----------------

Routing uses *routes* (implementations of :dn:iface:`~Microsoft.AspNetCore.Routing.IRouter`) to:
라우팅에서는 *라우트* (:dn:iface:`~Microsoft.AspNetCore.Routing.IRouter` 의 구현체) 를 다음과 같은 목적으로 사용합니다.:
Routing은 *routes* (:dn:iface:`~Microsoft.AspNetCore.Routing.IRouter` 의 구현)을 사용하여 다음과 같은 일을 한다 :

- map incoming requests to *route handlers*
- 인입되는 요청을 *라우트 핸들러* 로 연결
- 들어오는 request들을 *route handlers* 에 맵핑
- generate URLs used in responses
- 응답에서 사용할 URL 생성
- response에 사용되는 URL을 생성 

Generally an app has a single collection of routes. The route collection is processed in order. Requests look for a match in the route collection by :ref:`URL-Matching-ref`. Responses use routing to generate URLs.
일반적으로 하나의 어플리케이션에는 라우트들에 대한 콜렉션이 하나 있습니다. 라우트 콜렉션은 순서대로 처리됩니다. 요청 측면에서 볼 때는 요청의 URL 에 일치하는 라우트를 라우트 콜렉션에서 찾기 위해 :ref:`URL-Matching-ref` 을 사용합니다. 응답 측면에서 볼 때는 응답 내에서 사용할 URL 을 생성하기 위해 라우팅을 사용합니다.
일반적으로 app은 route들을 모아놓은 하나의 collection을 가진다. Route collection은 순서대로 처리된다. Request들은 :ref:`URL-Matching-ref` 에 의해 route collection에서 일치하는 것을 찾는다. Response는 URL 생성에 routing을 사용한다.

Routing is connected to the :doc:`middleware <middleware>` pipeline by the :dn:class:`~Microsoft.AspNetCore.Builder.RouterMiddleware` class. :doc:`ASP.NET MVC </mvc/overview>` adds routing to the middleware pipeline as part of its configuration. To learn about using routing as a standalone component, see using-routing-middleware_.
라우팅은 :doc:`미들웨어 <middleware>` 처리경로에 :dn:class:`~Microsoft.AspNetCore.Builder.RouterMiddleware` 클래스를 통해 연동할 수 있습니다. :doc:`ASP.NET MVC </mvc/overview>` 에서는 설정을 통해 미들웨어 처리경로에 라우팅을 추가합니다. 독립적인 컴포넌트로서 라우팅을 사용하는 방법을 확인하기 위해서는 using-routing-middleware_ 를 참고하세요.
Routing은 :dn:class:`~Microsoft.AspNetCore.Builder.RouterMiddleware` class에 의해서 :doc:`middleware <middleware>` pipeline에 연결된다. :doc:`ASP.NET MVC </mvc/overview>` 는 configuration을 구성하면서 routing을 middleware pipeline에 추가한다. 독립적인 component로 routing을 사용하는것데 대해 보려면 using-routing-middleware_을 참조하라.

.. _URL-Matching-ref:

URL matching
^^^^^^^^^^^^
URL matching is the process by which routing dispatches an incoming request to a *handler*. This process is generally based on data in the URL path, but can be extended to consider any data in the request. The ability to dispatch requests to separate handlers is key to scaling the size and complexity of an application.
URL 일치 판정은 라우팅이 인입되는 요청을 어떤 *핸들러* 에 전달할지를 결정하는 절차입니다. 이 절차는 일반적으로 URL 상의 데이터를 기반으로 합니다. 하지만 요청 내의 다른 종류의 데이터를 사용할 수도 있습니다. 요청을 각각의 핸들러로 전달하는 기능이야 말로 어플리케이션의 크기와 복잡도를 가늠하는 척도입니다.
URL matching은 routing이 들어오는 request를 *handler* 에 전달하는 과정이다. 이 처리는 일반적으로 URL path 정보에 기반하지만 request의 어떤 정보든 사용될 수 있다. 각각의 handler에 request를 전달하는 기능은 application의 복잡성과 크기를 스켕일링하는데 핵심이다.

Incoming requests enter the :dn:cls:`~Microsoft.AspNetCore.Builder.RouterMiddleware` which calls the :dn:method:`~Microsoft.AspNetCore.Routing.IRouter.RouteAsync` method on each route in sequence. The :dn:iface:`~Microsoft.AspNetCore.Routing.IRouter` instance chooses whether to *handle* the request by setting the :dn:cls:`~Microsoft.AspNetCore.Routing.RouteContext` :dn:prop:`~Microsoft.AspNetCore.Routing.RouteContext.Handler` to a non-null :dn:delegate:`~Microsoft.AspNetCore.Http.RequestDelegate`. If a handler is set a route, it will be invoked to process the request and no further routes will be processed. If all routes are executed, and no handler is found for a request, the middleware calls *next* and the next middleware in the request pipeline is invoked.
인입되는 요청이 :dn:cls:`~Microsoft.AspNetCore.Builder.RouterMiddleware` 로 진입하면, 미들웨어는 순서대로 각 라우트 (route) 의 :dn:method:`~Microsoft.AspNetCore.Routing.IRouter.RouteAsync` 를 호출합니다. :dn:iface:`~Microsoft.AspNetCore.Routing.IRouter` 의 구현체인 라우트 핸들러 개체는 ``RounteAsync`` 의 매개변수로 전달된 :dn:cls:`~Microsoft.AspNetCore.Routing.RouteContext` 상의 :dn:prop:`~Microsoft.AspNetCore.Routing.RouteContext.Handler` 속성에 null 이 아닌 :dn:delegate:`~Microsoft.AspNetCore.Http.RequestDelegate` 를 할당하는가 마는가를 통해 요청을 *처리할지 말지* 를 결정합니다. 라우트 핸들러에서 하나의 라우트를 지정하였다면, 요청을 처리하도록 호출될 것이고 미들웨어 내에서 그 이상의 라우트는 처리되지 않을 것입니다. 모든 라우트를 처리하였고 요청에 대한 핸들러가 더 없다면, 미들웨어는 *next* 를 호출하여 요청 처리경로 상의 다음 미들웨어를 호출합니다.

The primary input to ``RouteAsync`` is the :dn:cls:`~Microsoft.AspNetCore.Routing.RouteContext` :dn:prop:`~Microsoft.AspNetCore.Routing.RouteContext.HttpContext` associated with the current request. The ``RouteContext.Handler`` and :dn:cls:`~Microsoft.AspNetCore.Routing.RouteContext` :dn:prop:`~Microsoft.AspNetCore.Routing.RouteContext.RouteData` are outputs that will be set after a successful match.
``RouteAsync`` 메서드의 가장 중요한 입력값은 :dn:cls:`~Microsoft.AspNetCore.Routing.RouteContext` 의 :dn:prop:`~Microsoft.AspNetCore.Routing.RouteContext.HttpContext` 속성으로서, 현재 요청과 관련되어 있습니다. ``RouteContext.Handler`` 와 :dn:cls:`~Microsoft.AspNetCore.Routing.RouteContext` 의 :dn:prop:`~Microsoft.AspNetCore.Routi
``RouteAsync`` 의 주요 입력값은 현재 request와 관련된 :dn:cls:`~Microsoft.AspNetCore.Routing.RouteContext`.:dn:prop:`~Microsoft.AspNetCore.Routing.RouteContext.HttpContext` 이다. ``RouteContext.Handler`` 와 :dn:cls:`~Microsoft.AspNetCore.Routing.RouteContext`.:dn:prop:`~Microsoft.AspNetCore.Routing.RouteContext.RouteData` 는 매칭이 성공되면 설정되는 출력값이다.

A successful match during ``RouteAsync`` also will set the properties of the ``RouteContext.RouteData`` to appropriate values based on the request processing that was done. The ``RouteContext.RouteData`` contains important state information about the *result* of a route when it successfully matches a request.
``RouteAsync`` 실행 중에 URL 일치 판정에 성공한 경우, ``RouteContext.RouteData`` 의 속성들에 요청 처리 과정의 결과를 바탕으로 적절한 값을 할당합니다. 즉, ``RouteContext.RouteData`` 에는 라우트의 *결과* 에 대한 중요한 정보를 담고 있습니다.
``RouteAsync``를 호출하는동안 매칭이 성공하면 ``RouteContext.RouteData`` 의 propertie들 또한 완료된 request 처리에 기반한 적당한 값들로 설정된다. ``RouteContext.RouteData`` 는 request을 성공적으로 매칭시킨 route의 *result* 에 대한 중요한 상태정보를 담는다.

:dn:cls:`~Microsoft.AspNetCore.Routing.RouteData`.:dn:prop:`~Microsoft.AspNetCore.Routing.RouteData.Values` is a dictionary of *route values* produced from the route. These values are usually determined by tokenizing the URL, and can be used to accept user input, or to make further dispatching decisions inside the application.
:dn:cls:`~Microsoft.AspNetCore.Routing.RouteData` 의 :dn:prop:`~Microsoft.AspNetCore.Routing.RouteData.Values` 속성은 라우트에서 생성된 *라우트 값들*의 사전입니다. 이 값들은 보통 URL 을 토큰화하는 과정에서 결정되고, 사용자의 입력을 수용할 때 쓰이거나 혹은 어플리케이션 내에서 더 복잡한 어떤 선택을 할 때 쓰입니다.
:dn:cls:`~Microsoft.AspNetCore.Routing.RouteData`.:dn:prop:`~Microsoft.AspNetCore.Routing.RouteData.Values` 는 route로부터 만들어진 *route values* 의 사전이다. 이 값들은 보통 토근화 된 URL(by tokenizing the URL)로 결정되고 유저의 입력을 받는데 사용되거나 application 내부로 전달 할 수도 있다.

:dn:cls:`~Microsoft.AspNetCore.Routing.RouteData`.:dn:prop:`~Microsoft.AspNetCore.Routing.RouteData.DataTokens`  is a property bag of additional data related to the matched route. ``DataTokens`` are provided to support associating state data with each route so the application can make decisions later based on which route matched. These values are developer-defined and do **not** affect the behavior of routing in any way. Additionally, values stashed in data tokens can be of any type, in contrast to route values which must be easily convertable to and from strings.
:dn:cls:`~Microsoft.AspNetCore.Routing.RouteData` 의 :dn:prop:`~Microsoft.AspNetCore.Routing.RouteData.DataTokens` 속성은 일치한 라우트와 관련된 추가적인 데이터들의 ``PropertyBag`` 입니다. ``DataTokens`` 는 상태 데이터와 각 라우트를 연관지으므로, 어플리케이션에서 어떤 라우트에 일치하는가에 따라 선택을 해야 할 때 사용합니다. 이 값들은 개발자가 정의한 것으로서 라우팅의 행태에 *어떠한* 영향도 끼치지 않습니다. 게다가, ``DataTokens`` 속성 에 저장된 값들은 어떠한 형이라도 괜찮습니다. 이는 ``Values`` 속성 내의 라우트 값들은 문자열로 쉽게 변환 가능해야 한다는 점과는 차이가 있습니다.

:dn:cls:`~Microsoft.AspNetCore.Routing.RouteData`.:dn:prop:`~Microsoft.AspNetCore.Routing.RouteData.Routers` is a list of the routes that took part in successfully matching the request. Routes can be nested inside one another, and the ``Routers`` property reflects the path through the logical tree of routes that resulted in a match. 일반적으로 ``Routers`` 안의 첫번째 항목은 route collection이고 URL 생서에 사용된다. ``Routers`` 안의 마지막 항목은 매치된 route 이다.
:dn:cls:`~Microsoft.AspNetCore.Routing.RouteData` 의 :dn:prop:`~Microsoft.AspNetCore.Routing.RouteData.Routers` 속성은 요청 일치 판정에 성공한 모든 라우트의 목록입니다. 라우트들은 서로 중첩될 수 있고, ``Routers`` 속성은 URL 일치 판정 결과에 따라 생성된 라우트들의 로직 트리를 반영합니다. 일반적으로 ``Routers`` 속성의 첫 번째 항목은 라우트 콜렉션 개체로서, URL 생성 시에 사용되어야 합니다. 마지막 항목은 일치한 라우트 그 자체입니다.

URL generation
^^^^^^^^^^^^^^
URL generation is the process by which routing can create a URL path based on a set of route values. This allows for a logical separation between your handlers and the URLs that access them.
URL 생성은 라우팅 미들웨어가 라우트 값들을 기반으로 URL 을 만드는 절차입니다. 이를 통해 핸들러와 핸들러에 접근하는 URL 을 논리적으로 분리할 수 있습니다.

URL generation follows a similar iterative process, but starts with user or framework code calling into the :dn:method:`~Microsoft.AspNetCore.Routing.IRouter.GetVirtualPath` method of the route collection. Each *route* will then have its ``GetVirtualPath`` method called in sequence until a non-null :dn:cls:`~Microsoft.AspNetCore.Routing.VirtualPathData` is returned.
URL 생성은 다른 비슷한 반복적인 절차들과 마찬가지로서, 사용자나 프레임워크가 라우트 콜렉션에 대해 :dn:method:`~Microsoft.AspNetCore.Routing.IRouter.GetVirtualPath` 메서드를 호출합니다. 각각의 *라우트* 의 ``GetVirtualPath`` 메서드를 null 이 아닌 :dn:cls:`~Microsoft.AspNetCore.Routing.VirtualPathData` 이 반환될 때까지 호출합니다.

The primary inputs to ``GetVirtualPath`` are:
``GetVirtualPath`` 에 대한 가장 중요한 입력값들은 다음과 같습니다.:

- :dn:cls:`~Microsoft.AspNetCore.Routing.VirtualPathContext`.:dn:prop:`~Microsoft.AspNetCore.Routing.VirtualPathContext.HttpContext`
- :dn:cls:`~Microsoft.AspNetCore.Routing.VirtualPathContext`.:dn:prop:`~Microsoft.AspNetCore.Routing.VirtualPathContext.Values`
- :dn:cls:`~Microsoft.AspNetCore.Routing.VirtualPathContext`.:dn:prop:`~Microsoft.AspNetCore.Routing.VirtualPathContext.AmbientValues`

Routes primarily use the route values provided by the ``Values`` and ``AmbientValues`` to decide where it is possible to generate a URL and what values to include. The ``AmbientValues`` are the set of route values that were produced from matching the current request with the routing system. In contrast, ``Values`` are the route values that specify how to generate the desired URL for the current operation. The ``HttpContext`` is provided in case a route needs to get services or additional data associated with the current context.
라우트는 우선 ``Values`` 와 ``AmbientValues`` 로 제공되는 라우트 값을 사용하여, 어디서 URL 을 생성할 수 있고 어떤 값을 포함해야 할지를 결정합니다. ``AmbientValues`` 는 라우팅 시스템에서 현재 요청에 대한 일치 판정을 하는 과정에서 생성된 라우트 값들의 모음입니다. 대조적으로 ``Values`` 는 현재 동작에 적합한 URL 을 어떻게 생성할지를 지정하는 라우트 값들입니다. ``HttpContext`` 는 라우트에서 서비스나 현재의 컨텍스트와 관련 추가 데이터가 필요할 때 사용합니다. 

.. tip:: Think of ``Values`` as being a set of overrides for the ``AmbientValues``. URL generation tries to reuse route values from the current request to make it easy to generate URLs for links using the same route or route values.
.. tip:: ``Values`` 를 ``AmbientValues`` 에 우선하는 값으로 생각하세요. 또한 URL 생성에서는 현재 요청의 라우트 값들을 재사용하여, 동일한 라우트나 라우트 값들을 사용하는 링크에 대한 URL 생성을 원활하게 하려 할 수 있습니다.

The output of ``GetVirtualPath`` is a :dn:cls:`~Microsoft.AspNetCore.Routing.VirtualPathData`. ``VirtualPathData`` is a parallel of ``RouteData``; it contains the ``VirtualPath`` for the output URL as well as the some additional properties that should be set by the route.
``GetVirtualPath`` 의 출력값은 :dn:cls:`~Microsoft.AspNetCore.Routing.VirtualPathData` 입니다. ``VirtualPathData`` 는 ``RouteData`` 와 유사합니다. ``VirtualPathData`` 에는 출력 URL 에 대한 ``VirtualPath`` 가 들어있고, 그 외에 라우트에서 추가로 입력한 값들도 들어있습니다.

The :dn:cls:`~Microsoft.AspNetCore.Routing.VirtualPathData` :dn:prop:`~Microsoft.AspNetCore.Routing.VirtualPathData.VirtualPath`
property contains the *virtual path* produced by the route. Depending on your needs you may need to process the path further. For instance, if you want to render the generated URL in HTML you need to prepend the base path of the application.
:dn:cls:`~Microsoft.AspNetCore.Routing.VirtualPathData` 의 :dn:prop:`~Microsoft.AspNetCore.Routing.VirtualPathData.VirtualPath` 속성에는 라우트가 생성한 *가상 경로* 가 들어있습니다. 필요에 따라 가상 경로에 추가적인 처리해야 할 수 있습니다. 예를 들어, 생성된 URL 을 HTML 형태로 내보내고자 할 경우 어플리케이션의 기본 경로를 경로 문자열 앞에 붙여야 할 수 있습니다.

The :dn:cls:`~Microsoft.AspNetCore.Routing.VirtualPathData` :dn:prop:`~Microsoft.AspNetCore.Routing.VirtualPathData.Router` is a reference to the route that successfully generated the URL.
:dn:cls:`~Microsoft.AspNetCore.Routing.VirtualPathData` 의 :dn:prop:`~Microsoft.AspNetCore.Routing.VirtualPathData.Router` 속성은 URL 을 성공적으로 생성한 라우트에 대한 참조가 할당되어 있습니다.

The :dn:cls:`~Microsoft.AspNetCore.Routing.VirtualPathData` :dn:prop:`~Microsoft.AspNetCore.Routing.VirtualPathData.DataTokens` properties is a dictionary of additional data related to the route that generated the URL. This is the parallel of ``RouteData.DataTokens``.
The :dn:cls:`~Microsoft.AspNetCore.Routing.VirtualPathData` 의 :dn:prop:`~Microsoft.AspNetCore.Routing.VirtualPathData.DataTokens` 속성은 URL 을 생성한 라우트와 관련된 추가적인 데이터들의 사전입니다. 이는 ``RouteData.DataTokens`` 와 유사합니다.

Creating routes
^^^^^^^^^^^^^^^
Routing provides the :dn:cls:`~Microsoft.AspNetCore.Routing.Route` class as the standard implementation of ``IRouter``. ``Route`` uses the *route template* syntax to define patterns that will match against the URL path when :dn:method:`~Microsoft.AspNetCore.Routing.IRouter.RouteAsync` is called. ``Route`` will use the same route template to generate a URL when :dn:method:`~Microsoft.AspNetCore.Routing.IRouter.GetVirtualPath` is called.
라우팅에서는 ``IRouter`` 의 표준 구현으로서 :dn:cls:`~Microsoft.AspNetCore.Routing.Route` 클래스를 제공합니다. ``Route`` 는 *라우트 템플릿 (route template)* 문법을 사용하여 패턴을 정의하고, :dn:method:`~Microsoft.AspNetCore.Routing.IRouter.RouteAsync` 메서드가 호출된 경우에 URL 일치 여부를 판정할 때 이를 사용합니다. ``Route`` 는 :dn:method:`~Microsoft.AspNetCore.Routing.IRouter.GetVirtualPath` 메서드가 호출되었을 때도, 동일한 라우트 템플릿을 사용하여 URL 을 생성합니다.
Routing은 ``IRouter``의 표준 구현으로 :dn:cls:`~Microsoft.AspNetCore.Routing.Route` class를 제공한다. ``Route`` uses the *route template* syntax to define patterns that will match against the URL path when :dn:method:`~Microsoft.AspNetCore.Routing.IRouter.RouteAsync` is called. ``Route`` will use the same route template to generate a URL when :dn:method:`~Microsoft.AspNetCore.Routing.IRouter.GetVirtualPath` is called.

Most applications will create routes by calling ``MapRoute`` or one of the similar extension methods defined on :dn:iface:`~Microsoft.AspNetCore.Routing.IRouteBuilder`. All of these methods will create an instance of ``Route`` and add it to the route collection.
대부분의 어플리케이션에서는 ``MapRoute`` 메서드를 호출하거나 :dn:iface:`~Microsoft.AspNetCore.Routing.IRouteBuilder` 에 정의된 확장 메서드 중 하나를 호출하여 라우트를 생성합니다. 이런 메서드들에서는 ``Route`` 개체를 생성하고 라우트 콜랙션에 이를 추가합니다.
대부분의 application들은 ``MapRoute`` 나 :dn:iface:`~Microsoft.AspNetCore.Routing.IRouteBuilder` 에 정의된 비슷한 extension method들 중 하나를 호출하여 route들을 만들것이다. 이 method들 전부는 ``Route`` 의 instance를 만들고 route collection에 추가 할 것이다.

.. note:: :dn:method:`~Microsoft.AspNetCore.Builder.MapRouteRouteBuilderExtensions.MapRoute` doesn't take a route handler parameter - it only adds routes that will be handled by the :dn:prop:`~Microsoft.AspNetCore.Routing.IRouteBuilder.DefaultHandler`. Since the default handler is an :dn:iface:`~Microsoft.AspNetCore.Routing.IRouter`, it may decide not to handle the request. For example, ASP.NET MVC is typically configured as a default handler that only handles requests that match an available controller and action. To learn more about routing to MVC, see :doc:`/mvc/controllers/routing`.
.. note:: :dn:method:`~Microsoft.AspNetCore.Builder.MapRouteRouteBuilderExtensions.MapRoute` 메서드에는 라우트 핸들러를 매개변수로 전달받지 않습니다. - :dn:prop:`~Microsoft.AspNetCore.Routing.IRouteBuilder.DefaultHandler` 로 처리할 라우트를 추가할 뿐입니다. 기본 핸들러가 :dn:iface:`~Microsoft.AspNetCore.Routing.IRouter` 의 구현체이므로, 요청을 처리하지 않을 수도 있습니다. 예를 들어, 보통 ASP.NET MVC 를 기본 핸들러로 설정하므로 가능한 컨트롤러나 동작을 찾을 수 있는 요청 만 처리합니다. MVC 에 대한 라우팅에 대해 더 확인하시려면, :doc:`/mvc/controllers/routing` 를 참고하세요.
.. note:: :dn:method:`~Microsoft.AspNetCore.Builder.MapRouteRouteBuilderExtensions.MapRoute` 는 route handler parameter를 가지지 않는다. - :dn:prop:`~Microsoft.AspNetCore.Routing.IRouteBuilder.DefaultHandler` 에 의해 다뤄지는 route들을 추가하기만 한다. Since the default handler is an :dn:iface:`~Microsoft.AspNetCore.Routing.IRouter`, it may decide not to handle the request. 예를들어, ASP.NET MVC는 일반적으로 오직 가능한 controller와 action에 매치되는 request들을 다루는 기본 handler로 구성된다. To learn more about routing to MVC, see :doc:`/mvc/controllers/routing`.

This is an example of a ``MapRoute`` call used by a typical ASP.NET MVC route definition:
다음은 ASP.NET MVC 에서 라우트를 정의하는 전형적인 예시로서, ``MapRoute`` 메서드를 호출하고 있습니다.
다음예제는 일반적인 ASP.NET MVC route 정의를 사용하는 ``MapRoute`` call을 보여준다 :

.. code-block:: c#

    routes.MapRoute(
        name: "default",
        template: "{controller=Home}/{action=Index}/{id?}");

This template will match a URL path like ``/Products/Details/17`` and extract the route values ``{ controller = Products, action = Details, id = 17 }``. The route values are determined by splitting the URL path into segments, and matching each segment with the *route parameter* name in the route template. Route parameters are named. They are defined by enclosing the parameter name in braces ``{ }``.
이 템플릿은 ``/Products/Details/17`` 와 같은 URL 경로를 일치하는 것으로 판정하고, ``{ controller = Products, action = Details, id = 17 }`` 라는 라우트 값을 추출합니다. 이 라우트 값은 URL 경로를 분할하고 각 분할에 해당하는 라우트 템플릿 상의 *라우트 매개변수* 에 이름을 지정하여 생성합니다. 각 라우트 매개변수는 이름이 지정되어 있고, ``{ }`` 괄호로 쌓여있습니다.
이 template은 ``/Products/Details/17`` 과 같은 URL path를 매치하고  route values로 ``{ controller = Products, action = Details, id = 17 }`` 을 추출 할 것이다. Route values는 segment로 분할된 URL path에 의해 결정되고, 각 segment는 route template의 *route parameter* 이름과 매칭된다. Route parameter들은 braces ``{ }`` 에 둘러 쌓인 parameter name 으로 정의되어 이름이 지어진다.

The template above could also match the URL path ``/`` and would produce the values ``{ controller = Home, action = Index }``. This happens because the ``{controller}`` and ``{action}`` route parameters have default values, and the ``id`` route parameter is optional. An equals ``=`` sign followed by a value after the route parameter name defines a default value for the parameter. A question mark ``?`` after the route parameter name defines the parameter as optional. Route parameters with a default value *always* produce a route value when the route matches - optional parameters will not produce a route value if there was no corresponding URL path segment.
위의 템플릿은 URL 경로 ``/`` 도 일치하는 것으로 판정하고, ``{ controller = Home, action = Index }`` 이라는 값을 생성할 것입니다. 이는 ``{controller}`` 와 ``{action}`` 라우트 매개변수에는 기본값이 지정되어 있고, ``id`` 라우트 매개변수는 선택적이기 때문입니다. 매개변수 이름과 값 사이의 ``=`` 등호는 매개변수에 대한 기본값을 정의합니다. 라우트 매개변수 이름 뒤의 ``?`` 물음표는 해당 매개변수가 선택적이라고 정의합니다. 기본값을 지정한 라우트 매개변수는 *언제나* 라우트가 일치하는 것으로 판정될 때마다 라우트를 생성합니다. 반면에 선택적 매개변수의 경우에는 URL 경로에 관련된 분할 (segment) 이 없다면 라우트 값을 생성하지 않습니다.

See route-template-reference_ for a thorough description of route template features and syntax.
라우트 템플릿 기능과 문법에 대한 전반적인 설명을 확인하기 위해서는 route-template-reference_ 를 참고하세요.

This example includes a *route constraint*:
다음 예시에서는 라우트 제한조건을 포함하였습니다. :

.. code-block:: c#

    routes.MapRoute(
        name: "default",
        template: "{controller=Home}/{action=Index}/{id:int}");

This template will match a URL path like ``/Products/Details/17``, but not ``/Products/Details/Apples``. The route parameter definition ``{id:int}`` defines a *route constraint* for the ``id`` route parameter. Route constraints implement ``IRouteConstraint`` and inspect route values to verify them. In this example the route value ``id`` must be convertable to an integer. See route-constraint-reference_ for a more detailed explaination of route constraints that are provided by the framework.
이 템플릿은 ``/Products/Details/17`` 와 같은 URL 경로를 일치하는 것으로 판정할 것입니다. 하지만 ``/Products/Details/Apples`` 같은 경우에는 일치하지 않는 것으로 판정할 것입니다. ``{id:int}`` 라는 라우트 매개변수 정의는 ``id`` 라우트 매개변수에 *라우트 제한조건* 을 지정하고 있습니다. 라우트 제한조건은 ``IRouteConstraint`` 를 구현하고, 라우트 값이 올바른지 검사합니다. 이번 예시에서 라우트 값 ``id`` 는 정수 (int) 로 변환할 수 있어야 합니다. 프레임워크에서 제공하는 라우트 제한조건에 대해 더 자세히 확인하기 위해서는 route-constraint-reference_ 를 참고하세요.

Additional overloads of ``MapRoute`` accept values for ``constraints``, ``dataTokens``, and ``defaults``. These additional parameters of ``MapRoute`` are defined as type ``object``. The typical usage of these parameters is to pass an anonymously typed object, where the property names of the anonymous type match route parameter names.
``MapRoute`` 의 다른 오버로딩 메서드들의 경우 ``constraints`` 와 ``dataTokens``, ``defaults`` 에 대한 값을 매개변수로 받습니다. ``MapRoute`` 의 추가적인 매개변수들은 ``object`` 형으로 정의되어 있습니다. 이 매개변수들을 일반적인 사용 방법은 익명의 형인 개체로서 전달하는 것으로서, 해당 개체의 속성들은 라우트의 매개변수와 이름이 일치합니다.

The following two examples create equivalent routes:
다음 2가지 예시에서는 동일한 라우트를 생성하고 있습니다.

.. code-block:: c#

    routes.MapRoute(
        name: "default_route",
        template: "{controller}/{action}/{id?}",
        defaults: new { controller = "Home", action = "Index" });

    routes.MapRoute(
        name: "default_route",
        template: "{controller=Home}/{action=Index}/{id?}");

.. tip:: The inline syntax for defining constraints and defaults can be more convenient for simple routes. However, there are features such as data tokens which are not supported by inline syntax.
.. tip:: 제약사항과 기본값을 정의하는 인라인 형태의 문법이 간단한 라우트에 더 편리할 수 있습니다. 하지만 데이터 토큰와 같이 인라인 문법으로 지원하지 않는 기능도 있습니다.

.. review-required: changed template and add MVC controller sample

This example demonstrates a few more features:
다음 예시에서는 좀더 많은 기능을 확인할 수 있습니다. :

.. code-block:: c#

  routes.MapRoute(
    name: "blog",
    template: "Blog/{*article}",
    defaults: new { controller = "Blog", action = "ReadArticle" });

This template will match a URL path like ``/Blog/All-About-Routing/Introduction`` and will extract the values ``{ controller = Blog, action = ReadArticle, article = All-About-Routing/Introduction }``. The default route values for ``controller`` and ``action`` are produced by the route even though there are no corresponding route parameters in the template. Default values can be specified in the route template. The ``article`` route parameter is defined as a *catch-all* by the appearance of an asterix ``*`` before the route parameter name. Catch-all route parameters capture the remainder of the URL path, and can also match the empty string.
이 템플릿은 ``/Blog/All-About-Routing/Introduction`` 와 같은 URL 경로를 일치하는 것으로 판정할 것이고, ``{ controller = Blog, action = ReadArticle, article = All-About-Routing/Introduction }`` 와 같이 값을 추축할 것입니다. ``controller`` 와 ``action`` 관련 매개변수가 URL 경로에 없지만, 라우트에 의해 기본 라우트 값이 생성되었습니다. 기본 라우트 값은 라우트 템플릿 내에서 지정할 수도 있습니다. ``article`` 라우트 매개변수는 라우트 매개변수 이름 앞에 별표 ``*`` 를 붙여, ``포괄적 (catch-all)`` 이라고 정의하고 있습니다. 포괄적인 라우트 매개변수는 이후의 모든 URL 경로를 수집 (capture) 하고, 공백 문자도 일치하는 것으로 판정합니다. 

This example adds route constraints and data tokens:
다음 예시에서는 라우트 제약사항과 데이터 토큰을 포함하고 있습니다.

.. code-block:: c#

  routes.MapRoute(
      name: "us_english_products",
      template: "en-US/Products/{id}",
      defaults: new { controller = "Products", action = "Details" },
      constraints: new { id = new IntRouteConstraint() },
      dataTokens: new { locale = "en-US" });

This template will match a URL path like ``/en-US/Products/5`` and will extract the values ``{ controller = Products, action = Details, id = 5 }`` and the data tokens ``{ locale = en-US }``.
이 템플릿은 ``/en-US/Products/5`` 와 같은 URL 경로를 일치하는 것으로 판정할 것이고, ``{ controller = Products, action = Details, id = 5 }`` 와 같은 값과 ``{ locale = en-US }`` 와 같은 데이터 토큰을 추출할 것입니다.


.. image:: routing/_static/tokens.png

.. _url-generation:

URL generation
^^^^^^^^^^^^^^^
The ``Route`` class can also perform URL generation by combining a set of route values with its route template. This is logically the reverse process of matching the URL path.
``Route`` 클래스는 일련의 라우트 값과 라우트 템플릿을 조합하여 URL 생성도 수행할 수 있습니다. 이는 논리적으로 URL 경로의 일치 판정 과정의 정반대 과정입니다.

.. tip:: To better understand URL generation, imagine what URL you want to generate and then think about how a route template would match that URL. What values would be produced? This is the rough equivalent of how URL generation works in the ``Route`` class.
.. tip:: URL 생성에 대해 더 깊이 이해하기 위해서는, 여러분이 생성하고자 하는 URL 을 정하고 어떻게 라우트 템플릿을 통해 해당 URL 이 일치하는지 판정할 것인가에 대해 생각해보세요. 어떤 값을 생성될 것 같습니까? 이와 거의 동일한 과정이 ``Route`` 클래스 내부에서 URL 을 생성할 때 일어납니다.

This example uses a basic ASP.NET MVC style route:
다음 예시에서는 ASP.NET MVC 의 기본적인 형식인 라우트를 사용하고 있습니다.

.. code-block:: c#

    routes.MapRoute(
        name: "default",
        template: "{controller=Home}/{action=Index}/{id?}");

With the route values ``{ controller = Products, action = List }``, this route will generate the URL ``/Products/List``. The route values are substituted for the corresponding route parameters to form the URL path. Since ``id`` is an optional route parameter, it's no problem that it doesn't have a value.
이 라우트는 ``{ controller = Products, action = List }`` 라우트 값으로 URL ``/Products/List`` 를 생성할 것입니다. 라우트 값들은 관련된 라우트 매개변수에 대치되어 URL 경로를 형성합니다. ``id`` 는 선택적인 라우트 매개변수이기 때문에, 값이 없어도 문제없습니다.

With the route values ``{ controller = Home, action = Index }``, this route will generate the URL ``/``. The route values that were provided match the default values so the segments corresponding to those values can be safely omitted. Note that both URLs generated would round-trip with this route definition and produce the same route values that were used to generate the URL.
이 라우트는 ``{ controller = Home, action = Index }`` 라우트 값으로 URL ``/`` 를 생성할 것입니다. 라우트 값이 기본값과 동일하기 때문에, 관련된 분할을 안전하게 생략할 수 있습니다. 위 두 가지 생성된 URL들 모두가 라우트에 입력되었을 때, URL 을 생성할 때 사용된 것과 동일한 라우트 값을 생성할 것입니다.

.. tip:: An app using ASP.NET MVC should use :dn:cls:`~Microsoft.AspNetCore.Mvc.Routing.UrlHelper` to generate URLs instead of calling into routing directly.
.. tip:: ASP.NET MVC 를 사용하는 앱은 라우팅 기능을 직접 사용하지 말고 :dn:cls:`~Microsoft.AspNetCore.Mvc.Routing.UrlHelper` 을 사용하여 URL 을 생성해야 합니다.

For more details about the URL generation process, see url-generation-reference_.
URL 생성 절차에 대한 더 자세한 내용을 위해서는 url-generation-reference_ 을 참고하세요.

.. _using-routing-middleware:

Using Routing Middleware
-------------------------
To use routing middleware, add it to the **dependencies** in *project.json*:
라우팅 미들웨어를 사용하기 위해서는, *project.json* 내의 **dependencies** 에 추가하세요.

``"Microsoft.AspNetCore.Routing": <current version>``

Add routing to the service container in *Startup.cs*:
*Startup.cs* 에서 서비스 컨테이너에 라우팅을 추가하세요.

.. literalinclude:: routing/sample/RoutingSample/Startup.cs
  :dedent: 8
  :language: c#
  :lines: 11-14
  :emphasize-lines: 3

Routes must configured in the ``Configure`` method in the ``Startup`` class. The sample below uses these APIs:
경로들은 ``Startup`` 클래스의 ``Configure`` 메서드에서 설정해야 합니다. 아래의 예시에서는 다음과 같은 API를 사용합니다.

- :dn:cls:`~Microsoft.AspNetCore.Routing.RouteBuilder`
- :dn:method:`~Microsoft.AspNetCore.Routing.RouteBuilder.Build`
- :dn:method:`~Microsoft.AspNetCore.Routing.RequestDelegateRouteBuilderExtensions.MapGet`  Matches only HTTP GET requests
- :dn:method:`~Microsoft.AspNetCore.Routing.RequestDelegateRouteBuilderExtensions.MapGet` HTTP GET 요청에 대해서만 일치 판정을 수행합니다.
- :dn:method:`~Microsoft.AspNetCore.Builder.RoutingBuilderExtensions.UseRouter`

.. literalinclude:: routing/sample/RoutingSample/Startup.cs
  :dedent: 8
  :start-after: // Routes must configured in Configure
  :end-before: // Show link generation when no routes match.

The table below shows the responses with the given URIs.
아래의 표에서는 각 URI 에 대한 응답을 확인할 수 있습니다.

===================== ====================================================
URI                    Response
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
하나의 경로 만 설정하고자 한다면, ``app.UseRouter`` 에 ``IRouter`` 개체를 바로 전달하세요. ``RouteBuilder`` 를 호출할 필요없습니다.

The framework provides a set of extension methods for creating routes such as:
프레임워크에서는 라우트 생성을 위해 다음과 같은 일련의 확장 메서드를 제공합니다.

- :dn:method:`~Microsoft.AspNetCore.Builder.MapRouteRouteBuilderExtensions.MapRoute`
- :dn:method:`~Microsoft.AspNetCore.Routing.RequestDelegateRouteBuilderExtensions.MapGet`
- :dn:method:`~Microsoft.AspNetCore.Routing.RequestDelegateRouteBuilderExtensions.MapPost`
- :dn:method:`~Microsoft.AspNetCore.Routing.RequestDelegateRouteBuilderExtensions.MapPut`
- :dn:method:`~Microsoft.AspNetCore.Routing.RequestDelegateRouteBuilderExtensions.MapDelete`
- :dn:method:`~Microsoft.AspNetCore.Routing.RequestDelegateRouteBuilderExtensions.MapVerb`

Some of these methods such as ``MapGet`` require a :dn:delegate:`~Microsoft.AspNetCore.Http.RequestDelegate` to be provided. The ``RequestDelegate`` will be used as the *route handler* when the route matches. Other methods in this family allow configuring a middleware pipeline which will be used as the route handler. If the *Map* method doesn't accept a handler, such as ``MapRoute``, then it will use the :dn:prop:`~Microsoft.AspNetCore.Routing.IRouteBuilder.DefaultHandler`.
이 메서드들 중 일부 오버로딩 메서드의 경우 :dn:delegate:`~Microsoft.AspNetCore.Http.RequestDelegate` 를 필요로 합니다. ``RequestDelegate`` 는 라우트의 일치 판정 시에 *라우트 핸들러* 로서 사용될 것입니다. 이 외의 다른 오버로딩 메서드의 경우 라우트 핸들러로서 사용할 미들웨어 처리경로를 설정합니다. ``MapRoute`` 같은 *Map* 메서드에 핸들러를 전달하지 않는 경우, :dn:prop:`~Microsoft.AspNetCore.Routing.IRouteBuilder.DefaultHandler` 를 사용할 것입니다.

The ``Map[Verb]`` methods use constraints to limit the route to the HTTP Verb in the method name. For example, see `MapGet <https://github.com/aspnet/Routing/blob/1.0.0/src/Microsoft.AspNetCore.Routing/RequestDelegateRouteBuilderExtensions.cs#L85-L88>`__ and `MapVerb <https://github.com/aspnet/Routing/blob/1.0.0/src/Microsoft.AspNetCore.Routing/RequestDelegateRouteBuilderExtensions.cs#L156-L180>`__.
``Map[Verb]`` 메서드들은 제약사항을 사용하여 해당 라우트가 메서드 이름 내에 포함된 HTTP 동사 (Verb) 에서만 처리되도록 합니다. 예시를 확인하시려면, `MapGet <https://github.com/aspnet/Routing/blob/1.0.0/src/Microsoft.AspNetCore.Routing/RequestDelegateRouteBuilderExtensions.cs#L85-L88>`__ 와 `MapVerb <https://github.com/aspnet/Routing/blob/1.0.0/src/Microsoft.AspNetCore.Routing/RequestDelegateRouteBuilderExtensions.cs#L156-L180>`__ 를 참고하세요.

.. _route-template-reference:

Route Template Reference
------------------------
Tokens within curly braces (``{ }``) define *route parameters* which will be bound if the route is matched. You can define more than one route parameter in a route segment, but they must be separated by a literal value. For example ``{controller=Home}{action=Index}`` would not be a valid route, since there is no literal value between ``{controller}`` and ``{action}``. These route parameters must have a name, and may have additional attributes specified.
중괄호 (``{ }``) 내의 토큰은 *라우트 매개변수* 를 정의하고, 라우트가 일치하는 것으로 판정되었을 때 해당 토큰을 연결된 매개변수에 할당할 것입니다. 여러분은 하나의 라우트 분할 내에 하나 이상의 라우트 매개변수를 정의할 수 있습니다. 하지만 각각의 매개변수는 리터럴 값 (literal value) 으로 구분되어야 합니다. 예를 들어 ``{controller=Home}{action=Index}`` 는 ``{controller}`` 와 ``{action}`` 사이에 리터럴 값이 없으므로 적절한 경로가 아닙니다. 라우트 매개변수는 반드시 이름은 있어야 하고, 추가적인 특성의 경우에는 선택적으로 지정할 수 있습니다.

Literal text other than route parameters (for example, ``{id}``) and the path separator ``/`` must match the text in the URL. Text matching is case-insensitive and based on the decoded representation of the URLs path. To match the literal route parameter delimiter ``{`` or  ``}``, escape it by repeating the character (``{{`` or ``}}``).
라우트 매개변수 (예를 들어 ``{id}``) 와 경로 구분자 ``/`` 외의 리터럴 문자는 URL 경로 내에 일치하는 문자가 있어야 합니다. 문자 일치 확인은 대소문자를 구분하지 않고 URL 경로 내의 URL 인코딩이 디코딩된 표현을 기반으로 합니다. 라우트 매개변수 구분자인 ``{`` 이나 ``}`` 를 리터럴 문자로서 사용하기 위해서는, 문자를 반복하여 사용해야 합니다. (``{{`` 혹은 ``}}``)

URL patterns that attempt to capture a filename with an optional file extension have additional considerations. For example, using the template ``files/{filename}.{ext?}`` -
When both ``filename`` and ``ext`` exist, both values will be populated. If only ``filename`` exists in the URL, the route matches because the trailing period ``.`` is  optional. The following URLs would match this route:
파일 확장자가 선택적으로 존재하는 경우에 파일 이름을 수집하기 위한 URL 패턴의 경우에는 추가적으로 고려해야 할 부분이 있습니다. 예를 들어, ``files/{filename}.{ext?}`` 템플릿을 사용하는 경우에는 다음과 같은 경우가 존재할 것입니다. ``filename`` 과 ``ext`` 모두 있는 경우에는 두 가지 값 모두 결정할 것입니다. ``filename`` 만 있는 경우에는 뒷쪽의 마침표 ``.`` 가 선택적이기 때문에 라우트가 일치하는 것으로 판정될 것입니다. 다음 URL 은 모두 이 라우트에 일치하는 것으로 판정됩니다.

- ``/files/myFile.txt``
- ``/files/myFile.``
- ``/files/myFile``

You can use the ``*`` character as a prefix to a route parameter to bind to the rest of the URI - this is called a *catch-all* parameter. For example, ``blog/{*slug}`` would match any URI that started with ``/blog`` and had any value following it (which would be assigned to the ``slug`` route value). Catch-all parameters can also match the empty string.
``*`` 문자를 라우트 매개변수의 접두사로 사용하여, URI 의 나머지 부분을 모두 라우트 매개변수에 할당할 수 있습니다. - 이를 *포괄적 (catch-all)* 매개변수라고 합니다. 예를 들어, ``blog/{*slug}`` 의 경우 ``/blog`` 로 시작하는 URI 는 그 뒤에 어떤 문자열이 있더라도 일치하는 것으로 판정할 것입니다. (``/blog`` 뒤에 붙는 문자열은 ``slug`` 라우트 값에 할당될 것입니다.) 포괄적 변수에는 빈 문자열이 할당될 수도 있습니다.

Route parameters may have *default values*, designated by specifying the default after the parameter name, separated by an ``=``. For example, ``{controller=Home}`` would define ``Home`` as the default value for ``controller``. The default value is used if no value is present in the URL for the parameter. In addition to default values, route parameters may be optional (specified by appending a ``?`` to the end of the parameter name, as in ``id?``). The difference between optional and "has default" is that a route parameter with a default value always produces a value; an optional parameter has a value only when one is provided.
라우트 매개변수는 *기본값*을 가질 수도 있는데, 템플릿에서 매개변수의 이름 뒤의 ``=`` 다음에 지정하면 됩니다. 예를 들어, ``{controller=Home}`` 의 경우 ``Home`` 을 ``controller`` 의 기본값으로 지정합니다. 기본값은 URL 에 해당 매개변수에 대한 값이 없을 때 사용됩니다. 기본값 외에, 라우트 매개변수를 선택적으로 지정할 수 있습니다. (매개변수 이름 뒤에 ``?`` 를 붙여서 설정합니다. ``id?`` 와 같습니다.) 선택적과 "기본값 있음" 의 차이는 기본값이 있는 라우트 매개변수는 언제나 값을 있다는 점입니다. 그에 반해 선택적 매개변수는 URL 경로에 해당 부분이 있을 때에만 값이 할당됩니다.

Route parameters may also have constraints, which must match the route value bound from the URL. Adding a colon ``:`` and constraint name after the route parameter name specifies an *inline constraint* on a route parameter. If the constraint requires arguments those are provided enclosed in parentheses ``( )`` after the constraint name. Multiple inline constraints can be specified by appending another colon ``:`` and constraint name. The constraint name is passed to the :dn:iface:`~Microsoft.AspNetCore.Routing.IInlineConstraintResolver` service to create an instance of :dn:iface:`~Microsoft.AspNetCore.Routing.IRouteConstraint` to use in URL processing. For example, the route template ``blog/{article:minlength(10)}`` specifies the ``minlength`` constraint with the argument ``10``. For more description route constraints, and a listing of the constraints provided by the framework, see route-constraint-reference_.
라우트 매개변수에는 제약사항을 지정할 수도 있습니다. URL 에서 추출된 라우트 값은 반드시 제약사항을 지켜야 합니다. 라우트 매개변수 뒤에 콜론 ``:`` 과 제약사항 이름을 붙여서 라우트 매개변수에 대한 *인라인 제약사항*을 지정합니다. 제약사항에 인자가 필요한 경우에는 제약사항 이름 뒤에 괄호 ``( )`` 내에 넣어 지정합니다. 여러 개의 인라인 제약사항을 지정하고자 할 때는 제약사항들 사이에 콜론 ``:`` 을 넣어 추가하면 됩니다. 제약사항 이름은 :dn:iface:`~Microsoft.AspNetCore.Routing.IInlineConstraintResolver` 서비스에 전달되고 :dn:iface:`~Microsoft.AspNetCore.Routing.IRouteConstraint` 개체를 생성하여 URL 처리과정에서 사용됩니다. 예를 들어, 라우트 템플릿 ``blog/{article:minlength(10)}`` 은 인자 ``10`` 을 할당한 ``minlength`` 제약사항을 지정합니다. 라우트 제약사항과 프레임워크에서 제공하는 제약사항들의 목록에 대해 더 자세히 확인하기 위해서는, route-constraint-reference_ 를 참고하세요.

The following table demonstrates some route templates and their behavior.
다음 표에서 몇 가지 라우트 템플릿들과 그 행태를 확인할 수 있습니다.


+-----------------------------------+--------------------------------+------------------------------------------------+
| Route Template                    | Example Matching URL           | Notes                                          |
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
템플릿을 사용하는 방법은 일반적으로 라우팅을 사용하는 가장 간단한 방법입니다. 제약사항과 기본값은 라우티 템플릿 외부에서 지정할 수도 있습니다.

.. tip:: Enable :doc:`logging` to see how the built in routing implementations, such as ``Route``, match requests.
.. tip:: :doc:`logging` 을 허용하여 ``Route`` 와 같은 라우팅 구현체 내부에서 어떻게 요청들을 일치 판정하는지 확인할 수 있습니다.

.. _route-constraint-reference:

Route Constraint Reference
--------------------------
Route constraints execute when a ``Route`` has matched the syntax of the incoming URL and tokenized the URL path into route values. Route constraints generally inspect the route value associated via the route template and make a simple yes/no decision about whether or not the value is acceptable. Some route constraints use data outside the route value to consider whether the request can be routed. For example, the `HttpMethodRouteConstraint <https://docs.asp.net/projects/api/en/latest/autoapi/Microsoft/AspNetCore/Routing/Constraints/HttpMethodRouteConstraint/index.html#httpmethodrouteconstraint-class>`_ can accept or reject a request based on its HTTP verb.
라우트 제약사항은 ``Route`` 가 인입된 URL 이 일치하는지 판정하고 URL 경로를 토큰화하여 라우트 값에 할당할 때 사용됩니다. 라우트 제약사항은 일반적으로 라우트 템플릿과 관련된 라우트 값을 검사하고 해당 값을 사용해도 되는지 판정합니다. 일부 라우트 제약사항의 경우, 요청이 적합한지 판정하기 위해 라우트 값 이외의 데이터를 사용합니다. 예를 들어, `HttpMethodRouteConstraint <https://docs.asp.net/projects/api/en/latest/autoapi/Microsoft/AspNetCore/Routing/Constraints/HttpMethodRouteConstraint/index.html#httpmethodrouteconstraint-class>`_ 의 경우 HTTP 동사 (Verb) 를 바탕으로 요청을 허용할지 거부할지 판정합니다.

.. warning:: Avoid using constraints for **input validation**, because doing so means that invalid input will result in a 404 (Not Found) instead of a 400 with an appropriate error message. Route constraints should be used to **disambiguate** between similar routes, not to validate the inputs for a particular route.
.. warning:: **입력값 검증** 의 목적으로 라우트 제약사항을 사용하지 마십시오. 그렇게 사용할 경우, 부적절한 입력값이 인입되었을 때 적절한 오류 메시지를 담은 400 응답을 전달하지 않고 404 (페이지 없음) 응답을 전달할 것이기 때문입니다. 라우트 제약사항은 비슷한 라우트들을 **구분짓기 위해** 사용해야 합니다. 특정 라우트에 대한 입력값을 검증하기 위한 목적으로 사용해서는 안됩니다.

The following table demonstrates some route constraints and their expected behavior.
다음 표에서는 일부 라우트 제약사항들과 그 행태를 확인할 수 있습니다.

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
.. warning:: URL 이 CLR 형으로 전환될 수 있는지 검사하는 라우트 제약사항은 언제나 문화권에 상관없는 값을 사용합니다. 즉, URL 이 지역화 가능하지 않은 것으로 가정합니다. 프레임워크에서 제공하는 라우트 제약사항의 경우, 라우트 값에 포함된 값을 변경하지 않습니다. URL 에서 추출된 모든 라우트 값은 문자열로 저장될 것입니다. 예를 들어, `실수 (floar) 라우트 제약사항<https://github.com/aspnet/Routing/blob/1.0.0/src/Microsoft.AspNetCore.Routing/Constraints/FloatRouteConstraint.cs#L44-L60>`__ 의 경우, 라우트 값을 float 형으로 전환해봅니다. 그러나 이러한 값의 전환은 단지 전환이 가능한지 확인할 때만 사용할 뿐입니다.

.. tip:: To constrain a parameter to a known set of possible values, you can use a regular expression ( for example ``{action:regex(list|get|create)}``. This would only match the ``action`` route value to ``list``, ``get``, or ``create``. If passed into the constraints dictionary, the string "list|get|create" would be equivalent. Constraints that are passed in the constraints dictionary (not inline within a template) that don't match one of the known constraints are also treated as regular expressions.
.. tip:: 받을 수 있는 값들의 집합으로 매개변수를 한정 지으려면, 정규식을 사용하면 됩니다. 예를 들면, ``{action:regex(list|get|create)}`` 입니다. 이 정규식에 따르면 ``action`` 라우트 값에는 ``list`` 혹은 ``get`` 혹은 ``create`` 만 일치하는 것으로 판정합니다. 제약사항을 사전 (dictionary) 으로 전달할 경우, 문자열 "list|get|create" 이 사전의 내용에 해당합니다. 사전형으로 제약사항을 전달할 때, 내용 중에 ASP.NET 이 인지하지 못하는 것이 있는 경우 정규식으로서 인식합니다.

.. _url-generation-reference:

URL Generation Reference
------------------------
The example below shows how to generate a link to a route given a dictionary of route values and a ``RouteCollection``.
아래 예시에서는 라우트 값 사전과 ``RouteCollection`` 을 통해 어떤 라우트에 대한 링크를 생성하는 방법을 확인할 수 있습니다.

.. literalinclude:: routing/sample/RoutingSample/Startup.cs
  :start-after: // Show link generation when no routes match.
  :end-before: // End of app.Run
  :dedent: 12

The ``VirtualPath`` generated at the end of the sample above is ``/package/create/123``.
예시의 마지막에서 생성되는 ``VirtualPath`` 는 ``/package/create/123`` 입니다.

The second parameter to the :dn:cls:`~Microsoft.AspNetCore.Routing.VirtualPathContext` constructor is a collection of `ambient values`. Ambient values provide convenience by limiting the number of values a developer must specify within a certain request context. The current route values of the current request are considered ambient values for link generation. For example, in an ASP.NET MVC app if you are in the ``About`` action of the ``HomeController``, you don't need to specify the controller route value to link to the ``Index`` action (the ambient value of ``Home`` will be used).
:dn:cls:`~Microsoft.AspNetCore.Routing.VirtualPathContext` 생성자에 대한 두 번째 매개변수는 `환경 값 (ambient values)` 의 콜렉션입니다. 환경 값올 사용하면 개발자는 특정 요청 컨텍스트 내에서 본인지 지정해야 하는 값의 개수를 줄일 수 있어 편리합니다. 현재 요청의 현재 라우트 값의 경우, 링크 생성 시에 환경 값으로 간주합니다. 예를 들어, ASP.NET MVC 어플리케이션에서 ``HomeController` 의 ``About`` 기능에 위치해있다고 하면, ``Index`` 기능에 대한 링크를 생성할 때 컨트롤러에 대한 라우트 값을 지정하지 않아도 됩니다. (즉, 컨트롤러에 대해 환경 값인 ``Home`` 이 사용될 것입니다.)

Ambient values that don't match a parameter are ignored, and ambient values are also ignored when an explicitly-provided value overrides it, going from left to right in the URL.
일치하는 매개변수가 없는 환경 값은 무시됩니다. 또한 명시적으로 값을 지정한 경우에도 환경 값을 무시합니다. URL 내에서 왼쪽에서 오른쪽으로 값을 추출하면서 환경 값으로 간주합니다.

Values that are explicitly provided but which don't match anything are added to the query string. The following table shows the result when using the route template ``{controller}/{action}/{id?}``.
명시적으로 지정한 값이나, 일치하는 매개변수가 없는 경우에는 쿼리 문자열에 추가합니다. 다음 표에서는 라우트 템플릿 ``{controller}/{action}/{id?}`` 을 사용하는 경우에 환경 값과 외부 지정 값에 의한 결과를 보여주고 있습니다.

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
라우트에 매개변수와 관련없는 기본값을 지정한 상황에서 라우트 값을 명시적으로 전달하려 할 경우, 그 라우트 값은 반드시 기본값에 일치해야 합니다. 예를 들면 다음과 같습니다.

.. code-block:: c#

  routes.MapRoute("blog_route", "blog/{*slug}",
    defaults: new { controller = "Blog", action = "ReadPost" });

Link generation would only generate a link for this route when the matching values for controller and action are provided.
링크를 생성하려 할 때, 컨트롤러와 기능에 대해 일치하는 값을 전달한 경우에만 이 라우트에 대한 링크를 생성될 것입니다.
