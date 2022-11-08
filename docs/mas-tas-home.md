# TRIRIGA - Maximo Connector

The **TRIRIGA - Maximo Connector** is released in TAS under the name *"TAS Connector for Maximo"* and in MAS under the name *"MAS Connector for TRIRIGA"*. 
This connector supports the following capabilities through a collection of App Connect flows:

- Bi-directional loading, and continuous synchronization of portfolio data such as People, Locations and Assets.
- Bi-directional routing of service requests between TRIRIGA and Maximo
- Automatic shadowing of TRIRIGA Work Tasks with Maximo Work Orders to enable Facility teams to do planing, budgeting and project management in TRIRIGA while work tasks get reflected in Maximo as Work Orders for execution.

***
## Use case examples
   
   1. TRIRIGA is the system of record for People and Space data while Maximo is the system of record for Assets. The Maintenance team wants consistent reports across the entire organization and more streamlined operations. The Spaces flows provided can be used to bulk-load all Spaces data from TRIRIGA into Maximo Locations, and then continuosly keep the data in sync as it gets updated in TRIRIGA by a Facility Management team. This can apply to Assets, or People data in both directions. 
   2. Employees use TRIRIGA Request Central to submit a Service Request for a broken elevator, since elevators are maintained in Maximo, the flow will create a corresponding Service Request in Maximo that will be assigned to technicians and resolved. As the Maximo Service Request gets updated, the changes are automatically reflected in the TRIRIGA Service request so the originator of the request remains informed of its status. This flow can operate in reverse as well if a Service Request originates from Maximo against a maintenance operation that should be tracked in TRIRIGA.
   3. Facility Management team have a capital project in TRIRIGA to upgrade all the lights to LEDs. The project has the budget,  scope and tasks already defined. The flow can then create corresponding Work Orders in Maximo for each task to get executed by technicians using the Maximo Manage and Mobile applications. As the tasks get updated in Maximo, those updates flow back into TRIRIGA to reflect in the project plan.

***

## Connector architecture
**IBM App Connect** provides a flexible environment for integration solutions to transform, enrich, route, and process business messages and data. 

**App Connect Flows** enable specific integration use cases by connecting to predefined APIs to route and map data. Mapping has been pre-defined, but it can be customized.

**Native API framework** is used for Maximo and TRIRIGA and enabled thorugh provided packages that can be imported.




<img width="1280" alt="TRI-MAS Architecture" src="https://user-images.githubusercontent.com/20128676/199506541-1f7b96cd-a758-4011-802a-c1ea1a46e60d.png">

***

## Data Mapping

The image below illustrates the record types that are available the APIs and App Connect Flows Provided.



<img width="1280" alt="TRI-MAS Architecture" src="https://user-images.githubusercontent.com/20128676/199506793-e96635d7-4a18-4f4b-97df-633374c334c5.png">

***

## App Connect Flows

Included with this connector are 20+ flows that map TRIRIGA records to Maximo, and vice versa, along with all the required fields they contain. The table below shows all the flows you can use in differnet combinations to implement your desired integration use cases. For a more detailed mapping of all the fields please review the attached [data mapping spreadsheet](site/Resources/TRIRIGA_Maximo_Field_Mapping-Final.xlsx). 

File | Flow | Destination | Operation
-- | -- |--|--
PLUSITRIPerson2MX_v1_0_0.yaml | People | TRI to Max | Single Record at a time
PLUSITRIOrg2MX_v1_0_0.yaml | People | TRI to Max | Single Record at a time
PLUSIMXPerson2TRI_v1_0_0.yaml | People | Max to TRI | Batch & Individual Records
 |  |  | 
PLUSITRISpace2MX_v1_0_0.yaml | Spaces/Locations | TRI to Max | Single Record at a time
PLUSITRISpaceClass2MX_v1_0_0.yaml | Spaces/Locations | TRI to Max | Single Record at a time
PLUSITRILocPath2MX_v1_0_0.yaml | Spaces/Locations | TRI to Max | Single Record at a time
PLUSIMXLocation2TRI_v1_0_0.yaml | Spaces/Locations | Max to TRI | Batch & Individual Records
 |  |  | 
PLUSITRIAsset2MX_v1_0_0.yaml | Assets | TRI to Max | Single Record at a time
PLUSITRIAssetSpec2MX_v1_0_0.yaml | Assets | TRI to Max | Single Record at a time
PLUSIMXAsset2TRI_v1_0_0.yaml | Assets | Max to TRI | Batch & Individual Records
 |  |  | 
PLUSITRIServiceReq2MX_v1_0_0.yaml | Service Requests | TRI to Max | Single Record at a time
PLUSITRIReqClass2MX_v1_0_0.yaml | Service Requests | TRI to Max | Single Record at a time
PLUSIMXServiceReq2TRI_v1_0_0.yaml | Service Requests | Max to TRI | Single Record at a time
 |  |  | 
PLUSITRIWorkOrder2MX_v1_0_0.yaml | Tasks / Work Orders | TRI to Max | Single Record at a time
PLUSIMXWorkOrder2TRI_v1_0_0.yaml | Tasks / Work Orders | Max to TRI | Single Record at a time
 |  |  | 
PLUSITRIPEOPLEBATCH_v1_0_0.yaml | Batch People | TRI to Max | Batch
PLUSITRIORGANIZATIONBATCH_v1_0_0.yaml | Batch Org | TRI to Max | Batch
PLUSITRISPACEBATCH_v1_0_0.yaml | Batch Space | TRI to Max | Batch
PLUSITRIPROPERTYBATCH_v1_0_0.yaml | Batch Property | TRI to Max | Batch
PLUSITRIBUILDINGBATCH_v1_0_0.yaml | Batch Building | TRI to Max | Batch
PLUSITRIFLOORBATCH_v1_0_0.yaml | Batch Floors | TRI to Max | Batch
PLUSITRIASSETBATCH_v1_0_0.yaml | Batch Assets | TRI to Max | Batch

***

# Installation & Configuration Guide


>## Before you begin you will need:
>
>1. An instance of App Connect Enterprise or App Connect Pro with the Designer component.
>2. Admin access to your Maximo instance with an api key generated for this integration
>3. TRIRIGA with dedicated user/pw for this integration
>4. Secure connection between TRIRIGA, App Connect, and Maximo. If required IBM does provides a product that accomplishes this: Secure Gateway. Learn more about getting started with [Secure Gateway.](https://cloud.ibm.com/docs/SecureGateway?topic=SecureGateway-getting-started-with-sg)
>5. Import AppConnect Cert to Maximo to enable encrypted communication
>6. Import AppConnect Cert to TRIRIGA to enable encrypted communication

***

## Installation Steps Overview
1. Import the flows into App Connect 
2. TRIRIGA OM Package & Configuration
   - Point integration object to App Connect Flow
3. Maximo Configurations
   - Publish channel to point to App Connect Flow
   - Manually add necessary fields
4. Test 
  - MAS outbound connectivity
  - TRIRIGA POSTMAN

***

## Part 1. Installing the Right Flows into App Connect 

### App Connect Authentication

>Access to an instance of App Connect with a deployed instance of a Designer is required. This code pattern has been tested with App Connect version 3.0. 

1. Two accounts need to be created from the **Catalog** tab in order to connect the applications. Once all of the connectors have loaded, type in **http** to find the HTTP Application.
 
2. If this is the first account, select **Connect** to begin setting up the initial HTTP account. If this is not the first account, make sure to take note if there are any other generic account names present because the number of the one created will depend on what has already been created. App Connect creates an account with a generic name in sequential order (Example: if Account 1 and Account 2 are present, the new account will be Account 3).

3. Use the below table to associate the proper credentials for the accounts.
4. Once the account is connected, head back to the HTTP Application on the Catalog page and rename the new account according to the Account Name column in the table below.

Flow | Account Name | Username | Password | API key | API location | API key name
---|---|---|---|---|---|---
Max -> Tri | mxtririga | Your TRIRIGA Username | Your TRIRIGA Password | N/A | N/A | N/A
Tri -> Max | trimaximo | N/A | N/A | Your Maximo apikey | header | apikey 


<img src="https://media.github.ibm.com/user/348712/files/e23b1900-3801-11ed-8ab4-a5234d64325d" alt="App Connect Catalog">

*App Connect catalog*

### Importing App Connect Flows

...

## Part 2. TRIRIGA Configuration

> **Note** If you have not already done so, please import AppConnect Cert to TRIRIGA to enable encrypted communication. For instructions on how to do that please go here. 


1. Download the latest [OM Package](https://github.com/IBM/tririga-api/tree/main/docs/ompackages).
2. Navigate to **Tools -> Object Migration** and import the package into the instance of TRIRIGA.
3. The Date Time Format field in the user profile must be in UTC. Navigate to **Portfolio -> People -> My Profile** and select the user profile that will be triggering the action. The Date Time Format should be in UTC as shown below.

<img width="600" alt="TRI-UTC" src="https://media.github.ibm.com/user/348712/files/ec5d1780-3801-11ed-8f32-72fc453558f0">

*Date Time Format Field in My Profile*


***


## Part 3. Maximo Configuration

> **Note 1** If you have not already done so, please import AppConnect Cert to Maximo to enable encrypted communication. 


>**Note 2**: The following steps and prerequisites are done against a Maximo demo database. The naming conventions may slightly differ from this, but these are the necessary components.

Within Maximo, configure your instance to be ready to receive records from TRIRIGA. If these pre-requisites are not completed, the action will not be recorded.

#### 1. Create an Organization named TRIRIGA
 
 
a. Navigate to the **Organizations** page and click the blue + button on the top row.

b. Fill in the Organization name with TRIRIGA and the description as "TRIRIGA Organization".

c. Fill in the remaining required fields as such:
  
  Field Name | Value
  -- | --
  Base Currency 1 |<b>USD</b>
  Item Set | **SET1** 
  Company Set| **COMPSET1** 
  Default Item Status| **PENDING**
  Default Stock Category| **STK**
 
  
d. Click **Save Organization** on the left side of the screen under Common Actions. This will be set to Active later once there is a clearing account.
  

#### 2. Create a Testing clearing account in Chart of Accounts
  
**Un-require the GL Account fields**

a. Navigate to **Financial** -> **Chart of Accounts** and click on the previously created TRIRIGA org in the Organizations table. Currently, there should be no GL Accounts for TRIRIGA present. <br>

b. Click **GL Component Maintenance** on the left side under More Actions and add a New Row with the following values:

Field Name | Value
-- | --
GL Component Value| **1001**
Description| **Testing**
Active?| **Yes**

c. Click **OK**. Click **New Row** under GL Accounts for TRIRIGA and click the magnifying glass to search for that GL Component. Select it and it should populate in the GL Account and Description fields. The Active Date field should auto populate to the current date.<br>

d. Now that this account is present, head back to **Organizations** and update the TRIRIGA organization to show the just created Clearing Account, tick the Active box, and click **Save Organization**.<br>
 
#### 3. Create a site TRIMAIN and set it to active 
 
  a. On the Organization page, click on the **Sites** tab at the top of the page.

  b. Click **New Row** under **Sites** and enter TRIMAIN for Site and MAIN Site for Description. Set the site to Active.

  c. Click **Save Organization**.

#### 4. API Key
 
*Check to see if the version of Maximo comes with Maximo-X.* 

**a. Maximo-X<br>**
  - Navigate to **Administration -> Administration** and a new tab/window should open with the Maximo-X application.<br>
  - You should be on a page titled 'Integration'. Click on the tab at the top of the page that says API Keys and click on the button with the blue plus sign that reads **Add API key**<br>
  - Select user **MXINTADM** and click the **Add** button to generate an API key for this user. Securely store this API key for later use.

**b. No Maximo-X<br>**
  - Follow the steps in [this documentation](https://www.ibm.com/docs/en/mam/7.6.1.2?topic=components-api-keys) to generate an API key for the user
 
#### 5. Integration Controls
 
  a. Head to **Enterprise Services** and click on **Create Integration Controls**. There should be 5 X-Ref Integration Controls created with the following associations:
 
  Integration Control | MAXIMO Value | External Value | Description | Domain
  ---|---|---|---|---
  PLUSILOCSTATUS | ACTIVE | ACTIVE | Tririga Location Status mapping for inbound flows | LOCASSETSTATUS
  "" | INACTIVE | REVIEW IN PROGRESS | N/A | N/A
  "" | OPERATING | OPERATING | N/A | N/A
  PLUSIORG | TRIMAIN | IBM | Organization mapping for Tririga | N/A
  "" | TRIRIGA | TRIRIGA | N/A | N/A
  PLUSIORGEN | TRIRIGA | EAGLENA | Tririga Organization mapping for Inbound flow | N/A
  "" | TRIRIGA | IBM | N/A | N/A
  "" | TRIRIGA | MAXIMO ORG | N/A | N/A
  "" | TRIRIGA | TEST | N/A | N/A
  "" | TRIRIGA | TRIRIGA | N/A | N/A
  PLUSIPRIORITY | 1 | High | Priority mapping for Tririga | N/A
  "" | 2 | Medium | N/A | N/A
  "" | 3 | Low | N/A | N/A
  PLUSISITEEN | TRIMAIN | BEDFORD | Tririga Location mapping for inbound flows | N/A
  "" | TRIMAIN | SPACE 01 | N/A | N/A
  "" | TRIMAIN | TEST | N/A | N/A
  "" | TRIMAIN | TRIMAIN | N/A | N/A |
    
b. Once these Integration Controls are created, associate them in both the created Enterprise Services and Publish Channels by using the following two tables

Enterprise Service | Control
--|-- 
PLUSIMXASSETInterface | PLUSIORGEN
"" | PLUSIPRIORITY
"" | PLUSISITEEN 
PLUSIMXOPERLOCInterface | PLUSILOCSTATUS
 "" | PLUSISITEEN
 "" | PLUSISTATUS
 PLUSIMXPERSONInterface | PLUSIORGEN
 "" | PLUSISITEEN


Publish Channel | Control
--|--
PLUSIMXASSETInterface | PLUSIPRIORITY
PLUSIMXOPERLOCInterface | N/A
PLUSIMXPERSONInterface | PLUSIORG
 
c. Return to the PLUSITRIRIGA External System. On the left side of the External Systems page, select **Setup Integration Controls** under **More Actions** and make sure that all 5 Integration Controls are showing as present.

#### 6. Enable Object Structure Security

The user needs to be able to transact against the specific object structure in Manage. Navigate to Object Structures and search for MXPERSON. On the left side of the MXPERSON screen select **Configure Object Structure Storage** and turn on the button underneath **Use Object Structure for Authorization Name?**

<img width="1788" alt="Configure Object Security" src="https://media.github.ibm.com/user/348712/files/f1affe00-48a5-11ed-8488-a41686aafba5">

*Configure Object Security Screen*

The structure should save. Complete this process for the following objects:

  - MXASSET
  - MXOPERLOC
  - MXSR
  - MXWO
  - MXDOMAIN

Next, navigate to **Security -> Security Groups** and select the security group the current user is apart of. Click the **Object Structures** tab and filter search for MXPERSON. Once selected, click **Grant Listed Options for This Object Structure** and Save the Group. 

<img width="1785" alt="Security Group Page" src="https://media.github.ibm.com/user/348712/files/f2489480-48a5-11ed-8833-fc3297fb7881">

*Security Group Page*

Repeat this process for the other changed structures.


***



### **Reference for Pre-requisite**

### **Pre-requisite: add an App Connect Certificate in MAS 8.9+**

1. Extract the App Connect certificate from an imported flow URL. Navigate to the flow's page and click on **Test** and then **Try It** to get the proper url.

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

5. Once all three have been entered in, click **Retrieve signer information** and the information from the url will populate on screen. Click **Save** in the box at the top and then repeat the process for **NodeDefaultTrustStore**.

<img width="600" alt="Websphere-Home" src="https://media.github.ibm.com/user/348712/files/ee26db00-3801-11ed-8365-8604ad0a66df">

*Step 2: WebSphere Homepage*

<img width="600" alt="Websphere-Keystores" src="https://media.github.ibm.com/user/348712/files/eebf7180-3801-11ed-9253-85d4e8bed0ea">

*Step 3: Websphere Keystores*

<img width="600" alt="Websphere-Signercerts" src="https://media.github.ibm.com/user/348712/files/f121cb80-3801-11ed-9833-65fe541bcef5">
 
*Step 4: Websphere Signer Certs*


