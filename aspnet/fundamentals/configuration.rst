.. _fundamentals-configuration:

설정
=============
`Steve Smith`_, `Daniel Roth`_

ASP.NET Core 에서는 다양한 설정 방법을 제공합니다. 어플리케이션 설정 데이터는 JSON 이나 XML, INI 형식의 파일이나 시스템 환경변수, 커맨드 라인 매개변수, 메모리 상의 컬렉션으로 저장할 수 있습니다. 또한 :ref:`사용자 정의 설정 제공자 <custom-config-providers>` 를 통해 여러분 만의 형식을 사용할 수도 있습니다.

.. contents:: Sections:
  :local:
  :depth: 1

`샘플 코드를 확인하거나 다운로드 받으세요. <https://github.com/aspnet/docs/tree/master/aspnet/fundamentals/configuration/sample>`__

설정 상태를 얻거나 지정하기
------------------------------------------

ASP.NET Core 의 설정 시스템은 이전 버전의 ASP.NET 의 시스템을 재설계하여 제작되었습니다. 이전 버전에서는 ``System.Configuration`` 과 ``web.config`` 와 같은 XML 설정 파일에 의존하였습니다. 새 설정 시스템에서는 다양한 데이터 저장 형태를 지원하고 이를 통해 얻은 키-값 형태의 데이터에 대한 스트림라인 형태의 접근 방식을 제공합니다. 어플리케이션과 프레임워크에서 새로운 :ref:`옵션 패턴 <options-config-objects>` 과 강력한 형 (strongly typed) 에 기반한 방식으로 설정에 접근할 수 있습니다.

여러분의 ASP.NET 어플리케이션에서 설정을 사용하려 하는 경우, ``Startup`` 클래스의 ``Configuration`` 을 생성하는 방법 권장합니다. 그리고 각각의 설정에 접근할 때는 :ref:`옵션 패턴 <options-config-objects>` 을 사용하십시오.

At its simplest, ``Configuration`` is just a collection of sources, which provide the ability to read and write name/value pairs. If a name/value pair is written to ``Configuration``, it is not persisted. This means that the written value will be lost when the sources are read again.
가장 간단한 방법으로는 ``Configuration`` 

You must configure at least one source in order for ``Configuration`` to function correctly. The following sample shows how to test working with ``Configuration`` as a key/value store:

.. literalinclude:: configuration/sample/src/CodeSnippets/ConfigSummarySnippet.cs
  :language: c#
  :dedent: 12
  :start-after: // SNIPPET-START
  :end-before: // SNIPPET-END

.. note:: You must set at least one configuration source.

It's not unusual to store configuration values in a hierarchical structure, especially when using external files (e.g. JSON, XML, INI). In this case, configuration values can be retrieved using a ``:`` separated key, starting from the root of the hierarchy. For example, consider the following *appsettings.json* file:

.. _config-json:

.. literalinclude:: /../common/samples/WebApplication1/src/WebApplication1/appsettings.json
  :language: json

The application uses configuration to configure the right connection string. Access to the ``DefaultConnection`` setting is achieved through this key: ``ConnectionStrings:DefaultConnection``, or by using the :dn:method:`~Microsoft.Extensions.Configuration.ConfigurationExtensions.GetConnectionString` extension method and passing in ``"DefaultConnection"``.

The settings required by your application and the mechanism used to specify those settings (configuration being one example) can be decoupled using the :ref:`options pattern <options-config-objects>`. To use the options pattern you create your own options class (probably several different classes, corresponding to different cohesive groups of settings) that you can inject into your application using an options service. You can then specify your settings using configuration or whatever mechanism you choose.

.. note:: You could store your ``Configuration`` instance as a service, but this would unnecessarily couple your application to a single configuration system and specific configuration keys. Instead, you can use the :ref:`Options pattern <options-config-objects>` to avoid these issues.

Using the built-in sources
--------------------------

The configuration framework has built-in support for JSON, XML, and INI configuration files, as well as support for in-memory configuration (directly setting values in code) and the ability to pull configuration from environment variables and command line parameters. Developers are not limited to using a single configuration source. In fact several may be set up together such that a default configuration is overridden by settings from another source if they are present.

Adding support for additional configuration sources is accomplished through extension methods. These methods can be called on a :dn:class:`~Microsoft.Extensions.Configuration.ConfigurationBuilder` instance in a standalone fashion, or chained together as a fluent API. Both of these approaches are demonstrated in the sample below.

.. _custom-config:

.. literalinclude:: configuration/sample/src/CustomConfigurationProvider/Program.cs
  :dedent: 12
  :language: c#
  :lines: 12-25

The order in which configuration sources are specified is important, as this establishes the precedence with which settings will be applied if they exist in multiple locations. In the example below, if the same setting exists in both *appsettings.json* and in an environment variable, the setting from the environment variable will be the one that is used. The last configuration source specified "wins" if a setting exists in more than one location. The ASP.NET team recommends specifying environment variables last, so that the local environment can override anything set in deployed configuration files.

.. note:: To override nested keys through environment variables in shells that don't support ``:`` in variable names, replace them with ``__`` (double underscore).

It can be useful to have environment-specific configuration files. This can be achieved using the following:

.. literalinclude:: /../common/samples/WebApplication1/src/WebApplication1/Startup.cs
  :dedent: 8
  :language: none
  :lines: 20-35
  :emphasize-lines: 6

The :dn:iface:`~Microsoft.AspNetCore.Hosting.IHostingEnvironment` service is used to get the current environment. In the ``Development`` environment, the highlighted line of code above would look for a file named ``appsettings.Development.json`` and use its values, overriding any other values, if it's present. Learn more about :doc:`environments`.

When specifying files as configuration sources, you can optionally specify whether changes to the file should result in the settings being reloaded. This is configured by passing in a ``true`` value for the ``reloadOnChange`` parameter when calling :dn:method:`~Microsoft.Extensions.Configuration.JsonConfigurationExtensions.AddJsonFile` or similar file-based extension methods.

.. warning:: You should never store passwords or other sensitive data in configuration provider code or in plain text configuration files. You also shouldn't use production secrets in your development or test environments. Instead, such secrets should be specified outside the project tree, so they cannot be accidentally committed into the configuration provider repository. Learn more about :doc:`environments` and managing :doc:`/security/app-secrets`.

One way to leverage the order precedence of ``Configuration`` is to specify default values, which can be overridden. In the console application below, a default value for the ``username`` setting is specified in an in-memory collection, but this is overridden if a command line argument for ``username`` is passed to the application. You can see in the output how many different configuration sources are configured in the application at each stage of its execution.

.. literalinclude:: configuration/sample/src/ConfigConsole/Program.cs
  :emphasize-lines: 22,25
  :linenos:
  :language: none

When run, the program will display the default value unless a command line parameter overrides it.

.. image:: configuration/_static/config-console.png

.. _options-config-objects:

Using Options and configuration objects
---------------------------------------

The options pattern enables using custom options classes to represent a group of related settings. A class needs to have a public read-write property for each setting and a constructor that does not take any parameters (e.g. a default constructor) in order to be used as an options class.

It's recommended that you create well-factored settings objects that correspond to certain features within your application, thus following the `Interface Segregation Principle (ISP) <http://deviq.com/interface-segregation-principle/>`_ (classes depend only on the configuration settings they use) as well as `Separation of Concerns <http://deviq.com/separation-of-concerns/>`_ (settings for disparate parts of your app are managed separately, and thus are less likely to negatively impact one another).

A simple ``MyOptions`` class is shown here:

.. literalinclude:: configuration/sample/src/UsingOptions/Models/MyOptions.cs
  :language: c#
  :lines: 3-7
  :dedent: 4

Options can be injected into your application using the :dn:iface:`~Microsoft.Extensions.Options.IOptions\<TOptions>` accessor service. For example, the following :doc:`controller </mvc/controllers/index>`  uses ``IOptions<MyOptions>`` to access the settings it needs to render the ``Index`` view:

.. literalinclude:: configuration/sample/src/UsingOptions/Controllers/HomeController.cs
  :language: c#
  :lines: 9-20
  :dedent: 4
  :emphasize-lines: 3,5,8

.. tip:: Learn more about :doc:`dependency-injection`.

To setup the :dn:iface:`~Microsoft.Extensions.Options.IOptions\<TOptions>` service you call the :dn:method:`~Microsoft.Extensions.DependencyInjection.OptionsServiceCollectionExtensions.AddOptions` extension method during startup in your ``ConfigureServices`` method:

.. literalinclude:: configuration/sample/src/UsingOptions/Startup.cs
  :language: c#
  :lines: 26-30
  :emphasize-lines: 4
  :dedent: 8

.. _options-example:

The ``Index`` view displays the configured options:

.. image:: configuration/_static/index-view.png

You configure options using the :dn:method:`~Microsoft.Extensions.DependencyInjection.OptionsServiceCollectionExtensions.Configure\<TOptions>` extension method. You can configure options using a delegate or by binding your options to configuration:

.. literalinclude:: configuration/sample/src/UsingOptions/Startup.cs
  :language: c#
  :lines: 26-45
  :dedent: 8
  :emphasize-lines: 7,10-13,16

When you bind options to configuration, each property in your options type is bound to a configuration key of the form ``property:subproperty:...``. For example, the ``MyOptions.Option1`` property is bound to the key ``Option1``, which is read from the ``option1`` property in *appsettings.json*. Note that configuration keys are case insensitive.

Each call to :dn:method:`~Microsoft.Extensions.DependencyInjection.OptionsServiceCollectionExtensions.Configure\<TOptions>` adds an :dn:iface:`~Microsoft.Extensions.Options.IConfigureOptions\<TOptions>` service to the service container that is used by the :dn:iface:`~Microsoft.Extensions.Options.IOptions\<TOptions>` service to provide the configured options to the application or framework. If you want to configure your options using objects that must be obtained from the service container (for example, to read settings from a database) you can use the ``AddSingleton<IConfigureOptions<TOptions>>`` extension method to register a custom :dn:iface:`~Microsoft.Extensions.Options.IConfigureOptions\<TOptions>` service.

You can have multiple :dn:iface:`~Microsoft.Extensions.Options.IConfigureOptions\<TOptions>` services for the same option type and they are all applied in order. In the :ref:`example <options-example>` above, the values of ``Option1`` and ``Option2`` are both specified in `appsettings.json`, but the value of ``Option1`` is overridden by the configured delegate with the value "value1_from_action".

.. _custom-config-providers:

Writing custom providers
------------------------

In addition to using the built-in configuration providers, you can also write your own. To do so, you simply implement the :dn:iface:`~Microsoft.Extensions.Configuration.IConfigurationSource` interface, which exposes a :dn:method:`~Microsoft.Extensions.Configuration.IConfigurationSource.Build` method. The build method configures and returns an :dn:iface:`~Microsoft.Extensions.Configuration.IConfigurationProvider`.

Example: Entity Framework Settings
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

You may wish to store some of your application's settings in a database, and access them using Entity Framework Core (EF). There are many ways in which you could choose to store such values, ranging from a simple table with a column for the setting name and another column for the setting value, to having separate columns for each setting value. In this example, we're going to create a simple configuration provider that reads name-value pairs from a database using EF.

To start off we'll define a simple ``ConfigurationValue`` entity for storing configuration values in the database:

.. literalinclude:: configuration/sample/src/CustomConfigurationProvider/ConfigurationValue.cs
  :language: c#
  :lines: 3-7
  :dedent: 4

You need a ``ConfigurationContext`` to store and access the configured values using EF:

.. literalinclude:: configuration/sample/src/CustomConfigurationProvider/ConfigurationContext.cs
  :language: c#
  :lines: 5-12
  :dedent: 4
  :emphasize-lines: 7

Create an ``EntityFrameworkConfigurationSource`` that inherits from :dn:iface:`~Microsoft.Extensions.Configuration.IConfigurationSource`:

.. literalinclude:: configuration/sample/src/CustomConfigurationProvider/EntityFrameworkConfigurationSource.cs
  :language: c#
  :emphasize-lines: 7,16-19

Next, create the custom configuration provider by inheriting from :dn:class:`~Microsoft.Extensions.Configuration.ConfigurationProvider`. The configuration data is loaded by overriding the ``Load`` method, which reads in all of the configuration data from the configured database. For demonstration purposes, the configuration provider also takes care of initializing the database if it hasn't already been created and populated:

.. literalinclude:: configuration/sample/src/CustomConfigurationProvider/EntityFrameworkConfigurationProvider.cs
  :language: c#
  :emphasize-lines: 9,18-30,37-38

Note the values that are being stored in the database ("value_from_ef_1" and "value_from_ef_2"); these are displayed in the sample below to demonstrate the configuration is reading values from the database properly.

By convention you can also add an ``AddEntityFrameworkConfiguration`` extension method for adding the configuration source:

.. literalinclude:: configuration/sample/src/CustomConfigurationProvider/EntityFrameworkExtensions.cs
  :language: c#
  :emphasize-lines: 9

You can see an example of how to use this custom configuration provider in your application in the following example. Create a new :dn:class:`~Microsoft.Extensions.Configuration.ConfigurationBuilder` to set up your configuration sources. To add the ``EntityFrameworkConfigurationProvider``, you first need to specify the EF data provider and connection string. How should you configure the connection string? Using configuration of course! Add an *appsettings.json* file as a configuration source to bootstrap setting up the ``EntityFrameworkConfigurationProvider``. By adding the database settings to an existing configuration with other sources specified, any settings specified in the database will override settings specified in *appsettings.json*:

.. literalinclude:: configuration/sample/src/CustomConfigurationProvider/Program.cs
  :language: c#
  :emphasize-lines: 21-24

Run the application to see the configured values:

.. image:: configuration/_static/custom-config.png

Summary
-------

ASP.NET Core provides a very flexible configuration model that supports a number of different file-based options, as well as command-line, in-memory, and environment variables. It works seamlessly with the options model so that you can inject strongly typed settings into your application or framework. You can create your own custom configuration providers as well, which can work with or replace the built-in providers, allowing for extreme flexibility.
