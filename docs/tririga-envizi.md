# Tririga - Envizi integration

## Table of contents

<!--ts-->
  * [Overview](#overview)
  * [Tririga Configuration](#tririga-configuration)
  * [AppConnect Configuration](#appconnect-configuration)
    * [Adding Accounts](#adding-accounts)
    * [Importing the flow](#importing-the-flow)
    * [Configuring the flow to use the right accounts](#configuring-the-flow-to-use-the-right-accounts)
    * [Configuring the Scheduler](#configuring-the-scheduler)
    * [Using the flows](#using-the-flows)
      * [On-demand Flow](#on-demand-flow)
      * [Always-On Flow](#always-on-flow)
      * [Configuring the Flow Parameters](#configuring-the-flow-parameters)
    * [Starting and Stopping the flow](#starting-and-stopping-the-flow)
  * [Post-Setup Instructions](#post-setup-instructions)
<!--te-->

## Overview
- Flows included in this integration:
  - [TririgaBuildings_Always_On](/AppConnect%20Flows/TririgaBuildings_Always_On.yaml)
  - [TririgaBuildings_On_Demand](/AppConnect%20Flows/TririgaBuildings_On_Demand.yaml)

![Tririga-Envizi integration](https://media.github.ibm.com/user/375131/files/1a642100-01ed-11ed-8151-31e573db3664)


## Tririga Configuration

Before AppConnect flow is configured, OM package needs to be installed on the Tririga instance.

Follow the documentation [here](./tririga-configuration.md) for detailed instructions.


## AppConnect Configuration

Note: IBM Cloud AppConnect Professional or Enterprise is needed to run this flow.

Note: The names in the screenshots are generic, the elements in this integration will not have the same names during setup.


### Adding Accounts

Before importing the flow to AppConnect, add Accounts for SFTP and HTTP connectors. While adding the HTTP connector account, include credentials for the Tririga user which can consume the OSLC API.

- Navigate to Catalog section of the AppConnect instance
<img width="960" alt="Create Account 1" src="https://media.github.ibm.com/user/375131/files/eb8b5a00-eb1d-11ec-8401-35b47d561ce4">

- In the "Search application" field, type name of the connector.

- If the AppConnect instance does not have an account for the connector, click on "Connect" to create a new account
<img width="960" alt="Create Account 2a" src="https://media.github.ibm.com/user/375131/files/ec23f080-eb1d-11ec-9f62-af1addec3fd3">

- Else, open the account selection drop down, and click on "Add a new account ..."
<img width="948" alt="Create Account 2b" src="https://media.github.ibm.com/user/375131/files/ecbc8700-eb1d-11ec-9693-4ce83ffda2e6">

- Enter the necessary details for the connector
    - For SFTP, it will be the SFTP server and user account.
    - For HTTP, it will be the Authentication Key or username and password needed for Tririga.
<img width="960" alt="Create Account 3" src="https://media.github.ibm.com/user/375131/files/ed551d80-eb1d-11ec-9894-350e87fbb2b2">

- Click on Connect
<img width="960" alt="Create Account 4" src="https://media.github.ibm.com/user/375131/files/ed551d80-eb1d-11ec-983c-c276d43e0654">

- From the account selection drop down, select the newly created account. e.g., The default name will be `Account 2` if `Account 1` is already present
- Click on the kebab menu (three dots) and select "Rename Account"
<img width="960" alt="Create Account 5" src="https://media.github.ibm.com/user/375131/files/ededb400-eb1d-11ec-9a29-fd9fa65dbf8b">

- Enter an account name and click on "Rename Account". This name can now be used by the connector in the flow.
<img width="958" alt="Create Account 6" src="https://media.github.ibm.com/user/375131/files/ee864a80-eb1d-11ec-96d1-fdc42c679896">

### Importing the flow

- Open the Dashboard of the AppConnect instance
- Click on the "New" button and select "Import Flow"
<img width="861" alt="Flow Import 0" src="https://media.github.ibm.com/user/375131/files/ea582e00-eb19-11ec-9912-0881d321f3c7">

- Browse to the flow's YAML file and click on "Import"
<img width="960" alt="Flow Import 1" src="https://media.github.ibm.com/user/375131/files/ea582e00-eb19-11ec-88a7-abebff4e0534">

- The flow will now be imported and opened.
<img width="960" alt="Flow Import 2" src="https://media.github.ibm.com/user/375131/files/eaf0c480-eb19-11ec-885b-919da812499b">

### Configuring the flow to use the right accounts

When importing a flow, it is important to check if the flow is using the right accounts for the different connectors.

- Click on "Edit Flow"

- See if the connectors are using the right accounts.

- To change the account for any connector, select the connector and click on the dropdown icon next to the Account's name
<img width="960" alt="Select Account 1" src="https://media.github.ibm.com/user/375131/files/11186380-eb1e-11ec-9c92-5f892df61a57">

- Select the account name to use from the list
<img width="659" alt="Select Account 2" src="https://media.github.ibm.com/user/375131/files/11b0fa00-eb1e-11ec-88fd-8e4998f6304a">

### Configuring the Scheduler

- Click on the "Scheduler" node

<img width="659" alt="Scheduler node" src="https://media.github.ibm.com/user/375131/files/ea694d80-01ed-11ed-8852-16b625339675">

- Configure the schedule as needed

<img width="659" alt="Scheduler configuration 1" src="https://media.github.ibm.com/user/375131/files/d70ab200-01ee-11ed-9af7-4f66738aea7c">
<img width="659" alt="Scheduler configuration 2" src="https://media.github.ibm.com/user/375131/files/00c3d900-01ef-11ed-86aa-29008ff86450">

### Using the flows

This integration comes with two types of flows:

#### On-demand Flow

- This flow is for the initial sync or to sync data that was added/updated between specific dates.
- This flow is meant to be executed just once whenever needed and then stopped.
- To use this flow, the following parameters in "Set variable" node need to be configured:
  - Variable > config > OverrideFromDate:
    - The start timestamp of the window between which the data will be pulled from.
    - The date must be specified in ISO 8601 format.
    - e.g., `2022-06-26T23:09:30-07:00` where `-07:00` represents timezone offset of UTC-07:00 hours
  - Variable > config > OverrideToDate:
    - The end timestamp of the window between which the data will be pulled from.
    - The date must be specified in ISO 8601 format.
    - e.g., `2022-06-26T23:09:30-07:00` where `-07:00` represents timezone offset of UTC-07:00 hours

![Set variable configuration](https://media.github.ibm.com/user/375131/files/06799700-0932-11ed-90c4-5842714c7033)

#### Always-On Flow
- This flow is to keep syncing the data after the initial sync
- This flow is meant to be kept running
- This flow will only sync the data that has been added or updated after its previous execution event.
  - e.g., If the flow executes at 2 PM and it's previous execution was at 1 PM, the flow will pull data that has been added or updated after 1 PM.

#### Configuring the Flow Parameters

- Click on the "Set variable" node

![Set variable configuration](https://media.github.ibm.com/user/375131/files/06799700-0932-11ed-90c4-5842714c7033)

- In Variable > config > customer, enter the value provided by Envizi
- In Variable > config > triURL, enter URL for the Tririga instance
    - e.g., https://example.com:9080


### Starting and Stopping the flow

- If using AppConnect Dashboard, Click on the kebab menu (three dots) on the flow's tile.
<img width="656" alt="Flow start 1" src="https://media.github.ibm.com/user/375131/files/5be4ac00-eb1b-11ec-85b0-b7b2d400a7e1">

- If inside the flow, Click on the kebab menu (three dots) on top right of the screen.
<img width="959" alt="Flow start 2" src="https://media.github.ibm.com/user/375131/files/5be4ac00-eb1b-11ec-93f2-4031220e144f">

- Click on "Start API" or "Stop API" depending on which action is desired.

_Note: If the flow is not running, AppConnect will give Error 404 on the API call._

## Post-Setup Instructions

The Envizi implementation team will support the next steps of the integration by provisioning the file transfer details, enabling Envizi data connectors and validating the Envizi configuration ready to receive the data flow.

Please reach out to the primary contact at Envizi who can help coordinate next steps.
