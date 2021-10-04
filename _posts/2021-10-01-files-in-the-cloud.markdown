---
layout: post
title: "Exercise 8 - Files in the Cloud"
date: 2021-10-01 23:53
tags: Azure, Azure Storage, Azure Blob Storage, Razor Pages, Github Actions
---


## Preface

### The assignment this week was to:

Create a .NET console app with C# that can upload an image to a Azure blob storage account. Once the image has been uploaded, the app should display the URL to the image in the blob storage.

The three levels to the assignment (only **bronze** or **silver** is mandatory):

**Bronze (easy):** 

Test the console app locally with a storage emulator (choose between The Microsoft Azure Storage Emulator and Azurite), before connecting the app to Azure Storage.


**Silver (medium):**

Create a Web App fetch all images in the blob storage, and displays them in the web app.

The app, when running locally, can work against the storage emulator or directly with a blob container in Azure Storage.

**Gold (advanced):**

Complete the silver part of the assignment, and then set up automatic deployment of the app to Azure with GitHub Actions.


## The app

### The "bronze" app

The app for the "bronze" part of the assignment is a .NET 5 console app which uploads all images (with the extensions; .jpg, .png, .gif) it finds the "data" directory.

It uses Microsoft's "Azure.Storage.Blobs" library/nu-get package which takes care of connecting to a blob storage, and managing its container and blobs.

The bronze app has some level of error handling, and will for example inform the user if an image (a file with the same name) already exists in the Blob Storage.

#### Screenshots of the app in action

##### Successfully uploaded images:
![](/Molnapplikationer-Blogg/data/images/exercise-8-files-in-the-cloud/bronze-app-screenshot-successfully-uploaded.png)

##### Error handling:
![](/Molnapplikationer-Blogg/data/images/exercise-8-files-in-the-cloud/bronze-app-screenshot-error-handling.png)


### The "silver" (and "gold") app

The app for the "silver" and "gold" part of the assignment is a .NET 5 web app (Razor Pages) which fetch all of the image blobs from an Azure Storage account given a connection string and container name (hardcoded).

It uses Microsoft's "Azure.Storage.Blobs" library/nu-get package which takes care of connecting to a blob storage, and managing its container and blobs.

#### Screenshots of the app in action

![](/Molnapplikationer-Blogg/data/images/exercise-8-files-in-the-cloud/silver-app-screenshot.png)


## Data flow diagram

![](/Molnapplikationer-Blogg/data/images/exercise-8-files-in-the-cloud/diagram-data-flow.png)


## The code

Please refer to the data flow diagram above while reading the code below.
Explanations are embedded with the code below.

### "Bronze" app
```csharp
// usings and namespace removed for brevity

class Program
{
    // Note: Don't do this.
    private static string _temp_connectionString = "a-connection-string-for-Azure-Storage";

    static string[] AllowedExtensions { get; set; } = new[]
    {
        ".jpg",
        ".png",
        ".gif"
    };


    static async Task Main(string[] args)
    {
        string localPath = "./data/";
        var files = GetLocalImages(localPath);

        Console.WriteLine("Image Uploader 3000 Deluxe");

        Console.WriteLine("\nFound the following images:");
        foreach (var file in files)
        {
            Console.WriteLine($"file: {file.Name}");
        }

        // Create a container
        //// Note: Adds a unique id to the container name to upload images to a new container when testing.
        //// string containerName = "imagesfromconsoleapp" + Guid.NewGuid().ToString();
        string containerName = "imagesfromconsoleapp";
        var containerClient = await GetContainer(containerName);

        // Upload the images.
        await UploadFiles(containerClient, files);

        Console.WriteLine("\nPress any key to exit.");
        Console.ReadKey();
    }

    static IEnumerable<FileInfo> GetLocalImages(string path)
    {
        // Get all files in the provided path, and return an enumerable of files that are not hidden and are images (.jpg, .png, .gif).
        var files = new DirectoryInfo(path).GetFiles();
        return files.Where(f => !f.Attributes.HasFlag(FileAttributes.Hidden) && AllowedExtensions.Contains(f.Extension.ToLower()));
    }

    static async Task<BlobContainerClient> GetContainer(string containerName)
    {
        try
        {
            // Create BlobServiceClient instance with the conneciton string.
            BlobServiceClient blobServiceClient = new(_temp_connectionString);
            
            // Get a BlobContainerClient for the given container name.
            var containerClient = blobServiceClient.GetBlobContainerClient(containerName);

            // If the is not container with the given name, create one.
            if (!await containerClient.ExistsAsync())
            {
                containerClient = await blobServiceClient.CreateBlobContainerAsync(containerName);
                Console.WriteLine($"\nA blob container with the name \"{containerName}\" was successfully created.\n");
            }
            
            return containerClient;
        }
        catch (RequestFailedException e)
        {
            Console.WriteLine(e.Message);
            return null;
        }
    }

    static async Task UploadFiles(BlobContainerClient containerClient, IEnumerable<FileInfo> files)
    {
        int filesToUploadCount = files.Count();
        int filesUploaded = 0;

        if (filesToUploadCount == 0)
        {
            Console.WriteLine("There are no files to upload.");
            return;
        }

        foreach (var file in files)
        {
            try
            {
                // Create a BlobClient for the file.
                BlobClient blobClient = containerClient.GetBlobClient(file.Name);
                // Read the file to a FileStream
                using var fileStream = File.OpenRead(file.FullName);
                // Upload the file
                await blobClient.UploadAsync(fileStream);

                filesUploaded++;
                Console.WriteLine($"- \"{file.Name}\" was successfully uploaded.");
            }
            catch (RequestFailedException e)
            {
                Console.WriteLine($"{file.Name} failed to upload.");

                switch (e.ErrorCode)
                {
                    case "BlobAlreadyExists":
                        Console.WriteLine($"A blob with that name already exists in the container.");
                        break;
                    default:
                        Console.WriteLine(e.Message);
                        break;
                }

            }
        }

        // Print out status message after upload attempts.
        if (filesUploaded == 0)
        {
            Console.WriteLine("All files failed to upload");

        }
        else if (filesUploaded == filesToUploadCount)
        {
            Console.WriteLine("All files were successfully uploaded.");
        }
        else if (filesUploaded > 0 && filesUploaded <= filesToUploadCount)
        {
            Console.WriteLine($"{filesUploaded} uploaded successfully, {filesToUploadCount - filesUploaded} failed to upload");
        }
        else
        {
            throw new Exception("Unexpected error when uploading files.");
        }
    }
}
```


### The "silver" (and "gold") app:

#### AzureBlobStorageService

This service class handles fetching of containers and blobs.

```csharp
// usings and namespace skipped for brevity.

public class AzureBlobStorageService : IAzureBlobStorageService
{
    private readonly BlobServiceClient _blobServiceClient;

    public AzureBlobStorageService(string connectionString)
    {
        // Create a BlobServiceClient instance with the provided connection string.
        _blobServiceClient = new BlobServiceClient(connectionString);
    }

    public BlobContainerClient GetContainerClient(string containerName)
    {
        // Get a BlobContainerClient for a container with the given name.
        return _blobServiceClient.GetBlobContainerClient(containerName);
    }

    public async Task<List<BlobItem>> GetBlobsFromContainer(BlobContainerClient containerClient)
    {
        if (containerClient == null)
        {
            // todo: error handling
            return null;
        }

        List<BlobItem> blobs = new();

        try
        {
            await foreach (var blob in containerClient.GetBlobsAsync())
            {
                // Skip deleted blobs.
                if (blob.Deleted == true)
                {
                    continue;
                }

                blobs.Add(blob);
            }
        }
        catch (RequestFailedException e)
        {
            // todo: handle exception
        }

        return blobs;
    }

    public async Task<IEnumerable<string>> GetBlobUris(string containerName)
    {
        // Get a BlobContainerClient instance for the container with the given name.
        var containerClient = GetContainerClient(containerName);

        // If a BlobContainerClient cannot be created, return an empty enumerable
        if (containerClient == null)
        {
            return Enumerable.Empty<string>();
        }

        // Get all blobs from container.
        var blobItems = await GetBlobsFromContainer(containerClient);

        // Return the AbsoluteURI for all found blobs.
        return blobItems.Select(b => containerClient.GetBlobClient(b.Name).Uri.AbsoluteUri);
    }
}
```

#### Startup.cs

In **Startup.cs** if register my service class as a singleton dependency, and add a method `InitializeServices()` (more on this method below).

```csharp
public void ConfigureServices(IServiceCollection services)
{
    // Register our `AzureBlobStorageService` service class as a singleton.
    services.AddSingleton<IAzureBlobStorageService>(InitializeServices(Configuration.GetSection("AzureBlobStorage")));
    services.AddRazorPages();
}
```

##### Startup.InitializeServices()

This method reads the **Azure Storage "ConnectionString"** and with the connection string and creates an instance of `AzureBlobStorageService` for use with dependency injection.

```csharp
/// <summary>
/// Reads the config for the connection string and creates an AzureBlobService with it.
/// </summary>
/// <returns></returns>
private static AzureBlobStorageService InitializeServices(IConfigurationSection configurationSection)
{
    string connectionString = configurationSection.GetSection("ConnectionString").Value;

    return new AzureBlobStorageService(connectionString);
}
```

The connection string is located **appsettings.Development.json** (this file was added to `.gitignore`) while running locally and in the **Azure Web App**'s configuration store when deployed (see screenshot below).

![](/Molnapplikationer-Blogg/data/images/exercise-8-files-in-the-cloud/azure-portal-screenshot-app-config.png)



#### IndexModel

The model for the index page (Razor Page).

```csharp
// usings and namespace skipped for brevity.

public class IndexModel : PageModel
{
    private readonly ILogger<IndexModel> _logger;
    private readonly IAzureBlobStorageService _azureBlobStorageService;

    public IEnumerable<string> BlobUris{ get; set; }


    public IndexModel(ILogger<IndexModel> logger, IAzureBlobStorageService azureBlobStorageService)
    {
        _logger = logger;
        _azureBlobStorageService = azureBlobStorageService;
    }


    public async Task<IActionResult> OnGet()
    {
        // Get the URI of all blobs in the given container.
        BlobUris = await _azureBlobStorageService.GetBlobUris("imagesfromconsoleapp");

        return Page();
    }
}
```

#### Index.cshtml

The model for the index page (Razor Page).

```csharp
@page
@model IndexModel
@{
    ViewData["Title"] = "Image Viewer 3000 Deluxe";
}

<div class="text-left">
    <h1 class="display-4">Images from my Azure Blob Storage</h1>

    <div class="thumbnails">
        @foreach (var blobUri in Model.BlobUris)
        {
            <img src="@blobUri" />
        }
    </div>
</div>
```


## Cost of running a web app (App Service) which uses Azure Storage

### Web App (App Service)

![](/Molnapplikationer-Blogg/data/images/exercise-8-files-in-the-cloud/azure-pricing-calculator-estimate-monthly-app-service.png)


### Azure Storage

#### How did I get the numbers when I estimated the costs for Azure Storage?

With `1,000` users who each upload `100 MB` every day, this equals `0.1 TB` of data uploaded each month.

If we assume that each user has `1` **container** we'll have a total of `1,000` **containers**.

And let's say that the average file size is `2 MB`, this means every user uploads `50` images every days, and all users together upload `50,000` images.

If each upload equals `1` **write operation** this means roughly `1,500,000` **write operations** per month.

To upload blobs to a container you need to check so that the container exists, which means (at least) `1` operation, which I'll count under **"All other operations"** (in **Azure's Pricing Calculator**).

If a user uploads one image per session this means `50` operations per user per day, and `1500` operations per user per month. With `1,000` users that means `1,5000,000` operations.

And for **List and create operations** we'll take the same number and multiply it by `3` (since every image is downloaded 3 times every day), this means `4,500,000` **List and create operations**.

The same number is used for **Read operations**.

The only properties that change each month are **Capacity**, **List and Create Container Operations** and **Read operations**.


#### These are my estimates:

##### For the first month (**Azure App Service** & **Azure Storage**):

![](/Molnapplikationer-Blogg/data/images/exercise-8-files-in-the-cloud/azure-pricing-calculator-azure-storage-estimate-first-month.png)


##### For the first six months (only **Azure Storage**):

![](/Molnapplikationer-Blogg/data/images/exercise-8-files-in-the-cloud/azure-pricing-calculator-estimate-storage-only-first-6-months.png)


## How does Microsoft ensure that your Blob data is secure?

- To access Blob data you need to use an access key (if the Blob storage isn't set to anonymous access).
- Data is also encrypted, both **"at rest"** (meaning while stored on Azure's servers) and **"in flight"** with https (http with TLS) (data being sent to and from Azure's services and data centers).



## How did I get the app up and running in Azure (Gold part of assignment)?

To get it up and running I followed the same process as in assignment 4, [which I covered in an earlier blog post][myblog-assignment-4-github-actions-deploy]. 
There are two differences however:
1. I'm using an Azure Web App (App Service) instead of an Azure Function.
1. There are a few differences in my workflow file.

The workflow file for this project looks like this:

```yaml
name: Build Docker image, push to GHCR and publish to azure app app

# Trigger the workflow whenever something is pushed to the 'release' branch
on:
  push:
    branches: [ release ]

jobs:
  # The jobs which builds and pushes our images to two repositories, Github's container repository (GHCR) and Docker Hub.
  build-and-push-docker-image-and-publish-to-azure-web-app:
    runs-on: ubuntu-latest
    env:
      # Set the default working directory for all "run" steps on this workflow.
      working-directory: ./exercise-silver/

    steps:
    - uses: actions/checkout@v2

    # Login to GitHub Container Registry (GHCR) (with the login action by Docker).
    - name: Login to GitHub Container Registry
      uses: docker/login-action@v1.10.0
      with:
        # Specify the registry we want to push to.
        registry: ghcr.io
        # Set authentication information:
        # Set username, the username in this case is my Github username (specified with "github.actor").
        username: ${{ github.actor }}
        # And the password is set with "secrets.GITHUB_TOKEN" (which is a token automatically provided by Github which can be used on this (and only this repository).
        password: ${{ secrets.GITHUB_TOKEN }}  

    # The build and push task makes use of Docker's build-push action. This task builds an image and pushes it to GHCR.
    - name: Build and push
      id: docker_build
      # Here I specify to use Docker's build-push action, 
      uses: docker/build-push-action@v2.7.0
      with:
        push: true
        # Set the build context.
        context: ${{env.working-directory}}
        # Specify the registries we want to push our image to and set a "version tag" (e.g. "latest" or "9").
        # The format is: "registry/namespace/repository:version" (in my case the namespace on both registries is my username "johancz")
        tags: |
          ghcr.io/johancz/molnapplikationer-assignment-8:latest
          ghcr.io/johancz/molnapplikationer-assignment-8:${{ github.run_number }}
          
    - uses: azure/webapps-deploy@v2
      with:
        app-name: 'joch-assignmnent-8-images-app'
        publish-profile: ${{ secrets.AZURE_WEBAPP_PUBLISH_PROFILE }}
        images: 'ghcr.io/johancz/molnapplikationer-assignment-8:latest'
```



## Sources & Links
- [Average file size statistics - user:pilau - superuser.com][superuser-powershell-script-average-image-size]
- [Assignment 4 Serverless, FaaS and Azure Functions - This blog][myblog-assignment-4-github-actions-deploy]
- [How Does Azure Encrypt Data?](https://cloudacademy.com/blog/how-does-azure-encrypt-data/)

[superuser-powershell-script-average-image-size]: https://superuser.com/a/643542
[myblog-assignment-4-github-actions-deploy]: https://johancz.github.io/Molnapplikationer-Blogg/2021/09/17/exercise-4-serverless-faas-and-azure-functions#deploy-to-azure-functions-with-a-github-actions-pipeline
