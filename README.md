# Cognitive Speech
Azure cognitive services has a number of easy to use capabilities. This demonstration uses cognitive speech to transcribe audio.

A number of organisations would have uses for the automated transcription of audio. For example, financial services organisations often need to demonstrate that their selling processes are compliant with local regulations. So, the use of audio transcription can save a lot of manual effort listening to recordings of customer and advisor interactions. Sentiment analysis may also be used to flag specific recordings - allowing supervisors to concentrate on customer interactions which are different than normal.

# This Demonstration
This demonstation is a stand-alone one which uses cognitive services speech recognition with Azure serverless technologies to demonstrate audio transcription. To keep the demonstration self-contained, it uses audio MP3 file attachments to an email and then responds with the transcription in an email reply. This is not exactly how it would be done in production, but can easily be demonstrated.

![alt text](https://github.com/jometzg/cognitive-speech/blob/master/logic-apps/overview-diagram.png "Overview diagram")

More detail on the demonstration can be found here [demonstration flow](https://github.com/jometzg/cognitive-speech/blob/master/demo-flow/README.md)

This demonstration makes use of the Azure Cognitive Services Speech - specifically the batch transciption REST API. This is quite a simple set of REST APIs that allow you to do transcription and sentiment analysis. Here is a quick [sumary](https://github.com/jometzg/cognitive-speech/blob/master/REST%20API/Using%20transcription%20REST%20API.md).

# Getting Started
This demonstration requires access to an Azure subscription. If you don't have an Azure subscription, a trial one can be created here https://azure.microsoft.com/en-gb/free/

The demonstration requires a number of resources:

1. Cognitive services speech account
2. Key vault
3. A storage account for blobs
4. An email account to integrate
5. Logic apps

Detailed instructions on building the demonstration is [here](https://github.com/jometzg/cognitive-speech/tree/master/logic-apps)
