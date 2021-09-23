---
layout: post
title:  "Exercise 5 - Cloud Databases"
date:   2021-09-22 23:23
tags: Azure SQL, Cosmos DB, Azure Functions
---

<h1 style="color:red;">WIP, not complete!</h1>

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

## The application
## Beskriv kort applikationen, vad gör den?

The application is a very basic notes app. You can add a new note or fetch all notes.

## Beskriv koden
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

## Beskriv databasen
## The database

## Hur har du/ni fått den att köra i Azure functions? Screenshots, scrips, pipelines
## Getting it up and running in Azure Functions

## Hur har du tänkt runt uppdatring av databsen ifall scheman ändras? Migrations?
## Updating the database scheme

## Vad skulle det kosta att driva detta? Tänk gärna två scenarier: Nästan ingen använadere och jätte jätte mycket användere
   ### Använd Azure Calculator till att ta fram kostnad
## What would all of this cost?
   ### Small user base
   ### Massive user base


## Sources & Links
- [Microsoft.Azure.WebJobs.Extensions.CosmosDB - nuget.org/packages][link-nuget-package-azure-cosmosdb-bindings]
- [Azure Cosmos DB input binding for Azure Functions 2.x and higher - docs.microsoft.com][link-docs-microsoft-azure-functions-bindings-cosmosdb]
- [Query an Azure Cosmos container - docs.microsoft.com][link-docs-microsoft-cosmos-db-how-to-query-container]
- [DocumentQueryable.AsDocumentQuery<T>(IQueryable<T>) Method - docs.microsoft.com][link-docs-microsoft-azure-linq-document-queryable-as-document-query]
- [FeedOptions.MaxItemCount Property - docs.microsoft.com][link-docs-microsoft-azure-linq-document-client-feed-options-maxitemcount]
- [FeedOptions.EnableCrossPartitionQuery Property - docs.microsoft.com][link-docs-microsoft-azure-linq-document-client-feed-options-enablecrosspartitionquery]


[link-nuget-package-azure-cosmosdb-bindings]: https://www.nuget.org/packages/Microsoft.Azure.WebJobs.Extensions.CosmosDB
[link-docs-microsoft-azure-functions-bindings-cosmosdb]: https://docs.microsoft.com/en-us/azure/azure-functions/functions-bindings-cosmosdb-v2-input?tabs=csharp
[link-docs-microsoft-cosmos-db-how-to-query-container]: https://docs.microsoft.com/en-us/azure/cosmos-db/sql/how-to-query-container
[link-docs-microsoft-azure-linq-document-queryable-as-document-query]: https://docs.microsoft.com/en-us/dotnet/api/microsoft.azure.documents.linq.documentqueryable.asdocumentquery?view=azure-dotnet
[link-docs-microsoft-azure-linq-document-client-feed-options-maxitemcount]: https://docs.microsoft.com/en-us/dotnet/api/microsoft.azure.documents.client.feedoptions.maxitemcount?view=azure-dotnet
[link-docs-microsoft-azure-linq-document-client-feed-options-enablecrosspartitionquery]: https://docs.microsoft.com/en-us/dotnet/api/microsoft.azure.documents.client.feedoptions.enablecrosspartitionquery?view=azure-dotnet
