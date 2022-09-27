
# Service Request Configuration


## Summary

Enable automated synchronization of Service Request data from IBM Maximo to IBM TRIRIGA or vice versa.

## Description

In this code pattern, learn how to synchronize a Service Request created in Maximo with TRIRIGA using an AppConnect Designer flow. 

<img width="467" alt="Service-Request-Diagram" src="https://media.github.ibm.com/user/348712/files/490e0180-3805-11ed-922a-e18101228a1e" height=800px>

1. When a Service Request is created in Maximo Asset Management, it triggers the flow to populate the request in TRIRIGA. (There is also a flow that works in the reverse direction, that works in a similar way.)
2. App Connect sends a request with the new information through the flow towards the target system (TRIRIGA).
3. A JSON Parser sifts through the request and converts it to an object.
4. This object from the JSON Parser goes through Steps 5-8.
5. The data from the object is mapped to the corresponding fields in the target application (TRIRIGA).
6. The newly mapped data is sent to the target application (TRIRIGA) where the request is then created.
7. This record is then validated with the original application (Maximo Asset Management) via another Post request.
8. An ID is created within the original application (Maximo Asset Management).



At the end of this process, a Service Request can be created within Maximo and sent to TRIRIGA and vice versa.

## Pre-requisites

This configuration assumes the completion of the pre-requisites and steps outlined in the Maximo <-> TRIRIGA code pattern. [See here](https://developer.ibm.com/patterns/synchronize-databases-between-asset-workplace-management-solutions/) for those steps.


### Maximo

Within Maximo, some initial changes to the database and Service Request application need to be completed in order for the integration to work properly. Errors may arise if these steps are not completed.

#### 1. Create a Domain that will link to an attribute on the Ticket table.


<img width="1788" alt="Domains" src="https://media.github.ibm.com/user/348712/files/46aba780-3805-11ed-9a37-72039aa3a088">


|Field Name|Value  |
|--|--|
|Domain | PLUSIREQCLASS |
|Description | Tririga service request class |
|Domain Type | ALN |
|Data Type | ALN |
|Length | 10 |

This domain will need to be populated in order to send a service request out of Maximo. This will be completed in a later step.

#### 2. Head to Database Configuration and search for the 'Ticket' object. 

<img width="1784" alt="DB-config" src="https://media.github.ibm.com/user/348712/files/46131100-3805-11ed-81b1-16fa0f9ed5d7">


Go to 'Attributes' and create a new row:

|Field Name|Value  |
|--|--|
|Attribute | PLUSIREQCLASSID |
|Description | TRIRIGA request class of value |
| Type | ALN |
|Length | 10 |
|Required | No |
|Domain | *Select the PLUSIREQCLASS Domain from the previous step* |

** Make sure the length of this attribute has the same length as the domain that is linked


Save the attribute. Apply the configuration changes to the database by switching on Admin mode and Apply Database Configuration

#### 3. Configure the Publish Channel, Enterprise Service, End Point, and External System

##### Publish Channel
Navigate to Integration -> Publish Channels
1. Search for 'MXSRInterface' under the Publish Channel field. Click on the channel and from the left side of the screen select 'Duplicate Publish Channel' 
2. Rename the channel PLUSIMXSR
3. Click on 'Enable Event Listener' on the left side under More Actions
4. Make sure Publish JSON and Retain MBO's are checked, the Operation should default to Publish and the Adapter should default to MAXIMO.
5. Click 'Save Publish Channel' on the left under Common Actions

##### Enterprise Service
 Navigate to Integration -> Enterprise Services and click on the blue plus button at the top of the page
1. Under the System name fill in PLUSIMXSR and in the Description fill in "SERVICE REQUEST"
2. Select 'MXSR' under Object Structure which will populate the Object Structure Sub-Records table
3. Click 'Save Enterprise Service' on the left under Common Actions

Repeat this sequence for PLUSISRDOMAIN. In Description fill in "Service Request class records from Tririga" and select 'MXDOMAIN' for the Object Structure.

##### End Point 
 Navigate to Integration -> End Points and click on the blue plus bitton at the top of the page
1. Under End Point fill in PLUSISREQ and in the Description fill in "AppConnect SERVICE REQUEST outbound to TRIRIGA"
2. Select 'HTTP' for Handler
3. Click on 'Save End Point' on the left side under More Actions which will populate the Properties for the End Point
4. Until the flows have a destination url, we can only fill in certain fields:
   - HEADERS: "Content-Type: application/json"
   - HTTPMETHOD: POST
5. Save the End Point


##### External System
 Navigate to Integration -> External System and open up the 'PLUSITRIRIGA' external system that was previously set up. Associate the Publish Channel and Enterprise Services to the External System
 
1. Add a new row in Publish Channel with the newly created PLUSIMXSR and link the PLUSISREQ End Point as well
2. Add two new rows with the newly created Enterprise Services. 

**Make sure both are enabled**


#### 4. Create a relationship in the SR object


<img width="1791" alt="Relationship" src="https://media.github.ibm.com/user/348712/files/48756b00-3805-11ed-8c9d-ccfc631e04a8">


Go to DB Config -> SR object -> Relationships. Add a 'New Row' and enter the following values:

|Field Name|Value  |
|--|--|
|Relationship | PLUSIREQCLASS |
|Child Object | ALNDOMAIN |
|Where Clause | `domainid= 'PLUSIREQCLASS' and value=:PLUSIreqclassid` |
|Remarks | Relationship to PLUSIREQCLASS Alndomain |

#### 5. Application Designer


<img width="1788" alt="App-Designer" src="https://media.github.ibm.com/user/348712/files/457a7a80-3805-11ed-9484-133899bdfac4">


Go to System Configuration -> Platform Configuration -> Application Designer

Search for 'SR'

Switch to the Service Request Tab and scroll down to the Service Request Details section

At the top, click the icon labeled Control Palette and add a Multipart Textbox at the top of the right section. Add these values within the properties of the Multipart Textbox. 

** Be sure that the PLUSIREQCLASSID Attribute is taken from the TICKET Object. **

|Field Name|Value  |
|--|--|
|Attribute | PLUSIREQCLASSID |
|Attribute for Part 2 | PLUSIREQCLASS.DESCRIPTION |
| Lookup | VALUELIST |
|Input Mode for Part 2 | Readonly |

Click 'Save Definition' after the changes are added.

### AppConnect
 
The configuration of AppConnect from the previous code pattern should provide the 'mxtririga' and 'trimaximo' accounts within AppConnect needed for the flows to work properly.

Download and import the .yaml files for 'PLUSIMXServiceReq2TRI' and 'PLUSITRIReqClass2MX' and keep the urls handy for a later step. Use the following table for the parameters:

|Parameter Name|Value  |
|--|--|
|mxUrl | http://[host]:[port]/meaweb/esqueue/PLUSITRIRIGA/PLUSIMXSR |
|triUrl | http://[host]:[port]/oslc/so/triAPICServiceRequestCF |
|mxDomain | PLUSIREQCLASS|

### TRIRIGA

#### Populate the domain created in the Maximo pre-requisites

Go to Tools -> System Setup and select Integration Object under the Integration heading

Select triRequestClass - APIC - HTTP Post from the table or create it if it is not present. Fill in the required sections:

|Field Name|Value  |
|--|--|
|Name | triRequestClass - APIC- HTTP Post |
|Scheme | Http Post |
| Direction | Outbound |
|Post Type | JSON |
|Http URL | [the PLUSITRIReqClass2MX url from AppConnect with the correct parameters outlined in the AppConnect section] |
| Request Method | POST |
|Content-Type | application/json |

**If the AppConnect instance is based on cloud, include the api key in the Headers. If the instance is on-prem, include your basic authorization in UserName and Password


Once the correct values are filled in, click Execute at the top of the window. The process will take a few minutes since there is a large amount of files, but once it is completed you can check that the batch processed correctly under the specified domain.

#### Filter

To filter the request from TRIRIGA to Maximo, update filter in the triAPICServiceRequest - OSLC - Outbound query to trigger for selected Request Classes.

<img src="https://media.github.ibm.com/user/348712/files/49a69800-3805-11ed-9062-01b4ebbd4cc4" alt="Tri Filter 1" >

<img src="https://media.github.ibm.com/user/348712/files/49a69800-3805-11ed-82e0-38c2fc635983" alt="Tri Filter 2" >

Current filter is set to Request Class not null.


## (Optional) Step 1. Sync a Person to TRIRIGA

<i> ***This step is optional. If you are starting from scratch, run this flow to sync the person in both systems. If data is already aligned and present, move on to the next step </i>

A service request needs a member of the TRIRIGA organization on both Maximo and TRIRIGA. Accomplish this by creating a person record that will be requesting the SR and assign them to the TRIRIGA org. 


The PLUSIMXPerson2TRI flow can help with this. Navigate to Administration -> Resources -> Person and click on 'New Person' under 'Common Activities' on the left side of the screen. Create this new Person record with a 'Site' field of TRIMAIN as well as a primary email. Save the record and the MXPerson2TRI flow will sync the data into TRIRIGA and this user will be used in Step 2.

<img src="https://media.github.ibm.com/user/348712/files/47dcd480-3805-11ed-872a-6025189ef38c" alt="Person Sync" >


## Step 2. Create a Service Request
### Maximo to TRIRIGA

Go to Service Desk -> Service Request and click on the blue plus sign to create a new service request.

<img src="https://media.github.ibm.com/user/348712/files/490e0180-3805-11ed-959f-b0d9f985d16f" alt="Service Request" >

Assign a user to Reported By and Affected Person (the user created in Step 1 should be used here). From here, a request classification needs to be selected and the value and description will populate in. Click 'Save Service Request' and the flow should fire.



### TRIRIGA to Maximo
 
 Go to Requests -> Manage Requests -> Electrical & Lighting and click the 'Add' button on the top right. 
 
 <img src="https://media.github.ibm.com/user/348712/files/4a3f2e80-3805-11ed-9020-b097f3cb1251" alt="Tri Service Request" >
 
 Fill in the required fields and click 'Submit' at the top right of the newly opened window. The flow should fire upon submission.
 

## Troubleshooting

Common Errors and their resolutions:

### Maximo
 
 Common errors found in the Maximo system
 
 Error | Cause
  ---|---
 401: Bad Request | This usually means an aspect of the request was not sent correctly- double check what is being sent as well as the flow in AppConnect to make sure everything is correct and running.
 

 
### AppConnect
 The best way to troubleshoot with AppConnect is to use the logging function. While the flow is stopped, add a 'Log' node into the flow from the 'Toolbox' tab.
 
 <img src="https://media.github.ibm.com/user/348712/files/47dcd480-3805-11ed-9c5f-f68af91cbc41" alt="Log Designer" >
 
 This will allow mapping of any field to the 'Logging' section of the application. Select Info for the Log level and then map the field that needs debugging. In this example the Request Object has been mapped to see what is being sent through the flow. Click the icon to the right of the Message Detail filed to map the desired field. 
 
 <img src="https://media.github.ibm.com/user/348712/files/46aba780-3805-11ed-8db5-67c866d28126" alt="Log Content" >
 
 The Log node will compile the message and read out here:
 
 <img src="https://media.github.ibm.com/user/348712/files/47dcd480-3805-11ed-9efb-a9cb2dd074d4" alt="Log Location" >
 
 Diagnose the response that shows up in this section to learn what might be causing the issue.
 
 <img src="https://media.github.ibm.com/user/348712/files/47443e00-3805-11ed-9e6e-7fe53c4465c3" alt="Log Dashboard" >
 

 
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

