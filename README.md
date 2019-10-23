# Cognitive Speech
Azure cognitive services has a number of easy to use capabilities. This demonstration uses cognitive speech to transcribe audio.

A number of organisations would have uses for the automated transcription of audio. For example, financial services organisations often need to demonstrate that their selling processes are compliant with local regulations. So, the use of audio transcription can save a lot of manual effort listening to recordings of customer and advisor interactions. Sentiment analysis may also be used to flag specific recordings - allowing supervisors to concentrate on customer interactions which are different than normal.

# This Demonstration
This demonstation is a stand-alone one which uses cognitive services speech recognition with Azure serverless technologies to demonstrate audio transcription. To keep the demonstration self-contained, it uses audio MP3 file attachments to an email and then responds with the transcription in an email reply. This is not exactly how it would be done in production, but can easily be demonstrated.

![alt text](https://github.com/jometzg/cognitive-speech/blob/master/logic-apps/overview-diagram.png "Overview diagram")

More detail on the demonstration can be found here [demonstration flow](https://github.com/jometzg/cognitive-speech/blob/master/demo-flow/README.md)

This demonstration makes use of the Azure Cognitive Services Speech Service - specifically the batch transcription REST API. This is quite a simple set of REST APIs that allow you to do audio transcription and sentiment analysis. Here is a quick [summary](https://github.com/jometzg/cognitive-speech/blob/master/REST%20API/Using%20transcription%20REST%20API.md).

# Getting Started
This demonstration requires access to an Azure subscription. If you don't have an Azure subscription, a trial one can be created here https://azure.microsoft.com/en-gb/free/

The demonstration requires a number of resources:

1. Cognitive services speech account
2. Key vault
3. A storage account for blobs
4. An email account to integrate
5. Logic apps

Detailed instructions on building the demonstration is [here](https://github.com/jometzg/cognitive-speech/tree/master/logic-apps)

# Extending this demonstration
This demonstration is pretty basic in terms of what it does and how it works. Other areas where this may be extended could be:
1. Send a request received aknowledgement receipt to the origniator - useful for long transcriptions which could take some minutes
2. Use the sentiment analysis capability that is built into the batch API. For long transcriptions, the results are broken up into speech segments (roughly analogous to paragraphs) and sentiment can be returned for each of these segments. The demonstration could be extended to build a spreadsheet or graph of the sentiment across the timeline of the transcription. A practical use of this could be sentiment for customer interviews, so you can see at a glance where the sentiment changes across a number of minutes of an interview.
3. Handling problems. Right now this demonstration only looks at the positive path. It may be useful to send response emails in the case of where something has gone wrong.
4. Other integrations besides email. 
