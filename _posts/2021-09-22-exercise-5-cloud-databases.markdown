---
layout: post
title:  "Exercise 5 - Cloud Databases"
date:   2021-09-22 23:23
tags: Azure, Cosmos DB, Azure Functions, Serverless
---

## Preface
This is the assignment for the 5th session of the course "Cloud Applications" @ Teknikhögskolan Göteborg.

The goal of this assignment is write a simple Azure Functions API which writes and reads from a cloud database.
There are three levels (only one is mandatory):
- Bronze (easy):
  A REST API in Azure Functions using an SQL Server serverless database.
- Silver (medium):
  A REST API in Azure Functions using an CosmosDB.
- Gold (advanced):
  A REST API in Azure Functions using an SQL Server serverless database.

I chose to go straight to Silver/Gold.


## The application

The application is a very basic notes app in the form of a "Azure Functions" project (created with Visual Studio).

You can add a new note or fetch all notes through a REST Api.

There are currently two endpoints:
- `POST "/api/note"`
  > Add a new note.

  ### Example of usage (using REST Client in Visual Studio Code):
  ```
  POST http://localhost:7071/api/note

  {
      "title": "My first note",
      "data": "This is the data for my first note",
      "user": "JohanCz"
  }
  ```

- `GET "/api/note"`
  > Get all existing notes.

  ### Example of usage (using REST Client in Visual Studio Code):
  ```
  GET http://localhost:7071/api/note
  ```

It would be easy to expand the API in the future, with for example an endpoint for getting a specific note using its id. This endpoint might look like `api/notes/{id}`


## The code

My Azure Functions app has two functions, one handling `POST` requests on the "api/note" endpoint, and the other handles `GET` requets on the same endpoint ("api/note").

### The `POST` "api/note" handler:

The function's name is "AddNote" and it's job is to handle `POST` requests on the "api/note" endpoint, and using the data in the request body to add a new note to the database.
The function is triggered by an HTTP request, and it uses an instance of `DocumentClient` (which is provided by the `CosmosDB` binding we use) to asynchronously add a new document using the `CreateDocumentAsync` method (but more on that later).

* To use the `CosmosDB` binding we first need to be add [the relevant NuGet package][link-nuget-package-azure-cosmosdb-bindings], which we can install by adding the following to our project file:
  ```csharp
  <PackageReference Include="Microsoft.Azure.WebJobs.Extensions.CosmosDB" Version="3.0.0-beta7" />
  ```

  The binding is then used to specify our database connection string with the `ConnectionStringSetting` property.
  ```csharp
  [CosmosDB(ConnectionStringSetting = "CosmosDbConnectionString")] DocumentClient client,
  ```
  For local development we store the connection string in `local.settings.json` in our source directory, which looks like this:

  ![](/Molnapplikationer-Blogg/data/images/exercise-5-cloud-databases/azure-functions-vs-project-local-settings-json.png)

  and in the Azure Portal we stored this string in the function's "Application settings", see the image below:

  ![](/Molnapplikationer-Blogg/data/images/exercise-5-cloud-databases/azure-portal-azure-function-config-db-connection-string.png)


* Before the new document can be created we need to extract the data from the request body, which is done with a `StreamReader` instance.
The asynchronous `StreamReader.ReadToEndAsync()` method returns a string with all of the data in the body which we then need to deserialize into a `NoteDTO` object. From this object we create a new `Note` object which can be used by `CreateDocumentAsync` like so:

  > Before we create the new document, we need to create a document collection link in the form and a `Uri`, which is used by the `CreateDocumentAsync` method to create the document in the correct collection.

  ```csharp
  Uri driverCollectionUri = UriFactory.CreateDocumentCollectionUri(databaseId: "NotesApp", collectionId: "Items");
  Document doc = await client.CreateDocumentAsync(driverCollectionUri, note);
  ```


* What we get back is the newly inserted document (our note).
The document is then serialized to `JSON` and immediately deserialized to a `NoteDTO` data transfer object (DTO). I use a DTO to control what data is returned to the user.

  > A DTO might not be strictly necessary for our purposes, but I think it's good practice to always control what is returned to the user. The `Note` class might be changed in the future, which could lead to this method returning data we don't want to expose to the user (if it was returning a `Note` object directly that is).

* Finally this DTO is returned in an `OkObjectResult` object.
  ```csharp
  noteDto = JsonConvert.DeserializeObject<NoteDTO>(JsonConvert.SerializeObject(doc));
  return new OkObjectResult(noteDto);
  ```

* Should anything at any point fail inside our the try-catch statement, a `BadRequestObjectResult` is instead returned and the exception's message is logged.
  ```csharp
  catch (Exception e)
  {
      log.LogError($"Could not add new item to collection. Exception: {e.Message}");
      return new BadRequestObjectResult(e.Message);
  }
  ```

  #### The method in its entirety:

  ```csharp
  [FunctionName("AddNote")]
  public static async Task<IActionResult> AddNote(
      [HttpTrigger(AuthorizationLevel.Function, "post", Route = "note")] HttpRequest req,
      [CosmosDB(ConnectionStringSetting = "CosmosDbConnectionString")] DocumentClient client,
      ILogger log)
  {
      log.LogInformation("C# HTTP trigger function processed a request.");

      try
      {
          string requestBody = await new StreamReader(req.Body).ReadToEndAsync();
          NoteDTO noteDto = JsonConvert.DeserializeObject<NoteDTO>(requestBody);

          Note note = new Note
          {
              Title = noteDto.Title,
              Data = noteDto.Data,
              User = noteDto.User
          };

          Uri driverCollectionUri = UriFactory.CreateDocumentCollectionUri(databaseId: "NotesApp", collectionId: "Items");
          Document doc = await client.CreateDocumentAsync(driverCollectionUri, note);

          noteDto = JsonConvert.DeserializeObject<NoteDTO>(JsonConvert.SerializeObject(doc));
          return new OkObjectResult(noteDto);
      }
      catch (Exception e)
      {
          log.LogError($"Could not add new item to collection. Exception: {e.Message}");
          return new BadRequestObjectResult(e.Message);
      }
  }
  ```

### The `GET` "api/note" handler:

This function is similar to the `POST` handler described above, but significantly simpler.

<!-- Instead of creating a document collection link we specify the database and collection directly in the `CosmosDB` binding like so:
```csharp
[CosmosDB(
    databaseName: "NotesApp",
    collectionName: "Items",
    ConnectionStringSetting = "CosmosDbConnectionString")] DocumentClient client,
``` -->

* We still use the `DocumentClient` provided by the database binding, but this time we instead use the `CreateDocumentQuery` method to insert a new document into our collection ("Items").

* We then set up some query options:
  ```csharp
    FeedOptions queryOptions = new FeedOptions { MaxItemCount = -1, EnableCrossPartitionQuery = true };
  ```
  `MaxItemCount = -1` to get all matching documents. `MaxItemCount` is used if we want to use pagination.
  `EnableCrossPartitionQuery = true` to query across all physical partitions in the database.

* Next we create the query/queryable, again creating a document collection uri, and we use the query options here.
  ```csharp
      var queryable = client.CreateDocumentQuery<Note>(UriFactory.CreateDocumentCollectionUri("NotesApp", "Items"), queryOptions).AsDocumentQuery();
  ```

  > This query gives us all document in the collection "Items", and the resulting queryable is then converted to a `IDocumentQuery` which has support for pagination and asynchronous execution (the latter being especially important when an operation can return a lot of documents).

* Finally we fetch all documents asynchronously and add each document ("Note") to a list which can be returned to the user.
  ```csharp
  List<Note> notes = new List<Note>();

  // Asynchronously fetch all documents and add them to a list.
  while (queryable.HasMoreResults)
  {
      foreach (Note note in await queryable.ExecuteNextAsync<Note>())
      {
          notes.Add(note);
      }
  }

  return new OkObjectResult(notes);
  ```

  #### The method in its entirety:
  ```csharp
  [FunctionName("GetNotes")]
  public static async Task<IActionResult> GetNotes(
      [HttpTrigger(AuthorizationLevel.Function, "get", Route = "note")] HttpRequest req,
      [CosmosDB(ConnectionStringSetting = "CosmosDbConnectionString")] DocumentClient client,
      ILogger log)
  {
      // Query options
      FeedOptions queryOptions = new FeedOptions { MaxItemCount = -1, EnableCrossPartitionQuery = true };

      // Create the query/queryable for getting all documents from the "Items" collection.
      var queryable = client.CreateDocumentQuery<Note>(UriFactory.CreateDocumentCollectionUri("NotesApp", "Items"), queryOptions).AsDocumentQuery();

      List<Note> notes = new List<Note>();

      // Asynchronously fetch all documents and add them to a list.
      while (queryable.HasMoreResults)
      {
          foreach (Note note in await queryable.ExecuteNextAsync<Note>())
          {
              notes.Add(note);
          }
      }

      return new OkObjectResult(notes);
  }
  ```


## The database

The database is a serverless CosmosDB (SQL) database which I've named `NotesApp`, and it contains one collection `Items`. Each note is stored as a document in the `Items` collection.

The code above described two functions for creating new notes and fetching all notes, and this is the collection used by these two functions.

A post request might look like this (when using the REST Client extension in VS Code):

![](/Molnapplikationer-Blogg/data/images/exercise-5-cloud-databases/vs-code-rest-client-post-request.png)

And once it has been inserted into the database it looks like this in Azure Portal:

![](/Molnapplikationer-Blogg/data/images/exercise-5-cloud-databases/azure-portal-cosmosdb-data-explorer-document.png)

> A unique `id` has been automatically generated for the document.

The database, its account and the collection was created in Azure Portal.


### What do you need to consider when updating the database in the future? (Schema changes? Migrations?)

CosmosDB is a schema-free database, and thus "Schema Migrations" are not really a thing.
CosmosDB can store unstructured or semi-structured data, so it's preferred to treat each item (document) as self-contained. All relevant data, which would typically be stored in other tables in a relational database, can instead in be embedded in the same document. A document doesn't have to contain the same properties ("columns" in a relation database) as all of the other documents in the same collection.

> Note that it is not always recommended to embed all data in a single document. Some data might be too large for embedding, such as user comments on a website. It might be better to store these a separate collection. Another example might be when the data could be used by many items.


## Getting it up and running in Azure Functions

The entire app and its functions are stored in a GitHub repository. With GitHub Actions we can deploy the app to Azure Functions.

The process of setting this up is identical to the way I set it up in my previous blog post ([Click here if you'd like to know more][link-myblog-exercise-4-deploy-to-azure-functions-with-github-actions])

Below you can see my workflow file for this project:
```yml
name: Deploy DotNet project to Azure Function App

on:
  push:
    branches: [ release ]

env:
  AZURE_FUNCTIONAPP_NAME: joch-cosmos-db-sql-functions-app
  AZURE_FUNCTIONAPP_PACKAGE_PATH: './API for Cosmos DB/'
  DOTNET_VERSION: '3.1.x'

jobs:
  build-and-deploy:
    runs-on: windows-latest
    environment: dev
    steps:
    - name: 'Checkout GitHub Action'
      uses: actions/checkout@master

    - name: Setup DotNet ${{ env.DOTNET_VERSION }} Environment
      uses: actions/setup-dotnet@v1
      with:
        dotnet-version: ${{ env.DOTNET_VERSION }}

    - name: 'Resolve Project Dependencies Using Dotnet'
      shell: pwsh
      run: |
        pushd './${{ env.AZURE_FUNCTIONAPP_PACKAGE_PATH }}'
        dotnet build --configuration Release --output ./output
        popd
    - name: 'Run Azure Functions Action'
      uses: Azure/functions-action@v1
      id: fa
      with:
        app-name: ${{ env.AZURE_FUNCTIONAPP_NAME }}
        package: '${{ env.AZURE_FUNCTIONAPP_PACKAGE_PATH }}/output'
        publish-profile: ${{ secrets.AZURE_FUNCTIONAPP_PUBLISH_PROFILE }}
```

> One small difference from the way I set it up in the [previous exercise][link-myblog-exercise-4-deploy-to-azure-functions-with-github-actions] is that I chose to add a "Release" branch to my repository and only deploy when something is pushed to that branch.


## What would all of this cost?

This one is hard since I have no knowledge of how many RU/s (Request Units / second) I could expected per month, I will take the smallest possible number of RU (Request Units) for CosmosDB and set the same number of Executions for Azure Functions (although this would probably create a bottleneck since one Azure Functions execution in all likelihood would result in more than one RU).
> The RU cost depends on many things, such as the size of the data. For example, 1 KB = 1 RU and 100 KB = 10 RUs.
> Keep in mind that the first 400,000 GB/s of execution and 1,000,000 executions are free.

I have used the [Azure Calculator][link-azure-calculator]
Since I don't really know what numbers to input I will as mentioned use the smallest possible and then roughly multiplying those numbers by 10 and 100.
This means I compare the following in the image below:
| Plan      | RUs (CosmosDB)   | Transactional Storage (CosmosDB) | Memory Size (Azure Functions) | Executions per month |
| --------- | ------------------------ | -------------------------------- | ----------------------------- | -------------------- |
| minimum   | 1,000,000                | 1 GB                             | 128                           | 1,000,000            |
| x10       | 10,000,000               | 10 GB                            | 1024                          | 10,000,000           |
| x100      | 100,000,000              | 100 GB                           | 1535                          | 100,000,000          |


These are the estimates.
![](/Molnapplikationer-Blogg/data/images/exercise-5-cloud-databases/Azure-Calculator-estimates.png)

The Azure Functions uses the "Consumption" tier and the CosmosDB operations are serverless. All plans utilize 2 periodic backup copies


   ### Small user base (smallest possible number of CosmosDB RU/s):
   ### Massive user base:


## Sources & Links
- [Microsoft.Azure.WebJobs.Extensions.CosmosDB - nuget.org/packages][link-nuget-package-azure-cosmosdb-bindings]
- [Azure Cosmos DB input binding for Azure Functions 2.x and higher - docs.microsoft.com][link-docs-microsoft-azure-functions-bindings-cosmosdb]
- [Query an Azure Cosmos container - docs.microsoft.com][link-docs-microsoft-cosmos-db-how-to-query-container]
- [DocumentQueryable.AsDocumentQuery<T>(IQueryable<T>) Method - docs.microsoft.com][link-docs-microsoft-azure-linq-document-queryable-as-document-query]
- [FeedOptions.MaxItemCount Property - docs.microsoft.com][link-docs-microsoft-azure-linq-document-client-feed-options-maxitemcount]
- [FeedOptions.EnableCrossPartitionQuery Property - docs.microsoft.com][link-docs-microsoft-azure-linq-document-client-feed-options-enablecrosspartitionquery]
- [Data modeling in Azure Cosmos DB - docs.microsoft.com][link-docs-microsoft-cosmosdb-modeling-data]
- [Deploy to Azure Functions with a GitHub Actions pipeline - Blog Post #4][link-myblog-exercise-4-deploy-to-azure-functions-with-github-actions]
- [Optimize request cost in Azure Cosmos DB - docs.microsoft.com][link-docs-microsoft-cosmosdb-optimize-cost-reads-writes]


[link-nuget-package-azure-cosmosdb-bindings]: https://www.nuget.org/packages/Microsoft.Azure.WebJobs.Extensions.CosmosDB
[link-docs-microsoft-azure-functions-bindings-cosmosdb]: https://docs.microsoft.com/en-us/azure/azure-functions/functions-bindings-cosmosdb-v2-input?tabs=csharp
[link-docs-microsoft-cosmos-db-how-to-query-container]: https://docs.microsoft.com/en-us/azure/cosmos-db/sql/how-to-query-container
[link-docs-microsoft-azure-linq-document-queryable-as-document-query]: https://docs.microsoft.com/en-us/dotnet/api/microsoft.azure.documents.linq.documentqueryable.asdocumentquery?view=azure-dotnet
[link-docs-microsoft-azure-linq-document-client-feed-options-maxitemcount]: https://docs.microsoft.com/en-us/dotnet/api/microsoft.azure.documents.client.feedoptions.maxitemcount?view=azure-dotnet
[link-docs-microsoft-azure-linq-document-client-feed-options-enablecrosspartitionquery]: https://docs.microsoft.com/en-us/dotnet/api/microsoft.azure.documents.client.feedoptions.enablecrosspartitionquery?view=azure-dotnet
[link-docs-microsoft-cosmosdb-modeling-data]: https://docs.microsoft.com/en-us/azure/cosmos-db/sql/modeling-data
[link-myblog-exercise-4-deploy-to-azure-functions-with-github-actions]: https://johancz.github.io/Molnapplikationer-Blogg/2021/09/17/exercise-4-serverless-faas-and-azure-functions.html#deploy-to-azure-functions-with-a-github-actions-pipeline
[link-docs-microsoft-cosmosdb-optimize-cost-reads-writes]: https://docs.microsoft.com/en-us/azure/cosmos-db/optimize-cost-reads-writes
[link-azure-calculator]: https://azure.microsoft.com/en-us/pricing/calculator/
