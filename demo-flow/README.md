# Overall flow in the demonstration

The diagram illustrates the overall flow. Each step has been marked with a number to show the overall sequence of the flow.

![alt text](https://github.com/jometzg/cognitive-speech/blob/master/demo-flow/flow-diagram.png "Flow diagram")

## Steps
1. The diagram illustrates a mobile phone with some means of recording MP3 files and shareing these via email. This has been done as one of the quickest ways in which you can record your own MP3s, but you can "borrow" any MP3 voice recordings (provided these are public domain from elsewhere and email it as an attachment to an email address you configure for the logic app. 
2. An email is sent to an email address with a subject line of "audio". This will allow the logic app not to attempt to process other email arrivals to this account. The MP3 audio file is a normal email file attachment.
3. The "receive-email" logic app is setup to trigger from an email arrival to this account. It checks the subjec line to be "audio" and picks off the file attachment
4. The file attachment is stored in Azure blob storage. This MP3 file in blob storage needs to be "visible" to the Cognitive Services batch transcription service, so must in some way be public. It is not a good idea to make all of the blobs in a blob container public, so in this demonstration a shared-access signature (SAS) is created for the blob container.
5. The logic app makes an HTTP request to Cognitive Services passing in the URL to the MP3 blob just stored. This URL is of the form __storage account/container/blob?SAS__ The call to the batch API of Cognitive services is asynchronous and so this logic app completes here.
6. The batch transrcription API has been previously setup to use a **webhook**. This webhook is called by the batch transcription service on completion of the transcription. This can be several miunutes later. The second logic app is triggered on an HTTP resquest and its endpoint is the one registered with the batch transcription API. So the second logic app will now be triggered.
7. The second logic app has a few things to do. The trigger message from batch transcription returns with the status and another URL which actually contains the transcription details. So this logic app checks the status and if "Succeeded", then finds this URL and calls this using an HTTP request. The response from this contains the result and so is parsed to get the transcription text. Finally, the logic app sends an email back to the originator with the transcription text. It should be noted that the senders email is stored in blob storage in logic app one and is then used to formulate the "To" of the email send in this logic app. Ideally, a database should be used for this, but this is simpler for the purposes of a demonstration.
8. Finally, the user receives an email with the transcription

