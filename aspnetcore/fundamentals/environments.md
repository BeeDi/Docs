---
title: Use multiple environments in ASP.NET Core
author: rick-anderson
description: Learn how to control app behavior across multiple environments in ASP.NET Core apps.
ms.author: riande
ms.date: 06/21/2018
uid: fundamentals/environments
---
# Use multiple environments in ASP.NET Core

By [Rick Anderson](https://twitter.com/RickAndMSFT)

ASP.NET Core configures app behavior based on the runtime environment using an environment variable.

[View or download sample code](https://github.com/aspnet/Docs/tree/master/aspnetcore/fundamentals/environments/sample) ([how to download](xref:tutorials/index#how-to-download-a-sample))

## Environments

ASP.NET Core reads the environment variable `ASPNETCORE_ENVIRONMENT` at app startup and stores the value in [IHostingEnvironment.EnvironmentName](/dotnet/api/microsoft.aspnetcore.hosting.ihostingenvironment.environmentname). You can set `ASPNETCORE_ENVIRONMENT` to any value, but [three values](/dotnet/api/microsoft.aspnetcore.hosting.environmentname) are supported by the framework: [Development](/dotnet/api/microsoft.aspnetcore.hosting.environmentname.development), [Staging](/dotnet/api/microsoft.aspnetcore.hosting.environmentname.staging), and [Production](/dotnet/api/microsoft.aspnetcore.hosting.environmentname.production). If `ASPNETCORE_ENVIRONMENT` isn't set, it defaults to `Production`.

[!code-csharp[](environments/sample/WebApp1/Startup.cs?name=snippet)]

The preceding code:

* Calls [UseDeveloperExceptionPage](/dotnet/api/microsoft.aspnetcore.builder.developerexceptionpageextensions.usedeveloperexceptionpage) and [UseBrowserLink](/dotnet/api/microsoft.aspnetcore.builder.browserlinkextensions.usebrowserlink) when `ASPNETCORE_ENVIRONMENT` is set to `Development`.
* Calls [UseExceptionHandler](/dotnet/api/microsoft.aspnetcore.builder.exceptionhandlerextensions.useexceptionhandler) when the value of `ASPNETCORE_ENVIRONMENT` is set one of the following:

    * `Staging`
    * `Production`
    * `Staging_2`

The [Environment Tag Helper](xref:mvc/views/tag-helpers/builtin-th/environment-tag-helper) uses the value of `IHostingEnvironment.EnvironmentName` to include or exclude markup in the element:

[!code-cshtml[](environments/sample-snapshot/WebApp1/Pages/About.cshtml)]

On Windows and macOS, environment variables and values aren't case sensitive. Linux environment variables and values are **case sensitive** by default.

### Development

The development environment can enable features that shouldn't be exposed in production. For example, the ASP.NET Core templates enable the [developer exception page](xref:fundamentals/error-handling#the-developer-exception-page) in the development environment.

The environment for local machine development can be set in the *Properties\launchSettings.json* file of the project. Environment values set in *launchSettings.json* override values set in the system environment.

The following JSON shows three profiles from a *launchSettings.json* file:

[!code-json[](environments/sample/WebApp1/Properties/launchSettings.json?highlight=10,11,18,26)]

::: moniker range=">= aspnetcore-2.1"

> [!NOTE]
> The `applicationUrl` property in *launchSettings.json* can specify a list of server URLs. Use a semicolon between the URLs in the list:
>
> ```json
> "WebApplication1": {
>    "commandName": "Project",
>    "launchBrowser": true,
>    "applicationUrl": "https://localhost:5001;http://localhost:5000",
>    "environmentVariables": {
>      "ASPNETCORE_ENVIRONMENT": "Development"
>    }
> }
> ```

::: moniker-end

When the app is launched with [dotnet run](/dotnet/core/tools/dotnet-run), the first profile with `"commandName": "Project"` is used. The value of `commandName` specifies the web server to launch. `commandName` can be any one of the following:

* IIS Express
* IIS
* Project (which launches Kestrel)

When an app is launched with [dotnet run](/dotnet/core/tools/dotnet-run):

* *launchSettings.json* is read if available. `environmentVariables` settings in *launchSettings.json* override environment variables.
* The hosting environment is displayed.

The following output shows an app started with [dotnet run](/dotnet/core/tools/dotnet-run):

```bash
PS C:\Webs\WebApp1> dotnet run
Using launch settings from C:\Webs\WebApp1\Properties\launchSettings.json...
Hosting environment: Staging
Content root path: C:\Webs\WebApp1
Now listening on: http://localhost:54340
Application started. Press Ctrl+C to shut down.
```

The Visual Studio **Debug** tab provides a GUI to edit the *launchSettings.json* file:

![Project Properties Setting Environment variables](environments/_static/project-properties-debug.png)

Changes made to project profiles may not take effect until the web server is restarted. Kestrel must be restarted before it can detect changes made to its environment.

> [!WARNING]
> *launchSettings.json* shouldn't store secrets. The [Secret Manager tool](xref:security/app-secrets) can be used to store secrets for local development.

When using [Visual Studio Code](https://code.visualstudio.com/), environment variables can be set in the *.vscode/launch.json* file. The following example sets the environment to `Development`:

```json
{
   "version": "0.2.0",
   "configurations": [
        {
            "name": ".NET Core Launch (web)",

            ... additional VS Code configuration settings ...

            "env": {
                "ASPNETCORE_ENVIRONMENT": "Development"
            }
        }
    ]
}
```

A *.vscode/launch.json* file in the project isn't read when starting the app with `dotnet run` in the same way as *Properties/launchSettings.json*. When launching an app in development that doesn't have a *launchSettings.json* file, either set the environment with an environment variable or a command-line argument to the `dotnet run` command.

### Production

The production environment should be configured to maximize security, performance, and app robustness. Some common settings that differ from development include:

* Caching.
* Client-side resources are bundled, minified, and potentially served from a CDN.
* Diagnostic error pages disabled.
* Friendly error pages enabled.
* Production logging and monitoring enabled. For example, [Application Insights](/azure/application-insights/app-insights-asp-net-core).

## Setting the environment

It's often useful to set a specific environment for testing. If the environment isn't set, it defaults to `Production`, which disables most debugging features. The method for setting the environment depends on the operating system.

### Azure App Service

To set the environment in [Azure App Service](https://azure.microsoft.com/services/app-service/), perform the following steps:

1. Select the app from the **App Services** blade.
1. In the **SETTINGS** group, select the **Application settings** blade.
1. In the **Application settings** area, select **Add new setting**.
1. For **Enter a name**, provide `ASPNETCORE_ENVIRONMENT`. For **Enter a value**, provide the environment (for example, `Staging`).
1. Select the **Slot Setting** check box if you wish the environment setting to remain with the current slot when deployment slots are swapped. For more information, see [Azure Documentation: Which settings are swapped?](/azure/app-service/web-sites-staged-publishing).
1. Select **Save** at the top of the blade.

Azure App Service automatically restarts the app after an app setting (environment variable) is added, changed, or deleted in the Azure portal.

### Windows

To set the `ASPNETCORE_ENVIRONMENT` for the current session when the app is started using [dotnet run](/dotnet/core/tools/dotnet-run), the following commands are used:

**Command prompt**

```console
set ASPNETCORE_ENVIRONMENT=Development
```

**PowerShell**

```powershell
$Env:ASPNETCORE_ENVIRONMENT = "Development"
```

These commands only take effect for the current window. When the window is closed, the `ASPNETCORE_ENVIRONMENT` setting reverts to the default setting or machine value. To set the value globally in Windows, open the **Control Panel** > **System** > **Advanced system settings** and add or edit the `ASPNETCORE_ENVIRONMENT` value:

![System Advanced Properties](environments/_static/systemsetting_environment.png)

![ASPNET Core Environment Variable](environments/_static/windows_aspnetcore_environment.png)

**web.config**

See the *Setting environment variables* section of the [ASP.NET Core Module configuration reference](xref:host-and-deploy/aspnet-core-module#setting-environment-variables) topic.

**Per IIS Application Pool**

To set environment variables for individual apps running in isolated Application Pools (supported on IIS 10.0+), see the *AppCmd.exe command* section of the [Environment Variables &lt;environmentVariables&gt;](/iis/configuration/system.applicationHost/applicationPools/add/environmentVariables/#appcmdexe) topic.

### macOS

Setting the current environment for macOS can be performed in-line when running the app:

```bash
ASPNETCORE_ENVIRONMENT=Development dotnet run
```

Alternatively, set the environment with `export` prior to running the app:

```bash
export ASPNETCORE_ENVIRONMENT=Development
```

Machine-level environment variables are set in the *.bashrc* or *.bash_profile* file. Edit the file using any text editor. Add the following statement:

```bash
export ASPNETCORE_ENVIRONMENT=Development
```

### Linux

For Linux distros, use the `export` command at a command prompt for session-based variable settings and *bash_profile* file for machine-level environment settings.

### Configuration by environment

See [Configuration by environment](xref:fundamentals/configuration/index#configuration-by-environment) for more information.

## Environment-based Startup class and methods

When an ASP.NET Core app starts, the [Startup class](xref:fundamentals/startup) bootstraps the app. If a `Startup{EnvironmentName}` class exists, the class is called for that `EnvironmentName`:

[!code-csharp[](environments/sample/WebApp1/StartupDev.cs?name=snippet&highlight=1)]

[WebHostBuilder.UseStartup<TStartup>](/dotnet/api/microsoft.aspnetcore.hosting.webhostbuilderextensions.usestartup#Microsoft_AspNetCore_Hosting_WebHostBuilderExtensions_UseStartup__1_Microsoft_AspNetCore_Hosting_IWebHostBuilder_) overrides configuration sections.

[Configure](/dotnet/api/microsoft.aspnetcore.hosting.startupbase.configure) and [ConfigureServices](/dotnet/api/microsoft.aspnetcore.hosting.startupbase.configureservices) support environment-specific versions of the form `Configure{EnvironmentName}` and `Configure{EnvironmentName}Services`:

[!code-csharp[](environments/sample/WebApp1/Startup.cs?name=snippet_all&highlight=15,37)]

## Additional resources

* [Application startup](xref:fundamentals/startup)
* [Configuration](xref:fundamentals/configuration/index)
* [IHostingEnvironment.EnvironmentName](/dotnet/api/microsoft.aspnetcore.hosting.ihostingenvironment.environmentname)
