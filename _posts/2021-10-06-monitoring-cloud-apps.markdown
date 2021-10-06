---
layout: post
title:  "Exercise 9 - Monitoring Cloud Apps"
date:   2021-10-06 18:52
tags: Azure, Application Insights, Serilog, Logging
---

<h1 style="color:red;">WIP, not complete!</h1>


## The assignment

The assignment this week was to take an older project (e.g. the one from assignment #6), implement logging to **Application Insights**, deploy the app to **Azure App Service** and create at least **five** Kusto queries.


## The application

The application is the one I created for [assignment #6 (a simple note taking application)][blog-post-6].

To get my app up and running as an Azure Web App (App Service) I followed these steps:
1. Copy the code from the app I created for assignment #6 to a new directory and create and publish a GitHub repository for it.
1. Test the CI/CD workflow file by creating a "release" branch from the "main" branch. This workflow builds and pushes a Docker image to GHCR (GitHub Container Registry).
1. Create a new **Azure Web App** and download its **publish profile**.
1. Modify the workflow to also publish the image from **GHCR** to my **Azure App Service**.
    1. First add this step to the workflow file:
        ```yaml
        - uses: azure/webapps-deploy@v2
            with:
                app-name: 'joch-cloudapps-assignment-9-web-app'
                publish-profile: ${{ secrets.AZURE_WEBAPP_PUBLISH_PROFILE }}
                images: 'ghcr.io/johancz/assignment-9-azurewebappwithlogging:latest'

        ```
    1. Add the contents of the **publish profile** to a new GitHub Secret called `AZURE_WEBAPP_PUBLISH_PROFILE`.
1. Add the **CosmosDB** configuration data (CosmosDb account uri & key, database name and container name) to the **Web Webb**'s configuration store in Azure.
    ![](/Molnapplikationer-Blogg/data/images/exercise-9-monitoring-cloud-apps/azure-porta-web-app-configuration-cosmosdb.png)



## A diagram



## The code

Let's start in the Program class (Program.cs):
```csharp
public class Program
{
    public static int Main(string[] args)
    {
        Log.Logger = new LoggerConfiguration()
            .MinimumLevel.Override("Microsoft", LogEventLevel.Information)
            //.MinimumLevel.Override("Microsoft.AspNetCore", LogEventLevel.Warning)
            //.Enrich.WithMachineName()
            //.Enrich.WithProcessId()
            .Enrich.FromLogContext()
            .WriteTo.Console(new CompactJsonFormatter())
            .WriteTo.File(@"Logs\trace.log",
                fileSizeLimitBytes: 5_000_000,
                rollOnFileSizeLimit: true,
                shared: true,
                flushToDiskInterval: TimeSpan.FromSeconds(1))
            .WriteTo.ApplicationInsights(new TelemetryConfiguration { InstrumentationKey = "04efd465-67ba-4173-b3bb-a9ed1ded8f48" }, TelemetryConverter.Traces)
            .CreateLogger();

        try
        {
            Log.Information("Starting web host");
            CreateHostBuilder(args).Build().Run();
            return 0;
        }
        catch (Exception e)
        {
            Log.Fatal(e, "Host terminated unexpectedly");
            return 1;
        }
        finally
        {
            Log.CloseAndFlush();
        }
    }

    public static IHostBuilder CreateHostBuilder(string[] args) =>
        Host.CreateDefaultBuilder(args)
            .UseSerilog() // <------------------------------------------ Added
            .ConfigureWebHostDefaults(webBuilder =>
            {
                webBuilder.UseStartup<Startup>();
            });
}
```

### Startup.cs
```csharp
public void ConfigureServices(IServiceCollection services)
{
    Log.Logger.Information("Configuring Services");
    services.AddSingleton(Log.Logger);
    services.AddApplicationInsightsTelemetry("04efd465-67ba-4173-b3bb-a9ed1ded8f48");
    services.AddSingleton<ICosmosDbService>(InitializeCosmosClientInstanceAsync(Configuration.GetSection("CosmosDb"), Log.Logger).GetAwaiter().GetResult());
    services.AddRazorPages();
}
```

In the **Configure** method:

```csharp
// Enable Serilog's smarter HTTP request logging middleware.
app.UseSerilogRequestLogging();
```



### Index.cshtml.cs:

```csharp
public IndexModel(ILogger logger, ICosmosDbService cosmosDbService)
//public IndexModel(ILogger logger)
{
    _logger = logger.ForContext<IndexModel>();
    _cosmosDbService = cosmosDbService;
    //_cosmosDbService = null;
}
```

### appsettings.json
```json
"Serilog": {
    "MinimumLevel": {
        "Default": "Information",
        "Override": {
            "System": "Warning",
            "Microsoft": "Warning",
            "Microsoft.EntityFrameworkCore": "Warning"
        }
    }
}
```



## How can logging/monitoring your app improve its security?

With Azure Monitoring you can keep an eye of all of your resources. You can for example query for any VMs that are missing Window Updates, which should be extremely useful since outdated software can result in major security incidents. 


All pertinent data from logs and monitors can be presented on the organizations Azure Dashboard or in Azure Workbooks. If the data points to critical security issues, alerts can be set up to push out SMS or email messages to the security team.

The security team can monitor heartbeats to identify when servers are unavailable, which might indicate a denial-of-service (dos) attack.


## The queries

### Query 1: Get all logs with a severity level of warning or higher.
```kusto
traces | where severityLevel >= 2 
```


## Sources & Links
- [Exercise 6 - Web Apps in the Cloud - this blog][blog-post-6]
- [Analyze your Azure infrastructure by using Azure Monitor logs - Microsoft Learn @ docs.microsoft.com][docs.microsoft.com-learn-analyze-infrastructure-with-azure-monitor-logs]


[blog-post-6]: https://johancz.github.io/Molnapplikationer-Blogg/2021/09/24/exercise-6-web-apps-in-the-cloud.html
[docs.microsoft.com-learn-analyze-infrastructure-with-azure-monitor-logs]: https://docs.microsoft.com/en-us/learn/modules/analyze-infrastructure-with-azure-monitor-logs/
