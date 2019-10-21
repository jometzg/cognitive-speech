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
In the above request, extracted from Postman, you only need to update:
1. the region of the Cognitive services account (this is "westeurope" for West Europe).
2. For the header "Ocp-Apim-Subscription-Key", this needs the Cognitive Services account key which you can find in the portal for that account.
3. A useful name for the request. One idea is the filename of hte MP3 file of the audio recording
4. A publically available URL of the MP3 file. Blob storage is a really good location and this demo makes use of that. A public blob container could be used, but it's not the best practice. Blobs and containers can also have [shared access signatures (SAS)](https://docs.microsoft.com/en-us/azure/storage/common/storage-sas-overview). A good compromise betweeen security and convenience is to create a SAS on the container. This SAS can then be kept as a secret and re-used for each blob as needed.

## Getting the result for the transcription

There are a couple of REST calls that allow you to get transcription results. 

## List of all transcriptions
### Request
```
GET /api/speechtotext/v2.1/transcriptions/ HTTP/1.1
Host: <region>.cris.ai
Content-Type: application/json
Ocp-Apim-Subscription-Key: XXXXXXX-key-XXXXXXXXXX
```
### Response
```json
[
  {
    "results": [
      {
        "recordingsUrl": "<url to MP3 file (in blob storage)>",
        "resultUrls": [
          {
            "fileName": "transcription_0",
            "resultUrl": "**<URL of a blob that contains the actual results of the transcription>**"
          }
        ]
      }
    ],
    "datasets": [],
    "models": [
      {
        "modelKind": "AcousticAndLanguage",
        "datasets": [],
        "id": "**<result id>**",
        "createdDateTime": "2019-10-17T03:00:00Z",
        "lastActionDateTime": "2019-10-17T03:30:00Z",
        "status": "**Succeeded**",
        "locale": "en-US",
        "name": "20190520 (v4.14 Unified)",
        "description": "Unified",
        "properties": {
          "Purpose": "OnlineTranscription,BatchTranscription,LanguageAdaptation,LanguageOnlineInterpolation",
          "ModelClass": "unified-v4"
        }
      },
      {
        "modelKind": "Sentiment",
        "datasets": [],
        "id": "**<id>**",
        "createdDateTime": "2019-04-25T12:35:20Z",
        "lastActionDateTime": "2019-04-25T12:36:52Z",
        "status": "**Succeeded**",
        "locale": "en-US",
        "name": "sentiment-en-us-bert-v0-0",
        "description": "Sentiment Models for en-US",
        "properties": {
          "Purpose": "BatchTranscription,BatchSentimentAnalysis"
        }
      }
    ],
    "reportFileUrl": "**<url of sentiment analysis>**",
    "statusMessage": "None.",
    "id": "**<id>**",
    "createdDateTime": "2019-10-21T14:57:06Z",
    "lastActionDateTime": "2019-10-21T14:57:59Z",
    "status": "Succeeded",
    "locale": "en-US",
    "name": "__Recording 121236-051719.mp3__",
    "description": "<optional description of the transcription>",
    "properties": {
      "AddSentiment": "True",
      "AddWordLevelTimestamps": "True",
      "ProfanityFilterMode": "Masked",
      "PunctuationMode": "DictatedAndAutomatic",
      "CustomPronunciation": "False",
      "Duration": "00:00:53"
    }
  }
]
```
## Specific Transcription
If you have the transaction ID already, then you can request for this one specifically

```
GET /api/speechtotext/v2.1/transcriptions/<id> HTTP/1.1
Host: <region>.cris.ai
Content-Type: application/json
Ocp-Apim-Subscription-Key: XXXXXXX-key-XXXXXXXXXX

```

# Webhooks
Webhooks allow you to register an endpoint to receive transcription results once they are ready. A *webhook* is merely an HTTP endpoint that you register to receive results. This is web endpoint that responds to HTTP *POST* requests.

Azure web apps, functions and logic apps can recieve webhooks.

## Register a webhook
### Request
```
POST /api/speechtotext/v2.1/transcriptions/hooks HTTP/1.1
Host: <region>.cris.ai
Content-Type: application/json
Ocp-Apim-Subscription-Key: XXXXXXX-key-XXXXXXXXXX

{
  "configuration": {
    "url": "< URL of webhook. For a logic app, this can be copied from the HTTP trigger step >",
    "secret": "<my_secret>"
  },
  "events": [
    "TranscriptionCompletion"
  ],
  "active": true,
  "name": "TranscriptionCompletionWebHook",
  "description": "This is a Webhook created to trigger an HTTP POST request when my audio file transcription is completed.",
  "properties": {
      "Active" : "True"
  }

}

}
```
### Response
