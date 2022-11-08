

### Maximo

Within Maximo, a change to the Service Request application is needed to see additional fields.

#### 1. Service Request

<img width="1788" alt="App-Designer" src="https://media.github.ibm.com/user/348712/files/457a7a80-3805-11ed-9484-133899bdfac4">
*SR Application Designer*

a. Go to **System Configuration -> Platform Configuration -> Application Designer**
b. Search for **SR**
c. Switch to the Service Request Tab and scroll down to the Service Request Details section
d. At the top, click the icon labeled **Control Palette** and add a Multipart Textbox at the top of the right section. Add these values within the properties of the Multipart Textbox. 

** Be sure that the PLUSIREQCLASSID Attribute is taken from the TICKET Object. **

|Field Name|Value  |
|--|--|
|Attribute | PLUSIREQCLASSID |
|Attribute for Part 2 | PLUSIREQCLASS.DESCRIPTION |
| Lookup | VALUELIST |
|Input Mode for Part 2 | Readonly |

e. Click **Save Definition** after the changes are added.

### AppConnect
 
The configuration of AppConnect from the previous code pattern should provide the **mxtririga** and **trimaximo** accounts within AppConnect needed for the flows to work properly.

Download and import the .yaml files for **PLUSIMXServiceReq2TRI** and **PLUSITRIReqClass2MX** and keep the urls handy for a later step. Use the following table for the parameters:

|Parameter Name|Value  |
|--|--|
|mxUrl | http://[host]:[port]/meaweb/esqueue/PLUSITRIRIGA/PLUSIMXSR |
|triUrl | http://[host]:[port]/oslc/so/triAPICServiceRequestCF |
|mxDomain | PLUSIREQCLASS|

### TRIRIGA

#### Populate the domain created in the Maximo pre-requisites

1. Go to **Tools -> System Setup -> Integration -> Integration Object**

2. Select **triRequestClass - APIC - HTTP Post** from the table and fill in the URL with the correct parameters from the App Connect section.

3. Click **Execute** at the top of the window. The process will take a few minutes since there is a large amount of files, but once it is completed you can check that the batch processed correctly under the specified domain.

#### Filter

1. To filter the request from TRIRIGA to Maximo, update the **Filter** tab in the **triAPICServiceRequest - OSLC - Outbound** query to trigger for selected Request Classes. Select **System** for **triRequestClassCL** and then **Run Report**

<img>

## Step 1. Create a Service Request

### Prerequisite

A service request needs a member of the TRIRIGA organization on both Maximo and TRIRIGA. If you are starting from scratch, run the *PLUSITRIPERSONBATCH* flow to sync the person records in both systems. If data is already aligned and present, you are ready to run the integration.

### Maximo to TRIRIGA

1. Go to **Service Desk -> Service Request** and click on the blue plus sign to create a new service request.


2. Assign a user to Reported By and Affected Person. From here, a request classification needs to be selected and the value and description will populate in. **Click Save Service Request** and the flow should fire.

<img src="https://media.github.ibm.com/user/348712/files/490e0180-3805-11ed-959f-b0d9f985d16f" alt="Service Request" >

*Create Service Request in Maximo*


### TRIRIGA to Maximo
 
 1. Go to **Requests -> Manage Requests -> Electrical & Lighting** and click the **Add** button on the top right. 
 
 2. Fill in the required fields and click **Submit** at the top right of the newly opened window. The flow should fire upon submission.
 
 <img src="https://media.github.ibm.com/user/348712/files/4a3f2e80-3805-11ed-9020-b097f3cb1251" alt="Tri Service Request" >

 *TRIRIGA Service Request*
 

## Troubleshooting

Common Errors and their resolutions:

### Maximo
 
 Common errors found in the Maximo system
 
 Error | Cause
  ---|---
 401: Bad Request | This usually means an aspect of the request was not sent correctly- double check what is being sent as well as the flow in AppConnect to make sure everything is correct and running.
 

 
### AppConnect
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
  ERROR: Requested By Does not Exist | No People record exists with the triIdTX value mentioned in triRequestedByTX field of the payload
  ERROR: Building Does not Exist | No Building record exists with the triNameTX value mentioned in triBuildingTX field of the payload
  ERROR: Request Class Does not Exist | No Request Class record exists with the triNameTX value mentioned in triRequestClassCL field of the payload
  ERROR: Organization Does not Exist | No Organization record exists with the triPathTX value mentioned in triCustomerOrgTX field of the payload
  ERROR: Service Request Does not Exist | No Service Request record exists with the triIdTX value mentioned in triExternalReferenceTX field of the payload    

