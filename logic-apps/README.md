# Building the demonstration

This section will give detailed instructions on building the demonstration.

## Building the infrastructure
The infrastructure comprises:
1. Cognitive Services account
2. Storage account
3. Key vault
4. 2 logic apps and their connectors

![alt text](https://github.com/jometzg/cognitive-speech/blob/master/logic-apps/Azure-resources.png "resources in the portal")

These can be built in the portal, CLI or ARM/Terraform. 

An ARM template can be found [here](https://github.com/jometzg/cognitive-speech/blob/master/templates/template.json)

## Email account
This demonstration makes use of emails as a means of receiving MP3 recordings and for sending the reply with the transcription text. It therefore needs an email account for this. You can either use an existing account or create a new one with Outlook.com or with Gmail.
Some care needs to be taken with this account as sometimes the email provider sees this automation as a sign that the email account is in some way compromised. Please check this account to see if the account provider wants further verification. If this is the case, whilst this is happening the emails will become blocked.

## Storage account shared access signature (SAS)
The Cognitive Services batch API requires a URL to the MP3 audio recording. This is in blob storage. The API cannot use the blob storage keays, so a URL needs to be constructed that allows access to the MP3 file which in this demonstration will be in the *input* blob container. There are three choices:
1. Make the container public - this is not a good idea for potentially private recordings
2. Create a SAS for the blob - this has a coding overhead for each blob
3. Create a SAS for the input container, which will be common to all blobs in the container. This was chosen

Azure Storage Explorer is the simplest means. There is now a version of this in the Azure Portal

The Azure CLI may also be used [here](https://docs.microsoft.com/en-us/cli/azure/storage/container?view=azure-cli-latest#az-storage-container-generate-sas).
```
end=`date -u -d "30 minutes" '+%Y-%m-%dT%H:%MZ'`
sas=`az storage container generate-sas -n MyContainer --account-name MyStorageAccount --https-only --permissions dlrw --expiry $end -o tsv`
az storage blob upload -n MyBlob -c MyContainer --account-name MyStorageAccount -f file.txt --sas-token $sas
```

## Key vault secrets
The ARM template builds the infrastructure, but for this demonstration to work, there must be four key vault secrets of the correct name and these need to be populated after they have been created:
1. cognitive-account - needs to contain the URL to the Cogntivce Services batch API. For West Europe accounts, this is *https://westeurope.cris.ai*
2. cognitive-key - this is the key that was created when the Cognitive account was created
3. input-sas - this is a shared access signature of the *input* container of the blob storage account. This needs to be created in the Azure CLI or using the Azure Storage Explorer
4. storage-account - this needs to be the URL to the storage account e.g. *https://storageaccount.blob.core.windows.net* and is used to construct the blob URL alongside the blob name and the SAS for the batch REST call.

Just to be clear, only 2 of these are actual secrets, the other 2 being configuration. I thought it best to put all of this configuration in a single place. For production, there may be other means, such as injecting these values in a DevOps pipeline deployment.

## Setting up the webhook
Batch transcription of MP3 files can take minutes. Instead of waiting for the reults, it is much more efficient to use webhooks.
**This demonstration requires webhooks, so the webhook needs to be registered with the batch transcription API.** This is covered [here](https://github.com/jometzg/cognitive-speech/blob/master/REST%20API/Using%20transcription%20REST%20API.md).

The webhook itself needs to point to the second logic app's HTTP trigger endpoint. This can be found by clicking on the "webhook" logic app's HTTP trigger step to reveal the URL. It should be noted that this URL is auto-generated on logic app creation and so will differ on each implementation. Below shows the URL in the logic app designer:

![alt text](https://github.com/jometzg/cognitive-speech/blob/master/logic-apps/http-trigger-endpoint.png "HTTP trigger endpoint")


## Receive email logic app

![alt text](https://github.com/jometzg/cognitive-speech/blob/master/logic-apps/email-receive-trigger-2.png "receive_email logic app flow")

This logic app is triggered by inbound emails to a specific email account. The demonstration code uses an Outlook.com account, but you can  replace this with either a Gmail account or an Office365 account. These have a different trigger action. All of these though require you to authenticate with chosen email account at design-time and this results in an "API Connection" object being created in the same resource group as the logic app. This keeps credentials separate from the logic app - and so aids a cleaner CI/CD process.

The following steps retreive secrets from a keyvault. These are:
1. The cognitive account secret for the later API call to the batch trascription API
2. A storage container shared-access secret (SAS). This is a container-level SAS for the input container of the storage account and is used to construct the URL to the MP3 file attachment that later steps put in Azure storage. The SAS may be generated either by using [Azure Storage Explorer](https://azure.microsoft.com/en-gb/features/storage-explorer/) or by using the Azure [CLI](https://docs.microsoft.com/en-us/cli/azure/storage/container?view=azure-cli-latest#az-storage-container-generate-sas). when generating the SAS, make sure that the end date us sufficiently in the future that it does not later break the demo.

The following key vault secrets are not strictly-speaking secrets, but are configuration and have been put in key vault for convenience.
1. Cognitive services account URL - used to construct the REST API call later.
2. The storage account URL - used to construct the URL to the MP3 blob

The secrets used in the key vault are shown below:
![alt text](https://github.com/jometzg/cognitive-speech/blob/master/logic-apps/key-vault-secrets.png "key vault secrets needed")

If the subject line of the email is "audio", continue processing. This is to filter out other emails arriving at that email account.

Once this stage has been reached, the real processing starts. What happens next is:
1. The MP3 file attachment gets put into the **input** container in the blob storage account - using the attachment name
2. Another blob file gets created in the **output** container which is the name of the file attachment suffixed with **.return.txt**. This file's contents will be the *from* address from the email trigger. This is used by the other logic app later, to send the reply
3. The REST API request is constructed to the batch transcription API and sent.

This logic app then completes.

The completed [code view](https://github.com/jometzg/cognitive-speech/blob/master/logic-apps/receive-email.json)

## Webhook logic app

![alt text](https://github.com/jometzg/cognitive-speech/blob/master/logic-apps/webhook-email.png "webhook logic app flow")
This logic app is called by the Cognitive services batch transcription API when a transcription has completed. For this logic app to be called, **this logic app's HTTP trigger endpoint must be registered as a webhook**. This has been covered in more detail [here](https://github.com/jometzg/cognitive-speech/blob/master/REST%20API/Using%20transcription%20REST%20API.md).

**It should be noted that the webhook may contain more than one result, but for the purposes of the demonstration we are only interested in the first result returned. So there are several places in the returned JSON that are array results. We will always be picking the zero element [0] of these arrays. For production code, this may be different**

If the status of the response message is "Succeeded", then we will process the response. Otherwise do nothing.

The response message from the webhook call does not itself contain the transcription, but contains a URL that can then be used to retrieve the trasncription. The steps are therefore:
1. Pick out of the returned JSON, the URL *['results'][0].resultUrls[0].resultUrl* that contains the actual results and then retrieve these contents with an HTTP request.
2. This HTTP request returns JSON, so JSON parse the response to make it easier to pick out the text of the transcription
3. Pick out the transcription text *['AudioFileResults'][0].CombinedResults[0].Display* and save this in the *output* container in blob storage with the name of the MP3 file suffixed by **.trans.txt**. It should be noted that we do not strictly-speaking need to store this response, but it makes easier debugging.
4. Using the name of the MP3 as a key, load the blob with the suffix **.return.txt** - this contains the email return address.
5. Send an email using the retrieved return address and the transcription contents.

The overall workflow is now complete and the sender of the MP3 should now receive and email with the transcription.

The completed [code view](https://github.com/jometzg/cognitive-speech/blob/master/logic-apps/webhook.json)

## Debugging the overall flow
It can be sometimes difficult to work out what is happing with an synchronous process and this is no different. There are several main sources of debugging what has happened:
1. Logic apps themselves have a run history and within a specific run each step may be inspected and the values it uses seen (secrets are, however, not visible). You can also "Resubmit" a logic app, which means you can update a logic app and run it with the previous input values. This will alow you to check changes without the need to either send emails or HTTP requests.

![alt text](https://github.com/jometzg/cognitive-speech/blob/master/logic-apps/run-history.png "Webhook Run history")
As can be seen above, a step may be opened and its values inspected.

2. Look in Azure storage and see what files are in the *input* and *output* containers. See below:

### Input container
![alt text](https://github.com/jometzg/cognitive-speech/blob/master/logic-apps/input-container.png "input container")


### Output container
![alt text](https://github.com/jometzg/cognitive-speech/blob/master/logic-apps/output-container.png "input container")

3. Postman to generate HTTP requests.
4. Check the email account to see what has been sent and what has been received


## Troubleshooting

