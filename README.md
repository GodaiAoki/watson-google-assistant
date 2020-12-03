[![Build Status](https://travis-ci.com/IBM/watson-google-assistant.svg?branch=master)](https://travis-ci.com/IBM/watson-google-assistant)

# Create an Action for Google Assistant using Watson Assistant for conversational actions

This code pattern includes a Watson Assistant workspace to demonstrate an implementation of a retail agent that can ask for reservation schedules and specifics. To demonstrate how to test it with Google Assistant devices, we will setup a Google Action that calls out to our Node.js server which interacts with Watson Assistant.

![architecture](doc/source/images/architecture.png)

## Flow

1. User talks or types to Google Assistant.
2. Google Assistant posts text to an HTTPS endpoint.
3. IBM Cloud functions calls Watson Assistant to get the response.
4. The response is returned to Google Assistant.
5. Google Assistant replies to the user.

# Steps

## Run locally

1. [Clone the repo](#1-clone-the-repo)
1. [Create a Watson Assistant skill](#2-create-a-watson-assistant-skill)
1. [Configure credentials](#3-configure-credentials)
1. [Create the OpenWhisk action](#4-create-the-openwhisk-action)
1. [Create an Conversational Action](#5-create-conversational-action)
1. [Talk to it](#6-talk-to-it)

### 1. Clone the repo

### 2. Create a Watson Assistant skill

Sign up for [IBM Cloud](https://cloud.ibm.com/registration/) if you don't have an IBM Cloud account yet.


#### Using the provided rent_a_car.json file

Create the service by following this link and hitting `Create`:

* [**Watson Assistant**](https://cloud.ibm.com/catalog/services/watson-assistant)

Import the Assistant rent_a_car.json:

* Find the Assistant service in your IBM Cloud Dashboard.
* Click on the service and then click on `Launch Watson Assistant`.
* Go to the `Skills` tab.
* Click `Create skill`
* Click the `Import skill` tab.
* Click `Choose JSON file`, go to your cloned repo dir, and `Open` the rent_a_car.json file in [`data/assistant/rent_a_car.json`](data/assistant/rent_a_car.json).
* Click `Import`.

To find the `SKILL_ID` for Watson Assistant:

* Go back to the `Skills` tab.
* Find the card for the workspace you would like to use. Look for `rent-a-car`, if you uploaded rent_a_car.json. The name will vary if you used BAE.
* Click on the three dots in the upper right-hand corner of the card and select `View API Details`.
* Copy the `Skill ID` GUID. Save it for the .params file in [Step 5](#5-configure-credentials).

![view_api_details](doc/source/images/view_api_details.png)

### 3. Configure credentials

The default runtime parameters need to be set for the action.
These can be set on the command line or via the IBM Cloud UI.
Here we've provided a params.sample file for you to copy and use
with the `-param-file .params` option (which is used in the instructions below).

Copy the [`params.sample`](params.sample) to `.params`.

```bash
cp params.sample .params
```

Edit the `.params` file and add the required settings as described below.

#### `params.sample:`

```json
{
  "ASSISTANT_APIKEY": "<add_assistant_apikey>",
  "ASSISTANT_URL": "<add_assistant_url>",
  "SKILL_ID": "<add_assistant_skill_id>",
}
```

#### Finding the credentials

The credentials for IBM Cloud services can be found in the IBM Cloud UI.

* Go to your IBM Cloud Dashboard.
* Find your Assistant service in the `Services` list.
* Click on the service name.
* `Manage` should be selected in the sidebar.
* Use the copy icons to copy the `API Key` and `URL` and paste them into your .params file.
* For `SKILL_ID`, use the Skill ID for Watson Assistant from [Step 2](#2-create-a-watson-assistant-skill).

### 4. Create the OpenWhisk action

As a prerequisite, [install the Cloud Functions (IBM Cloud OpenWhisk) CLI](https://cloud.ibm.com/docs/openwhisk?topic=cloud-functions-cli_install)

#### Create the OpenWhisk action

Run these commands to gather Node.js requirements, zip the source files, and upload the zipped files
to create a raw HTTP web action in OpenWhisk.

> Note: You can use the same commands to update the action if you modify the code or the .params.

```sh
npm install
rm action.zip
zip -r action.zip main.js package* node_modules
ibmcloud wsk action update google-watson action.zip --kind nodejs:default --web raw --param-file .params
```

#### Determine your IBM Cloud endpoint

To find this URL, navigate to [IBM Cloud Functions - Actions](https://cloud.ibm.com/openwhisk/actions), click on your
`google-watson` action and use the sidebar to navigate to `Endpoints`.

![functions_endpoints](doc/source/images/functions_endpoints.png)

### 5. Create an Converastional Action

help is here.
https://developers.google.com/assistant/conversational/build/projects

Go to https://console.actions.google.com and click the `New Project` button.

Provide a name, Choose a language for your action `English`, hit the `create project` button.

Select the **Custom** template and hit the `Next` button.

Select the **Blank Project** template and hit the `start building` button.

Provide an invocation name

Register Webhook and set for No Match Intents:

* In the left sidebar menu, click on `Webhook`, and choose `HTTPS Endpoint`.

* Enter Endpoint of google-watson, and `Save`

* In the left sidebar menu, click on `Main invocation`

* Select `call your Webhook` and type event name.

* In the left sidebar menu, click on `Intents` and select `NO_MATCH`

* Select `call your Webhook` and type event name.

### 6. Talk to it

Use the `Test` tab in the Actions console.


# Sample output

Here is a sample conversation flow using the provided Watson Assistant rent_a_car.json:

![sample_conversation](doc/source/images/sample_conversation.png)

# Troubleshooting

* Want to see debug logging

  > Use the IBM Cloud UI to monitor logs, or use this CLI command to show the latest activation log:

  ```bash
  ibmcloud wsk activation list -l1 | tail -n1 | cut -d ' ' -f1 | xargs ibmcloud wsk activation logs
  ```

* Testing invoke from CLI

  > Use these commands to invoke the action (named alexa-watson in the example) without any input, then check the latest logs. **Expect an error ("Must be called from Alexa")**.

  ```bash
  ibmcloud wsk action invoke alexa-watson -bvd
  ibmcloud wsk activation list -l1 | tail -n1 | cut -d ' ' -f1 | xargs ibmcloud wsk activation logs
  ```

# License

This code pattern is licensed under the Apache License, Version 2. Separate third-party code objects invoked within this code pattern are licensed by their respective providers pursuant to their own separate licenses. Contributions are subject to the [Developer Certificate of Origin, Version 1.1](https://developercertificate.org/) and the [Apache License, Version 2](https://www.apache.org/licenses/LICENSE-2.0.txt).

[Apache License FAQ](https://www.apache.org/foundation/license-faq.html#WhatDoesItMEAN)
