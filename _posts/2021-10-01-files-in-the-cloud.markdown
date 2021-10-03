---
layout: post
title: "Exercise 8 - Files in the Cloud"
date: 2021-10-01 23:53
tags: Azure, Azure Storage, Azure Blob Storage, Razor Pages, Github Actions
---

<h1 style="color:red;">WIP, not complete!</h1>


## Preface

Gör en .NET-konsol applikation med C# som kan ladda upp ett bild till en Azure blob storage. När man har laddat upp bilden ska applikationen visa URLen till bilden i bloben.

**Brons (enkel):** Konsol applikation

Testa lokalt med en storage emulator (välj själv emellam The Microsoft Azure Storage Emulator och Azurite), innan ni ansluttar mot Azure.


**Silver (meddel):** Skåpa en web app som läser läser vilka bilder som finns i eran blob, läsa deras URL och använn denna till att vises bilderna i en HTML sida.

Lokalt kan webb applikationen jobba mot eran storage emulator eller mot en container i Azure.


**Guld (avancerat):** Bygg som i silver en web app som läser av era filer i blob storage. Lägg till automatisk deploy med GitHub actions till Azure.




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


## Cost of running an app which uses Azore Storage

With 1,000 users who each upload 100 MB every day, this equals 0.1 TB of data uploaded each month.

And let's say that the average file size is 2 MB, this means every user uploads `50` images every days, and all users combined upload `50,000` images.

If each upload equals **1 write operation** this means 100,000 write operations per month.

This is my estimate for the first month:

![](/Molnapplikationer-Blogg/data/images/exercise-8-files-in-the-cloud/azure-pricing-calculator-estimate-first-month.png)


The estimates for the first six months:

![](/Molnapplikationer-Blogg/data/images/exercise-8-files-in-the-cloud/azure-pricing-calculator-estimate-storage-only-first-6-months.png)


## How does Microsoft ensure that your Blob data is secure?



## How did I get the app up and running in Azure (Gold part of assignment)?

To get the average file size of an image I took an average of all images (.png. jpg and .gif) in the **Picures** directory on my PC with the following powershell script:
```powershell
$items = Get-ChildItem -path "C:\Users\username\Pictures" -recurse -include *png,*jpg,*gif
$count = ($items | measure-object | select -expand Count)
$size =(($items | measure-object -property length -sum).sum /1MB)
$avrg = $size / $count
$avrg
1.30598176799048
```


## Sources & Links
- [Average file size statistics - user:pilau - superuser.com][superuser-powershell-script-average-image-size]


[superuser-powershell-script-average-image-size]: https://superuser.com/a/643542
