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



### Libraries (nu-get packages):

We use a number of libraries, such as like **Serilog** for structured logging, sinks for serilog and a library to read configurations for  serilog from the config file (appsettings.json).

### Serilog Sinks
Sinks are essentially logging targets, i.e. Console, File or ApplicationInsights. They route the logging data to various targets and formats.


### Serilog Enrichers:

You can use extensions for Serilog called "Enrichers" that each provide access to more data, such as domain-name, computer name, username and environment name with **Serilog.Enrichers.Environment**
I could've used more libraries for collecting data such as "Serilog.Enrichers.Environment" to collect more data such as the MachineName, EnvironmentUserName and EnvironmentName the error originates from.


#### Serilog.Settings.Configuration

A settings provider for Serilog which reads from `Microsoft.Extensions.Configuration` sources, e,g. "appsettings.json" and "appsettings.Development.json, for any Serilog configuration.



Let's start in the Program class (Program.cs):
```csharp
public class Program
{
    public static int Main(string[] args)
    {
        // To make Serilog available to us immediatelly we can configurate it here first and then read from `Microsoft.Extensions.Configuration` sources later.
        // First we configure Serilog
        Log.Logger = new LoggerConfiguration()
            // Here we say that the minimum default logging level should "Debug" (which is the 2nd lowest severity level in Serilog). This included internal system events
            .MinimumLevel.Debug()
            // Next we override the default minimum logging level for the "Microsoft" namespace. "Information" is one severity level above "Debug"
            .MinimumLevel.Override("Microsoft", LogEventLevel.Information)
            // Enrich log events with properties from Serilog's LogText.
            .Enrich.FromLogContext()
            // Configure our logger to use the "Console" sink, i.e. log to console.
            .WriteTo.Console(new CompactJsonFormatter())
            // To write to "Azure Diagnostic Log Stream" we need to use the "File" sink and write logs to either "/LogFiles/" or a file with an extension of; ".txt", ".log", or ".htm".
            .WriteTo.File("/LogFiles/trace.log",
                fileSizeLimitBytes: 1_000_000,
                rollOnFileSizeLimit: true,
                shared: true,
                flushToDiskInterval: TimeSpan.FromSeconds(1))
            // Setup writing log events to Azure Application Insight. The InstrumentationKey links our telemetry data with an Application Insights resources.
            // It is also important to note that tis key should not be hardcoded or pushed to the git repo hosting the code.
            .WriteTo.ApplicationInsights(new TelemetryConfiguration { InstrumentationKey = "04efd465-67ba-4173-b3bb-a9ed1ded8f48" }, TelemetryConverter.Traces)
            .CreateLogger();

        // We add a try-catch block to catch any configuraiton or startup exceptions, which we log with 
        try
        {
            Log.Information("Starting web host");
            CreateHostBuilder(args).Build().Run();
            return 0;
        }
        catch (Exception e)
        {
            // Let's log any configuration errors as "Fatal" ("Critical" in Application Insights).
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
            // Set Serilog as the logging provider.
            .UseSerilog()
            .ConfigureWebHostDefaults(webBuilder =>
            {
                webBuilder.UseStartup<Startup>();
            });
}
```

### Startup.cs

#### Startup.ConfigureServices():

We update the **ConfigureServices** method by adding the services we need to log and send data to **Application Insights**

```csharp
public void ConfigureServices(IServiceCollection services)
{
    Log.Logger.Information("Configuring Services");
    // Register Serilog's logger as a `Singleton` for dependency injection.
    services.AddSingleton(Log.Logger);
    // Enable Application Insights telemetry collection for this app. 
    services.AddApplicationInsightsTelemetry("04efd465-67ba-4173-b3bb-a9ed1ded8f48");
    //services.AddSingleton<TelemetryClient>

    // We register an instance of our service class `CosmosDbService` as a `Singleton` for dependency injection.
    services.AddSingleton<ICosmosDbService>(InitializeCosmosClientInstanceAsync(Configuration.GetSection("CosmosDb"), Log.Logger).GetAwaiter().GetResult());
    services.AddRazorPages();
}
```

In the **Configure** method:

```csharp
// Enable Serilog's smarter HTTP request logging middleware.
// It's important that this call appears before calls to any middleware or handlers we want to enable logging for.
app.UseSerilogRequestLogging(); // <--------- Added
```



### Index.cshtml.cs:

```csharp
// Because we registered the Serilog logger service as a singleton dependency in `Startup.ConfigureServices()` it is available to us here.
public IndexModel(ILogger logger, ICosmosDbService cosmosDbService)
{
    // We set the `IndexModel` as the source type.
    _logger = logger.ForContext<IndexModel>();
    _cosmosDbService = cosmosDbService;
}

public async Task<IActionResult> OnGetAsync()
{
    _logger.Information("*** SERILOG *** IndexModel.OnGetAsync() was called.");

    await GetNotesAsync();

    return Page();
}

public async Task<IActionResult> OnPostAsync()
{
    _logger.Information("*** SERILOG *** IndexModel.OnPostAsync() was called");

    await GetNotesAsync();

    if (NewNote != null && ModelState.IsValid)
    {
        _logger.Information("*** SERILOG *** Adding new note to database.");
        NewNote.Id = Guid.NewGuid().ToString();
        NewNote.Created = DateTime.Now;
        await _cosmosDbService.AddItemAsync(NewNote);
        return RedirectToPage("index");
    }
    else
    {
        // Here it is not obvious what severity level to use, "Error" is too severe since the user is notified that they haven't completed the form.
        // "Information" might be better suited for this, but if we reach this step often the developers might want to know about it,
        // since it might indicate that the form can be improved (e.g. not enough informationfor the user of what values are required to complete the form).
        _logger.Warning("*** SERILOG *** ModelState for \"NewNote\" is invalid or \"NewNote\" is null. \"NewNote\" = {@Note}", NewNote);
        return Page();
    }
}

/// <summary>
/// Helper method to fetch all notes from the db, for use by both the GET and POST methods above.
/// </summary>
/// <returns></returns>
public async Task GetNotesAsync()
{
    if (_cosmosDbService == null)
    {
        // This should be at least an "Error" since this error (no database connection) indicates that something is seriously wrong.
        _logger.Error("*** SERILOG *** Could not find a connection to the database.");
        Notes = null;
    }
    else
    {
        _logger.Information("*** SERILOG *** Fetching all notes");

        Notes = await _cosmosDbService.GetItemsAsync("SELECT * FROM c");

        if (Notes != null)
        {
            Notes.OrderBy(n => n.Created);
        }
    }
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

### App Insights application dashboard

Let's look at an example of how logging can help us identity potential security issues.

If we take a look at the application dashboard for our Application Insight resource in Azure Portal we se a card titled "Failed requests". Clicking this card will show us all failed requests for the past 24 hours (this time range filter can be changed). We can further zoom in and look closer at individual failures, for example like on this image:

![](/Molnapplikationer-Blogg/data/images/exercise-9-monitoring-cloud-apps/azure-portal-app-insights-failures.png)

In this case we see two failures types:
1. The first failure is a 404 response code to a request. This response code was sent in response to the request `GET /robots933456.txt`, which is a "Dummy URL path" used by **Azure App Service** to check if the container is capable of serving requests <sup>[1](#source-azure-docs-robot933456)</sup>. The 404 response in this case indicates to **App Service** that the container is healthy and ready to response the requests.
2. The second and more worrisome failure is a `CryptographicException`. This exception was thrown because: "The antiforgery token could not be decrypted.". This sounds important to investigate further.


## The queries

### Query 1: Get all logs with a severity level of warning or higher.
```kusto
traces | where severityLevel >= 2 
```


## Sources & Links
- [Exercise 6 - Web Apps in the Cloud - this blog][blog-post-6]
- [Analyze your Azure infrastructure by using Azure Monitor logs - Microsoft Learn @ docs.microsoft.com][docs.microsoft.com-learn-analyze-infrastructure-with-azure-monitor-logs]
- <a name="source-azure-docs-robot933456">1</a>: [robots933456 in logs -  MicrosoftDocs/azure-docs @ github][azure-docs-app-service-robot933456.txt]

[blog-post-6]: https://johancz.github.io/Molnapplikationer-Blogg/2021/09/24/exercise-6-web-apps-in-the-cloud.html
[docs.microsoft.com-learn-analyze-infrastructure-with-azure-monitor-logs]: https://docs.microsoft.com/en-us/learn/modules/analyze-infrastructure-with-azure-monitor-logs/
[azure-docs-app-service-robot933456.txt]: https://github.com/MicrosoftDocs/azure-docs/blob/master/includes/app-service-web-configure-robots933456.md
