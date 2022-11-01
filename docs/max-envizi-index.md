# IBM MAS Connector for Envizi

## Overview

Enable on-demand and scheduled synchronization of Meters & Meter Reading data between Maximo and Envizi systems.

![Maximo-Envizi Integration](https://media.github.ibm.com/user/375131/files/213f0180-e8d8-11ec-8d47-a276b27d6fb0)

*Maximo to Envizi Integration Diagram*

## Pre-requisites

### Maximo Configuration

#### Pre-checks on your data

*Data for Location Meter will not be exported to Envizi if the Unit of Measurement is not configured.*

The current integration with Envizi only supports the following Units of Measurement for the Location Meter data:

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

Service Address must be configured in Location with the following fields for proper functioning of Envizi's features:

- Street Address
- City
- State
- Post Code
- Country
- Latitude
- Longitude

#### Deploying Maximo Build

Within Maximo, the integration provides integration components (via .dbc script files) and Java Classes as part of the solution. These components need to be installed in the customer's Maximo environment. Components and other content created for this integration solution will be identified by names that begin with PLUSZ.

##### Maximo 7.6.1.2+

1. On the Maximo Admin workstation, overlay the Maximo SMP directory (`/opt/IBM/SMP/maximo`) with the contents from the solution zip file that is provided. This will lay down the Java Classes and .dbc files provided with the solution.

2. Shutdown the MXServer

3. Run UpdateDB command to install the solution components<br>
    a. Navigate to `/opt/IBM/SMP/maximo/tools/maximo`<br>
    b. Run `./updatedb.sh`

4. Build the Maximo EAR file<br>
    a. Navigate to `/opt/IBM/SMP/maximo/deployment`<br>
    b. Run `./buildmaximoear.sh`

5. Deploy the new Maximo EAR File on all Maximo servers<br>
    a. In WebSphere console, navigate to **Applications->Application types->Websphere enterprise applications**<br>
    b. Select **MAXIMO** and then hit **Update** button<br>
    c. Select **Browse** and select the EAR file from Step 4<br>
    d. Hit **Next** and accept defaults from all pages<br>
    e. After deployment, click on **Save**<br>

6. Start the applications and MXServer

##### MAS 8.8+

1. Navigate to the Admin dashboard of the instance of Manage within MAS.
2. Select the workspace the instance is deployed on and update the configuration. 
3. Scroll down to **Customization** and link to the location of the .zip file in the field ([Additional details](https://www.ibm.com/docs/en/maximo-manage/continuous-delivery?topic=manage-setting-customizations) on setting customizations)
4. Click **Apply Changes** and Manage will update the instance with the customization. 

#### Configuring Artifacts

##### Meter Groups
Make sure all the Meters that need to sync with Envizi are in the right meter groups:

Meter Group | Meters
--|--
`PLUSZ_ELEC`| Electric Meters
`PLUSZ_GAS` | Natural Gas Meters
`PLUSZ_WTR` | Water Meters

##### End Points

1. In the Maximo menu, click on **Integration -> End Points**
<img width="580" alt="End Points - Menu" src="https://media.github.ibm.com/user/375131/files/b7315700-e8ed-11ec-8e95-885b57b97a66">
*End Point Navigation*

2. Under the **End Point** column, type **PLUSZ** and hit **Enter** to search.
<img width="692" alt="End Points - Search" src="https://media.github.ibm.com/user/375131/files/aed91c00-e8ed-11ec-9e22-99cece3dadbc">
*End Point Search*

3. Click on the **PLUSZEXPORT** End Point and configure the parameters required to execute the App Connect flow. Click on **Save End Point** on the left side once finished.

Field | Value
-- | --
URL | Full URL to the flow's API as show in App Connect **API Documentation Link**
HEADERS | Replace the `YourAPIKeyHere` with the App Connect flow's API Key. Do not remove `,Content-Type:application/json`
HTTPMETHOD | Do not change the default value **POST**

<img width="946" alt="End Points - Configuration" src="https://media.github.ibm.com/user/375131/files/5787e680-f7dd-11ec-93af-d44216ffbf86">

*End Point Configuration Page*

### AppConnect Configuration

*Note: You need IBM Cloud AppConnect Professional or Enterprise to run this flow.*

*Note: The names in the screenshots are generic, other instances will not have the same names during setup.*

#### Adding Accounts

Before importing the flow to App Connect, add Accounts for SFTP and HTTP connectors.

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

#### Importing the flow

- Open the Dashboard of the AppConnect instance
- Click on the "New" button and select "Import Flow"
<img width="861" alt="Flow Import 0" src="https://media.github.ibm.com/user/375131/files/ea582e00-eb19-11ec-9912-0881d321f3c7">

- Browse to the flow's YAML file and click on "Import"
<img width="960" alt="Flow Import 1" src="https://media.github.ibm.com/user/375131/files/ea582e00-eb19-11ec-88a7-abebff4e0534">

- The flow will now be imported and opened.
<img width="960" alt="Flow Import 2" src="https://media.github.ibm.com/user/375131/files/eaf0c480-eb19-11ec-885b-919da812499b">

#### Configuring the flow to use the right accounts

When importing a flow, it is important to check if the flow is using the right accounts for the different connectors.

- Click on "Edit Flow"

- See if the connectors are using the right accounts.

- To change the account for any connector, select the connector and click on the dropdown icon next to the Account's name
<img width="960" alt="Select Account 1" src="https://media.github.ibm.com/user/375131/files/11186380-eb1e-11ec-9c92-5f892df61a57">

- Select the account name that you want to use from the list
<img width="659" alt="Select Account 2" src="https://media.github.ibm.com/user/375131/files/11b0fa00-eb1e-11ec-88fd-8e4998f6304a">


#### Creating API Key

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
