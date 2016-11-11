:version: 1.0.0

Working with Multiple Environments (류지형님)
==================================

By `Steve Smith`_

ASP.NET Core introduces improved support for controlling application behavior across multiple environments, such as development, staging, and production. Environment variables are used to indicate which environment the application is running in, allowing the app to be configured appropriately.

.. contents:: Sections:
  :local:
  :depth: 1

`View or download sample code <https://github.com/aspnet/Docs/tree/master/aspnet/fundamentals/environments/sample>`__

Development, Staging, Production
--------------------------------

ASP.NET Core references a particular `environment variable <https://github.com/aspnet/Home/wiki/Environment-Variables>`_, ``ASPNETCORE_ENVIRONMENT`` to describe the environment the application is currently running in. This variable can be set to any value you like, but three values are used by convention: ``Development``, ``Staging``, and ``Production``. You will find these values used in the samples and templates provided with ASP.NET Core.

The current environment setting can be detected programmatically from within your application. In addition, you can use the Environment :doc:`tag helper </mvc/views/tag-helpers/index>` to include certain sections in your :doc:`view </mvc/views/index>` based on the current application environment.

.. note:: The specified environment name is case insensitive. Whether you set the variable to ``Development`` or ``development`` or ``DEVELOPMENT`` the results will be the same.

Development
^^^^^^^^^^^

This should be the environment used when developing an application. When using Visual Studio, this setting can be specified in your project's debug profiles, such as for IIS Express, shown here:

.. image:: environments/_static/project-properties-debug.png

When you modify the default settings created with the project, your changes are persisted in *launchSettings.json* in the ``Properties`` folder. This file holds settings specific to each profile Visual Studio is configured to use to launch the application, including any environment variables that should be used. (Debug profiles are discussed in more detail in :doc:`servers`). For example, after adding another profile configured to use IIS Express, but using an ``ASPNETCORE_ENVIRONMENT`` value of ``Staging``, the ``launchSettings.json`` file in our sample project is shown below:

.. literalinclude:: environments/sample/src/Environments/Properties/launchSettings.json
  :language: javascript
  :caption: launchSettings.json
  :emphasize-lines: 15,22

.. note:: Changes made to project profiles or to *launchSettings.json* directly may not take effect until the web server used is restarted (in particular, Kestrel must be restarted before it will detect changes made to its environment).

You can create multiple different launch profiles for various different configurations of your application, including those that require other environment variables.

.. warning:: Environment variables stored in *launchSettings.json* are not secured in any way and will be part of the source code repository for your project, if you use one. **Never store credentials or other secret data in this file.** If you need a place to store such data, use the *Secret Manager* tool described in :ref:`security-app-secrets`.

Staging
^^^^^^^

By convention, a ``Staging`` environment is a pre-production environment used for final testing before deployment to production. Ideally, its physical characteristics should mirror that of production, so that any issues that may arise in production occur first in the staging environment, where they can be addressed without impact to users.

Production
^^^^^^^^^^

The ``Production`` environment is the environment in which the application runs when it is live and being used by end users. This environment should be configured to maximize security, performance, and application robustness. Some common settings that a production environment might have that would differ from development include:

- Turn on caching
- Ensure all client-side resources are bundled, minified, and potentially served from a CDN
- Turn off diagnostic ErrorPages
- Turn on friendly error pages
- Enable production logging and monitoring (for example, `Application Insights <https://azure.microsoft.com/en-us/documentation/articles/app-insights-asp-net-five/>`_)

This is by no means meant to be a complete list. It's best to avoid scattering environment checks in many parts of your application. Instead, the recommended approach is to perform such checks within the application's ``Startup`` class(es) wherever possible

Determining the environment at runtime
--------------------------------------

The :dn:iface:`~Microsoft.AspNetCore.Hosting.IHostingEnvironment` service provides the core abstraction for working with environments. This service is provided by the ASP.NET hosting layer, and can be injected into your startup logic via :doc:`dependency-injection`. The ASP.NET Core web site template in Visual Studio uses this approach to load environment-specific configuration files (if present) and to customize the app's error handling settings. In both cases, this behavior is achieved by referring to the currently specified environment by calling :dn:prop:`~Microsoft.AspNetCore.Hosting.IHostingEnvironment.EnvironmentName` or :dn:method:`~Microsoft.AspNetCore.Hosting.HostingEnvironmentExtensions.IsEnvironment` on the instance of :dn:iface:`~Microsoft.AspNetCore.Hosting.IHostingEnvironment` passed into the appropriate method.

.. note:: If you need to check whether the application is running in a particular environment, use ``env.IsEnvironment("environmentname")`` since it will correctly ignore case (instead of checking if ``env.EnvironmentName == "Development"`` for example).

For example, you can use the following code in your Configure method to setup environment specific error handling:

.. literalinclude:: environments/sample/src/Environments/Startup.cs
  :language: c#
  :dedent: 8
  :start-after: STARTSNIP:error-handling
  :end-before: ENDSNIP:error-handling

If the app is running in a ``Development`` environment, then it enables the runtime support necessary to use the "BrowserLink" feature in Visual Studio, development-specific error pages (which typically should not be run in production) and special database error pages (which provide a way to apply migrations and should therefore only be used in development). Otherwise, if the app is not running in a development environment, a standard error handling page is configured to be displayed in response to any unhandled exceptions.

You may need to determine which content to send to the client at runtime, depending on the current environment. For example, in a development environment you generally serve non-minimized scripts and style sheets, which makes debugging easier. Production and test environments should serve the minified versions and generally from a CDN. You can do this using the Environment :doc:`tag helper </mvc/views/tag-helpers/intro>`. The Environment tag helper will only render its contents if the current environment matches one of the environments specified using the ``names`` attribute.

.. literalinclude:: environments/sample/src/Environments/Views/Shared/_Layout.cshtml
  :language: html
  :dedent: 4
  :start-after: STARTSNIP:env-tag-helper
  :end-before: ENDSNIP:env-tag-helper

To get started with using tag helpers in your application see :doc:`/mvc/views/tag-helpers/intro`.

Startup conventions
-------------------

ASP.NET Core supports a convention-based approach to configuring an application's startup based on the current environment. You can also programmatically control how your application behaves according to which environment it is in, allowing you to create and manage your own conventions.

When an ASP.NET Core application starts, the ``Startup`` class is used to bootstrap the application, load its configuration settings, etc. (:doc:`learn more about ASP.NET startup <startup>`). However, if a class exists named ``Startup{EnvironmentName}`` (for example ``StartupDevelopment``), and the ``ASPNETCORE_ENVIRONMENT`` environment variable matches that name, then that ``Startup`` class is used instead. Thus, you could configure ``Startup`` for development, but have a separate ``StartupProduction`` that would be used when the app is run in production. Or vice versa.

In addition to using an entirely separate ``Startup`` class based on the current environment, you can also make adjustments to how the application is configured within a ``Startup`` class. The ``Configure()`` and ``ConfigureServices()`` methods support environment-specific versions similar to the ``Startup`` class itself, of the form ``Configure{EnvironmentName}()`` and ``Configure{EnvironmentName}Services()``. If you define a method ``ConfigureDevelopment()`` it will be called instead of ``Configure()`` when the environment is set to development. Likewise, ``ConfigureDevelopmentServices()`` would be called instead of ``ConfigureServices()`` in the same environment.

Summary
-------

ASP.NET Core provides a number of features and conventions that allow developers to easily control how their applications behave in different environments. When publishing an application from development to staging to production, environment variables set appropriately for the environment allow for optimization of the application for debugging, testing, or production use, as appropriate.

Additional Resources
--------------------

- :doc:`configuration`
- :doc:`/mvc/views/tag-helpers/intro`
