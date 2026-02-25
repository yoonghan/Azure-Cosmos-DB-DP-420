# Hands-on Labs Exercises
Practice via the following lab links from [Microsoft Learning](https://github.com/MicrosoftLearning/dp-420-cosmos-db-dev)

**Notes on Lab**
1. It might cost you money to run the lab. Becareful to clean up the resources after you finish the lab.
2. Always use free tier and make sure you have only 1 Cosmos DB account, not 2. Keep checking DB Account.
3. Set a billing budget in Azure Portal via Cost Management -> Budgets. You can set alert of 0.1 USD if you're paranoid.
4. To view your charges to Cost Management -> Cost analysis to see what is charged after labs.
5. Becareful of 16 and 17 Denormalize. It creates a database that opts out free tier and creates a charge of at least USD20 a day. Each execution creates a new DB account with random name.


## Lab settings to know

1. Assign Data roles
```
az cosmosdb sql role assignment create --resource-group "<RESOURCE_GROUP_NAME>" --account-name "<COSMOS_DB_ACCOUNT_NAME>" --role-definition-id "00000000-0000-0000-0000-000000000002" --principal-id $(az ad signed-in-user show --query id -o tsv) --scope "/"
```

2. Create the database named `cosmicworks` manually.
You must create the database manually "cosmicworks", there are no options for you to skip this only via Az CLI/ARM/Bicep/Terraform. There is no permission available to create DB.

3. (Doesn't work) You need to assign DB Operator custom role required to insert/update/delete container. However this does not work in SDK as SDK is a Data Plane SDK. Use CLI/master key/ARM/Bicep/Terraform instead to create/update a container. Below will not work even assigned.
```
az cosmosdb sql role definition create \
    --account-name "<COSMOS_DB_ACCOUNT_NAME>" \
    --resource-group "<RESOURCE_GROUP_NAME>" \
    --body '{
        "RoleName": "DatabaseCreator",
        "Type": "CustomRole",
        "AssignableScopes": ["/"],
        "Permissions": [{
            "DataActions": [
                "Microsoft.DocumentDB/databaseAccounts/readMetadata",
                "Microsoft.DocumentDB/databaseAccounts/sqlDatabases/containers/*",
                "Microsoft.DocumentDB/databaseAccounts/sqlDatabases/containers/items/*"
            ]
        }]
    }
# You can also find the id in the output, kind of long with /subscriptions<SUBSCRIPTION>/resourceGroups/<RESOURCE_GROUP_NAME>/providers/Microsoft.DocumentDB/databaseAccounts/<COSMOS_DB_ACCOUNT_NAME>/sqlRoleDefinitions/xx-xxxx-xx-xx-xxxxxx
``` 

```
az cosmosdb sql role assignment create --resource-group "<RESOURCE_GROUP_NAME>" --account-name "<COSMOS_DB_ACCOUNT_NAME>" --role-definition-id "$( \
az cosmosdb sql role definition list \
    --account-name "<COSMOS_DB_ACCOUNT_NAME>" \
    --resource-group "<RESOURCE_GROUP_NAME>" \
    --query "[?roleName=='DatabaseCreator'].id" \
    --output tsv \
)" --principal-id $(az ad signed-in-user show --query id -o tsv) --scope "/"
```

4. Enable auth, default is true but this allows the use of master key.
```
az resource update --resource-type "Microsoft.DocumentDB/databaseAccounts" --resource-group dp-420 --name cosmicworks --set properties.disableLocalAuth=false 
```

