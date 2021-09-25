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

I opted to keep the app simple and thus it was sufficient to utilize the existing "Index" Razor Page.


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

Startup.cs was only modified with the addition of a method and registration of a dependency.

#### Startup.cs - InitializeCosmosClientInstanceAsync()
Mext we add a method which reads the database configuration data from the appsettings.json/appsettings.Development.json file. The databasebase configuration data includes; the URI to Cosmos DB account and its primary key, the name of the database and container we'll use. Storing secrets or sensitive data in "appsettings.json" is not recommended, but for the purpose of this assignment it will do since th GitHub repository is private and the database account will be deleted shortly after this blog-post goes live.

It then creates a `CosmosClient` instance (provided by the "Microsoft.Azure.Cosmos" libraray) using the account URI and the primary key. The instance is a "client-side" (Azure Web App) representation of the database account and which can execute requests against our database.

Next we create an instance of our `CosmosDbService` class and feed it our `CosmosClient` instance, database name and container name.
Finally we create the database and/or container if either does not already exist, and return the `CosmosdbService` instance.

#### Startup.cs - InitializeCosmosClientInstanceAsync()
We register our service class `CosmosDbService` and a `Singleton` dependency by adding the following to the `ConfigureServices` method:
```csharp
services.AddSingleton<ICosmosDbService>(InitializeCosmosClientInstanceAsync(Configuration.GetSection("CosmosDb")).GetAwaiter().GetResult());
```

This calls the `InitializeCosmosClientInstanceAsync()` method described above.

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


The constructor takes a `CosmosClient` instance, a database name and a container name and it creates a `Container` proxy which is saves to a private field.

This `Container` class has a number of built in methods for writing to and reading from a container, and we use our `Container` proxy to 



## Getting it up and running in Azure App Service

- screenshots
- scripts
- pipelines


## What would all of this cost?



## Sources & Links
- [ Microsoft.Azure.Cosmos - nuget.com][nuget-package-microsoft-azure-cosmos]


[nuget-package-microsoft-azure-cosmos]: https://www.nuget.org/packages/Microsoft.Azure.Cosmos
