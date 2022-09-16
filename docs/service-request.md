
# Service Request Configuration


### Summary

### Description

## Pre-requisites

This configuration assumes you have already completed the pre-requisites and steps outlined in the Maximo <-> TRIRIGA code pattern. [See here](https://developer.ibm.com/patterns/synchronize-databases-between-asset-workplace-management-solutions/) for those steps.


<details><summary><b>Maximo</b></summary>

Within Maximo, some initial changes to the database and Service Request application need to be completed in order for the integration to work properly. Errors may arise if these steps are not completed.

### 1. 

Service request is a view. The changes will be on the Ticket Table.

We need to create a Domain first that will link to an attribute on the Ticket table.

Domain: PLUSTREQCLASS
Description: Tririga service request class
Domain Type: ALN
Data Type: ALN
Length: 10

This domain needs to be populated in order to send a service request out of Maximo. (Batch info still being configured)

To start, head to Database Configuration and search for the 'SR' object. This is a view rather than a table. 

Head to the Ticket table. Go to the attributes section and create a new row:

Attribute: PLUSTREQCLASSID
Description: TRIRIGA request class of value
Type: ALN
Length: 10
Required: No
Domain: *Select the PLUSTREQCLASS Domain from the previous step

** Make sure the length of this attribute has the same length as the domain that is linked

Save the attribute. Apply the configuration changes to your database by switching on Admin mode and Apply Database Configuration

External System -> PlustTririga

Publish Channel: PLUSTMXSR
Enterprise System: PLUSTMXSR
 PLUSTSRDOMAIN
 Endpoint: PLUSTSREQ (needs to have mxUrl and triUrl in the url)

Make sure there is a relationship created before adding a field to the application. Go to DB Config -> SR object -> Relationships.

Relationship: PLUSTREQCLASS
Child Objct: ALNDOMAIN
Where Clause: `domainid= 'PLUSTREQCLASS' and value=:plustreqclassid`
Remarks: Relationship to PLUSTREQCLASS Alndomain

Application Designer -> SR

Switch to the Service Request Tab and scroll down to the Service Request Details section

Add TRIRIGA Request Classification as a Multipart Textbox. 

Attribute: PLUSTREQCLASSID 

**Make sure this attribute is from the SR Object and not the TICKET Object

Attribute for Part 2: PLUSTREQCLASS.DESCRIPTION
Lookup: VALUELIST
Input Mode for Part 2: Readonly



</details>
 
 <details><summary><b>AppConnect</b></summary>

We will be supplying the mxUrl as well as the mxDomain to the flow. Domain name is the name that was configured in the Maximo pre-requisite section (PLUSTREQCLASS)

</details>

<details><summary><b>TRIRIGA</b></summary>

We will run a batch command to populate the domain created in the Maximo pre-requisites

Go to Tools -> System Setup and select Integration Object under the Integration heading

Select triRequestClass - APIC - HTTP Post from the table or create it if it is not present. Fill in the required sections:

Name: triRequestClass - APIC- HTTP Post
Scheme: Http Post
Direction: Outbound
Post Type: JSON
Http URL: [the flow url from AppConnect with the correct parameters outlined in the AppConnect section]
Request Method: POST
Content-Type: application/json

**If the AppConnect instance is based on cloud, include the api key in the Headers. If the instance is on-prem, include your basic authorization in UserName and Password

Once the correct values are filled in, click Execute at the top of the window. The process will take a few minutes since there is a large amount of files, but once it is completed you can check that the batch processed correctly under the specified domain.

</details>

## Step 1. Create a Service Request in Maximo

Go to Service Desk -> Service Request and click on the blue plus sign to create a new service request.

You need to assign a user to Reported By and Affected Person. These users need to be part of the TRIRIGA organization in order to work properly.

From here, the request classification needs to be selected. The value and description will populate in. Click 'Save Service Request' and the flow should fire.

## Troubleshooting

