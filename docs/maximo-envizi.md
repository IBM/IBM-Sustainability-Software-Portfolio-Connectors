# Maximo - Envizi integration

## Table of contents

<!--ts-->
  * [Overview](#overview)
  * [Maximo Configuration](#maximo-configuration)
    * [Pre-checks on your data](#pre-checks-on-your-data)
    * [Deploying Maximo Build](#deploying-maximo-build)
    * [Configuring Artifacts](#configuring-artifacts)
        * [Meter Groups](#meter-groups)
        * [End Points](#end-points)
        * [Crons](#crons)
            * [Always-on Cron (Always-Cron)](#always-on-cron-always-cron)
            * [On-demand Cron (Cron-demand)](#on-demand-cron-cron-demand)
            * [Basic Parameters](#basic-parameters)
            * [Advanced Parameters](#advanced-parameters)
            * [Configuring Cron Task](#configuring-cron-task)
            * [Start/Stop the Cron Task Instance](#startstop-the-cron-task-instance)
  * [AppConnect Configuration](#appconnect-configuration)
    * [Adding Accounts](#adding-accounts)
    * [Importing the flow](#importing-the-flow)
    * [Configuring the flow to use the right accounts](#configuring-the-flow-to-use-the-right-accounts)
    * [Starting and Stopping the flow](#starting-and-stopping-the-flow)
    * [Creating API Key](#creating-api-key)
  * [Working of the flow](#working-of-the-flow)
<!--te-->

## Overview
![Maximo-Envizi Integration](https://media.github.ibm.com/user/375131/files/213f0180-e8d8-11ec-8d47-a276b27d6fb0)

## Maximo Configuration

### Pre-checks on your data

- Data for Location Meter will not be exported to Envizi if the Unit of Measurement is not configured.
- The current integration with Envizi only supports the following Units of Measurement for the Location Meter data:
    - Electric Meters
        - GJ
        - kWh
        - MWh
    - Natural Gas Meters
        - GJ
        - MJ
        - kWh
        - m3
        - mmbtu
        - therms
    - Water Meters
        - litres
        - kliter
        - m3
        - gallons

- Service Address must be configured in Location with the below fields for proper functioning of Envizi's features:
    - Street Address
    - City
    - State
    - Post Code
    - Country
    - Latitude
    - Longitude

### Deploying Maximo Build

Within Maximo, the integration provides integration components (via .dbc script files) and Java Classes as part of the solution. These components need to be installed in the customer's Maximo environment. Components and other content created for this integration solution will be identified by names that begin with PLUSZ.

1. On the Maximo Admin workstation, overlay the Maximo SMP directory (`/opt/IBM/SMP/maximo`) with the contents from the solution zip file that is provided. This will lay down the Java Classes and .dbc files provided with the solution.

2. Shutdown the MXServer

3. Run UpdateDB command to install the solution components
    - Navigate to `/opt/IBM/SMP/maximo/tools/maximo`
    - Run `./updatedb.sh`

4. Build the Maximo EAR file
    - Navigate to `/opt/IBM/SMP/maximo/deployment`
    - Run `./buildmaximoear.sh`

5. Deploy the new Maximo EAR File on all Maximo servers
    - In WebSphere console, navigate to: Applications->Application types->Websphere enterprise applications
    - Select MAXIMO and then hit "Update" button
    - Select "Browse" and select the EAR file from Step 4
    - Hit "Next" and accept defaults from all pages
    - After deployment, click on "Save"

6. Start the applications and MXServer

### Configuring Artifacts

#### Meter Groups
- Make sure all the Meters that need to sync with Envizi are in the right meter groups:
    - `PLUSZ_ELEC` - Electric Meters
    - `PLUSZ_GAS` - Natural Gas Meters
    - `PLUSZ_WTR` - Water Meters

#### End Points

- In the Maximo menu, click on Integration > End Points
<img width="580" alt="End Points - Menu" src="https://media.github.ibm.com/user/375131/files/b7315700-e8ed-11ec-8e95-885b57b97a66">

- Under the "End Point" column, type "PLUSZ" and hit enter to search

- Click on the "PLUSZEXPORT" End Point
<img width="692" alt="End Points - Search" src="https://media.github.ibm.com/user/375131/files/aed91c00-e8ed-11ec-9e22-99cece3dadbc">

- Configure the parameters required to execute the AppConnect flow
    - `URL`: Full URL to the flow's API as show in AppConnect "API Documentation Link"
    - `HEADERS`: Replace the `YourAPIKeyHere` with the AppConnect flow's API Key
        - By default the AppConnect flow will be looking for the API key in the `X-IBM-Client-Id` header
        - If the API Key is `123`, the final value in this field will be `X-IBM-Client-Id: 123,Content-Type: application/json`
        - **Do not** remove `,Content-Type:application/json`
    - `HTTPMETHOD`: Do not change the default value `POST`
<img width="946" alt="End Points - Configuration" src="https://media.github.ibm.com/user/375131/files/5787e680-f7dd-11ec-93af-d44216ffbf86">

- Click on "Save End Point" in "Common Actions" on the left side

The End Point is now configured 

#### Crons

For every Object Structure that gets pulled out of Maximo and pushed to Envizi, there are two types of Cron jobs:

##### Always-on Cron (Always-Cron)
This is for pulling the live or recent data, which is supposed to run frequently. This will pull the data that has come into the system after the last time it ran.

Based on its running frequency, the system will automatically decide the window of dates/timestamps to pull the data from.

These Cron Task Instances will use the suffix `_ALWAYS_ON`

<img width="360" alt="Cron task instance - always on" src="https://media.github.ibm.com/user/375131/files/27732000-f7aa-11ec-82f3-0b1e39ea5c02">


##### On-demand Cron (Cron-demand)
Even though this is a Cron, this is supposed to run just once, on-demand, when the customer or the user wants to pull data between fixed dates/timestamps. Once the first cron event executes, it should be stopped in order to prevent the same data getting pulled again and again. _This is a stop-gap for now as we dont have another way from Maximo to call a preconfigured API on-demand._

This might be needed during initial setup of the system, as the Always-on Cron will start pulling data that has arrived after it has been started. This can also be used anytime in the future if there is any need to pull any historical data on-demand.

The start date (`OVERRIDESTARTDATE`) and end date (`OVERRIDEENDDATE`) have to be configured before the cron is started. 

These Cron Task Instances will use the suffix `ON_DEMAND`

<img width="356" alt="Cron task instance - on demand" src="https://media.github.ibm.com/user/375131/files/6e611580-f7aa-11ec-80ac-5378b76aa2a9">


##### Basic Parameters

- `MXURL`: The base URL of the Maximo instance. This would include the protocol, domain name and the port number. Do not put a trailing `/` at the end. In the absense of port number, the default port for the protocol will be used _(80 for http, 443 for https)_.
    - e.g. http://example.maximo.com:9080

- `CUSTOMER`: Name of the customer. This will be used to name the CSV files. To be supplied by Envizi.

- `OVERRIDESTARTDATE`: _(Only for On-demand Cron)_ The start timestamp of the window between which the data will be pulled from.
    - The date must be specified in ISO 8601 format.
    - e.g. `2022-06-26T23:09:30-07:00` where `-07:00` represents timezone offset of UTC-07:00 hours
- `OVERRIDEENDDATE`: _(Only for On-demand Cron)_ The end timestamp of the window between which the data will be pulled from.
    - The date must be specified in ISO 8601 format.
    - e.g. `2022-06-26T23:09:30-07:00` where `-07:00` represents timezone offset of UTC-07:00 hours


##### Advanced Parameters

These will be set to the right values by default. DO NOT change these parameters without consulting with Envizi.

- `SELECT`: Names of the fields from the Object Structure
- `FROM`: Name of the Object Structure
- `WHERE`: _(Optional)._ OSLC Condition for data selection
- `ORDERBY`: _(Optional)._ Sequence by which the data should be pulled
- `TARGETDATECOLUMN`: Name of the DateTime Column in the Object structure that will be used to filter records between a date-time window.
- `SAVEDQUERY`
- `ENDPOINTNAME` - Name of the Maximo End Point configured to connect to the AppConnect flow.
- `PAGESIZE` - Maximum number of records pulled from Maximo in one API call. This will also be the maximum number of records in the CSV file.

##### Configuring Cron Task

- In the Maximo menu, click on System Configuration > Platform Configuration > Cron Task Setup
<img width="638" alt="Cron task - Menu" src="https://media.github.ibm.com/user/375131/files/204c1200-f7aa-11ec-902b-56a6cacbbbb1">

- Under the "Cron Task" column, type "PLUSZ" and hit enter to search
<img width="754" alt="Cron task - Search" src="https://media.github.ibm.com/user/375131/files/217d3f00-f7aa-11ec-8d77-1339ac049a2d">

- Click on the "PLUSZEXPORT" Cron Task
- In the Cron Task Instances, click on the Schedule/Calendar icon to set the schedule for the Cron Task Instance
- Once it is configured, click on "OK"
- Select the Cron Task Instance to configure Parameters for
- Scroll down to the "Cron Task Parameters" section and configure the "Value" column as needed. Ideally, only the `CUSTOMER` and `MXURL` parameters will need to be edited for all instances. For the "On-demand" Crons, the `OVERRIDESTARTDATE` and `OVERRIDEENDDATE` parameters will need to be edited.
- Click on Save

##### Start/Stop the Cron Task Instance
- Navigate inside the Cron Task as mentioned above
- Enable/Disable the checkbox in the column "Active?" in front of the desired Cron Task Instance
- Click on Save

## AppConnect Configuration

Note: You need IBM Cloud AppConnect Professional or Enterprise to run this flow.

Note: The names in the screenshots are generic, other instances will not have the same names during setup.

### Adding Accounts

Before importing the flow to AppConnect, add Accounts for SFTP and HTTP connectors.

- Navigate to Catalog section of the AppConnect instance
<img width="960" alt="Create Account 1" src="https://media.github.ibm.com/user/375131/files/eb8b5a00-eb1d-11ec-8401-35b47d561ce4">

- In the "Search application", type name of the connector to add the account for

- If the AppConnect instance does not have an account for the connector, click on "Connect" to create a new account
<img width="960" alt="Create Account 2a" src="https://media.github.ibm.com/user/375131/files/ec23f080-eb1d-11ec-9f62-af1addec3fd3">

- Else, open the account selection drop down, and click on "Add a new account ..."
<img width="948" alt="Create Account 2b" src="https://media.github.ibm.com/user/375131/files/ecbc8700-eb1d-11ec-9693-4ce83ffda2e6">

- Enter the necessary details for the connector
    - For SFTP, it will be the SFTP server and user account.
    - For HTTP, it will be the Authentication Key or username and password needed for Maximo.
<img width="960" alt="Create Account 3" src="https://media.github.ibm.com/user/375131/files/ed551d80-eb1d-11ec-9894-350e87fbb2b2">

- Click on Connect
<img width="960" alt="Create Account 4" src="https://media.github.ibm.com/user/375131/files/ed551d80-eb1d-11ec-983c-c276d43e0654">

- From the account selection drop down, select the newly created account. e.g., The default name will be `Account 2` if you already have `Account 1`
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

- Select the account name that you want to use from the list
<img width="659" alt="Select Account 2" src="https://media.github.ibm.com/user/375131/files/11b0fa00-eb1e-11ec-88fd-8e4998f6304a">


### Starting and Stopping the flow

- If using AppConnect Dashboard, Click on the kebab menu (three dots) on the flow's tile.
<img width="656" alt="Flow start 1" src="https://media.github.ibm.com/user/375131/files/5be4ac00-eb1b-11ec-85b0-b7b2d400a7e1">

- If inside the flow, Click on the kebab menu (three dots) on top right of the screen.
<img width="959" alt="Flow start 2" src="https://media.github.ibm.com/user/375131/files/5be4ac00-eb1b-11ec-93f2-4031220e144f">

- Click on "Start API" or "Stop API" depending on what action you want to perform.

_Note: If the flow is not running, AppConnect will give Error 404 on the API call._

### Creating API Key

- Click on "Manage".
<img width="960" alt="Flow API Key 1" src="https://media.github.ibm.com/user/375131/files/12479180-eb1a-11ec-86ee-1adea1c7836f">

- Scroll to the bottom of the page. Click on "Create API key and documentation link"
<img width="960" alt="Flow API Key 2" src="https://media.github.ibm.com/user/375131/files/12e02800-eb1a-11ec-9da1-36c9b85d002c">

- Enter name for the API key and click on "Create"
<img width="960" alt="Flow API Key 3" src="https://media.github.ibm.com/user/375131/files/1378be80-eb1a-11ec-9c2f-b700d0e55d9e">

- Make a note of this key. Maximo End Point will be configured to send this API key as value for the `X-IBM-Client-Id` header while calling the AppConnect flow API.

- Open the "API Documentation Link" in another tab.

- Click on the Route to see the URL
<img width="958" alt="Flow API Key 4" src="https://media.github.ibm.com/user/375131/files/14115500-eb1a-11ec-8988-315fe623751c">

- Copy the link show in details tab next to the HTTP Method
<img width="960" alt="Flow API Key 5" src="https://media.github.ibm.com/user/375131/files/14115500-eb1a-11ec-8bce-d7751c8e9141">

It is helpful to use a tool like Postman to test this API prior to testing it for the first time from Maximo.

## Working of the flow

<img width="893" alt="PLUSZMXTOSFTP_P1" src="https://media.github.ibm.com/user/375131/files/4383bb00-f7c4-11ec-9d13-c85b6f8ce6ee">
<img width="884" alt="PLUSZMXTOSFTP_P2" src="https://media.github.ibm.com/user/375131/files/441c5180-f7c4-11ec-9cbd-229152cebafd">
<img width="902" alt="PLUSZMXTOSFTP_P3" src="https://media.github.ibm.com/user/375131/files/467eab80-f7c4-11ec-9558-276e2708db02">


#### If
Checks for the mandatory fields that are required for the flow to function. If any of the fields are missing, the flow returns Error 400.

#### If 3
Checks the presence of `OVERRIDESTARTDATE` and `OVERRIDEENDDATE` in the Request. It also checks if they are valid dates. If all checks are passed, `dateQuery` with these override dates is generated.
If the override date checks fail, it checks if `LASTSTARTDATE` is present. If it is, `dateQuery` with it is generated.
If all checks fail, the flow returns Error 400

#### Set variable 5

- `where`: The final `oslc.where` query created using `WHERE` from Request and `dateQuery` from the output of "If 3"
- `batchTimestamp`: Current Epoc timestamp in milliseconds. This will be included in every filename to indicate that they are from the same batch of exported data.

#### Set variable

- `query`: The query to be sent to OSLC API to fetch the required data
- `pagesize`: The total number of records that will be fetched in one page. If it is not specified in the request, the flow will use default value of `100`.

#### HTTP Invoke Method
Does an HTTP GET to the Maximo's OSLC API to fetch the total number of records that match the query. `countonlyparams` from "Set variable" will be used here.

#### Set variable 2
`totalCount`: Total Count received in the response of the "HTTP Invoke Method"

#### Set variable 3
`totalPages`: Total number of pages to be fetched from Maximo OSLC API that match the query. This is calculated using the `totalCount` from "Set variable 2" and `pagesize` from "Set variable".

#### For each: page
- Summary: Fetches data from Maximo OSLC API page by page and writes CSV file to the SFTP server.
- Input: An array of numbers from 1 to `totalPages` from "Set variable 3"
- Output: `fileWritten` - Array of string containing names of files written to the SFTP server.

#### HTTP Invoke method 2
Does an HTTP GET call to Maximo OSLC API to fetch the data that matches `queryparams` from "Set variable" for the current page from "For each: page"

#### Set variable 4
`filename`: File name as per format specified by Envizi team using `CUSTOMER` and `FROM` from the Request, `batchTimestamp` from "Set variable 5" and current page from "For each: page".

eg. `DemoCorporation_PLUSZLOCATIONS_1654856600431_1.csv`

#### SFTP Create file
Writes Response body from "HTTP Invoke method 2" to the configured SFTP Server with the `filename` from "Set variable 4" at the preconfigured file path.

#### Response
Returns HTTP Status 200 with JSON body containing:
- `filesWritten`: Array of file names written to the SFTP server, obtained from the output of "For each: page"
- `queryparams`: `query` from "Set variable"

The JSON in response body is not used or stored by Maximo.
