# Introduction
There are a number of ways of interacting with Azure Cognitive Services speech services. The batch API was chosen because:

1. This is a REST interface so does not rely in an SDK
2. The REST API can take MP3 files directly
3. You can take advantage of webhook callbacks. This is especially important in serverless implemetations as you don't want long-running serverless calls.

For more details on this see [here](https://docs.microsoft.com/en-us/azure/cognitive-services/speech-service/batch-transcription)

## Calling the REST API
### Request
```
POST /api/speechtotext/v2.1/transcriptions/ HTTP/1.1
Host: <region>.cris.ai
Content-Type: application/json
Ocp-Apim-Subscription-Key: XXXXXXX-key-XXXXXXXXXX
User-Agent: PostmanRuntime/7.17.1
Accept: */*
Cache-Control: no-cache
Host: westeurope.cris.ai
Accept-Encoding: gzip, deflate
Content-Length: 545
Connection: keep-alive
cache-control: no-cache

{
  "description": "<optional description of the transcription>",
  "locale": "en-US",
  "models": [],
  "name": "<name>",
  "properties": {
    "AddSentiment": "True",
    "AddWordLevelTimestamps": "True",
    "ProfanityFilterMode": "Masked",
    "PunctuationMode": "DictatedAndAutomatic"
  },
  "recordingsUrls": ["<url to MP3 file (in blob storage)>"]
}
```
### Response
```
202 Accepted
```
In the above request, exptracted from Postman, you only need to update:
1. the region of the Cognitive services account (this is "westeurope" for West Europe.
2. For the header "Ocp-Apim-Subscription-Key", this needs the Cognitive services account key
3. A useful name for the request. One idea is the filename of hte MP3 file of the audio recording
4. A publically available URL of the MP3 file. Blob storage is a really good location and this demo makes use of that. A public blob container could be used, but it's not the best practice. Blobs and containers can also have [shared access signatures (SAS)](https://docs.microsoft.com/en-us/azure/storage/common/storage-sas-overview). A good compromise betweeen security and convenience is to create a SAS on the container. This SAS can then be kept as a secret and re-used for each blob as needed.
