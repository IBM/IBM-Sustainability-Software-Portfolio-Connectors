# TRIRIGA - Maximo Connector

The **TRIRIGA - Maximo Connector** is released in TAS under the name *"TAS Connector for Maximo"* and in MAS under the name *"MAS Connector for TRIRIGA"*. 
This connector supports the following capabilities through a collection of App Connect flows:

- Bi-directional loading, and continuous synchronization of portfolio data such as People, Locations and Assets.
- Bi-directional routing of service requests between TAS and Maximo
- Automatic shadowing of TAS Work Tasks with Maximo Work Orders to enable Facility teams to do planing, budgeting and project management in TAS while work tasks get reflected in Maximo as Work Orders for execution.

***
## Use case examples
   
   1. TAS is the system of record for People and Space data while Maximo is the system of record for Assets. The Maintenance team wants consistent reports across the entire organization and more streamlined operations. The Spaces flows provided can be used to bulk-load all Spaces data from TAS into Maximo Locations, and then continuously keep the data in sync as it gets updated in TAS by a Facility Management team. This can apply to Assets, or People data in both directions. 
   2. Employees use Request Central to submit a Service Request for a broken elevator. Since elevators are maintained in Maximo, the flow will create a corresponding Service Request in Maximo that will be assigned to technicians and resolved. As the Maximo Service Request gets updated, the changes are automatically reflected in the TAS Service request so the originator of the request remains informed of its status. This flow can operate in reverse as well if a Service Request originates from Maximo against a maintenance operation that should be tracked in TAS.
   3. Facility Management team have a capital project in TAS to upgrade all the lights to LEDs. The project has the budget, scope and tasks already defined. The flow can then create corresponding Work Orders in Maximo for each task to get executed by technicians using the Maximo Manage and Mobile applications. As the tasks get updated in Maximo, those updates flow back into TAS to reflect in the project plan.

***

## Connector architecture
**IBM App Connect** provides a flexible environment for integration solutions to transform, enrich, route, and process business messages and data. 

**App Connect Flows** enable specific integration use cases by connecting to predefined APIs to route and map data. Mapping has been pre-defined, but it can be customized.

**Native API framework** is used for Maximo and TAS and enabled thorugh provided packages that can be imported.




<img width="1280" alt="TRI-MAS Architecture" src="https://user-images.githubusercontent.com/20128676/199506541-1f7b96cd-a758-4011-802a-c1ea1a46e60d.png">

***

## Data Mapping

The image below illustrates the record types that are available the APIs and App Connect Flows Provided.



<img width="1280" alt="TRI-MAS Architecture" src="https://user-images.githubusercontent.com/20128676/199506793-e96635d7-4a18-4f4b-97df-633374c334c5.png">

***

## App Connect Flows

Included with this connector are 20+ flows that map TAS records to Maximo, and vice versa, along with all the required fields they contain. The table below shows all the flows you can use in differnet combinations to implement your desired integration use cases. For a more detailed mapping of all the fields please review the [data mapping spreadsheet](https://github.com/IBM/IBM-Sustainability-Software-Portfolio-Connectors/blob/TRIMAS-Updates/site/Resources/TRIRIGA_Maximo_Field_Mapping-Final.xlsx). 

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

## Installation & Configuration Guide

>### Before you begin you will need:
>
>1. An instance of App Connect Enterprise or App Connect Pro with the Designer component.
>2. Admin access to your Maximo instance with an API key generated for this integration. Storage will need to be properly configured to store the images as they are coming in. Refer to the [Doclinks](https://www.ibm.com/docs/en/maximo-manage/continuous-delivery?topic=documents-configuring-system-properties-persistent-storage) setup page
>3. TRIRIGA with dedicated user/pw for this integration
>4. Secure connection between TRIRIGA, App Connect, and Maximo. If required IBM does provides a product that accomplishes this: Secure Gateway. Learn more about getting started with [Secure Gateway.](https://cloud.ibm.com/docs/SecureGateway?topic=SecureGateway-getting-started-with-sg)
>5. [Import AppConnect Cert to Maximo](#pre-requisite-add-an-app-connect-certificate-in-mas-89) to enable encrypted communication
>6. [Import AppConnect Cert to TRIRIGA](#pre-requisite-add-certificate-in-tririga) to enable encrypted communication

## Downloadable Resources

Download the [zip file](https://github.com/IBM/mas-tas-appconnect-flows/releases/tag/v1.0.0) that has all of the flows and configuration files.

***

## Installation Steps Overview
1. Install Connector<br>
2. Configure App Connect<br> 
   a. Authenticate<br>
   b. Import Flows<br>
3. Configure TRIRIGA<br>
   a. Import TRIRIGA OM Package <br>
   b. Point integration object to App Connect Flow<br>
4. Configure Maximo<br>
   a.  Add necessary fields<br>
   b.  Publish channel to point to App Connect Flow<br>
5. Test <br>
   a. MAS outbound connectivity<br>
   b. TRIRIGA POSTMAN<br>

***

## Part 1. Install Connector

### MAS 8.10+
The MAS Connector to TAS in Manage can be installed on the MAS administration dashboard through the tile. 

1. Search for TRIRIGA from the Catalog page and select the **TRIRIGA Connector** tile.
2. On the tile page, click **Configure** and it will take you to the update configuration page. Select the latest version of the connector under the **Components** section and then click **Apply Changes** at the top of the page and **Confirm** on the next window.

<img src="https://media.github.ibm.com/user/348712/files/aff7df19-4804-4407-9994-bb82fedd1c6e" alt="Tile dashboard">
*Step 1: Select Connector Tile*

<img src="https://media.github.ibm.com/user/348712/files/9c550273-474a-41b4-bf9b-b266c90014ac" alt="Component Section">
*Step 2: Apply Changes for Connector*

The process can take anywhere between 20 minutes to 1-2 hours depending on the cluster resources. Monitor the logs in the created `build` pods in the Manage namespace in the OpenShift cluster to see the progress.

### MAS 8.8 & 8.9

The MAS Connector to TAS in Manage can be installed by using the proper customization .zip file provided from Passport Advantage. This .zip file will have all of the configurations necessary to complete step 4 in the Installation process.

1. Navigate to the Admin dashboard of the instance of Manage within MAS.
2. Select the workspace the instance is deployed on and update the configuration. 
3. Scroll down to **Customization** and link to the location of the .zip file in the field ([Additional details](https://www.ibm.com/docs/en/maximo-manage/continuous-delivery?topic=manage-setting-customizations) on setting customizations)
4. Click **Apply Changes** and Manage will update the instance with the customization.

The process can take anywhere between 20 minutes to 1-2 hours depending on the cluster resources. Monitor the logs in the created `build` pods in the Manage namespace in the OpenShift cluster to see the progress.

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

## Part 2. Configure App Connect

### App Connect Authentication

*Note: IBM App Connect Professional or Enterprise is needed to run these flows. The flows have been tested on IBM Cloud App Connect, AWS App Connect, as well as the containerized version of App Connect.*


1. Two accounts need to be created from the **Catalog** tab in order to connect the applications. Once all of the connectors have loaded, type in **http** to find the HTTP Application.
 
2. If this is the first account, select **Connect** to begin setting up the initial HTTP account. If this is not the first account, make sure to take note if there are any other generic account names present because the number of the one created will depend on what has already been created. App Connect creates an account with a generic name in sequential order (Example: if Account 1 and Account 2 are present, the new account will be Account 3).

3. Use the below table to associate the proper credentials for the accounts.
4. Once the account is connected, head back to the HTTP Application on the Catalog page and rename the new account according to the Account Name column in the table below.

Flow | Account Name | Username | Password | API key | API location | API key name
---|---|---|---|---|---|---
Max -> Tri | mxtririga | Your TAS Username | Your TAS Password | N/A | N/A | N/A
Tri -> Max | trimaximo | N/A | N/A | Your Maximo apikey | header | apikey 


<img src="https://media.github.ibm.com/user/348712/files/e23b1900-3801-11ed-8ab4-a5234d64325d" alt="App Connect Catalog">
*App Connect catalog*

### Importing App Connect Flows

1. Locate the .yaml files for the desired flows from the provided zip.
2. Import the Selected Flows into **App Connect Enterprise**, or **App Connect SaaS on IBM Cloud** using the instructions below


#### Import Steps - App Connect Enterprise (CP4I) & App Connect SaaS (AWS)

  1. From the App Connect Dashboard, click **New** and select **Import Flow** from the drop down menu.
  2. Either drag and drop or select the flow for import. In this example, the MX2TRI Person flow will be used.
  3. The flow should now be uploaded onto the App Connect instance. From this screen navigate using the **Edit flow** button to see the individual nodes of this flow. Be sure to select the HTTP account that was configured for Maximo to TRIRIGA for the connector. 
  4. Click **Done** on the top right of the screen then click on the three dots in the top right corner and select **Start API**.
  5. Go to the **Test** tab once it shows that the flow is **Running** and select the **POST** option on the left side of the screen. Click on **Try It** and grab the URL and security credentials from this screen for the next step.
  
  <img src="https://media.github.ibm.com/user/348712/files/e23b1900-3801-11ed-9bf0-334b0f78fb60" alt="App Connect Dashboard"> 
  *Step 1: App Connect Dashboard*

  <img width="300" alt="Uploaded_Flow" src="https://media.github.ibm.com/user/348712/files/ecf5ae00-3801-11ed-886a-b5bb80a59523">
  *Step 2: Import a Flow* 

  <img width="1771" alt="Completed_Flow" src="https://media.github.ibm.com/user/348712/files/e6673680-3801-11ed-9883-661b31bad8b5">
  *Step 3: Completed Flow*

  <img width="251" alt="Start_API" src="https://media.github.ibm.com/user/348712/files/ea935400-3801-11ed-826c-318a039b6bec">
  *Step 4: Start Flow*
  
  <img src="https://media.github.ibm.com/user/348712/files/e49d7300-3801-11ed-83f4-a6fd5e41a3da" alt="App Connect Config Full" >
  *Step 5: Try It*

#### Import Steps - App Connect SaaS (IBM Cloud)
  1. If your instance of AppConnect is through the cloud, your page will look a bit different.
  2. The interface is mostly the same, but instead of **Try It** you'll see a tab called **Manage**. This page contains a few important pieces of information that you'll need to complete the configuration. First, at the top of the page under **API Info** you'll see a field called **Route**. Piece this together with the correct path of the flow that you're implementing in order to create the proper flow URL.
  3. Collect the API key at the bottom of the page from a field called **Sharing outside of Cloud Foundry organization**. Click on **Create API key and documentation link**. Provide a name and it will generate an apikey for you to use with this flow along with a documentation link that looks like the **Test** page from the on-prem configuration.
  4. The end of the URL at the top of the page should have the path that will complete the route URL. Copy the end of the URL that begins with **tri** and add it to the piece gathered earlier. The 6 character string that precedes the path on the documentation should match the last 6 characters of the route piece from before.

  
  <img src="https://media.github.ibm.com/user/348712/files/e36c4600-3801-11ed-8744-4aa3b105af0d" alt="AppConnect Cloud Manage Tab" >
  *Step 1: Manage Tab*
  
  <img src="https://media.github.ibm.com/user/348712/files/e404dc80-3801-11ed-95de-e332d10a5597" alt="AppConnect Cloud Route" >
  *Step 2: Cloud Route*

  <img src="https://media.github.ibm.com/user/348712/files/e49d7300-3801-11ed-9084-943aa4afe16c" alt="AppConnect Cloud Sharing" >
  *Step 3: API key creation*
  
  <img src="https://media.github.ibm.com/user/348712/files/e2d3af80-3801-11ed-8ff7-4e319d52da99" alt="AppConnect Cloud Documentation Link" > 
  *Step 4: API Documentation Link*


#### Running in Production

Once the configuration of the below applications is complete and the connectivity between systems is working as expected, you will need to deploy the flow as an integration server via the App Connect Dashboard. 

1. From the dashboard view of the Designer application, click the three dots on the flow's tile, select **Export**, and then **Runtime flow asset (BAR)**.

2. In the App Connect Dashboard instance, select **Create Server** and then the type of integration you would like to run. The flows work with all types of integration sizes, so pick the one that is right for your deployment. Import the bar file you just downloaded into the section on the **Integrations** tab.
3. On the Configuration screen, refer to [securing a REST-based integration](https://www.ibm.com/docs/en/app-connect/containers_cd?topic=examples-securing-rest-based-integration-in-app-connect) to properly secure the server.

4. Give the intended details of the server such as the name, version, and correct license to use. Click **Create** when you're ready to deploy. It may take upwards of 5-10 minutes for the server to be up and running.

<img src="https://media.github.ibm.com/user/348712/files/100feeb6-e960-43b0-abc1-ac9b5568a150" alt="Export Bar file" >
*Step 1: Export Bar file*

<img src="https://media.github.ibm.com/user/348712/files/d81a2a41-5d31-4c0b-9b7d-24ff3286810b" alt="Create Integration Server" >
*Step 2: Create Integration Server*

<img src="https://media.github.ibm.com/user/348712/files/6e75ff15-92fc-4829-9198-6fb378c75c3a" alt="Details Page" >
*Step 3: Final details*

## Part 3. Configure TAS

#### Import Object Migration Package

1. Import the OM Package labeled *APIConnector* into the TAS instance. Go to **Tools -> Administration -> Object Migration** and select **New Import Package** to begin the import process.
**Refer to the [Object Migration Overview](https://www.ibm.com/docs/en/tap/3.6.1?topic=objects-object-migration-overview ) for more information on Object Migration**

#### TAS API User Access

In order for App Connect to be able to use TAS APIs, it will need a user with certain permissions. These user's credentials will be configured in App Connect.

1. Create an integration user by following the steps given in [Chapter 2](https://www.ibm.com/docs/en/SSFCZ3_11.2/pdf/pdf_tri_app_admin.pdf). This user should be a non-admin user and not part of the Admin security group.

- Assign that user to a new or existing group for the integration. If you need to create a new security group, follow the steps given in [Chapter 1](https://www.ibm.com/docs/en/SSFCZ3_11.2/pdf/pdf_tri_app_admin.pdf).

- Add the "TAS Base License" to the **License Details** section on the user Profile.

- Select the newly created group or desired existing group and switch to the **Access** tab and add the appropriate access for the integration.

##### Outbound Traffic from TAS
For outbound traffic from TAS, grant at least READ access on the Business Objects that will be used. The table below shows the various supported business Objects the API can pull from: 

Module | Business Object Label
--|--
Asset | [Building Equipment](https://github.com/IBM/tririga-api/blob/main/markdowns/Asset.md)
Classification | [Request Class](https://github.com/IBM/tririga-api/blob/main/markdowns/RequestClass.md)
Classification | [Space Class Current](https://github.com/IBM/tririga-api/blob/main/markdowns/SpaceClass.md)
Classification | [Asset Spec Class](https://github.com/IBM/tririga-api/blob/main/markdowns/AssetSpecClass.md)
People | [People](https://github.com/IBM/tririga-api/blob/main/markdowns/People.md)
Location |[Property](https://github.com/IBM/tririga-api/blob/main/markdowns/Property.md)
Location |[Building](https://github.com/IBM/tririga-api/blob/main/markdowns/Building.md)
Location |[Floor](https://github.com/IBM/tririga-api/blob/main/markdowns/Floor.md)
Location |[Space](https://github.com/IBM/tririga-api/blob/main/markdowns/Space.md)
Organization |[Organization](https://github.com/IBM/tririga-api/blob/main/markdowns/Organization.md)
Request |[Service Request](https://github.com/IBM/tririga-api/blob/main/markdowns/ServiceRequest.md)
Task |[Work Task](https://github.com/IBM/tririga-api/blob/main/markdowns/WorkTask.md)

In the example below, the API user is able to pull data from the Building Business Object: 

<img width="1097" alt="Outbound Business Object" src="https://media.github.ibm.com/user/348712/files/e5633b80-3b58-11ed-81a4-46540898ff50">

##### Inbound traffic to TAS

For inbound traffic, Data Access needs to be enabled as well as Application Access permissions to the **triAPIConnect Module** or the individual Objects. To enable an API user to create a building, grant access to the triAPICBuilding Business object as shown below: 

<img width="1000" alt="Inbound Traffic Business Object" src="https://media.github.ibm.com/user/348712/files/e5633b80-3b58-11ed-842e-cea368ad86dd">

#### Date Time Format

2. The Date Time Format field in the user profile must be in UTC. Navigate to **Portfolio -> People -> My Profile** and select the user profile that will be triggering the action. The Date Time Format should be in UTC as shown below.

<img width="600" alt="TRI-UTC" src="https://media.github.ibm.com/user/348712/files/ec5d1780-3801-11ed-8f32-72fc453558f0">
*Date Time Format Field in My Profile*

#### Configure Integration Object
4. Configure the Integration Object - From the main page of TAS, click on **Tools -> System Setup -> Integration -> Integration Object**. Under the **Name** column, type in **apic**, and select the integration object that pertains to the record that is getting sent. 
5. Click on the object and fill in the credentials in the pop-up box.
 
<img src="https://media.github.ibm.com/user/348712/files/ecf5ae00-3801-11ed-9ea5-af57b4059a25" alt="TAS End Point">
*TAS End Point*

***


## Part 4. Maximo Configuration

> **Note 1** If you have not already done so, please import AppConnect Cert to Maximo to enable encrypted communication. 

>**Note 2**: The following steps and prerequisites are done against a Maximo demo database. The naming conventions may slightly differ from this, but these are the necessary components.

Within Maximo, configure your instance to be ready to receive records from TAS. If these pre-requisites are not completed, the action will not be recorded.

#### 1. Create an Organization named TAS
 
a. Navigate to the **Organizations** page and click the blue + button on the top row.<br>
b. Fill in the Organization name with TAS and the description as "TAS Organization".<br>
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

a. Navigate to **Financial** -> **Chart of Accounts** and click on the previously created TAS org in the Organizations table. Currently, there should be no GL Accounts for TAS present. <br>
b. Click **GL Component Maintenance** on the left side under More Actions and add a New Row with the following values:

Field Name | Value
-- | --
GL Component Value| **1001**
Description| **Testing**
Active?| **Yes**

c. Click **OK**. Click **New Row** under GL Accounts for TAS and click the magnifying glass to search for that GL Component. Select it and it should populate in the GL Account and Description fields. The Active Date field should auto populate to the current date.<br>
d. Now that this account is present, head back to **Organizations** and update the TAS organization to show the just created Clearing Account, tick the Active box, and click **Save Organization**.<br>
 
#### 3. Create a site TRIMAIN and set it to active 
 
  a. On the Organization page, click on the **Sites** tab at the top of the page.<br>
  b. Click **New Row** under **Sites** and enter TRIMAIN for Site and MAIN Site for Description. Set the site to Active.<br>
  c. Click **Save Organization**.

#### 4. API Key
 
##### MAS

  - Navigate to **Integration -> API Keys**.<br>
  - Click on the button that reads **Add API key**<br>
  - Select user **MXINTADM** and click the **Add** button to generate an API key for this user. Securely store this API key for later use.

##### Maximo 7.6.1.2

  - Navigate to **Administration -> Administration** and a new tab/window should open with the Maximo-X application.<br>
  - You should be on a page titled 'Integration'. Click on the tab at the top of the page that says API Keys and click on the button with the blue plus sign that reads **Add API key**<br>
  - Select user **MXINTADM** and click the **Add** button to generate an API key for this user. Securely store this API key for later use.
 
#### 5. Integration Controls
 
  a. Head to **Enterprise Services** and click on **Create Integration Controls**. These integration controls help translate specific external values into values Maximo understands. The following tables contain values to set up this demo, but can be customized to fit your TAS and Maximo naming conventions. There should be 5 X-Ref Integration Controls created with the following associations:
 
  Integration Control | MAXIMO Value | External Value | Description | Domain
  ---|---|---|---|---
  PLUSILOCSTATUS | ACTIVE | ACTIVE | TAS Location Status mapping for inbound flows | LOCASSETSTATUS
  PLUSILOCSTATUS | INACTIVE | REVIEW IN PROGRESS | N/A | N/A
  PLUSILOCSTATUS | OPERATING | OPERATING | N/A | N/A
  PLUSIORG | TRIMAIN | IBM | Organization mapping for TAS | N/A
  PLUSIORG | TAS | TAS | N/A | N/A
  PLUSIORGEN | TAS | EAGLENA | TAS Organization mapping for Inbound flow | N/A
  PLUSIORGEN | TAS | IBM | N/A | N/A
  PLUSIORGEN | TAS | MAXIMO ORG | N/A | N/A
  PLUSIORGEN | TAS | TAS | N/A | N/A
  PLUSIPRIORITY | 1 | High | Priority mapping for TAS | N/A
  PLUSIPRIORITY | 2 | Medium | N/A | N/A
  PLUSIPRIORITY | 3 | Low | N/A | N/A
  PLUSISITEEN | TRIMAIN | BEDFORD | TAS Location mapping for inbound flows | N/A
  PLUSISITEEN | TRIMAIN | SPACE 01 | N/A | N/A
  PLUSISITEEN | TRIMAIN | TRIMAIN | N/A | N/A |
    
b. Once these Integration Controls are created, associate them in both the created Enterprise Services and Publish Channels by using the following two tables

Enterprise Service | Control
--|-- 
PLUSIASSET | PLUSIORGEN
PLUSIASSET | PLUSIPRIORITY
PLUSIASSET | PLUSISITEEN 
PLUSILOCATION | PLUSILOCSTATUS
 PLUSILOCATION | PLUSISITEEN
 PLUSILOCATION | PLUSISTATUS
 PLUSIPERSON | PLUSIORGEN
 PLUSIPERSON | PLUSISITEEN
  PLUSIWO | PLUSIWOPRIORITY



Publish Channel | Control
--|--
PLUSIASSET | PLUSIPRIORITY
PLUSILOCATION | N/A
PLUSIPERSON | PLUSIORG
PLUSISR | N/A
PLUSIWO | PLUSIWOPRIORITY
PLUSIWO | PLUSIWOSTART
 
c. Return to the PLUSITRIRIGA External System. On the left side of the External Systems page, select **Setup Integration Controls** under **More Actions** and make sure that all 5 Integration Controls are showing as present.

#### 6. Enable Object Structure Security

The user needs to be able to transact against the specific object structureS in Manage. Navigate to Object Structures and search for MXPERSON. On the left side of the MXPERSON screen select **Configure Object Structure Storage** and turn on the button underneath **Use Object Structure for Authorization Name?**

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

>**Note**
>Repeat this process for the other changed structures.

### 7. Application Designer Changes

<!-- <img width="1788" alt="App-Designer" src="https://media.github.ibm.com/user/348712/files/457a7a80-3805-11ed-9484-133899bdfac4"> -->

Go to **System Configuration -> Platform Configuration -> Application Designer**

#### Person

1. Search for **PERSON**, switch to the **Person** Tab and select a section to add three new boxes.
2. At the top, click the icon labeled **Control Palette** and add a Multipart Textbox at the bottom of the section. Add the values from the below table within the properties of the Multipart Textbox and click **Save Definition** 
> **Note**
> *Be sure that the Attributes are taken from the PERSON Object.*

|Type of Control | Label | Attribute | Attribute for Part 2 (*If Multipart Textbox*) | Lookup | Input Mode for Part 2 (*If Multipart Textbox*)
|--|--|--|--|--|--|
| Multipart Textbox |TAS Location Path | PLUSIPRIMARYLOCPATH | PLUSILOCATIONPATH.DESCRIPTION | VALUELIST | Readonly
| Multipart Textbox | TAS Primary Organization | PLUSIORGPATH | PLUSIORGANIZATIONPATH.DESCRIPTION | VALUELIST | Readonly
|Textbox | TAS Record ID | EXTERNALREFID (*From the Person Object*) | N/A | N/A | N/A


Follow the same directions using the following information for the other objects that will be used in the integration

#### Asset

Search for **ASSET** in Application Designer
> **Note**
> *Be sure that the Attributes are taken from the ASSET Object.*

|Type of Control | Label | Attribute | Attribute for Part 2 (*If Multipart Textbox*) | Lookup | Input Mode for Part 2 (*If Multipart Textbox*)
|--|--|--|--|--|--|
| Multipart Textbox |TAS Building Equipment Spec | PLUSIASSETSPECNAME | PLUSIASSETSPECCLASS.DESCRIPTION | VALUELIST | Readonly
| Multipart Textbox | TAS Primary Organization | PLUSIORGPATH | PLUSIORGANIZATIONPATH.DESCRIPTION | VALUELIST | Readonly
| Multipart Textbox | TAS Location Path | PLUSILOCPATH | PLUSILOCATIONPATH.DESCRIPTION | VALUELIST | Readonly
|Textbox | TAS Record ID | EXTERNALREFID (*From the Asset Object*) | N/A | N/A | N/A

#### Location

Search for **LOCATION** in Application Designer
> **Note**
> *Be sure that the Attributes are taken from the LOCATION Object.*

|Type of Control | Label | Attribute | Attribute for Part 2 (*If Multipart Textbox*) | Lookup | Input Mode for Part 2 (*If Multipart Textbox*)
|--|--|--|--|--|--|
| Multipart Textbox |TAS Space Classification | PLUSISPACECLASSIFICATION | PLUSISPCCLASSIFICATION.DESCRIPTION | VALUELIST | Readonly
| Multipart Textbox | TAS Parent Location | PLUSIPARENTLOCATION | PLUSIPARENTPATH.DESCRIPTION | VALUELIST | Readonly
|Textbox | TAS Record ID | EXTERNALREFID (*From the Location Object*) | N/A | N/A | N/A

#### Service Request

Search for **SR** in Application Designer
> **Note**
> Be sure that the PLUSIREQCLASSID Attribute is taken from the TICKET Object.


|Type of Control | Label | Attribute | Attribute for Part 2 (*If Multipart Textbox*) | Lookup | Input Mode for Part 2 (*If Multipart Textbox*)
|--|--|--|--|--|--|
| Multipart Textbox |TAS Request Classification | PLUSIREQCLASSID | PLUSIREQCLASS.DESCRIPTION | VALUELIST | Readonly
| Multipart Textbox |TAS Space Classification | PLUSISPACECLASSIFICATION | PLUSISPCCLASSIFICATION.DESCRIPTION | VALUELIST | Readonly
| Multipart Textbox | TAS Parent Location | PLUSIPARENTLOCATION | PLUSIPARENTPATH.DESCRIPTION | VALUELIST | Readonly
|Textbox | TAS Record ID | EXTERNALREFID (*From the Ticket Object*) | N/A | N/A | N/A

e. Click **Save Definition** after the changes are added.


#### Work Order
Search for **WOTRACK** in Application Designer
> **Note**
> Be sure that the PLUSIREQCLASSID Attribute is taken from the WORKORDER Object.

|Type of Control | Label | Attribute | Attribute for Part 2 (*If Multipart Textbox*) | Lookup | Input Mode for Part 2 (*If Multipart Textbox*)
|--|--|--|--|--|--|
| Multipart Textbox |TAS Location Path | PLUSILOCPATH | PLUSILOCATIONPATH.DESCRIPTION | VALUELIST | Readonly
| Multipart Textbox | TAS Primary Organization | PLUSIORGPATH | PLUSIORGANIZATIONPATH.DESCRIPTION | VALUELIST | Readonly
|Textbox | External Ref ID | EXTERNALREFID (*From the WorkOrder Object*) | N/A | N/A | N/A

Click **Save Definition** after the changes are added.

### 8. TAS to Maximo Service Request Comment & Attachment Support

The PLUSITRIServiceRequest2MX_v_1_1_0 integration supports both Comments and Attachments from TAS. This includes pictures or images that are attached as either Comments or Attachments in TAS.

Some configuration will need to be done in order to allow for this.

#### DB Config

1. Go to Database Configuration and pull up the WORKLOG object.
- Add 'EXTERNALREFID' as an Attribute with a type ALN and 50 length.
- Turn on Admin Mode and Run Database Configuration for this change as it will need to be referenced in the Index.
- Once EXTERNALREFID has been added, navigate to Indexes and add the index 'PLUSIEXTREF'. Under the Columns table make sure to reference the EXTERNALREFID that was just created as well as to leave 'Enforce Uniqueness' unchecked.
- Turn on Admin Mode and run Database Configuration again to have these changes take effect.

#### Object Structure

1. Navigate to Object Structure and find MXSR. Duplicate this Object Structure and rename it PLUSISRDOCS.
- Add two new Source Objects to the Structure underneath SR: "DOCLINKS" and "WORKLOG". Make sure the Relationships match with the following image.

<img>
- Select the WORKLOG Source Object and in the 'Details' section add the 'PLUSIEXTREF' Index to the Alternate Key field. Then navigate to Exclude/Include Fields. Click on Non-Persistent Fields in the window and make sure 'Description_LongDescription' is selected as Included.

#### Enterprise Services

1. Now that the changes are made in database and object structure level, they need to be referenced by the Enterprise Service. To do this, duplicate the Default PLUSISR Enterprise Service and name this new one 'PLUSISRDOCS' that references the newly created Object Structure.
- Remove the default PLUSISR Enterprise Service from the PLUSITRIRIGA External System in order to Delete the Enterprise Service. Once deleted, duplicate PLUSISRDOCS and name it PLUSISR to replace the deleted Enterprise Service.
- Add the new PLUSISR back into the External System.


Now when a Service Request comes in with a comment or attachment, Maximo will keep track of the comments in the Worklog and images/documents in the Attachments.

***

## Part 5: Testing

To test that the configuration is complete, send a test payload in order to test connectivity.

### Maximo
1. Go to the **End Points** application and click **Test** at the bottom of the integration you would like to test.
- Send a test payload that is a valid object. ```{"Hello":"World"}``` would work.
- If the response is anything other than **Bad Request**, see [what might be causing the error](#testing-the-end-point)

### TAS
Use a tool like POSTMan to test the connectivity of the App Connect flow. You can use a Sample JSON Payload from this [open source repository](https://github.com/IBM/tririga-api). 

## Part 6: Data Pre-requisites for the Integrations

Because of internal references within the TRIRIGA and Maximo data models some flows depend on other flows to function. The following tables describe these relationships. You will need to run the pre-requisite flows first before running the main flows.

### Maximo to TAS

Flow | Flows to run first
--|--
MXPerson2TRI | TRISpace2MX, TRILocPath2MX, TRIOrg2MX
MXAsset2TRI | TRISpace2MX, TRILocPath2MX, TRIOrg2MX, TRIAssetSpec2MX
MXLocation2TRI | TRILocPath2MX, TRISpaceClass2MX
MXServiceReq2TRI | TRILocPath2MX, TRIReqClass2MX, TRIPerson2MX
MXWorkOrder2TRI | TRILocPath2MX, TRIOrg2MX

### TAS to Maximo

Flow |  Flows to run first
--|--
TRIPerson2MX | TRISpace2MX, TRILocPath2MX, TRIOrg2MX
TRIAsset2MX | TRISpace2MX, TRILocPath2MX, TRIOrg2MX, TRIAssetSpec2MX
TRISpace2MX | TRILocPath2MX, TRISpaceClass2MX
TRIServiceReq2MX | TRILocPath2MX, TRIReqClass2MX, TRIPerson2MX
TRIWorkOrder2MX | TRILocPath2MX, TRIOrg2MX



## Troubleshooting

Depending on the direction of the flow, cross-referencing errors from two systems can help identify the root cause of an issue with the integration. For example:
 
 - If there is a Not Found error in Maximo Message Reprocessing when running the flow, double check the logs in App Connect to see if there is a corresponding error. If there is, the root cause might be related to what is being sent out of Maximo. If there isn't, then the message never left Maximo and the flow should be checked to make sure it is running.

### Where to find logs in App Connect

#### App Connect Enterprise

The logs in App Connect Enterprise are found in the Integration Server pod. If running from App Connect Designer, this integration server is found in the App Connect namespace in OpenShift. Typically the Designer integration server has 4 pods and the logs can be found in the pod with a similar naming convention to `des-01-quickstart-ma-designer-designer-flows`.

<img src="https://media.github.ibm.com/user/348712/files/259886b8-45d5-4fff-9d31-18145cf61c03" alt="App Connect Enterprise">
*App Connect Enterprise*

#### App Connect SaaS & AWS

The logs in App Connect SaaS and AWS can be found within the application itself by clicking on the clipboard icon on the left side task bar.

<img src="https://media.github.ibm.com/user/348712/files/32ea395f-a001-4529-ae96-8615a35bd5c1" alt="App Connect SaaS or AWS">
*App Connect SaaS or AWS*
 
### Common errors that arise from Maximo
 
#### Testing the End Point
 
When testing that the end point is entered correctly on the **End Points** application, there are two common errors:

Error | Cause | Resolution
 -- |-- |--
Response code received from the HTTP request from the endpoint is not successful | Invalid URL in the Integration Object | Double check the URL that all of the components are entered correctly. Make sure there are no accidental spaces at the beginning or the end in event of a copy/paste.
404: Not Found | Flow is not Running | Make sure the flow is Running before starting the cron tasks
PKSync error| Certificate error | Confirm the certificate is configured correctly
 
#### Message Reprocessing

Errors in Maximo can be found in **Message Reprocessing**

 Error | Cause | Resolution
 -- |-- |--
Bad Request error | Either the payload being sent out is incorrect or the account configurations are wrong |Double check to make sure the Accounts from the App Connect pre-requisite section are correct and that the item in Maximo has all of the mapped fields

### Common errors that arise from TAS
 
 Error | Cause | Resolution
 -- |-- |--
 404 - Not Found: Cannot POST | Invalid URL in the Integration Object | Double check the URL in the specific Integration Object that all of the components are entered correctly
 404 - API doesn't exist | Flow is not running | Double check that the flow is Active in App Connect
 404 - The HTTP request returned with an error 404 "Not Found" | Incorrect App Connect connector config | Double check that the credentials being used in the HTTP post node in App Connect are correct
 401 - Authorization error | Too many user sessions open in TAS | Open the Admin dashboard on the TAS environment and check the Users logged in. This issue can arise after a number of requests are made to TAS and then gives a 401 error even with the proper credentials. Clear the users logged in and the issue should clear.
 
 - Clear OSLC Cache in TAS Admin Console in case the integrations do not work in intended manner.
 


## Reference for Pre-requisite

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

### Pre-Requisite: Update TAS Custom Resource with App Connect Certificates:

In OpenShift, import the following yaml with the App Connect certificates added under the spec.alias.crt section:

```
cat <<EOF | oc create -f -
apiVersion: truststore-mgr.ibm.com/v1
kind: Truststore
metadata:
    name: my-tas-truststore
spec:
    license:
        accept: true
    includeDefaultCAs: true
    servers:
    - "example.com:443"
    certificates:
    - alias: alias_1 
      crt: |
        -----BEGIN CERTIFICATE-----
        ...
        Certificate 1   
        ...
        -----END CERTIFICATE-----

        ...

    - alias: alias_n 
      crt: |
        -----BEGIN CERTIFICATE-----
        ...
        Certificate n   
        ...
        -----END CERTIFICATE-----
EOF        

```

If there is already a truststore for TAS, update the truststore with the App Connect certificate.