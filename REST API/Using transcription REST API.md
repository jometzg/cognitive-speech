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
### Request
```
GET /api/speechtotext/v2.1/transcriptions/<id> HTTP/1.1
Host: <region>.cris.ai
Content-Type: application/json
Ocp-Apim-Subscription-Key: XXXXXXX-key-XXXXXXXXXX

```
### Response
```json
{
  "results": [
    {
      "recordingsUrl": "https://jjspeechsa.blob.core.windows.net/input/Recording 091535-073119.mp3?XXX-SAS-XXXX",
      "resultUrls": [
        {
          "fileName": "transcription_0",
          "resultUrl": "https://spsvcprodweu.blob.core.windows.net/bestor-acc02701-cb45-42dc-9d32-2843911017ca/TranscriptionData/b2d494b1-e142-4943-9a20-84a7a52ce8db.json?sv=2017-04-17&sr=b&sig=07P%2FSvqwQOv7dup43fVIuDXLsijhW8gYM%2ByFmvowskg%3D&st=2019-10-21T09:32:13Z&se=2019-10-24T09:37:13Z&sp=rl"
        }
      ]
    }
  ],
  "datasets": [],
  "models": [
    {
      "modelKind": "AcousticAndLanguage",
      "datasets": [],
      "id": "6a69a05c-7753-4d78-a698-79429d946e6a",
      "createdDateTime": "2019-10-17T03:00:00Z",
      "lastActionDateTime": "2019-10-17T03:30:00Z",
      "status": "Succeeded",
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
      "id": "13413008-c050-4af0-bd13-92af6464bb7e",
      "createdDateTime": "2019-04-25T12:35:20Z",
      "lastActionDateTime": "2019-04-25T12:36:52Z",
      "status": "Succeeded",
      "locale": "en-US",
      "name": "sentiment-en-us-bert-v0-0",
      "description": "Sentiment Models for en-US",
      "properties": {
        "Purpose": "BatchTranscription,BatchSentimentAnalysis"
      }
    }
  ],
  "reportFileUrl": "https://spsvcprodweu.blob.core.windows.net/bestor-acc02701-cb45-42dc-9d32-2843911017ca/TranscriptionData/860997ed-947f-4ed2-8d6d-1b312ba0d24b.txt?sv=2017-04-17&sr=b&sig=14edOt8Zdrd6NoFxZBEhPqpAOp2kRp6yMjimp5nluvM%3D&st=2019-10-21T09:32:13Z&se=2019-10-24T09:37:13Z&sp=rl",
  "statusMessage": "None.",
  "id": "dedffd77-218f-4052-b07a-4a4061530fba",
  "createdDateTime": "2019-10-18T15:48:03Z",
  "lastActionDateTime": "2019-10-18T15:48:38Z",
  "status": "Succeeded",
  "locale": "en-US",
  "name": "shortone",
  "description": "An optional description of the transcription.",
  "properties": {
    "ProfanityFilterMode": "Masked",
    "PunctuationMode": "DictatedAndAutomatic",
    "AddSentiment": "True",
    "CustomPronunciation": "False",
    "Duration": "00:00:24"
  }
}
```
Note that the response from the trasncription request does not itself contain the text of the transcription, but the __resultUrl__ field points to a URL where the response from the transcription can be found. This is another storage area (that belongs to the Speech Service) and if you open that URL you will get the response data, a sample of which is shown below:
```json
{
  "AudioFileResults": [
    {
      "AudioFileName": "transcription_0",
      "AudioFileUrl": "https://jjspeechsa.blob.core.windows.net/input/Recording%20091535-073119.mp3?XXX-SAS-XXXX",
      "AudioLengthInSeconds": 24.43,
      "CombinedResults": [
        {
          "ChannelNumber": "0",
          "Lexical": "ten democratic presidential hopefuls have clashed in a televised US debate that laid bare the parties deep divisions over how best to in twenty twenty elizabeth warren and bernie sanders the most liberal candidates in the crowd crowded field came under attack from their more moderate colleagues three other women and seven men on stage in michigan debated healthcare border policy and how to defeat donald trump",
          "ITN": "10 Democratic presidential hopefuls have clashed in a televised US debate that laid bare the parties deep divisions over how best to in 2020 Elizabeth Warren and Bernie Sanders the most liberal candidates in the crowd crowded field came under attack from their more moderate colleagues Three other women and 7 men on stage in Michigan debated healthcare border policy and how to defeat Donald Trump",
          "MaskedITN": "10 Democratic presidential hopefuls have clashed in a televised US debate that laid bare the parties deep divisions over how best to in 2020 Elizabeth Warren and Bernie Sanders the most liberal candidates in the crowd crowded field came under attack from their more moderate colleagues Three other women and 7 men on stage in Michigan debated healthcare border policy and how to defeat Donald Trump",
          "Display": "10 Democratic presidential hopefuls have clashed in a televised US debate that laid bare the parties deep divisions over how best to in 2020. Elizabeth Warren and Bernie Sanders, the most liberal candidates in the crowd crowded field came under attack from their more moderate colleagues. Three other women and 7 men on stage in Michigan, debated healthcare border policy and how to defeat Donald Trump."
        }
      ],
      "SegmentResults": [
        {
          "RecognitionStatus": "Success",
          "ChannelNumber": "0",
          "SpeakerId": null,
          "Offset": 16800000,
          "Duration": 219200000,
          "OffsetInSeconds": 1.68,
          "DurationInSeconds": 21.92,
          "NBest": [
            {
              "Confidence": 0.898624659,
              "Lexical": "ten democratic presidential hopefuls have clashed in a televised US debate that laid bare the parties deep divisions over how best to in twenty twenty elizabeth warren and bernie sanders the most liberal candidates in the crowd crowded field came under attack from their more moderate colleagues three other women and seven men on stage in michigan debated healthcare border policy and how to defeat donald trump",
              "ITN": "10 Democratic presidential hopefuls have clashed in a televised US debate that laid bare the parties deep divisions over how best to in 2020 Elizabeth Warren and Bernie Sanders the most liberal candidates in the crowd crowded field came under attack from their more moderate colleagues Three other women and 7 men on stage in Michigan debated healthcare border policy and how to defeat Donald Trump",
              "MaskedITN": "10 Democratic presidential hopefuls have clashed in a televised US debate that laid bare the parties deep divisions over how best to in 2020 Elizabeth Warren and Bernie Sanders the most liberal candidates in the crowd crowded field came under attack from their more moderate colleagues Three other women and 7 men on stage in Michigan debated healthcare border policy and how to defeat Donald Trump",
              "Display": "10 Democratic presidential hopefuls have clashed in a televised US debate that laid bare the parties deep divisions over how best to in 2020. Elizabeth Warren and Bernie Sanders, the most liberal candidates in the crowd crowded field came under attack from their more moderate colleagues. Three other women and 7 men on stage in Michigan, debated healthcare border policy and how to defeat Donald Trump.",
              "Sentiment": {
                "Negative": 0.150532,
                "Neutral": 0.849462,
                "Positive": 0.0
              },
              "Words": null
            }
          ]
        }
      ]
    }
  ]
}
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
