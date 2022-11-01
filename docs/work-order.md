
# Work Order Integration


## Summary

Enable automated synchronization of Work Order data from IBM Maximo to IBM TRIRIGA or vice versa.

## Description

In this code pattern, learn how to synchronize a Work Order created in Maximo with TRIRIGA using an App Connect Designer flow. 

<img src="https://media.github.ibm.com/user/348712/files/f6354980-3806-11ed-977c-310048b1e763" alt="Work Order Diagram">

*A diagram of the Work Order Integration*

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

This configuration assumes the completion of the pre-requisites and steps outlined in the [Maximo <-> TRIRIGA code pattern](https://ibm.github.io/IBM-Sustainability-Software-Portfolio-Connectors/mas-tas-home/) for those steps.


### Maximo

Within Maximo, a change to the Work Order Application is needed to see additional fields.

#### 1. Application Designer


<img src="https://media.github.ibm.com/user/348712/files/f46b8600-3806-11ed-8a5d-43e6c8db4760" alt="App Designer" >

*Work Order Application Designer*


Go to **System Configuration -> Platform Configuration -> Application Designer**

Search for **WOTRACK**

Switch to the **Work Order** Tab and scroll down to the Work Order Details section

At the top, click the icon labeled **Control Palette** and drag the respective controls into the first main section on the right of the screen. Add these values within the properties of the control. 

**Be sure that the PLUSIREQCLASSID Attribute is taken from the WORKORDER Object.**

|Type of Control | Label | Attribute | Attribute for Part 2 (*If Multipart Textbox*) | Lookup | Input Mode for Part 2 (*If Multipart Textbox*)
|--|--|--|--|--|--|
| Multipart Textbox |Tririga Location Path | PLUSILOCPATH | PLUSILOCATIONPATH.DESCRIPTION | VALUELIST | Readonly
| Multipart Textbox | Tririga Primary Organization | PLUSIORGPATH | PLUSIORGANIZATIONPATH.DESCRIPTION | VALUELIST | Readonly
|Textbox | External Ref ID | EXTERNALREFID (*From the WorkOrder Object*) | N/A | N/A | N/A

Click **Save Definition** after the changes are added.


### App Connect
 
*The configuration of App Connect from the previous code pattern should provide the **mxtririga** and **trimaximo** accounts within App Connect needed for the flows to work properly.*

1. Download and import the .yaml files and keep the urls handy for a later step. Use the following table for the parameters of the flows:


|Parameter Name|Value  |
|--|--|
|mxUrl | http://[host]:[port]/meaweb/esqueue/PLUSITRIRIGA/PLUSIMXWO |
|triUrl | http://[host]:[port]/oslc/so/triAPICWorkTaskCF |
|mxDomain | PLUSILOCPATH (*For Loc Path flow*) / PLUSIORGPATH (*For Org Path flow*)|


### TRIRIGA

**Populate the domains created in the Maximo pre-requisites**

1. Go to **Tools -> System Setup -> Integration -> Integration Object**.

2. Select **Organization - APIC - HTTP Post** from the table. Fill in the required sections:
3. Click **Execute** at the top of the window. The process will take a few minutes since there are a large amount of files, but once it is completed you can check that the batch processed correctly under the specified domain.

4. Repeat the process with **triBuilding**, **triProperty**, & **triFloor** using the **PLUSITRILocPath2MX** url and parameters.

5. Verify the data is in sync by checking the corresponding Domain in Maximo. The populated table should look like this:

<img width="1786" alt="Populated-Domain" src="https://media.github.ibm.com/user/348712/files/3f728c80-0ce1-11ed-87cb-f3079380ad56">

*Populated Domains*


## Step 1. Create a Work Order
### Maximo to TRIRIGA

1. Go to **Work Orders -> Work Order Tracking** and click on the blue plus sign to create a new Work Order.

2. Input the desired name/number of the WO along with the description and assign the corresponding Primary Org and Location Path. Click **Save Work Order** and the flow should fire.

3. The flow also supports the cost calculation of associated actuals within a Work Order. Costs can only be transacted against a Work Order with a valid GL Account. Once the proper GL Account is associated, navigate to the **Actuals** tab within the desired Work Order to assign costs. The supported actuals are **Labor**, **Materials**, **Services**, and **Tools**.

<img src="https://media.github.ibm.com/user/348712/files/f59cb300-3806-11ed-8bdb-d30c653a6894" alt="MX Work Order" >

*Work Order Tracking Page*

#### A. Labor

1. Select the correct Labor record to associate with the Work Order. Enter in the required Start and End Time fields and **Save** the Work Order. The flow will update the TRIRIGA application with the correct cost associated with the Labor based on the time entered.

<img width="1792" alt="Labor" src="https://media.github.ibm.com/user/348712/files/3e415f80-0ce1-11ed-9933-273f160fd177">

*Labor Actuals*

#### B. Materials

Select the correct Material record to associate with the Work Order. Enter in the required **Storeroom** field and **Save** the Work Order. The flow will update the TRIRIGA application with the correct cost associated with the Material.

<img width="1789" alt="Materials" src="https://media.github.ibm.com/user/348712/files/3da8c900-0ce1-11ed-82eb-d77145b0c93c">

*Materials*

#### C. Services

The Service actual will only populate if a PO is received, and the PO has a Service line associated with the Work Order. 

1. Create a new Purchasing Order and add a new PO Line. Select **Service** as the Line Type and enter the Line Cost. 
2. Associate the desired WO for this Service to be charged and approve the PO. 
3. In the **Receiving** application, select the approved PO and switch to the **Service Receipts** tab at the top. Click on **Select Ordered Services**, select the created Service, and click **Save Receipt** on the left-hand side of the screen. The Service will then appear as an actual in both Maximo and TRIRIGA via the flow.

<img width="1792" alt="PO-Charges" src="https://media.github.ibm.com/user/348712/files/3bdf0580-0ce1-11ed-941e-66396f0f92a7">

*Step 1: PO Charges*

<img width="1792" alt="Service Receipts" src="https://media.github.ibm.com/user/348712/files/3c779c00-0ce1-11ed-9d5b-6b96edde539b">

*Step 3: Service Receipts*


#### D. Tools

1. Select the correct Tool record to associate with the Work Order. Enter in the required Bin field and **Save** the Work Order. The flow will update the TRIRIGA application with the correct cost associated with the Material.

<img width="1792" alt="Tools" src="https://media.github.ibm.com/user/348712/files/3bdf0580-0ce1-11ed-86e5-8099602fea45">

*Tools Actual*

<i>A full breakdown of costs can be found by going to **View -> Costs** from the left side of the screen under **More Actions**</i>

<img width="1792" alt="View Costs" src="https://media.github.ibm.com/user/348712/files/3d103280-0ce1-11ed-9993-b769a66da5db">

*View Cost Breakdown*

### TRIRIGA to Maximo
 
 1. Go to **Tasks -> Manage Tasks -> Work Tasks** and click the **Add** button on the top right. 
 
 
 2. Fill in the required fields and click **Submit** at the top right of the newly opened window. The flow should fire upon submission.
 
 <img src="https://media.github.ibm.com/user/348712/files/f6354980-3806-11ed-9486-47977fdfdc12" alt="Tri Work Order" >

 *TRIRIGA Work Order*



## Troubleshooting

Common Errors and their resolutions:

### Maximo
 
 Common errors found in the Maximo system
 
 Error | Cause
  ---|---
 401: Bad Request | This usually means an aspect of the request was not sent correctly- double check what is being sent as well as the flow in App Connect to make sure everything is correct and running.
 
### App Connect
 Troubleshoot App Connect with the logging function. 
 
 1. While the flow is stopped, add a **Log** node into the flow from the **Toolbox** tab. This will allow mapping of any field to the **Logging** section of the application. 

 2. Select **Info** for the Log level and then map the field that needs debugging. In this example the Request Object has been mapped to see what is being sent through the flow. Click the icon to the right of the Message Detail to map the desired field. The **Log** node will compile the message and read it out in the **Logging** section.

 3. Diagnose the response that shows up in this section to learn what might be causing the issue.
 
 <img src="https://media.github.ibm.com/user/348712/files/47dcd480-3805-11ed-9c5f-f68af91cbc41" alt="Log Designer" >

 *Step 1: Log Node*
 
 <img src="https://media.github.ibm.com/user/348712/files/46aba780-3805-11ed-8db5-67c866d28126" alt="Log Content" >

 *Step 2: Log Content*
 
 <img src="https://media.github.ibm.com/user/348712/files/47dcd480-3805-11ed-9efb-a9cb2dd074d4" alt="Log Location" >
 
 *Logging section*

 <img src="https://media.github.ibm.com/user/348712/files/47443e00-3805-11ed-9e6e-7fe53c4465c3" alt="Log Dashboard" >

 *Step 3: Logging Dashboard*
 

 
### TRIRIGA
 
 Common errors found in the TRIRIGA system

Error | Cause
  ---|---
  ERROR: Requested For Does not Exist | No People record exists with the triIdTX value mentioned in triRequestedForTX field of the payload
  ERROR: Building Does not Exist | No Building record exists with the triNameTX value mentioned in triBuildingTX field of the payload
  ERROR: Location Does not Exist | No Location record exists with the triNameTX value mentioned in triParentLocationTX field of the payload
  ERROR: Organization Does not Exist | No Organization record exists with the triPathTX value mentioned in triCustomerOrgTX field of the payload
  400 - Bad Request :: {"error":{"statusCode":400,"message":"API creation is failed with not_found","name":"Error"}} | *To be filled in*