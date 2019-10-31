# Installing the logic apps
The logic apps use API connections which are authorised against other services - key vault, blob storage and outlook.com. This makes it difficult to deploy the logic apps directly from an ARM template export. See [here](https://www.feval.ca/posts/logicapp-ci/).

The ARM templates have been processed and for the logic apps and their connections, the file *speechall.json* has both logic apps and their connectors.

## Deployment Sequence
### Provision the other infastructure
Either manually of from the main template deploy:
1. Create a resource group 
2. Cognitive Services Account
2. Key vault
3. Storage account

Elsewhere in these documents further configuration of these is described, such as creating the containers in storage and the secrets in key vault.

### Provision the logic apps
This can be done in the portal, but this has been shown to not return detailed enough errors. lternatively the CLI  gives much better errors than via the portal

```
az group deployment create -g "trans-rg"   --template-file speechall.json  --name "speechall"   --query 'properties.provisioningState' -o tsv
```
Where *trans-rg* is the name of the resource group. Your's may be different.

### Amend the logic app connections
The two logic apps shoould be deployed alongside three connector. These connectors will not be correct, but this can be fixed by going in turn to the two logic apps and fixing any connection errors.

Once they have their connections amended the logic apps should save correctly and can then be tested.

