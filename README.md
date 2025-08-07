# How to create Azure Storage Account and Grant Permissions

## 1. Create a New Console C# application in VSCode

Open VSCode and run this command to create a new C# console application with **.NET 10**:

dotnet new console --framework net10.0

<img width="1287" height="578" alt="image" src="https://github.com/user-attachments/assets/29d89951-e8df-491e-9328-a4301829c9fc" />

## 2. Login in Azure with VSCode Terminal window

In the VSCode Terminal window run the command

```
az login
```

It is better way to set in the login the subscription number

We get the subscritiponId from Azure Portal

<img width="1919" height="493" alt="image" src="https://github.com/user-attachments/assets/0e458a87-f75c-40a9-bd24-fd271f7b0b82" />

We run this command to set the SubscriptionId:

<img width="876" height="29" alt="image" src="https://github.com/user-attachments/assets/45f07262-6cec-4dd9-a51a-734d90b92873" />

## 3. We load the libraries

We add this libraries:

```
dotnet add package Azure.Identity
dotnet add package Azure.ResourceManager
dotnet add package Azure.ResourceManager.Storage
```

We review the csproj file:

```csproj
<Project Sdk="Microsoft.NET.Sdk">

  <PropertyGroup>
    <OutputType>Exe</OutputType>
    <TargetFramework>net10.0</TargetFramework>
    <ImplicitUsings>enable</ImplicitUsings>
    <Nullable>enable</Nullable>
  </PropertyGroup>

  <ItemGroup>
    <PackageReference Include="Azure.Identity" Version="1.14.2" />
    <PackageReference Include="Azure.ResourceManager" Version="1.13.2" />
    <PackageReference Include="Azure.ResourceManager.Storage" Version="1.4.4" />
  </ItemGroup>

</Project>
```

We run the command:

```
dotnet restore
```

## 4. We input the source code

```csharp
using System;
using System.Threading.Tasks;
using Azure;
using Azure.Identity;
using Azure.Core;
using Azure.ResourceManager;
using Azure.ResourceManager.Resources;
using Azure.ResourceManager.Storage;
using Azure.ResourceManager.Storage.Models;

class Program
{
    static async Task Main(string[] args)
    {
        string subscriptionId = "e5bd93f3-xxxxxxxxxxxxxxxxxxxxxxxxxx";
        string resourceGroupName = "myluiscocoRG";
        string location = "westeurope";
        string storageAccountName = "luisstor987"; // must be globally unique
        string containerName = "newblob";

        // Authenticate using DefaultAzureCredential (make sure you're logged in with `az login`)
        var credential = new AzureCliCredential();
        var armClient = new ArmClient(credential, subscriptionId);

        // Get the subscription and resource group
        SubscriptionResource subscription = await armClient.GetDefaultSubscriptionAsync();
        ResourceGroupResource rg;

        // Create resource group if it doesn't exist
        var rgLro = await subscription.GetResourceGroups().CreateOrUpdateAsync(
            WaitUntil.Completed,
            resourceGroupName,
            new ResourceGroupData(location));

        rg = rgLro.Value;

        // Prepare storage account config
        var storageParams = new StorageAccountCreateOrUpdateContent(
            new StorageSku(StorageSkuName.StandardLrs),
            StorageKind.StorageV2,
            new AzureLocation(location))
        {
            AccessTier = StorageAccountAccessTier.Hot
        };

        // Create the storage account
        var storageLro = await rg.GetStorageAccounts()
            .CreateOrUpdateAsync(WaitUntil.Completed, storageAccountName, storageParams);

        var storageAccount = storageLro.Value;
        Console.WriteLine($"✅ Storage account '{storageAccount.Data.Name}' created.");

        // Get the Blob Service resource
        BlobServiceResource blobService = await storageAccount.GetBlobService().GetAsync();

        // Create the container
        var containerLro = await blobService.GetBlobContainers()
            .CreateOrUpdateAsync(WaitUntil.Completed, containerName, new BlobContainerData());

        Console.WriteLine($"✅ Blob container '{containerName}' created.");
    }
}
```

## 5. We run the application

We execute the application with the following command:

```
dotnet run
```

<img width="1296" height="777" alt="image" src="https://github.com/user-attachments/assets/541d338a-0588-4b96-9cac-7f645b85d260" />

## 6. We confirm in Azure Portal

We review the ResourceGroups

<img width="1652" height="492" alt="image" src="https://github.com/user-attachments/assets/23a07a7d-bfed-4f0a-a334-7a545ed3831c" />

We review the StorageAccounts

<img width="1721" height="491" alt="image" src="https://github.com/user-attachments/assets/dfd19c63-7e86-41bc-8292-ff467f70b2f2" />




