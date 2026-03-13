# Hands-on Labs Exercises
Practice via the following lab links from [Microsoft Learning](https://github.com/MicrosoftLearning/dp-420-cosmos-db-dev)

**Notes on Lab**
  1. It might cost you money to run the lab. Becareful to clean up the resources after you finish the lab.
  2. Always use free tier and make sure you have only 1 Cosmos DB account, not 2. Keep checking DB Account.
  3. Set a billing budget in Azure Portal via Cost Management -> Budgets. You can set alert of 0.1 USD if you're paranoid.
  4. To view your charges to Cost Management -> Cost analysis to see what is charged after labs.
  5. Becareful of 16 (Measure) and 17 (Denormalize). It creates a database that opts out free tier and creates a charge of at least USD20 a day. Each execution creates a new DB account with random name.


## Lab settings to know

1. Assign Data roles
```
az cosmosdb sql role assignment create --resource-group "<RESOURCE_GROUP_NAME>" --account-name "<COSMOS_DB_ACCOUNT_NAME>" --role-definition-id "00000000-0000-0000-0000-000000000002" --principal-id $(az ad signed-in-user show --query id -o tsv) --scope "/"
```

2. Create the database named `cosmicworks` manually if master/primary/secondary key are not used.
You must create the database manually "cosmicworks", there are no options for you to skip this only via Az CLI/ARM/Bicep/Terraform. There is no permission available to create DB.

3. The SDK is only for dataplane control (meaning only for data plane not control plane) for RBAC, using master key is fine. It is only possible via CLI/ARM/Bicep/Terraform to create db or container. To change via code you need ARM library
```
using System;
using System.Threading.Tasks;
using Azure.Core;
using Azure.Identity;
using Azure.ResourceManager;
using Azure.ResourceManager.CosmosDB;
using Azure.ResourceManager.CosmosDB.Models;

// 1. Authenticate using Entra ID (RBAC)
var credential = new DefaultAzureCredential();
var armClient = new ArmClient(credential);

string resourceGroupName = "dp-420";
string accountName = "costly-db";

// 2. Get a reference to your Cosmos DB Account via the Control Plane
var subscription = await armClient.GetDefaultSubscriptionAsync();
var resourceGroup = await subscription.GetResourceGroupAsync(resourceGroupName);
var cosmosAccount = await resourceGroup.Value.GetCosmosDBAccountAsync(accountName);

// 3. Create the Database using the Management SDK
string databaseName = "cosmicworks";
var databaseData = new CosmosDBSqlDatabaseCreateOrUpdateContent(
    new AzureLocation("eastus"), 
    new ExtendedCosmosDBSqlDatabaseResourceInfo(databaseName));

var databaseResource = await cosmosAccount.Value.GetCosmosDBSqlDatabases().CreateOrUpdateAsync(Azure.WaitUntil.Completed, databaseName, databaseData);
Console.WriteLine($"Database {databaseName} created/verified.");

// 4. Create the Container using the Management SDK
string containerName = "products";
var containerData = new CosmosDBSqlContainerCreateOrUpdateContent(
    new AzureLocation("eastus"), 
    new ExtendedCosmosDBSqlContainerResourceInfo(containerName)
    {
        PartitionKey = new CosmosDBContainerPartitionKey()
        {
            Paths = { "/categoryId" },
            Kind = CosmosDBPartitionKind.Hash
        }
    })
{
    Options = new CosmosDBCreateUpdateConfig { Throughput = 400 }
};

var containerResource = await databaseResource.Value.GetCosmosDBSqlContainers().CreateOrUpdateAsync(Azure.WaitUntil.Completed, containerName, containerData);
Console.WriteLine($"Container {containerName} created/verified.");
```

4. Enable auth, default is false, incase it fails it means a master key is not allowed.
```
az resource update --resource-type "Microsoft.DocumentDB/databaseAccounts" --resource-group dp-420 --name cosmicworks --set properties.disableLocalAuth=false 
```

