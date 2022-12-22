# IBM MAS Connector for Envizi

The **IBM MAS Connector for Envizi** is released under the name "MAS Connector for Envizi". This connector supports the following capabilities through an App Connect flow:

-Automatically synchronize Asset location and meter readings from MAS to Envizi to automate tracking of energy usage and calculating its related scope 1 and 2 emissions of Electricity, Natural Gas, and Water.

***

## Use Case Examples

Corporate sustainability managers can gather and maintain accurate sustainability data for their portfolio of assets directly from Maximo where those assets are managed. Location data from Maximo serves as a baseline for all other Sustainability reports in Envizi; continuously updated meter readings data captured in Maximo (Electricity, Gas, Water) enables accurate GHG accounting and performance reporting in Envizi.

***

## Connector Architecture

**IBM App Connect** provides a flexible environment for integration solutions to transform, enrich, route, and process business messages and data. 

**App Connect Flows** enable specific integration use cases by connecting to predefined APIs to route and map data. Mapping has been pre-defined, but it can be customized.

**Native API framework** is used for Maximo and enabled thorugh provided packages that can be imported.

![Maximo-Envizi Integration](https://media.github.ibm.com/user/348712/files/c2494984-60cb-4cfb-ab5a-16602e969cbb)

*Maximo to Envizi Integration Diagram*

***

## Data Mapping

The image below illustrates the type of data that is being sent by the API and App Connect Flows.

<img width="450" height="850" alt="MAS-Envizi Architecture" src="https://media.github.ibm.com/user/348712/files/0c4fda65-2306-476b-8d57-331e53654ea8">

*Maximo to Envizi Data Map*

***

## App Connect Flows

Included with this connector is a flow that exports locations and meter readings, along with all the required fields they contain. The table below shows the naming convention for this flow and the current integration use case.

File | Flow | Destination | Operation
-- | -- |--|--
PLUSZMXTOS3_v1_1_0.yaml | Locations and Meter Reading | Max to Envizi | Bulk Load or Load Changes

***

## Installation & Configuration Guide


>## Before you begin you will need:
>
>1. An instance of App Connect Enterprise or App Connect Pro with the Designer component.
>2. Admin access to your Maximo instance with an api key generated for this integration.<br>
>3. Envizi instance with a AWS S3 Bucket
>4. [Import AppConnect Cert to Maximo](#pre-requisite-add-an-app-connect-certificate-in-mas-89) to enable encrypted communication
>5. [Check that meter data uses supported units of measure](#pre-requisite-pre-check-that-data-is-correct)

***

## Installation Steps Overview
1. Configure App Connect<br>
   a. Import Flows into App Connect<br>
   b. Configure Flows<br>
2. Configure Maximo<br> 
   a. Upload Configuration File <br>
   b. Categorize existing meters into groups <br>
   c. Set up End Points <br>
   d. Configure Cron Task <br>
3. Test <br>
   a. MAS outbound connectivity<br>

***


## Part 1: App Connect Configuration

*Note: You need IBM Cloud AppConnect Professional or Enterprise to run this flow.*

*Note: The names in the screenshots are generic, other instances will not have the same names during setup.*

### Adding Accounts

Before importing the flow to App Connect, add Accounts for S3 and HTTP connectors.

1. Navigate to Catalog section of the AppConnect instance
<img width="960" alt="Create Account 1" src="https://media.github.ibm.com/user/375131/files/eb8b5a00-eb1d-11ec-8401-35b47d561ce4">

- In the **Search application**, type name of the connector to add the account for

- If the AppConnect instance does not have an account for the connector, click on **Connect** to create a new account. Else, open the account selection drop down, and click on **Add a new account ...**
<img width="948" alt="Create Account 2b" src="https://media.github.ibm.com/user/348712/files/02ef8867-555d-4949-a87b-5ffc68d5dca2">

- Enter the necessary details for the connector<br>
    a. For S3, it will be the S3 server and user account.<br>
    b. For HTTP, it will be the Authentication Key or username and password needed for Maximo.
<img width="960" alt="Create Account 3" src="https://media.github.ibm.com/user/348712/files/07bdaf7e-7538-46e8-aaff-27b6acfb8738">

- Click on **Connect**
<img width="960" alt="Create Account 4" src="https://media.github.ibm.com/user/348712/files/80626bfe-53a9-4708-bbc9-e5a110127182">

- From the account selection drop down, select the newly created account. e.g., The default name will be `Account 2` if you already have `Account 1`
- Click on the kebab menu (three dots) and select **Rename Account**
<img width="960" alt="Create Account 5" src="https://media.github.ibm.com/user/348712/files/f0d73883-70c5-4892-a996-fae0cd33a00f">

- Enter an account name and click on **Rename Account**. This name can now be used by the connector in the flow.
<img width="958" alt="Create Account 6" src="https://media.github.ibm.com/user/348712/files/020ecda5-db0a-4380-aa2b-d0c23f969bf5">

### Importing the flow

1. Open the Dashboard of the AppConnect instance
- Click on the **New** button and select **Import Flow**
<img width="861" alt="Flow Import 0" src="https://media.github.ibm.com/user/375131/files/ea582e00-eb19-11ec-9912-0881d321f3c7">

- Browse to the flow's YAML file and click on **Import**
<img width="960" alt="Flow Import 1" src="https://media.github.ibm.com/user/375131/files/ea582e00-eb19-11ec-88a7-abebff4e0534">

- The flow will now be imported and opened.
<img width="960" alt="Flow Import 2" src="https://media.github.ibm.com/user/375131/files/eaf0c480-eb19-11ec-885b-919da812499b">

### Configuring the flow to use the right accounts

*When importing a flow, it is important to check if the flow is using the right accounts for the different connectors.*

1. Click on **Edit Flow**

- See if the connectors are using the right accounts.

- To change the account for any connector, select the connector and click on the dropdown icon next to the Account's name
<img width="960" alt="Select Account 1" src="https://media.github.ibm.com/user/348712/files/e61b9ed4-61ab-4782-b0c4-a42bb432467b">

- Select the account name that you want to use from the list
<img width="659" alt="Select Account 2" src="https://media.github.ibm.com/user/348712/files/52de25b1-d449-4aa8-a1d7-2b6af7e8b24e">


### Authentication

#### On-prem based App Connect

If the instance of App Connect is on-prem, basic authentication will be used. 

1. Click on **Test**.
<img width="960" alt="Flow API Key 1" src="https://media.github.ibm.com/user/375131/files/12479180-eb1a-11ec-86ee-1adea1c7836f">

- Click on **POST** from the left hand side and then the **Try It** tab to get the username and password for the flow.
<img width="960" alt="Flow API Key 2" src="https://media.github.ibm.com/user/375131/files/12e02800-eb1a-11ec-9da1-36c9b85d002c">

- Use this authentication in the End Point section of the Maximo Configuration.

#### Cloud based App Connect
*If the instance of App Connect is on cloud*

1. Click on **Manage**.
<img width="960" alt="Flow API Key 1" src="https://media.github.ibm.com/user/375131/files/12479180-eb1a-11ec-86ee-1adea1c7836f">

- Scroll to the bottom of the page. Click on **Create API key and documentation link**
<img width="960" alt="Flow API Key 2" src="https://media.github.ibm.com/user/375131/files/12e02800-eb1a-11ec-9da1-36c9b85d002c">

- Enter name for the API key and click on **Create**
<img width="960" alt="Flow API Key 3" src="https://media.github.ibm.com/user/375131/files/1378be80-eb1a-11ec-9c2f-b700d0e55d9e">

- Make a note of this key. Maximo End Point will be configured to send this API key as value for the `X-IBM-Client-Id` header while calling the AppConnect flow API.

- Open the **API Documentation Link** in another tab.

- Click on the Route to see the URL
<img width="958" alt="Flow API Key 4" src="https://media.github.ibm.com/user/375131/files/14115500-eb1a-11ec-8988-315fe623751c">

- Copy the link show in details tab next to the HTTP Method
<img width="960" alt="Flow API Key 5" src="https://media.github.ibm.com/user/375131/files/14115500-eb1a-11ec-8bce-d7751c8e9141">

It is helpful to use a tool like Postman to test this API prior to testing it for the first time from Maximo.

***

## Part 2: Maximo Configuration

### Deploying Maximo Build

Within Maximo, the integration provides integration components (via .dbc script files) and Java Classes as part of the solution. These components need to be installed in the customer's Maximo environment. Components and other content created for this integration solution will be identified by names that begin with PLUSZ.

#### Maximo 7.6.1.2+

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

#### MAS 8.8+

1. Navigate to the Admin dashboard of the instance of Manage within MAS.
2. Select the workspace the instance is deployed on and update the configuration. 
3. Scroll down to **Customization** and link to the location of the .zip file in the field ([Additional details](https://www.ibm.com/docs/en/maximo-manage/continuous-delivery?topic=manage-setting-customizations) on setting customizations)
4. Click **Apply Changes** and Manage will update the instance with the customization. 

### Configuring Artifacts

#### Meter Groups
Make sure all the Meters that need to sync with Envizi are in the right meter groups:

Meter Group | Meters
--|--
`PLUSZ_ELEC`| Electric Meters
`PLUSZ_GAS` | Natural Gas Meters
`PLUSZ_WTR` | Water Meters

#### End Points

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

#### Cron Tasks

The integration is equipped to export Locations and Location Meter Readings out of Maximo. As part of the PLUSZEXPORT cron task, there are four individual cron tasks that are configurable- two for Locations and two for Meter Readings. These are the Always-on Cron Task and the On-Demand Cron Task.

##### Always-on Cron Task
This is for pulling the live or recent data from Locations and Meter Readings, and is supposed to be run frequently. This will pull the data that has come into the system after the last time it ran.

Based on its running frequency, the system will automatically decide the window of dates/timestamps to pull the data from.

These Cron Task Instances will use the suffix `_ALWAYS_ON`

<img width="360" alt="Cron task instance - always on" src="https://media.github.ibm.com/user/375131/files/27732000-f7aa-11ec-82f3-0b1e39ea5c02">


##### On-demand Cron Task
Even though this is a Cron Task, this is supposed to run just once, on-demand, when the user wants to pull data between fixed dates/timestamps. Once the first cron event executes, it should be stopped in order to prevent the same data getting pulled again and again. 

_This is a stop-gap for now as we dont have another way from Maximo to call a preconfigured API on-demand._

This might be needed during initial setup of the system, as the Always-on Cron will start pulling data that has arrived after it has been started. This can also be used anytime in the future if there is any need to pull any historical data on-demand.

The start date (`OVERRIDESTARTDATE`) and end date (`OVERRIDEENDDATE`) have to be configured before the cron is started. 

These Cron Task Instances will use the suffix `ON_DEMAND`

<img width="356" alt="Cron task instance - on demand" src="https://media.github.ibm.com/user/375131/files/6e611580-f7aa-11ec-80ac-5378b76aa2a9">


##### Basic Cron Task Parameters

- `MXURL`: The base URL of the Maximo instance. This would include the protocol, domain name and the port number. Do not put a trailing `/` at the end. In the absense of port number, the default port for the protocol will be used _(80 for http, 443 for https)_.
    - e.g. http://example.maximo.com:9080
    - For MAS deployments, this route should be to the Manage instance domain. You can find this route in OpenShift under **Networking -> Routes** within the Manage namespace. The route should have the format **"workspace".manage."instanceid"**

- `CUSTOMER`: Name of the customer. This will be used to name the CSV files. To be supplied by Envizi.

- `OVERRIDESTARTDATE`: _(Only for On-demand Cron)_ The start timestamp of the window between which the data will be pulled from.
    - The date must be specified in ISO 8601 format.
    - e.g. `2022-06-26T23:09:30-07:00` where `-07:00` represents timezone offset of UTC-07:00 hours
- `OVERRIDEENDDATE`: _(Only for On-demand Cron)_ The end timestamp of the window between which the data will be pulled from.
    - The date must be specified in ISO 8601 format.
    - e.g. `2022-06-26T23:09:30-07:00` where `-07:00` represents timezone offset of UTC-07:00 hours


##### Advanced Parameters

These will be set to the right values by default. **DO NOT** change these parameters without consulting with Envizi.

- `SELECT`: Names of the fields from the Object Structure
- `FROM`: Name of the Object Structure
- `WHERE`: _(Optional)._ OSLC Condition for data selection
- `ORDERBY`: _(Optional)._ Sequence by which the data should be pulled
- `TARGETDATECOLUMN`: Name of the DateTime Column in the Object structure that will be used to filter records between a date-time window.
- `SAVEDQUERY`
- `ENDPOINTNAME` - Name of the Maximo End Point configured to connect to the AppConnect flow.
- `PAGESIZE` - Maximum number of records pulled from Maximo in one API call. This will also be the maximum number of records in the CSV file.

##### Configuring Cron Task

1. In the Maximo menu, click on **System Configuration > Platform Configuration > Cron Task Setup**
<img width="638" alt="Cron task - Menu" src="https://media.github.ibm.com/user/375131/files/204c1200-f7aa-11ec-902b-56a6cacbbbb1">

- Under the **Cron Task** column, type **PLUSZ** and hit enter to search
<img width="754" alt="Cron task - Search" src="https://media.github.ibm.com/user/375131/files/217d3f00-f7aa-11ec-8d77-1339ac049a2d">

- Click on the **PLUSZEXPORT** Cron Task
- In the Cron Task Instances, click on the Schedule/Calendar icon to set the schedule for the Cron Task Instance
- Once it is configured, click on **OK**
- Select the Cron Task Instance to configure Parameters
- Scroll down to the **Cron Task Parameters** section and configure the **Value** column as needed. Ideally, only the `CUSTOMER` and `MXURL` parameters will need to be edited for all instances. For the **On-demand** Crons, the `OVERRIDESTARTDATE` and `OVERRIDEENDDATE` parameters will need to be edited.
- Click on **Save**

***

## Operating the connector

<!-- - <a href="https://github.com/IBM/maximo-envizi-appconnect-flows/blob/main/AppConnect%20Flows/PLUSZMXTOSFTP.yml" target="_blank">PLUSZMXTOSFTP</a> -->

##### Start/Stop the Cron Task Instance
1. Navigate inside the desired Cron Task.
- Enable/Disable the checkbox in the column **Active?** in front of the desired Cron Task Instance
- Click on **Save**

### Starting and Stopping the flow

1. If using AppConnect Dashboard, Click on the kebab menu (three dots) on the flow's tile.
<img width="656" alt="Flow start 1" src="https://media.github.ibm.com/user/375131/files/5be4ac00-eb1b-11ec-85b0-b7b2d400a7e1">

- If inside the flow, Click on the kebab menu (three dots) on top right of the screen.
<img width="959" alt="Flow start 2" src="https://media.github.ibm.com/user/375131/files/5be4ac00-eb1b-11ec-93f2-4031220e144f">

- Click on **Start API** or **Stop API** depending on what action you want to perform.

<img width="893" alt="PLUSZMXTOSFTP_P1" src="https://media.github.ibm.com/user/375131/files/4383bb00-f7c4-11ec-9d13-c85b6f8ce6ee">
<img width="884" alt="PLUSZMXTOSFTP_P2" src="https://media.github.ibm.com/user/375131/files/441c5180-f7c4-11ec-9cbd-229152cebafd">
<img width="902" alt="PLUSZMXTOSFTP_P3" src="https://media.github.ibm.com/user/348712/files/985426bc-11fe-4864-8094-3cc1dc2de611">

## Part 3: Testing

To test that the configuration is complete, send a test payload in order to test connectivity.

1. Go to the **End Points** application and click **Test** at the bottom of the PLUSZExport end point.
- Send a test payload that is a valid object. ```{"Hello":"World"}``` would work.
- If the response is anything other than **Bad Request**, see [what might be causing the error](#troubleshooting)

## Troubleshooting
 
When testing that the end point is entered correctly on the **End Points** application, there are a few common errors:

Error | Cause | Resolution
 -- |-- |--
Response code received from the HTTP request from the endpoint is not successful | Invalid URL in the Integration Object | Double check the URL that all of the components are entered correctly. Make sure there are no accidental spaces at the beginning or the end in event of a copy/paste.
404: Not Found | Flow is not Running | Make sure the flow is Running before starting the cron tasks
PKSync error| Certificate error | Confirm the certificate is configured correctly
Unable to Verify Leaf Signature | Certificate Error |Confirm the HTTP account is set with "Allow Self-Signed Certificates" as True.
Status Code 400: BMXAA8744E - The OSLC query was not parsed. | Incorrect character in cron task parameter field, such as a decimal point in Page Size |Check the parameters in the cron task.
“Cannot GET /maximo/api/os/PLUSZLOCATIONS” | MXUrl is not valid |Verify the MXURL parameter in Cron task that it is pointing to the correct url.


## Reference for Pre-requisite

### **Pre-requisite: Pre-check that Data is correct**

*Data for Location Meter will not be exported to Envizi if the Unit of Measurement is not configured.*

The current integration with Envizi only supports the following Units of Measurement for the Location Meter data:

Meter Type | Supported Unit of Measure
-- | --
Electric | GJ, kWh, MWh
Natural Gas | GJ, MJ, kWh, m3, mmbtu, therms
Water | litres, kliter, m3, gallons

If these Units of Measure are not present on the instance of Maximo, they can be added in the **Item Master** application under **Inventory** on the drop down menu.

Select **Unit of Measure and Conversion -> Add/Modify Units of Measure** and add the missing units.

<img width="800" alt="Screen Shot 2022-11-16 at 6 31 58 PM" src="https://media.github.ibm.com/user/348712/files/0570811c-9472-4bb8-9208-9d20d50f78fe">
<img width="800" alt="Screen Shot 2022-11-16 at 6 32 11 PM" src="https://media.github.ibm.com/user/348712/files/62ba84ab-d29a-4349-b546-290cd61c075e">

A Service Address must be configured in Location with the following fields for proper functioning of Envizi's features:

- Street Address
- City
- State
- Post Code
- Country
- Latitude
- Longitude

If the Location does not have a Service Address, navigate to **Address Information** under the Location and either select or create an associated Service Address.

<img width="800" alt="Service Address" src="https://media.github.ibm.com/user/348712/files/e6d257ba-d8da-458f-858b-d2077c7a8481">

If one needs to be created, navigate to **Administration -> Service Address** and click **New Service Address** and enter the required fields. Then return to the desired location and associate the Service Address in **Address Information**.

<img width="800" alt="Service Address-2" src="https://media.github.ibm.com/user/348712/files/a0b4fabe-dbb9-48c8-9f08-2d893bc35a47">
<img width="800" alt="Service Address-1" src="https://media.github.ibm.com/user/348712/files/a4dbdcda-3c6a-4d70-a751-ee0c9863454b">

### **Pre-requisite: add an App Connect Certificate in MAS 8.9+**

1. Extract the App Connect certificate from an imported flow URL. Navigate to the flow's page and click on **Test** and then **Try It** to get the proper URL.

2. Navigate to the Admin dashboard for MAS and go to the workspace where Manage is deployed.


3. Update the configuration and scroll down to **Imported Certificates**. Untick the **System managed** button and fill in the extracted certificate in the fields.


4. Click **Apply Changes** at the top of the page and MAS will update the truststore with the new certificate.

<img width="600" alt="Manage-Workspace" src="https://media.github.ibm.com/user/348712/files/33502d00-43fe-11ed-801b-a454fa4b8f7e">

*Step 2: Manage Workspace*

<img width="600" alt="Imported-Certificates" src="https://media.github.ibm.com/user/348712/files/33e8c380-43fe-11ed-91b9-68b37462c897">

*Step 3: Imported Certificates*

### Pre-requisite: add an App Connect Certificate in Maximo 7.6.1.2+:


Configure WebSphere Certificates. This makes a test connection to a Secure Sockets Layer (SSL) port and retrieves the signer from the server during the handshake.

1. Log into the WebSphere console that is hosting the Maximo server.

2. Click on **Security** -> **SSL certificate & key management**. Under **Related Items** click on **Key stores and certificates**.

3. Click on **CellDefaultTrustStore** and on the next page under **Additional Properties** click on **Signer certificates**. 

4. From this page, click on the button that says **Retrieve from Port** and fill in the required fields using the table below:

    Field | Value
    ---|---
    Host | The host from the imported flow URL
    Port | 443
    Alias | appconnect

5. Once all three have been entered in, click **Retrieve signer information** and the information from the URL will populate on screen. Click **Save** in the box at the top and then repeat the process for **NodeDefaultTrustStore**.

<img width="600" alt="Websphere-Home" src="https://media.github.ibm.com/user/348712/files/ee26db00-3801-11ed-8365-8604ad0a66df">

*Step 2: WebSphere Homepage*

<img width="600" alt="Websphere-Keystores" src="https://media.github.ibm.com/user/348712/files/eebf7180-3801-11ed-9253-85d4e8bed0ea">

*Step 3: Websphere Keystores*

<img width="600" alt="Websphere-Signercerts" src="https://media.github.ibm.com/user/348712/files/f121cb80-3801-11ed-9833-65fe541bcef5">
 
*Step 4: Websphere Signer Certs*
