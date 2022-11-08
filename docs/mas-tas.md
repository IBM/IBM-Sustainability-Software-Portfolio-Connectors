# Portfolio Data Integration - To be deleted

### Summary

Enable automated synchronization of portfolio data between TRIRIGA and Maximo Asset Management systems to integrate asset and facility management operations.

### Description

In this code pattern, learn how to synchronize bi-directional portfolio data of IBM TRIRIGA and IBM Maximo Asset Management using App Connect Designer flows for synchronizing People, Locations, and Assets. Using the provided flows, enable a variety of use cases: from a simple trigger-action integration that updates or populates the records in the other application, to more complex scripted cron jobs that orchestrate these objects into their respective primary system of record.

<img alt="MX2TRI Graph"src="https://media.github.ibm.com/user/348712/files/ea935400-3801-11ed-9dc2-9f5a0d4e38a8">

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

## Prerequisites

### 1. Application Designer Changes


<!-- <img width="1788" alt="App-Designer" src="https://media.github.ibm.com/user/348712/files/457a7a80-3805-11ed-9484-133899bdfac4"> -->


Go to **System Configuration -> Platform Configuration -> Application Designer**

#### Person

1. Search for **PERSON**, switch to the **Person** Tab and select a section to add three new boxes.

2. At the top, click the icon labeled **Control Palette** and add a Multipart Textbox at the bottom of the section. Add the values from the below table within the properties of the Multipart Textbox and click **Save Definition** 


> **Note**
> *Be sure that the Attributes are taken from the PERSON Object.*

|Type of Control | Label | Attribute | Attribute for Part 2 (*If Multipart Textbox*) | Lookup | Input Mode for Part 2 (*If Multipart Textbox*)
|--|--|--|--|--|--|
| Multipart Textbox |TRIRIGA Location Path | PLUSIPRIMARYLOCPATH | PLUSILOCATIONPATH.DESCRIPTION | VALUELIST | Readonly
| Multipart Textbox | TRIRIGA Primary Organization | PLUSIORGPATH | PLUSIORGANIZATIONPATH.DESCRIPTION | VALUELIST | Readonly
|Textbox | TRIRIGA Record ID | EXTERNALREFID (*From the Person Object*) | N/A | N/A | N/A


Follow the same directions using the following information for Asset and Location

#### Asset

1. Search for **ASSET** in Application Designer

> **Note**
> *Be sure that the Attributes are taken from the ASSET Object.*

|Type of Control | Label | Attribute | Attribute for Part 2 (*If Multipart Textbox*) | Lookup | Input Mode for Part 2 (*If Multipart Textbox*)
|--|--|--|--|--|--|
| Multipart Textbox |TRIRIGA Building Equipment Spec | PLUSIASSETSPECNAME | PLUSIASSETSPECCLASS.DESCRIPTION | VALUELIST | Readonly
| Multipart Textbox | TRIRIGA Primary Organization | PLUSIORGPATH | PLUSIORGANIZATIONPATH.DESCRIPTION | VALUELIST | Readonly
| Multipart Textbox | TRIRIGA Location Path | PLUSILOCPATH | PLUSILOCATIONPATH.DESCRIPTION | VALUELIST | Readonly
|Textbox | TRIRIGA Record ID | EXTERNALREFID (*From the Asset Object*) | N/A | N/A | N/A

#### Location

Search for **LOCATION** in Application Designer

> **Note**
> *Be sure that the Attributes are taken from the LOCATION Object.*

|Type of Control | Label | Attribute | Attribute for Part 2 (*If Multipart Textbox*) | Lookup | Input Mode for Part 2 (*If Multipart Textbox*)
|--|--|--|--|--|--|
| Multipart Textbox |TRIRIGA Space Classification | PLUSISPACECLASSIFICATION | PLUSISPCCLASSIFICATION.DESCRIPTION | VALUELIST | Readonly
| Multipart Textbox | TRIRIGA Parent Location | PLUSIPARENTLOCATION | PLUSIPARENTPATH.DESCRIPTION | VALUELIST | Readonly
|Textbox | TRIRIGA Record ID | EXTERNALREFID (*From the Location Object*) | N/A | N/A | N/A


## Step 1 - Select an App Connect Flow for deployment

1. Locate the .yaml file for the deployment direction (MX2TRI or TRI2MX) based on the system of record and download to the local machine. From the previous example, use the MX2TRI flow in the Person row.

Asset | Maximo | TRIRIGA
---|---|---
Person | [MX2TRI](/docs/MAX2Tririga/PLUSTMXPerson2TRI.yaml) | [TRI2MX](/docs/TRI2Maximo/PLUSTTRIPerson2MX.yaml)
Asset | [MX2TRI](/docs/MAX2Tririga/PLUSTMXAsset2TRI.yaml) | [TRI2MX](/docs/TRI2Maximo/PLUSTTRIAsset2MX.yaml)
Location | [MX2TRI](/docs/MAX2Tririga/PLUSTMXLocation2TRI.yaml) | [TRI2MX](/docs/TRI2Maximo/PLUSTTRISpace2MX.yaml)

Make sure to check the [Mapping Document](/docs/TRIRIGA_Maximo_Field_Mapping-Final.xlsx) that the correct fields are being mapped in the selected flow.


## Step 2 - Import the Selected Flow in App Connect

Follow the below steps to import the flow.

### Import Steps (ACE)

  1. From the App Connect Dashboard, click **New** and select **Import Flow** from the drop down menu.


  2. Either drag and drop or select the flow for import. In this example, the MX2TRI Person flow will be used.

  3. The flow should now be uploaded onto the App Connect instance. From this screen navigate using the **Edit flow** button to see the individual nodes of this flow. Be sure to select the HTTP account that was configured for Maximo to TRIRIGA for the connector. 


  4. Click **Done** on the top right of the screen then click on the three dots in the top right corner and select **Start API**.

  5. Go to the **Test** tab once it shows that the flow is **Running** and select the **POST** option on the left side of the screen. Click on **Try It** and grab the url and security credentials from this screen for the next step.
  
  <img src="https://media.github.ibm.com/user/348712/files/e23b1900-3801-11ed-9bf0-334b0f78fb60" alt="App Connect Dashboard"> 

  *Step 1: App Connect Dashboard*

  <img width="300" alt="Uploaded_Flow" src="https://media.github.ibm.com/user/348712/files/ecf5ae00-3801-11ed-886a-b5bb80a59523">
  
  *Step 2: Import a Flow* 

  <img width="1771" alt="Completed_Flow" src="https://media.github.ibm.com/user/348712/files/e6673680-3801-11ed-9883-661b31bad8b5">

  *Step 3: Completed Flow*

  <img width="251" alt="Start_API" src="https://media.github.ibm.com/user/348712/files/ea935400-3801-11ed-826c-318a039b6bec">
  
  <img src="https://media.github.ibm.com/user/348712/files/e49d7300-3801-11ed-83f4-a6fd5e41a3da" alt="App Connect Config Full" >

### Import Steps (App Connect Cloud)
  1. If your instance of AppConnect is through the cloud, your page will look a bit different
  
  2. The interface is mostly the same, but instead of **Try It** you'll see a tab called **Manage**. This page contains a few important pieces of information that you'll need to complete the configuration. First, at the top of the page under **API Info** you'll see a field called **Route**. Piece this together with the correct path of the flow that you're implementing in order to create the proper flow url.
  
  3. Collect the API key at the bottom of the page from a field called **Sharing outside of Cloud Foundry organization**. Click on **Create API key and documentation link**. Provide a name and it will generate an apikey for you to use with this flow along with a documentation link that looks like the **Test** page from the on-prem configuration.
  
  4. The end of the url at the top of the page should have the path that will complete the route url. Copy the end of the url that begins with **tri** and add it to the piece gathered earlier. The 6 character string that precedes the path on the documentation should match the last 6 characters of the route piece from before.

  
  <img src="https://media.github.ibm.com/user/348712/files/e36c4600-3801-11ed-8744-4aa3b105af0d" alt="AppConnect Cloud Manage Tab" >
  *Step 1: Manage Tab*
  
  <img src="https://media.github.ibm.com/user/348712/files/e404dc80-3801-11ed-95de-e332d10a5597" alt="AppConnect Cloud Route" >

  *Step 2: Cloud Route*

  <img src="https://media.github.ibm.com/user/348712/files/e49d7300-3801-11ed-9084-943aa4afe16c" alt="AppConnect Cloud Sharing" >

  *Step 3: API key creation*
  
  <img src="https://media.github.ibm.com/user/348712/files/e2d3af80-3801-11ed-8ff7-4e319d52da99" alt="AppConnect Cloud Documentation Link" > 

  *Step 4: API Documentation Link*

## Step 3 - Configure Maximo and TRIRIGA instances with App Connect urls

Using the credentials from the end of Step 2, populate the instances of Maximo and TRIRIGA with the correct End Points of the recently created flow.

### Maximo

  1. From the main page of Maximo, click the menu icon on the top left and navigate to **Integration -> End Points**.

  2. Fill in the properties with the url, username, and password from Step 2.
  
  3. Click the **Test** button at the bottom right of the screen and send a simple {"hello":"world"}. With the proper configuration, there will be an expected error that ends with **Bad Request**. If there is a different error than Bad Request in the Response window, refer to the Troubleshooting section to debug.
  
*If the instance is based in the cloud, there is a slight difference in authorization.*
  
  1. Since the authorization is with an api key, remove the basic authorization values (USERNAME & PASSWORD) and in the **HEADERS** field add the API key after Content-Type. The **Headers** field should look something like this:
  
  Content-Type: application/json, X-IBM-Client-Id: [apikey from step 2]

  <img src="https://media.github.ibm.com/user/348712/files/e830fa00-3801-11ed-8366-1d1c57561fba" alt="Maximo End Point Navigation" >

  *Step 1: Maximo End Point app*
  
  <img src="https://media.github.ibm.com/user/348712/files/e8c99080-3801-11ed-91c8-96763b471911" alt="Maximo End Point Properties" >
  
  *On-prem End Point properties*

  <img src="https://media.github.ibm.com/user/348712/files/e5360980-3801-11ed-99bb-64636c034d61" alt="Cloud End Point" > 

  *Cloud End Point properties*
 
 
### TRIRIGA

1. From the main page of TRIRIGA, click on **Tools -> System Setup -> Integration -> Integration Object**. Under the **Name** column, type in **apic**, and select the integration object that pertains to the record that is getting sent. 

2. Click on the object and fill in the credentials in the pop-up box.
 
<img src="https://media.github.ibm.com/user/348712/files/ecf5ae00-3801-11ed-9ea5-af57b4059a25" alt="TRIRIGA End Point">

*TRIRIGA End Point*


## Step 4 - Test the Flow

With these 3 steps completed, test the flow with a payload.

1. Head back to the **Try It** page in the deployed flow (or the documentation page if cloud-based) and scroll down to the bottom of the page under Parameters.

2. In here, fill in **mxUrl** and **triUrl** with the urls for the respective applications, generate a test payload to send through the flow and monitor if the information populates in the desired application.

3. If there is a response other than 200 from the Test, refer to Troubleshooting.

<img src="https://media.github.ibm.com/user/348712/files/e2d3af80-3801-11ed-9e0e-0aba1b46eefa" alt="App Connect Test" >

*App Connect Test*

## References
[Mapping Document](/docs/TRIRIGA_Maximo_Field_Mapping-Final.xlsx)

## Troubleshooting

### Common errors that arise from App Connect

 
 #### Testing the whole flow
 
 - If there is a 404 Not Found error when trying to test the flow, this could mean that the flow is not running. Double check to make sure the flow shows a green dot and says **Running** after edits have been made.
 
 - If there is a 400 Bad Request error when testing the flow, this could mean the wrong account configurations. Double check to make sure the Accounts from the App Connect pre-requisite section are correct.
 
### Common errors that arise from Maximo
 
 #### Testing the End Point
 
 - If an error reads "The response code received from the HTTP request from the endpoint is not successful.", this is related to the configured End Point. Double check and make sure the values in **End Points** are correct. Make sure there are no accidental spaces at the beginning or end in the event of the values being copy/pasted.
 
 - If an error comes back as a **PKSync error**, this is related to the certificates in WebSphere. Double check and confirm that the certificates from Step 4 are correctly configured. 
 
### Common errors that arise from TRIRIGA
 
 - Clear OSLC Cache in TRIRIGA Admin Console in case the integrations do not work in intended manner.
 
