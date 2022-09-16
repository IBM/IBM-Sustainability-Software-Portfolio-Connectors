
# Work Order Configuration


### Summary

Enable automated synchronization of Work Order data from IBM Maximo to IBM TRIRIGA or vice versa.

### Description

In this code pattern, learn how to synchronize a Work Order created in Maximo with TRIRIGA using an AppConnect Designer flow. 

<img src="/Images/Work-Order-Diagram.png">

1. When a Work Order is created in Maximo Asset Management, it triggers the flow to populate the request in TRIRIGA. (There is also a flow that works in the reverse direction, that works in a similar way.)
2. App Connect sends a request with the new information through the flow towards the target system (TRIRIGA).
3. A JSON Parser sifts through the request and converts it to an object.
4. This object from the JSON Parser goes through Steps 5-8.
5. The data from the object is mapped to the corresponding fields in the target application (TRIRIGA).
6. The newly mapped data is sent to the target application (TRIRIGA) where the request is then created.
7. This record is then validated with the original application (Maximo Asset Management) via another Post request.
8. An ID is created within the original application (Maximo Asset Management).



At the end of this process, a Work Order can be created within Maximo and sent to TRIRIGA and vice versa.

## Pre-requisites

This configuration assumes the completion of the pre-requisites and steps outlined in the Maximo <-> TRIRIGA code pattern. [See here](https://developer.ibm.com/patterns/synchronize-databases-between-asset-workplace-management-solutions/) for those steps.


<details><summary><b>Maximo</b></summary>

Within Maximo, some initial changes to the database and Work Order application need to be completed in order for the integration to work properly. Errors may arise if these steps are not completed.

## First, create a Domain that will link to an attribute on the WORKORDER table.


<img src="/Images/Domains.png" >

#

|Domain | Description | Domain Type | Data Type | Length |
|--|--|--|--|--|
|PLUSILOCPATH | Tririga location path details | ALN | ALN | 10
|PLUSIORG | Primary organization path | ALN | ALN | 10

These domain will need to be populated in order to send a Work Order out of Maximo. This will be completed in a later step.

## Next, head to Database Configuration and search for the 'WORKORDER' object. 

<img src="/Images/DB-config.png" >

#

Go to 'Attributes' and create a new row:

|Attribute | Description | Type | Length | Required | Domain |
|--|--|--|--|--|--|
|PLUSILOCPATH|Tririga location path details | ALN | ALN | No | *Select the PLUSILOCPATH Domain from the previous step*
|PLUSIORGPATH | Primary organization path | ALN | ALN | No | *Select the PLUSIORG Domain from the previous step*

** Make sure the length of this attribute has the same length as the domain that is linked

#

Save the attribute. Apply the configuration changes to the database by switching on Admin mode and Apply Database Configuration

### Configure the Publish Channel, Enterprise Service, End Point, and External System

#### Publish Channel
Navigate to Integration -> Publish Channels
1. Search for 'MXWOInterface' under the Publish Channel field. Click on the channel and from the left side of the screen select 'Duplicate Publish Channel' 
2. Rename the channel PLUSIMXWO
3. Click on 'Enable Event Listener' on the left side under More Actions
4. Make sure Publish JSON and Retain MBO's are checked, the Operation should default to Publish and the Adapter should default to MAXIMO.
5. Click 'Save Publish Channel' on the left under Common Actions

#### Enterprise Service
 Navigate to Integration -> Enterprise Services and click on the blue plus button at the top of the page
1. Under the System name fill in PLUSIWO and in the Description fill in "Work Order"
2. Select 'MXWO' under Object Structure which will populate the Object Structure Sub-Records table
3. Click 'Save Enterprise Service' on the left under Common Actions

Repeat this sequence for PLUSIORGDOMAIN and PLUSILOCDOMAIN. In Description fill in "Organization records from Tririga" & "Location path records from Tririga" respectively and select 'MXDOMAIN' for both Object Structures.

#### End Point 
 Navigate to Integration -> End Points and click on the blue plus bitton at the top of the page
1. Under End Point fill in PLUSIWO and in the Description fill in "AppConnect Work Order outbound to TRIRIGA"
2. Select 'HTTP' for Handler
3. Click on 'Save End Point' on the left side under More Actions which will populate the Properties for the End Point
4. Until the flows have a destination url, we can only fill in certain fields:
   - HEADERS: "Content-Type: application/json"
   - HTTPMETHOD: POST
5. Save the End Point


#### External System
 Navigate to Integration -> External System and open up the 'PLUSITRIRIGA' external system that was previously set up. Associate the Publish Channel and Enterprise Services to the External System
 
1. Add a new row in Publish Channel with the newly created PLUSIMXWO and link the PLUSIWO End Point as well
2. Add three new rows with the newly created Enterprise Services. 

** Make sure all are enabled


### Create a relationship in the WORKORDER object


<img src="/Images/Relationship.png" >

#

Go to DB Config -> WORKORDER object -> Relationships. Add a 'New Row' and enter the following values:

|Relationship | Child Object | Where Clause | Remarks |
|--|--|--|--|
|PLUSILOCATIONPATH| ALNDOMAIN | domainid='PLUSILOCPATH' and value=:plusilocpath | Relationship to PLUSILOCPATH Alndomain
|PLUSIORGANIZATIONPATH | ALNDOMAIN | domainid='PLUSIORG' and value=:plusiorgpath | Relationship to PLUSIORGPATH Alndomain


### Application Designer


<img src="/Images/App-Designer.png" >

#

Go to System Configuration -> Platform Configuration -> Application Designer

Search for 'WOTRACK'

Switch to the Work Order Tab and scroll down to the Work Order Details section

At the top, click the icon labeled Control Palette and drag the respective controls into the first main section on the right of the screen. Add these values within the properties of the control. 

** Be sure that the PLUSIREQCLASSID Attribute is taken from the WORKORDER Object. **

|Type of Control | Label | Attribute | Attribute for Part 2 (*If Multipart Textbox*) | Lookup | Input Mode for Part 2 (*If Multipart Textbox*)
|--|--|--|--|--|--|
| Multipart Textbox |Tririga Location Path | PLUSILOCPATH | PLUSILOCATIONPATH.DESCRIPTION | VALUELIST | Readonly
| Multipart Textbox | Tririga Primary Organization | PLUSIORGPATH | PLUSIORGANIZATIONPATH.DESCRIPTION | VALUELIST | Readonly
|Textbox | External Ref ID | EXTERNALREFID (*From the WorkOrder Object*) | N/A | N/A | N/A

Click 'Save Definition' after the changes are added.

</details>
 
 <details><summary><b>AppConnect</b></summary>
 
The configuration of AppConnect from the previous code pattern should provide the 'mxtririga' and 'trimaximo' accounts within AppConnect needed for the flows to work properly.

Download and import the .yaml files and keep the urls handy for a later step. Use the following table for the parameters of the flows:


|Parameter Name|Value  |
|--|--|
|mxUrl | http://[host]:[port]/meaweb/esqueue/PLUSITRIRIGA/PLUSIMXWO |
|triUrl | http://[host]:[port]/oslc/so/triAPICWorkTaskCF |
|mxDomain | PLUSILOCPATH (*For Loc Path flow*) / PLUSIORGPATH (*For Org Path flow*)|


</details>

<details><summary><b>TRIRIGA</b></summary>

### Populate the domains created in the Maximo pre-requisites

Go to Tools -> System Setup and select Integration Object under the Integration heading

Select Organization - APIC - HTTP Post from the table. Fill in the required sections:

|Field Name|Value  |
|--|--|
|Http URL | [the *PLUSITRIOrgPath2MX* url from AppConnect with the correct parameters outlined in the AppConnect section] |
| Request Method | POST |
|Content-Type | application/json |

**If the AppConnect instance is based on cloud, include the api key in the Headers. If the instance is on-prem, include your basic authorization in UserName and Password

#

Once the correct values are filled in, click Execute at the top of the window. The process will take a few minutes since there is a large amount of files, but once it is completed you can check that the batch processed correctly under the specified domain.

Repeat the process with *triBuilding*, *triProperty*, & *triFloor* using the *PLUSITRILocPath2MX* url and parameters.

Verify the data is in sync by checking the corresponding Domain in Maximo. The populated table should look like this:

<img width="1786" alt="Populated-Domain" src="https://media.github.ibm.com/user/348712/files/3f728c80-0ce1-11ed-87cb-f3079380ad56">

</details>


## Step 1. Create a Work Order
<details><summary><b>Maximo to TRIRIGA</b></summary>

Go to Work Orders -> Work Order Tracking and click on the blue plus sign to create a new Work Order.

<img src="/Images/MX-Work-Order.png" >

Input the desired name/number of the WO along with the description and assign the corresponding Primary Org and Location Path. Click 'Save Work Order' and the flow should fire.

The flow also supports the cost calculation of associated actuals within a Work Order. Costs can only be transacted against a Work Order with a valid GL Account. Once the proper GL Account is associated, navigate to the 'Actuals' tab within the desired Work Order to assign costs. The supported actuals are 'Labor', 'Materials', 'Services', and 'Tools'.

#### A. Labor

Select the correct Labor record to associate with the Work Order. Enter in the required Start and End Time fields and 'Save' the Work Order. The flow will update the Tririga application with the correct cost associated with the Labor based on the time entered.

<img width="1792" alt="Labor" src="https://media.github.ibm.com/user/348712/files/3e415f80-0ce1-11ed-9933-273f160fd177">

#### B. Materials

Select the correct Material record to associate with the Work Order. Enter in the required 'Storeroom' field and 'Save' the Work Order. The flow will update the Tririga application with the correct cost associated with the Material.

<img width="1789" alt="Materials" src="https://media.github.ibm.com/user/348712/files/3da8c900-0ce1-11ed-82eb-d77145b0c93c">

#### C. Services

The Service actual will only populate if a PO is received, and the PO has a Service line associated with the Work Order. In order to initialize this, create a new Purchasing Order and add a new PO Line. Select 'Service' as the Line Type and enter the Line Cost. Associate the desired WO for this Service to be charged and approve the PO. 

<img width="1792" alt="PO-Charges" src="https://media.github.ibm.com/user/348712/files/3bdf0580-0ce1-11ed-941e-66396f0f92a7">

In the 'Receiving' application, select the approved PO and switch to the 'Service Receipts' tab at the top. Click on 'Select Ordered Services', select the created Service, and click 'Save Receipt' on the left-hand side of the screen.

<img width="1792" alt="Service Receipts" src="https://media.github.ibm.com/user/348712/files/3c779c00-0ce1-11ed-9d5b-6b96edde539b">

The Service will then appear as an actual in both Maximo and Tririga via the flow.

#### D. Tools

Select the correct Tool record to associate with the Work Order. Enter in the required Bin field and 'Save' the Work Order. The flow will update the Tririga application with the correct cost associated with the Material.

<img width="1792" alt="Tools" src="https://media.github.ibm.com/user/348712/files/3bdf0580-0ce1-11ed-86e5-8099602fea45">



A full breakdown of costs can be found by going to View -> Costs from the left side of the screen under 'More Actions'

<img width="1792" alt="View Costs" src="https://media.github.ibm.com/user/348712/files/3d103280-0ce1-11ed-9993-b769a66da5db">

</details>

<details><summary><b>TRIRIGA to Maximo</b></summary>
 
 Go to Tasks -> Manage Tasks -> Work Tasks and click the 'Add' button on the top right. 
 
 <img src="/Images/Tri-Work-Order.jpeg" >
 
 Fill in the required fields and click 'Submit' at the top right of the newly opened window. The flow should fire upon submission.
 
 </details>

## Troubleshooting

Common Errors and their resolutions:

<details><summary><b>Maximo</b></summary>
 
 Common errors found in the Maximo system
 
 Error | Cause
  ---|---
 401: Bad Request | This usually means an aspect of the request was not sent correctly- double check what is being sent as well as the flow in AppConnect to make sure everything is correct and running.
 
 </details>
 
<details><summary><b>AppConnect</b></summary>
 The best way to troubleshoot with AppConnect is to use the logging function. While the flow is stopped, add a 'Log' node into the flow from the 'Toolbox' tab.
 
 <img src="/Images/Log-Designer.jpeg" >
 
 This will allow mapping of any field to the 'Logging' section of the application. Select Info for the Log level and then map the field that needs debugging. In this example the Request Object has been mapped to see what is being sent through the flow. Click the icon to the right of the Message Detail filed to map the desired field. 
 
 <img src="/Images/Log-Content.jpeg" >
 
 The Log node will compile the message and read out here:
 
 <img src="/Images/Log-Location.jpeg" >
 
 Diagnose the response that shows up in this section to learn what might be causing the issue.
 
 <img src="/Images/Log-Dashboard.jpeg" >
 
 </details>
 
<details><summary><b>TRIRIGA</b></summary>
 
 Common errors found in the TRIRIGA system

Error | Cause
  ---|---
  ERROR: Requested For Does not Exist | No People record exists with the triIdTX value mentioned in triRequestedForTX field of the payload
  ERROR: Building Does not Exist | No Building record exists with the triNameTX value mentioned in triBuildingTX field of the payload
  ERROR: Location Does not Exist | No Location record exists with the triNameTX value mentioned in triParentLocationTX field of the payload
  ERROR: Organization Does not Exist | No Organization record exists with the triPathTX value mentioned in triCustomerOrgTX field of the payload
    
 </details>
