---
layout: post
title:  "Exercise 6 - Web Apps in the Cloud"
date:   2021-09-24 22:39
tags: Web Apps, Razor Pages, ASP.NET, Azure App Service
---

<h1 style="color:red;">WIP, not complete!</h1>


## Preface
This is the assignment for the 6th session of the course "Cloud Applications" @ Teknikhögskolan Göteborg.

The goal of this assignment is write a simple Razor Pages Web App that can write to and read from a cloud database (the same database used in the previous assignment).
There are four "levels" to the assignment (only one is mandatory):
- Bronze (easy):
  Build a Web App with Razor Pages and publish it to Azure App Services with Visual Studio or Visual Studio Code.

- Silver (medium - recommended):

  Build a Web App with Razor Pages, add a Dockerfile so it can be build an image, push the image to ACR (Azure Container Registry). And finally configure a new Azure Web App Service to fetch and run the image from ACR.

- Gold (advanced):
  Build a Web App with Razor Pages, add a Dockerfile so it can be build an image, push the code to a GitHub repository and configure a GitHub Action to build an image and push it to GitHub Packages --(GitHub Container Registry)--. to ACR (Azure Container Registry). And finally configure a new Azure Web App Service to fetch and run the image from Github Packages.

- Platina (very advanced):
  Build the Web App as a static web application, which fetches data through Azure Functions. It is recommended to use React as the frontend framework.

I opted to start with Silver and finally move on to Gold. Platina might happen at some point.


## The application

The application is a very basic notes app in the form of a "ASP.NET Core Web App" project (created with Visual Studio) utilizing Razor Pages. The App is then hosted as an "Azure Web App".

The app allows the user to add a new note through a basic html form and it will fetch all notes from the database. It uses the [Microsoft.Azure.Cosmos][nuget-package-microsoft-azure-cosmos] library to write to and read from the database.

The database is an "Azure Cosmos DB (Core) SQL" database which has one container called "NoteItems".


## The code

### Services/CosmosDbService and Services/ICosmosDbService
First I created a class which contains all necessary logic for connecting to and using a Cosmos DB database. This class is later registered in "Startup.cs" as a `Singleton` and can be injected into wherever it is required via dependency injection. The service class implements an interface `ICosmosDbService` and looks like this (please see the comments in the code below for any explanations):

```csharp
public class CosmosDbService : ICosmosDbService
{
    private Container _container;

    // The constructor takes a CosmosClient, a database name and a container name. It then creates a `Container` instance (this class is provided by the "Microsoft.Azure.Cosmos" library) and saves it to a private field.
    // The `Container` class has methods for reading, inserting, deleting and updating a CosmosDb container.
    public CosmosDbService(
        CosmosClient dbClient,
        string databaseName,
        string containerName)
    {
        this._container = dbClient.GetContainer(databaseName, containerName);
    }

    // Takes a `Note` instance and asynchronously adds it to the "NotesItem" container.
    public async Task AddItemAsync(Note note)
    {
        // The partition key is the id of the document in the "NotesItem" container.
        await this._container.CreateItemAsync<Note>(note, new PartitionKey(note.Id));
    }

    // Fetch all documents in the "NoteItems" container. The methods takes an sql query string, e.g. "SELECT * FROM c" which will get us all document from the database.container specified in the constructor of this class.
    public async Task<IEnumerable<Note>> GetItemsAsync(string queryString)
    {
        // The `Container.GetItemQueryIterator()` method takes a query string in this case and returns an iterator.
        var query = this._container.GetItemQueryIterator<Note>(new QueryDefinition(queryString));
        List<Note> results = new List<Note>();
        // As long as there are more items, asynchronously get the next set of results from the query and add it to the List<Note> declared above.
        while (query.HasMoreResults)
        {
            var response = await query.ReadNextAsync();

            results.AddRange(response.ToList());
        }

        return results;
    }
}
```

### Startup.cs

Startup.cs was only modified with the addition of a method `InitializeCosmosClientInstanceAsync()` and registration of a dependency.

#### Startup.cs - InitializeCosmosClientInstanceAsync()
```csharp
private static async Task<CosmosDbService> InitializeCosmosClientInstanceAsync(IConfigurationSection configurationSection)
{
    string databaseName = configurationSection.GetSection("DatabaseName").Value;
    string containerName = configurationSection.GetSection("ContainerName").Value;
    string account = configurationSection.GetSection("Account").Value;
    string key = configurationSection.GetSection("Key").Value;

    CosmosClient client = new CosmosClient(account, key);
    CosmosDbService cosmosDbService = new CosmosDbService(client, databaseName, containerName);
    DatabaseResponse database = await client.CreateDatabaseIfNotExistsAsync(databaseName);
    await database.Database.CreateContainerIfNotExistsAsync(containerName, "/id");

    return cosmosDbService;
}
```
This method reads the database configuration data from the appsettings.json/appsettings.Development.json file. The databasebase configuration data includes; the URI to Cosmos DB account and its primary key, the name of the database and container we'll use. Storing secrets or sensitive data in "appsettings.json" is not recommended, but for the purpose of this assignment it will do since th GitHub repository is private and the database account will be deleted shortly after this blog-post goes live.

It then creates a `CosmosClient` instance (provided by the "Microsoft.Azure.Cosmos" libraray) using the account URI and the primary key. The instance is a "client-side" (Azure Web App) representation of the database account and which can execute requests against our database.

Next we create an instance of our `CosmosDbService` class and feed it our `CosmosClient` instance, database name and container name.
Finally we create the database and/or container if either does not already exist, and return the `CosmosdbService` instance.

#### Startup.cs - InitializeCosmosClientInstanceAsync()
We register am instance of our service class `CosmosDbService` as a `Singleton` dependency by adding the following to the `ConfigureServices` method (This calls the `InitializeCosmosClientInstanceAsync()` method described above):
```csharp
services.AddSingleton<ICosmosDbService>(InitializeCosmosClientInstanceAsync(Configuration.GetSection("CosmosDb")).GetAwaiter().GetResult());
```

### The "Note" Model class
```csharp
public class Note
{
    [JsonProperty(PropertyName = "id")]
    public string Id { get; set; }

    [JsonProperty(PropertyName = "title")]
    [Display(Name = "Note Title")]
    public string Title { get; set; }

    [JsonProperty(PropertyName = "data")]
    [Display(Name = "Note Contents")]
    public string Data { get; set; }

    [JsonProperty(PropertyName = "created")]
    [Display(Name = "Created")]
    public DateTime Created { get; set; }
}
```
### Index.cshtml and Index.cshtml.cs

I opted to keep the app simple and thus it was sufficient to reuse the existing "Index" Razor Page to; present a form for adding a new note, and to render an HTML table for displaying all of the notes fetched from the database.

```cshtml
@page
@model IndexModel
@{
    ViewData["Title"] = "Notes";
}

<h2>New Note</h2>

<form method="post">
    <div class="form-group">
        @Html.LabelFor(m => m.NewNote.Title):
        <br />
        @Html.TextBoxFor(m => m.NewNote.Title)

        <br />

        @Html.LabelFor(m => m.NewNote.Data):
        <br />
        @Html.TextAreaFor(m => m.NewNote.Data)
    </div>

    <div class="form-group">
        <button type="submit" class="btn btn-primary">Create</button>
    </div>
</form>

<h2>All Notes</h2>

<table class="table">
    <tr>
        <th>
            @Html.DisplayNameFor(m => m.Notes.ElementAt(0).Title)
        </th>
        <th>
            @Html.DisplayNameFor(m => m.Notes.ElementAt(0).Data)
        </th>
        <th>
            @Html.DisplayNameFor(m => m.Notes.ElementAt(0).Created)
        </th>
        <th></th>
    </tr>

    @foreach (var item in Model.Notes)
    {
        <tr>
            <td>
                @Html.DisplayFor(m => item.Title)
            </td>
            <td>
                @Html.DisplayFor(m => item.Data)
            </td>
            <td>
                @Html.DisplayFor(m => item.Created)
            </td>
        </tr>
    }

</table>
```

This uses the html tag helper `@Html.LabelFor()` to render labels with the name of the properties of the `Note` class (or "Display Names" in our case since we use the `Display` attribute (e.g. `[Display(Name = "Note Title")]`)). The empty `input` and `textarea` elements are rendered with the tag helpers `TextBoxFor` and `TextAreaFor` respectively.

We then loop through all `Notes` fetched with the help of our `CosmosDbService` class and render each note in a loop with the `@Html.DisplayNameFor` and `@Html.DisplayFor` tag helpers.

```csharp
public class IndexModel : PageModel
{
    private readonly ILogger<IndexModel> _logger;
    private readonly ICosmosDbService _cosmosDbService;

    public IEnumerable<Note> Notes { get; set; }

    [BindProperty]
    public Note NewNote { get; set; }

    // Inject the `CosmosDbService` instance for use by this page model.
    public IndexModel(ILogger<IndexModel> logger, ICosmosDbService cosmosDbService)
    {
        _logger = logger;
        _cosmosDbService = cosmosDbService;
    }

    public async Task<ActionResult> OnGetAsync()
    {
        // Fetch all `Notes` from the database and store them in an IEnumerable<Note>
        Notes = await _cosmosDbService.GetItemsAsync("SELECT * FROM c");
        // Render the page.
        return Page();
    }

    public async Task<RedirectToPageResult> OnPostAsync()
    {
        // Since the page's form automatically gets bound to `NewNote` when the [BindProperty] attribute is used we immediately have access to the data posted from the form.

        // No validations was setup, but this was left for future work
        if (ModelState.IsValid)
        {
            // Create a unique id for the new `Note`.
            NewNote.Id = Guid.NewGuid().ToString();
            // Set the date & time for when the `note` was created.
            NewNote.Created = DateTime.Now;
            // Add the new `Note` as a document to the `NoteItems` container.
            await _cosmosDbService.AddItemAsync(NewNote);
        }

        // Effectively reloads the page we're on so that the new note is shown in the list below the form.
        return RedirectToPage("index");
    }
}
```

In the `IndexModel` we inject the `CosmosDbService` singleton dependency which is then used by the `OnGetAsync` and `OnPostAsync` methods for fetching all notes and posting a new one.


## Getting it up and running in Azure App Service

<!-- First of all here are the Cosmos DB settings I used  -->

### For the "Silver" part of this assignment
1. To get my Web App up and running in Azure App Service I first created an Azure Container Registry with these settings:
    ![](/Molnapplikationer-Blogg/data/images/exercise-6-web-apps/azure-portal-container-registry-settings.png)

1. With my ACR (Azure Container Registry) up and running, and my app ready to go live I turned to the command prompt in Windows to push my image to the registry:
    1. I began my signing into Azure with the Azure CLI:
        ```bash
        az login
        ```
        This opened an Azure sign-in page in my default browser.
    1. After signing into Azure I could login to my Azure Container Registry "jochStudentCR":
        ```bash
        az acr login --name jochStudentCR
        ```
    1. I then built my image and tagged it:
        ```bash
        docker build . -t azurewebapp
        docker tag azurewebapp jochstudentcr.azurecr.io/azurewebapp:v1
        ```
    1. I could then my image to my ACR:
        ```bash
        docker push jochstudentcr.azurecr.io/azurewebapp:v1
        ```
    1. The final step I took was to enable "admin user" on my ACR to enable me to deploy my image from my ACR to Azure App Services:
        ```bash
        az acr update -n jochStudentCR --admin-enabled true
        ```

1. I could then create an Azure Web App in the Azure Portal with these settings (on the "Basics" tab):
    ![](/Molnapplikationer-Blogg/data/images/exercise-6-web-apps/azure-portal-create-resources-create-web-app-details.png)

    And these settings (on the "Docker" tab, to set up the Web App to pull an image from the my ACR):
    ![](/Molnapplikationer-Blogg/data/images/exercise-6-web-apps/azure-portal-create-resources-create-web-app-details-docker-silver-acr.png)



## What would all of this cost?

This are my cost estimate for...

### a small user base:
> With 
![](/Molnapplikationer-Blogg/data/images/exercise-6-web-apps/azure-cost-estimates-few-users.png)

### a very large user base:
> App Service is on the `Premium v3` tier with three instances of `P3V3: 8 cores, 32 GB RAM 250 GB Storage` and utilizing the 55% discount (3 year reserved instances).
> Azure Cosmos DB (Serverless) is setup for `100,000,000 RUs (Request Units)` and with `100 GB` of transactional storage.
![](/Molnapplikationer-Blogg/data/images/exercise-6-web-apps/azure-cost-estimates-many-users.png)

### Further details on choices:
- `Azure Cosmos DB` uses the serverless model in both user base cases. This is used accordingly with spec of the assignment, but in a real world scenario another model such as `Autoscaled Provisioned Throughout` should probably be used for the large use base case.
- `North Europe` is the selected location wherever it is possible to set.
- `Linux` is the `App Service` OS in both cases.
- Both estimates have`Azure Container Regisry -> Bandwidth -> Outbound Data Transfer` set to 5 GB.
- `Azure Container Regisry -> Bandwidth -> Extra Storage` is set to 1 GB and 5 GB for the small and large user bases respectively.

### Summary
```
                 | Upfront Cost | Month Cost
Small user base: |        $0.00 |     $18.67
Large user base: |        $0.00 |    $765.24
```


## Sources & Links
- [Microsoft.Azure.Cosmos - nuget.com][nuget-package-microsoft-azure-cosmos]
- [Pushing Docker images to ACR - Azure Container Registry - blog.hildenco.com][link-blog.hildenco-pushing-docker-images-to-azure]
- [Azure Web App for Containers - youtube.com (Microsoft Student Accelerator)][link-youtube-microsoft-student-accelerator-Azure-Web-App-for-Containers]


[nuget-package-microsoft-azure-cosmos]: https://www.nuget.org/packages/Microsoft.Azure.Cosmos
[link-blog.hildenco-pushing-docker-images-to-azure]: https://blog.hildenco.com/2020/10/pushing-docker-images-to-azure.html
[link-youtube-microsoft-student-accelerator-Azure-Web-App-for-Containers]: https://www.youtube.com/watch?v=xnUOu-yPEzo
