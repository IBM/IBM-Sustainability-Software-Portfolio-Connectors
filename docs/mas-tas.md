# App Connect Flows for MAXIMO/TRIRIGA Application Suites

### Summary

Enable automated synchronization of portfolio data between TRIRIGA and Maximo Asset Management systems to integrate asset and facility management operations.

### Description

In this code pattern, learn how to synchronize bi-directional portfolio data of IBM TRIRIGA and IBM Maximo Asset Management using App Connect Designer flows for synchronizing People, Locations, and Assets. Using the provided flows, enable a variety of use cases: from a simple trigger-action integration that updates or populates the records in the other application, to more complex scripted cron jobs that orchestrate these objects into their respective primary system of record.

<img src="/Pics/MX2TRI-Graph.png" >

1. When a record is updated in Maximo Asset Management, it triggers the flow to synchronize with TRIRIGA. (There is also a flow that works in the reverse direction, that works in a similar way.)
2. App Connect sends a request with the updated information through the flow towards the target system (TRIRIGA).
3. A JSON Parser sifts through the request and converts it to an object.
4. Each object that is returned from the JSON Parser goes through Steps 5-8.
5. The data from the object is mapped to the corresponding fields in the target application (TRIRIGA).
6. The newly mapped data is sent to the target application (TRIRIGA) where the record is created or updated.
7. This record is then validated with the original application (Maximo Asset Management) via another Post request.
8. An ID is either created or updated within the original application (Maximo Asset Management).
9. Steps 4-8 are repeated for each object that is present in the original request (Maximo Asset Management).

At the end of this process, a Person, Asset, or Location can be sent from Maximo to TRIRIGA and TRIRIGA to Maximo utilizing App Connect.

A detailed breakdown of the specific fields being mapped within each flow can be found in this [Mapping Document](/docs/TRIRIGA_Maximo_Field_Mapping-Final.xlsx)

## Pre-requisites

 <details><summary><b>MAXIMO</b></summary>
 
### Credentials and access to an instance of Maximo as well as the WebSphere application which hosts Maximo are required. This code pattern has been tested on Maximo 7.6.1.2 as well as MAS 8.6 
 
The following steps and pre-requisites are done against a Maximo demo database. The naming conventions may slightly differ from this, but these are the necessary components. 

Within Maximo, configure your instance to be ready to receive records from TRIRIGA. If these pre-requisites are not completed, the action will not be recorded.

### 1. Create an Organization named TRIRIGA
 
  - Navigate to the 'Organizations' page and click the blue + button on the top row.
  - Fill in the Organization name with TRIRIGA and the description as "TRIRIGA Organization".
  - Fill in the remaining required fields as such
    1. Base Currency 1: USD
    2. Item Set: SET1
    3. Company Set: COMPSET1
    4. Default Item Status: PENDING
    5. Default Stock Category: STK
  - Click Save Organization on the left side of the screen under Common Actions. This will be set to Active later once there is a clearing account.

### 2. Create a Testing clearing account in Chart of Accounts
  
  - Navigate to Financial -> Chart of Accounts and click on the previously created TRIRIGA org in the Organizations table. Currently, there should be no GL Accounts for TRIRIGA present
  - Click 'GL Component Maintenance' on the left side under More Actions and add a New Row with the following values:
    1. GL Component Value: 1001
    2. Description: Testing
    3. Active?: Yes
  - Click OK. Click New Row under GL Accounts for TRIRIGA and click the magnifying glass to search for that GL Component. Select it and it should populate in the GL Account & Description fields. The Active Date field should auto populate to the current date.
 - Now that this account is present, head back to Organizations and update the TRIRIGA organization to show the just created Clearing Account, tick the Active box, and click Save Organization.
 
 
 ### 3. Create a site TRIMAIN and set it to active
 
  - On the Organization page, click on the 'Sites' tab at the top of the page.
  - Click New Row under 'Sites' and enter TRIMAIN for Site and "MAIN Site" for Description. Set the site to Active.
  - Click Save Organization.

### 4. Create the PLUSTTRIRIGA External System

  - Navigate to Integration -> External Systems and click on the blue plus button at the top of the page.
  - Under the System name fill in PLUSTTRIRIGA and in the Description fill in "To integrate Maximo with TRIRIGA"
  - Enable the System and then fill in the Queues on the right hand side as follows:
    1. Outbound Sequential Queue: jms/maximo/int/queues/sqout
    2. Inbound Sequential Queue: jms/maximo/int/queues/sqin
    3. Inbound Continuous Queue: jms/maximo/int/queues/cqin
  - Save the External System

### 5. Create the Publish Channels for each integration
 
 
 
 <img src="/Pics/Publish-Channel-Config.png">
 

  - Navigate to Integration -> Publish Channels
  - For Asset
    1. Search for 'MXASSETInterface' under the Publish Channel field. Click on the channel and from the left side of the screen select 'Duplicate Publish Channel' 
    2. Rename the channel PLUSTMXASSETInterface
    3. Click on 'Enable Event Listener' on the left side under More Actions
    4. Make sure Publish JSON and Retain MBO's are checked, the Operation should default to Publish and the Adapter should default to MAXIMO.
    5. Click 'Save Publish Channel' on the left under Common Actions
  - For Location
    1. Search for 'MXOPERLOCInterface' under the Publish Channel field. Click on the channel and from the left side of the screen select 'Duplicate Publish Channel' 
    2. Rename the channel PLUSTMXOPERLOCInterface
    3. Click on 'Enable Event Listener' on the left side under More Actions
    4. Make sure Publish JSON and Retain MBO's are checked, the Operation should default to Publish and the Adapter should default to MAXIMO.
    5. Click 'Save Publish Channel' on the left under Common Actions
  - For Person
    1. Search for 'MXPERSONInterface' under the Publish Channel field. Click on the channel and from the left side of the screen select 'Duplicate Publish Channel' 
    2. Rename the channel PLUSTMXPERSONInterface
    3. Click on 'Enable Event Listener' on the left side under More Actions
    4. Make sure Publish JSON and Retain MBO's are checked, the Operation should default to Publish and the Adapter should default to MAXIMO.
    5. Click 'Save Publish Channel' on the left under Common Actions

### 6. Create the Enterprise Services for each integration
 
 
 
 <img src="/Pics/Enterprise-Service-Config.png">
 
  - Navigate to Integration -> Enterprise Services and click on the blue plus button at the top of the page
  - For Asset
    1. Under the System name fill in PLUSTMXASSETInterface and in the Description fill in "ASSETS"
    2. Select 'MXASSET' under Object Structure which will populate the Object Structure Sub-Records table
    3. Click 'Save Enterprise Service' on the left under Common Actions
  - For Location
    1. Under the System name fill in PLUSTMXOPERLOCInterface and in the Description fill in "OPERATION LOCATION"
    2. Select 'MXOPERLOC' under Object Structure which will populate the Object Structure Sub-Records table
    3. Click 'Save Enterprise Service' on the left under Common Actions
  - For Person
    1. Under the System name fill in PLUSTMXPERSONInterface and in the Description fill in "PERSON"
    2. Select 'MXPERSON' under Object Structure which will populate the Object Structure Sub-Records table
    3. Click 'Save Enterprise Service' on the left under Common Actions

### 7. Create the End Points for each integration 
 
 
 <img src="/Pics/End-Point-Config.png">
 
 
  - Navigate to Integration -> End Points and click on the blue plus button at the top of the page
  - For Asset
    1. Under End Point fill in PLUSTASSET and in the Description fill in "AppConnect ASSET outbound to TRIRIGA"
    2. Select 'HTTP' for Handler
    3. Click on 'Save End Point' on the left side under More Actions which will populate the Properties for the End Point
    4. Until the flows have a destination url, we can only fill in certain fields:
       - HEADERS: "Content-Type: application/json"
       - HTTPMETHOD: POST
    5. Save the End Point
 
  - For Location
    1. Under End Point fill in PLUSTLOCATION and in the Description fill in "AppConnect LOCATION outbound to TRIRIGA"
    2. Select 'HTTP' for Handler
    3. Click on 'Save End Point' on the left side under More Actions which will populate the Properties for the End Point
    4. Until the flows have a destination url, we can only fill in certain fields:
       - HEADERS: "Content-Type: application/json"
       - HTTPMETHOD: POST
    5. Save the End Point
  - For Person
    1. Under End Point fill in PLUSTPERSON and in the Description fill in "AppConnect PERSON outbound to TRIRIGA"
    2. Select 'HTTP' for Handler
    3. Click on 'Save End Point' on the left side under More Actions which will populate the Properties for the End Point
    4. Until the flows have a destination url, we can only fill in certain fields:
       - HEADERS: "Content-Type: application/json"
       - HTTPMETHOD: POST
    5. Save the End Point


### 8. Link the Publish Channels & Enterprise Services to the PLUSTTRIRIGA External System
 
  - On the External Systems page, search and select PLUSTTRIRIGA. Switch over to the Publish Channels tab. One at a time, click New Row and select the Publish Channel for the just created integrations. Make sure the Publish Channel name matches the End Point and it is Enabled like the below image:
 
 <img src="/Pics/Link-Publish-Channel.png">
 
 
 Once finished, the linked Publish Channels should look like the table below:
 
  Channel | Description | Adaptor | End Point | User Defined | Enabled
  ---|---|---|---|---|---
  PLUSTMXASSETInterface| ASSETS | MAXIMO | PLUSTASSET | Yes | Yes
  PLUSTMXOPERLOCInterface | OPERATION LOCATION | MAXIMO | PLUSTLOCATION | Yes | Yes
  PLUSTMXPERSONInterface | PERSON | MAXIMO | PLUSTPERSON | Yes | Yes
 
  - Save the External System
  - Switch over to the Enterprise Services tab. One at a time, click New Row and select the Enterprise Service for the integrations you just created. It should look similar to the below image:
 
 
 <img src="/Pics/Link-Enterprise-Service.png">
 
 When you have finished, your linked Enterprise Services should look like the table below:
 
  Service | Description | Adaptor | Operation | User Defined | Enabled | Use Continuous Queue?
  ---|---|---|---|---|---|---
  PLUSTMXASSETInterface| ASSETS | MAXIMO | Sync | Yes | Yes | Yes
  PLUSTMXOPERLOCInterface | OPERATION LOCATION | MAXIMO | Sync | Yes | Yes | Yes
  PLUSTMXPERSONInterface | PERSON | MAXIMO | Sync | Yes | Yes | Yes
 
  - Save the External System

### 9a. API Key (Maximo-X)
 
  - First, check to see if the version of Maximo comes with Maximo-X. Navigate to Administration -> Administration and a new tab/window should open with the Maximo-x application. If there is trouble reaching this page or it is not installed, follow 9b in order to create an API key.
  - You should be on a page titled 'Integration'. Click on the tab at the top of the page that says API Keys and click on the button with the blue plus sign that reads 'Add API key'
  -  Select user 'MXINTADM' and click the Add button to generate an API key for this user. Securely store this API key for later use.
 
### 9b. API Key (No Maximo-X)
 
  - Follow the steps in [this documentation](https://www.ibm.com/docs/en/mam/7.6.1.2?topic=components-api-keys) to generate an API key for the user
 
### 10. Integration Controls
 
  - There should be 5 Integration Controls created with the following associations:
 
    Integration Control | MAXIMO Value | External Value | Description | Domain
    ---|---|---|---|---
    PLUSTLOCSTATUS | ACTIVE | ACTIVE | Tririga Location Status mapping for inbound flows | LOCASSETSTATUS
    "" | INACTIVE | REVIEW IN PROGRESS | N/A | N/A
    "" | OPERATING | OPERATING | N/A | N/A
    PLUSTORG | TRIMAIN | IBM | Organization mapping for Tririga | N/A
    "" | TRIRIGA | TRIRIGA | N/A | N/A
    PLUSTORGEN | TRIRIGA | EAGLENA | Tririga Organization mapping for Inbound flow | N/A
    "" | TRIRIGA | IBM | N/A | N/A
    "" | TRIRIGA | MAXIMO ORG | N/A | N/A
    "" | TRIRIGA | TEST | N/A | N/A
    "" | TRIRIGA | TRIRIGA | N/A | N/A
    PLUSTPRIORITY | 1 | High | Priority mapping for Tririga | N/A
    "" | 2 | Medium | N/A | N/A
    "" | 3 | Low | N/A | N/A
    PLUSTSITEEN | TRIMAIN | BEDFORD | Tririga Location mapping for inbound flows | N/A
    "" | TRIMAIN | SPACE 01 | N/A | N/A
    "" | TRIMAIN | TEST | N/A | N/A
    "" | TRIMAIN | TRIMAIN | N/A | N/A |
    
- Once these Integration Controls are created, associate them in both the created Enterprise Services and Publish Channels by using the following two tables

Enterprise Service | Control
--|-- 
PLUSTMXASSETInterface | PLUSTORGEN
"" | PLUSTPRIORITY
"" | PLUSTSITEEN 
PLUSTMXOPERLOCInterface | PLUSTLOCSTATUS
 "" | PLUSTSITEEN
 "" | PLUSTSTATUS
 PLUSTMXPERSONInterface | PLUSTORGEN
 "" | PLUSTSITEEN


Publish Channel | Control
--|--
PLUSTMXASSETInterface | PLUSTPRIORITY
PLUSTMXOPERLOCInterface | N/A
PLUSTMXPERSONInterface | PLUSTORG
 
 
  - Return to the PLUSTTRIRIGA External System. On the left side of the External Systems page, select Setup Integration Controls under 'More Actions' and make sure that all 5 Integration Controls are showing as present.

  </details>
  
 <details><summary><b>TRIRIGA</b></summary>

### Credentials and access to an instance of TRIRIGA are required. This code pattern has been tested with TRIRIGA version 10.6.1 as well as TAS 11.0

1. Download the latest TRIRIGA APIs OM Package from this [TRIRIGA APIs GitHub Repository](https://github.com/IBM/tririga-api/blob/main/README.md). Navigate to Tools > Object Migration and import the OM Package. 
2. The Date Time Format field in the user profile must be in UTC. Navigate to Portfolio > People > My Profile and select the user profile that will be triggering the action. The Date Time Format should be in UTC as shown below.

<img src= "/Pics/UTC.jpeg>

 
  </details>

 <details><summary><b>App Connect</b></summary>

### Access to an instance of App Connect with a deployed instance of a Designer is required. This code pattern has been tested with AppConnect version 3.0

Two accounts need to be created from the 'Catalog' tab in order to connect the applications.

Once all of the connectors have loaded, type in 'http' to find the HTTP Application.
 
<img src="/Pics/App-Connect-Catalog.jpeg">

If this is the first account, select 'Connect' to begin setting up the initial HTTP account. If this is not the first account, make sure to take note if there are any other generic account names present because the number of the one created will depend on what has already been created. App Connect creates an account with a generic name in sequential order (Example: if Account 1 and Account 2 are present, the new account will be Account 3).

See the below table for credentials:

Flow | Account Name | Username | Password | API key | API location | API key name
---|---|---|---|---|---|---
Max -> Tri | mxtririga | Your TRIRIGA Username | Your TRIRIGA Password | N/A | N/A | N/A
Tri -> Max | trimaximo | N/A | N/A | Your Maximo apikey | header | apikey 

Once the account is connected, head back to the HTTP Application on the Catalog page and rename the new account according to the Account Name column in the above table.
  

</details>

These pre-requisites are assuming that all three applications are behind the same firewall. If any of these three applications do not share the same firewall, a secure connection between the applications will need to be established. IBM does provide a product that accomplishes this- Secure Gateway. Learn more about getting started with Secure Gateway [here](https://cloud.ibm.com/docs/SecureGateway?topic=SecureGateway-getting-started-with-sg)

## Step 1 - Select an App Connect Flow for deployment

Locate the .yaml file for the deployment direction (MX2TRI or TRI2MX) based on the system of record and download to the local machine. From the previous example, use the MX2TRI flow in the Person row.

Asset | Maximo | TRIRIGA
---|---|---
Person | [MX2TRI](/docs/MAX2Tririga/PLUSTMXPerson2TRI.yaml) | [TRI2MX](/docs/TRI2Maximo/PLUSTTRIPerson2MX.yaml)
Asset | [MX2TRI](/docs/MAX2Tririga/PLUSTMXAsset2TRI.yaml) | [TRI2MX](/docs/TRI2Maximo/PLUSTTRIAsset2MX.yaml)
Location | [MX2TRI](/docs/MAX2Tririga/PLUSTMXLocation2TRI.yaml) | [TRI2MX](/docs/TRI2Maximo/PLUSTTRISpace2MX.yaml)

Make sure to check the [Mapping Document](/docs/TRIRIGA_Maximo_Field_Mapping-Final.xlsx) that the correct fields are being mapped in the selected flow.


## Step 2 - Import the Selected Flow in App Connect

Follow the below steps to import the flow.

<details><summary><b>Import Steps</b></summary>

From the App Connect Dashboard, click 'New' and select 'Import Flow' from the drop down menu.

<img src="/Pics/App-Connect-Dashboard.jpeg"> 

Either drag and drop or select the flow for import. In this example, the MX2TRI Person flow will be used.

<img src="/Pics/Uploaded_Flow.png" width=300>

The flow should now be uploaded onto the App Connect instance. From this screen navigate using the 'Edit flow' button to see the individual nodes of this flow. Be sure to select the HTTP account that was configured for Maximo to TRIRIGA for the connector. 

<img src="/Pics/Completed_Flow.png">

Click 'Done' on the top right of the screen then click on the three dots in the top right corner and select 'Start API'.

<img src="/Pics/Start_API.png" width=250>

Go to the 'Test' tab once it shows that the flow is 'Running' and select the 'POST' option on the left side of the screen.
 
Click on 'Try It' and grab the url and security credentials from this screen for the next step.
 
<img src="/Pics/AppConnect-Config-Full.jpeg" >
 
 
If your instance of AppConnect is through the cloud, your page will look a bit different
 
The interface is mostly the same, but instead of 'Try It' you'll see a tab called 'Manage'
 
<img src="/Pics/AppConnect-Cloud-Manage-Tab.jpeg" >
 
This page contains a few important pieces of information that you'll need to complete the configuration. First, at the top of the page under 'API Info' you'll see a field called 'Route'
 
<img src="/Pics/AppConnect-Cloud-Route.jpeg" >
 
This is the route for your flow, you'll need to piece this together with the correct path of the flow that you're implementing in order to create the proper flow url.
 
At the bottom of the page is a field called 'Sharing outside of Cloud Foundry organization'.
 
<img src="/Pics/AppConnect-Cloud-Sharing.jpeg" >

Here is where you will create your apikey for authentication. The on-prem set up uses basic authentication but the cloud configuration uses an apikey to authorize access. Click on 'Create API key and documentation link' 
  
Provide a name and it will generate an apikey for you to use with this flow along with a documentation link that looks like the 'Test' page from the on-prem configuration.
 
<img src="/Pics/AppConnect-Cloud-Documentation-Link.jpeg" > 
 
The end of the url at the top of the page should have the path that will complete the route url. Copy the end of the url that begins with 'tri' and add it to the piece gathered earlier. The 6 character string that precedes the path on the documentation should match the last 6 characters of the route piece from before.

</details>

## Step 3 - Configure Maximo and TRIRIGA instances with App Connect urls

Using the credentials from the end of Step 2, populate the instances of Maximo and TRIRIGA with the correct End Points of the recently created flow.

<details><summary><b>Maximo</b></summary>

From the main page of Maximo, click the menu icon on the top left and navigate to Integration -> End Points
 
<img src="/Pics/Maximo-EndPoint-Navigation.jpeg" >
 
Fill in the properties with the url, username, and password from Step 2
 
<img src="/Pics/Maximo-EndPoint-Properties.png" >
 
Click the 'Test' button at the bottom right of the screen and send a simple {"hello":"world"}. With the proper configuration, there will be an expected error that ends with 'Bad Request'. If there is a different error than Bad Request in the Response window, refer to the Troubleshooting section to debug.
 
 
If the instance is based in the cloud, there is a slight difference in authorization.
 
Since the authorization is with an api key, remove the basic authorization values (USERNAME & PASSWORD) and in the 'HEADERS' field add the following after Content-Type: 
 
X-IBM-Client-Id: [apikey from Step 2] 
 
Make sure to separate these two values with a comma. It should look something like this:
 
Content-Type: application/json, X-IBM-Client-Id: [apikey]
 
<img src="/Pics/Cloud-End-Point.png" > 
 
</details>
 
 <details><summary><b>TRIRIGA</b></summary>

From the main page of TRIRIGA, click on Tools -> System Setup -> Integration -> Integration Object.
 
Under the 'Name' column, type in 'apic', and select the integration object that pertains to the record that is getting sent. 
 
<img src="/Pics/TRIRIGA-EndPoint.png">
 
Click on the object and fill in the credentials in the pop-up box.
 
</details>

## Step 4 - Configure WebSphere Certificates

This step makes a test connection to a Secure Sockets Layer (SSL) port and retrieves the signer from the server during the handshake.

<details><summary><b>Websphere</b></summary>

Login to the Websphere console that is hosting the Maximo server

<img src="/Pics/Websphere-Home.png">

Click on Security -> SSL certificate & key management. Under 'Related Items' click on 'Key stores and certificates'

<img src="/Pics/Websphere-Keystores.png">

Click on 'CellDefaultTrustStore' and on the next page under 'Additional Properties' click on 'Signer certificates'. 

<img src="/Pics/Websphere-Signercerts.png">

From this page, click on the button that says 'Retrieve from Port' and fill in the required fields using the table below

Field | Value
---|---
Host | The host from the url in Step 2
Port | 443
Alias | appconnect

Once all three have been entered in, click 'Retrieve signer information' and the information from the url will populate on screen. Click save in the box at the top and then repeat the process for 'NodeDefaultTrustStore'
 
 </details>

## Step 5 - Test the Flow

With these 4 steps completed, test the flow with a payload.

Head back to the 'Try It' page in the deployed flow (or the documentation page if cloud-based) and scroll down to the bottom of the page under Parameters.

<img src="/Pics/App-Connect-Test.jpeg" >

In here, fill in mxUrl and triUrl with the urls for the respective applications, generate a test payload to send through the flow and monitor if the information populates in the desired application.

If there is a response other than 200 from the Test, refer to Troubleshooting.

## References
[Mapping Document](/docs/TRIRIGA_Maximo_Field_Mapping-Final.xlsx)

## Troubleshooting

<details><summary><b>Common errors that arise from App Connect</b></summary>

 
 ## Testing the whole flow
 
 - If there is a 404 Not Found error when trying to test the flow, this could mean that the flow is not running. Double check to make sure the flow shows a green dot and says 'Running' after edits have been made.
 
 - If there is a 400 Bad Request error when testing the flow, this could mean the wrong account configurations. Double check to make sure the Accounts from the App Connect pre-requisite section are correct.
 
</details>
<details><summary><b>Common errors that arise from Maximo</b></summary>
 
 ## Testing the End Point
 
 - If an error reads "The response code received from the HTTP request from the endpoint is not successful.", this is related to the configured End Point. Double check and make sure the values in 'End Points' are correct. Make sure there are no accidental spaces at the beginning or end in the event of the values being copy/pasted.
 
 - If an error comes back as a 'PKSync error', this is related to the certificates in WebSphere. Double check and confirm that the certificates from Step 4 are correctly configured. 
 
</details>

<details><summary><b>Common errors that arise from TRIRIGA</b></summary>
 
 - Clear OSLC Cache in TRIRIGA Admin Console in case the integrations do not work in intended manner.
 
</details>
