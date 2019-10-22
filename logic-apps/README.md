# Building the demonstration

This section will give detailed instructions on building the demonstration.

## Building the infrastructure
The infrastructure comprises:
1. Cognitive Services account
2. Storage account
3. Key vault
4. 2 logic apps and their connectors

These can be built in the portal, CLI or ARM/Terraform.

## Receive email logic app

![alt text](https://github.com/jometzg/cognitive-speech/blob/master/logic-apps/email-receive-trigger-2.png "receive_email logic app flow")

The completed [code view](https://github.com/jometzg/cognitive-speech/blob/master/logic-apps/receive-email.json)

## webhook logic app

![alt text](https://github.com/jometzg/cognitive-speech/blob/master/logic-apps/webhook-email.png "webhook logic app flow")

The completed [code view](https://github.com/jometzg/cognitive-speech/blob/master/logic-apps/webhook.json)
