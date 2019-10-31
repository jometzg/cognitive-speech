# Installing the logic apps
The logic apps use API connections which are authorised against other services - key vault, blob storage and outlook.com. This makes it difficult to deploy the logic apps directly from an ARM template export. See [here](https://www.feval.ca/posts/logicapp-ci/).

The ARM templates have been processed and exported as two separate templates:
1. The API connections - *speechconnectors.json*
2. The logic apps themselves - *speech.json"
These will need to be deployed separately.

## Deployment Sequence
### Provision the other infastructure
Either manually of from the main template deploy:
1. Create a resource group 
2. Cognitive Services Account
2. Key vault
3. Storage account
Elsewhere in these documents further configuration is described.

### Provision the API Connections
This is really best done with the CLI as this gives much better errors than via the portal

```
az group deployment create -g "trans-rg"   --template-file speechconnectors.json  --name "speechconnectors"   --query 'properties.provisioningState' -o tsv
```
Where *trans-rg* is the name of the resource group. Your's can be different.

### Reauthorise each API Connection
There should be three API connection objects. These will not have valid connections.
Go into the portal and inside "Edit API connection" update the API connection for each of these for the key vault, email and storage accounts you have.

### Create the logic apps
This is near identical to creating the API connections:
```
az group deployment create -g "trans-rg"   --template-file speech.json  --name "speech"   --query 'properties.provisioningState' -o tsv
```

### Amend the logic app connections
The two logic apps shoould be deployed. The actions in the logic apps that use connections will need opening and the connections fixing. 

Once they have their connections amended the logic apps should save correctly and can then be tested.

