# IBM MAS Connector for TRIRIGA

## Overview

Portfolio Data is able to be synced bi-directionally across Maximo and TRIRIGA with the use of a TRIRIGA Application Suite (TAS) or Maximo Application Suite (MAS) Connector. The data that is supported at this time includes People, Assets, Locations/Spaces, Service Requests, and Work Orders/Work Tasks. After configuring the environments with the proper prerequisites, select the desired connector below and follow the instructions to implement the integration.

**Note: IBM MAS Connector for TRIRIGA also refers to the IBM TAS Connector for Maximo**

## Prerequisites

These code patterns all require initial configuration of an instance of Maximo, App Connect, and TRIRIGA. These prerequisites also require that all three applications are behind the same firewall. 

If any of these three applications do not share the same firewall, a secure connection between the applications will need to be established. IBM does provide a product that accomplishes this: Secure Gateway. Learn more about getting started with [Secure Gateway.](https://cloud.ibm.com/docs/SecureGateway?topic=SecureGateway-getting-started-with-sg)

Certificates for the App Connect instance will need to be imported into both systems:

### Certificates for Maximo

#### Maximo 7.6.1.2+:

**Configure WebSphere Certificates**

This makes a test connection to a Secure Sockets Layer (SSL) port and retrieves the signer from the server during the handshake.

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

<img width="1280" alt="Websphere-Home" src="https://media.github.ibm.com/user/348712/files/ee26db00-3801-11ed-8365-8604ad0a66df">

*Step 2: WebSphere Homepage*

<img width="1282" alt="Websphere-Keystores" src="https://media.github.ibm.com/user/348712/files/eebf7180-3801-11ed-9253-85d4e8bed0ea">

*Step 3: Websphere Keystores*

<img width="1283" alt="Websphere-Signercerts" src="https://media.github.ibm.com/user/348712/files/f121cb80-3801-11ed-9833-65fe541bcef5">
 
*Step 4: Websphere Signer Certs*


#### MAS 8.8+

1. Extract the App Connect certificate from an imported flow URL. Navigate to the flow's page and click on **Test** and then **Try It** to get the proper url.

2. Navigate to the Admin dashboard for MAS and go to the workspace where Manage is deployed.


3. Update the configuration and scroll down to **Imported Certificates**. Untick the **System managed** button and fill in the extracted certificate in the fields.


4. Click **Apply Changes** at the top of the page and MAS will update the truststore with the new certificate.

<img width="1280" alt="Manage-Workspace" src="https://media.github.ibm.com/user/348712/files/33502d00-43fe-11ed-801b-a454fa4b8f7e">

*Step 2: Manage Workspace*

<img width="1280" alt="Imported-Certificates" src="https://media.github.ibm.com/user/348712/files/33e8c380-43fe-11ed-91b9-68b37462c897">

*Step 3: Imported Certificates*

### Certificates for TRIRIGA

*To be filled in with proper information*
<br>
<hr>


See below for the pre-requisites of each system:

### Maximo

**Credentials and access to an instance of Maximo as well as the WebSphere application which hosts Maximo are required. This code pattern has been tested on Maximo 7.6.1.2 as well as MAS 8.6** 
 
The following steps and prerequisites are done against a Maximo demo database. The naming conventions may slightly differ from this, but these are the necessary components. 

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

### App Connect

** Access to an instance of App Connect with a deployed instance of a Designer is required. This code pattern has been tested with App Connect version 3.0. **

1. Two accounts need to be created from the **Catalog** tab in order to connect the applications. Once all of the connectors have loaded, type in **http** to find the HTTP Application.
 
2. If this is the first account, select **Connect** to begin setting up the initial HTTP account. If this is not the first account, make sure to take note if there are any other generic account names present because the number of the one created will depend on what has already been created. App Connect creates an account with a generic name in sequential order (Example: if Account 1 and Account 2 are present, the new account will be Account 3).

3. Use the below table to associate the proper credentials for the accounts.
4. Once the account is connected, head back to the HTTP Application on the Catalog page and rename the new account according to the Account Name column in the above table.

Flow | Account Name | Username | Password | API key | API location | API key name
---|---|---|---|---|---|---
Max -> Tri | mxtririga | Your TRIRIGA Username | Your TRIRIGA Password | N/A | N/A | N/A
Tri -> Max | trimaximo | N/A | N/A | Your Maximo apikey | header | apikey 


<img src="https://media.github.ibm.com/user/348712/files/e23b1900-3801-11ed-8ab4-a5234d64325d" alt="App Connect Catalog">

*App Connect catalog*

### TRIRIGA

1. Download the latest [OM Package](https://github.com/IBM/tririga-api/tree/main/docs/ompackages).
2. Navigate to **Tools -> Object Migration** and import the package into the instance of TRIRIGA.
3. The Date Time Format field in the user profile must be in UTC. Navigate to **Portfolio -> People -> My Profile** and select the user profile that will be triggering the action. The Date Time Format should be in UTC as shown below.

<img width="1792" alt="TRI-UTC" src="https://media.github.ibm.com/user/348712/files/ec5d1780-3801-11ed-8f32-72fc453558f0">

*Date Time Format Field in My Profile*
