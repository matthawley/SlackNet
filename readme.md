# SlackNet
An easy-to-use and comprehensive API for writing Slack apps in .NET.

This project builds on the [Slack API](https://api.slack.com/). You should [read through Slack's documentation](https://api.slack.com/start) to get an understanding of how Slack apps work and what you can do with them before using SlackNet.

- [Getting Started](#getting-started)
  - [SlackNet](#slacknet)
    - [Socket Mode](#socket-mode)
  - [SlackNet.AspNetCore](#slacknetaspnetcore)
    - [Developing with Socket Mode](#developing-with-socket-mode)
    - [Azure Functions](#azure-functions)
    - [Endpoints naming convention](#endpoints-naming-convention)
- [Examples](./Examples)
- [Contributing](#contributing)

## Getting Started
There are two main NuGet packages available to install, depending on your use case.
  - [SlackNet](https://www.nuget.org/packages/SlackNet/): A comprehensive Slack API client for .NET, including a socket mode client for receiving events.
  - [SlackNet.AspNetCore](https://www.nuget.org/packages/SlackNet.AspNetCore/): ASP.NET Core integration for receiving requests from Slack.

A [SlackNet.Bot](SlackNet.Bot#slacknetbot) package for using Slack's deprecated RTM API is also available, but you're probably better off using the [Socket Mode client](#socket-mode) instead.

### SlackNet
To use the Web API, build the API client:
```c#
var api = new SlackServiceBuilder()
    .UseApiToken("<your bot or user OAuth token here>")
    .GetApiClient();
```
then call any of Slack's [many API methods](https://api.slack.com/methods):
```c#
await api.Chat.PostMessage(new Message { Text = "Hello, Slack!", Channel = "#general" });
```

#### Socket Mode
To use the socket mode client:
```c#
var client = new SlackServiceBuilder()
    .UseAppLevelToken("<app-level OAuth token required for socket mode>")
    /* Register handlers here */
    .GetSocketModeClient();
await client.Connect();
```

A range of handler registration methods are available, but all require that you construct the handlers manually. You can simplify handler registration by integrating with a DI container. Integrations are provided for Autofac, Microsoft.Extensions.DependencyInjection, and SimpleInjector. [Examples are provided](./Examples) for each of these options. 

### SlackNet.AspNetCore
Configure SlackNet in your Startup class:
```c#
public void ConfigureServices(IServiceCollection services)
{
    services.AddSlackNet(c => c.UseApiToken("<your OAuth access token here>"));
}

public void Configure(IApplicationBuilder app, IHostingEnvironment env)
{
    app.UseSlackNet(c => c.UseSigningSecret("<your signing secret here>"));
}
```

or in the setup of your ASP.NET Core 6.0 app:
```c#
builder.Services.AddSlackNet(c => c.UseApiToken("<your bot or user OAuth token here>"));
var app = builder.Build();
app.UseSlackNet(c => c.UseSigningSecret("<your signing secret here>"));
```

Add event handler registrations inside the AddSlackNet callback. See the [SlackNetDemo](Examples/SlackNetDemo) project for more detail.

#### Developing with Socket Mode

While developing an ASP.NET application, you can use socket mode instead of needing to host the website publicly, by enabling socket mode with:

```c#
app.UseSlackNet(c => c.UseSocketMode());
```

#### Azure Functions
SlackNet.AspNetCore can be used in Azure Functions as well, although it's a little more manual at the moment.

You'll need to [enable dependency injection](https://docs.microsoft.com/en-us/azure/azure-functions/functions-dotnet-dependency-injection) in your project, then include in your Startup class:
```c#
public override void Configure(IFunctionsHostBuilder builder)
{
    builder.Services.AddSlackNet(c => c.UseApiToken("<your bot or user OAuth token here>"));
    builder.Services.AddSingleton(new SlackEndpointConfiguration()
        .UseSigningSecret("<your signing secret here>"));
}
```

Copy the [SlackEndpoint](Examples/AzureFunctionExample/SlackEndpoints.cs) class into your project.
This provides the functions for Slack to call, and delegates request handling the same way the regular ASP.NET integration does, with the same methods for registering event handlers as above.

See the [AzureFunctionExample](Examples/AzureFunctionExample) project for more detail.

#### Endpoints naming convention

SlackNet requires the Slack app endpoints to be named after the following convention:

| Endpoint            | Route                    |
|---------------------|--------------------------|
| Event subscriptions | `{route_prefix}/event`   |
| Interactivity       | `{route_prefix}/action`  |
| Select menus        | `{route_prefix}/options` |
| Slash commands      | `{route_prefix}/command` |

By default, the value of `{route_prefix}` is `slack`, but this can be configured like so:

```c#
public void Configure(IApplicationBuilder app, IHostingEnvironment env)
{
    app.UseSlackNet(c => c.UseSigningSecret("<your signing secret here>").MapToPrefix("api/slack"));
}
```

## Examples
[Several example projects](./Examples) are provided to help you get started.

## Contributing
Contributions are welcome. Currently, changes must be made on a feature branch, otherwise the CI build will fail.

Slack's API is large and changes often, and while their documentation is very good, it's not always 100% complete or accurate, which can easily lead to bugs or missing features in SlackNet.
Raising issues or submitting pull requests for these sorts of discrepencies is highly appreciated, as realistically I have to rely on the documentation unless I happen to be using a particular API myself.
