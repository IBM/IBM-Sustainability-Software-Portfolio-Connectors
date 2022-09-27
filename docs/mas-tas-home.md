# MAS-TAS Portfolio Data Integrations

## Overview

Portfolio Data is able to be synced bi-directionally across Maximo & TRIRIGA with the use of these integrations. The data that is supported at this time includes People, Assets, Locations/Spaces, Service Requests, and Work Orders/Work Tasks. After configuring the environments with the proper prerequisites, please select the desired connector below and follow the instructions to implement the integration.

## Prerequisites

These code patterns all require initial configuration of an instance of Maximo, App Connect, and TRIRIGA. These pre-requisites are also assuming that all three applications are behind the same firewall. If any of these three applications do not share the same firewall, a secure connection between the applications will need to be established. IBM does provide a product that accomplishes this- Secure Gateway. Learn more about getting started with Secure Gateway [here](https://cloud.ibm.com/docs/SecureGateway?topic=SecureGateway-getting-started-with-sg)


See below for the pre-requisites of each system:

### Maximo

** Credentials and access to an instance of Maximo as well as the WebSphere application which hosts Maximo are required. This code pattern has been tested on Maximo 7.6.1.2 as well as MAS 8.6 ** 
 
The following steps and pre-requisites are done against a Maximo demo database. The naming conventions may slightly differ from this, but these are the necessary components. 

Within Maximo, configure your instance to be ready to receive records from TRIRIGA. If these pre-requisites are not completed, the action will not be recorded.

#### 1. Create an Organization named TRIRIGA
 
  - Navigate to the 'Organizations' page and click the blue + button on the top row.
  - Fill in the Organization name with TRIRIGA and the description as "TRIRIGA Organization".
  - Fill in the remaining required fields as such
    1. Base Currency 1: USD
    2. Item Set: SET1
    3. Company Set: COMPSET1
    4. Default Item Status: PENDING
    5. Default Stock Category: STK
  - Click Save Organization on the left side of the screen under Common Actions. This will be set to Active later once there is a clearing account.

#### 2. Create a Testing clearing account in Chart of Accounts
  
  - Navigate to Financial -> Chart of Accounts and click on the previously created TRIRIGA org in the Organizations table. Currently, there should be no GL Accounts for TRIRIGA present
  - Click 'GL Component Maintenance' on the left side under More Actions and add a New Row with the following values:
    1. GL Component Value: 1001
    2. Description: Testing
    3. Active?: Yes
  - Click OK. Click New Row under GL Accounts for TRIRIGA and click the magnifying glass to search for that GL Component. Select it and it should populate in the GL Account & Description fields. The Active Date field should auto populate to the current date.
 - Now that this account is present, head back to Organizations and update the TRIRIGA organization to show the just created Clearing Account, tick the Active box, and click Save Organization.
 
#### 3. Create a site TRIMAIN and set it to active 
 
  - On the Organization page, click on the 'Sites' tab at the top of the page.
  - Click New Row under 'Sites' and enter TRIMAIN for Site and "MAIN Site" for Description. Set the site to Active.
  - Click Save Organization.

#### 4. Create the PLUSTTRIRIGA External System

  - Navigate to Integration -> External Systems and click on the blue plus button at the top of the page.
  - Under the System name fill in PLUSTTRIRIGA and in the Description fill in "To integrate Maximo with TRIRIGA"
  - Enable the System and then fill in the Queues on the right hand side as follows:
    1. Outbound Sequential Queue: jms/maximo/int/queues/sqout
    2. Inbound Sequential Queue: jms/maximo/int/queues/sqin
    3. Inbound Continuous Queue: jms/maximo/int/queues/cqin
  - Save the External System

#### 5. Create the Publish Channels for each integration
 
 
 
 <img alt="Publish Channel Config" src="https://media.github.ibm.com/user/348712/files/ea935400-3801-11ed-9326-b40304b0b7a4">
 

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

#### 6. Create the Enterprise Services for each integration
 
 
 
 <img alt="Enterprise Service Config" src="https://media.github.ibm.com/user/348712/files/e7986380-3801-11ed-8cde-1e6a47b09a0c">
 
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

#### 7. Create the End Points for each integration 
 
 
 <img src="https://media.github.ibm.com/user/348712/files/e6ffcd00-3801-11ed-8aee-523e5de087c4" alt="End Point Config">
 
 
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


#### 8. Link the Publish Channels & Enterprise Services to the PLUSTTRIRIGA External System
 
  - On the External Systems page, search and select PLUSTTRIRIGA. Switch over to the Publish Channels tab. One at a time, click New Row and select the Publish Channel for the just created integrations. Make sure the Publish Channel name matches the End Point and it is Enabled like the below image:
 
 <img src="https://media.github.ibm.com/user/348712/files/e830fa00-3801-11ed-879f-6e9ad68dde4a" alt="Link Publish Channel">
 
 
 Once finished, the linked Publish Channels should look like the table below:
 
  Channel | Description | Adaptor | End Point | User Defined | Enabled
  ---|---|---|---|---|---
  PLUSTMXASSETInterface| ASSETS | MAXIMO | PLUSTASSET | Yes | Yes
  PLUSTMXOPERLOCInterface | OPERATION LOCATION | MAXIMO | PLUSTLOCATION | Yes | Yes
  PLUSTMXPERSONInterface | PERSON | MAXIMO | PLUSTPERSON | Yes | Yes
 
  - Save the External System
  - Switch over to the Enterprise Services tab. One at a time, click New Row and select the Enterprise Service for the integrations you just created. It should look similar to the below image:
 
 
 <img src="https://media.github.ibm.com/user/348712/files/e7986380-3801-11ed-848e-2440c90fb049" alt="Link Enterprise Service">
 
 When you have finished, your linked Enterprise Services should look like the table below:
 
  Service | Description | Adaptor | Operation | User Defined | Enabled | Use Continuous Queue?
  ---|---|---|---|---|---|---
  PLUSTMXASSETInterface| ASSETS | MAXIMO | Sync | Yes | Yes | Yes
  PLUSTMXOPERLOCInterface | OPERATION LOCATION | MAXIMO | Sync | Yes | Yes | Yes
  PLUSTMXPERSONInterface | PERSON | MAXIMO | Sync | Yes | Yes | Yes
 
  - Save the External System

#### 9a. API Key (Maximo-X)
 
  - First, check to see if the version of Maximo comes with Maximo-X. Navigate to Administration -> Administration and a new tab/window should open with the Maximo-x application. If there is trouble reaching this page or it is not installed, follow 9b in order to create an API key.
  - You should be on a page titled 'Integration'. Click on the tab at the top of the page that says API Keys and click on the button with the blue plus sign that reads 'Add API key'
  -  Select user 'MXINTADM' and click the Add button to generate an API key for this user. Securely store this API key for later use.
 
#### 9b. API Key (No Maximo-X)
 
  - Follow the steps in [this documentation](https://www.ibm.com/docs/en/mam/7.6.1.2?topic=components-api-keys) to generate an API key for the user
 
#### 10. Integration Controls
 
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

### App Connect

** Access to an instance of App Connect with a deployed instance of a Designer is required. This code pattern has been tested with AppConnect version 3.0 **

Two accounts need to be created from the 'Catalog' tab in order to connect the applications.

Once all of the connectors have loaded, type in 'http' to find the HTTP Application.
 
<img src="https://media.github.ibm.com/user/348712/files/e23b1900-3801-11ed-8ab4-a5234d64325d" alt="App Connect Catalog">

If this is the first account, select 'Connect' to begin setting up the initial HTTP account. If this is not the first account, make sure to take note if there are any other generic account names present because the number of the one created will depend on what has already been created. App Connect creates an account with a generic name in sequential order (Example: if Account 1 and Account 2 are present, the new account will be Account 3).

See the below table for credentials:

Flow | Account Name | Username | Password | API key | API location | API key name
---|---|---|---|---|---|---
Max -> Tri | mxtririga | Your TRIRIGA Username | Your TRIRIGA Password | N/A | N/A | N/A
Tri -> Max | trimaximo | N/A | N/A | Your Maximo apikey | header | apikey 

Once the account is connected, head back to the HTTP Application on the Catalog page and rename the new account according to the Account Name column in the above table.
### TRIRIGA

Navigate to Tools > Object Migration and import the latest OM Package found [here](https://github.com/IBM/tririga-api/tree/main/docs/ompackages).


The Date Time Format field in the user profile must be in UTC. Navigate to Portfolio > People > My Profile and select the user profile that will be triggering the action. The Date Time Format should be in UTC as shown below.

<img width="1792" alt="TRI-UTC" src="https://media.github.ibm.com/user/348712/files/ec5d1780-3801-11ed-8f32-72fc453558f0">

## Connectors

Name | Link
-- | --
Portfolio Data | [Code Pattern](./mas-tas.md)
Service Request | [Code Pattern](./service-request.md)
Work Order | [Code Pattern](./work-order.md)