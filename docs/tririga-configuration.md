# Tririga Configuration for Envizi Integration

## Table of Contents

<!--ts-->
  * [Overview](#overview)
  * [Security Role Configuration](#security-role-configuration)
    * [Object Migration Access](#object-migration-access)
    * [Import OM Package](#import-om-package)
    * [Tririga API User Access](#tririga-api-user-access)
    * [Outbound Traffic](#outbound-traffic-from-tririga)
    * [Inbound Traffic](#inbound-traffic-to-tririga)
  * [Group Name Configuration](#group-name-configuration)
    * [Data Modeler](#data-modeler)
    * [Form Builder](#form-builder)
    * [Security Manager](#security-manager)
    * [Workflow Builder](#workflow-builder)
    * [My Reports/OSLC](#my-reports-and-oslc)
  * [How to Use](#how-to-use)
    * [TimeStamp Pre-Requisite](#time-stamp-pre-requisite)
    * [Using the Integration](#using-the-integration) 

## Overview
This solution adds four fields that hold the group name values and the path for the integration. It also makes sure that the payload is sent when there is an update.

## Security Role Configuration
In order to properly configure Tririga, a user needs to be configured with the proper security access for Object Migration and Tririga APIs

### Object Migration Access

Object Migration is a task managed by administrators. If a Tririga User needs to have full access to Object Migration in Tririga, access is granted at the group level. Follow the steps given in [Chapter 1](https://www.ibm.com/docs/en/SSFCZ3_11.2/pdf/pdf_tri_app_admin.pdf) to create a new security group.

1. Select the newly created group or desired existing group and switch to the **Access** tab and add the appropriate access for importation of Object Migration Packages. There are 2 panes in **Access**: **Object** and **Permissions**. 
2. Scroll and find the Object Migration Object on the left pane and click **Full Access** on the right pane. The users in this group, granted through the Members tab, will be able to import the Tririga API Object Migration package. 

<img width="1002" alt="Access Permissions" src="https://media.github.ibm.com/user/348712/files/baf46d80-1a45-11ed-924e-acda1e9bfe24">

*Access Permissions*


Additional information on [Tririga Security groups](https://www.ibm.com/docs/en/tap/3.6.1?topic=security-groups) 

### Import OM Package

Import the most recent [OM Package](https://github.com/IBM/tririga-api/tree/main/docs/ompackages) into the Tririga instance. Go to **Tools -> Administration -> Object Migration** and select **New Import Package** to begin the import process.

Please refer to [official IBM® Tririga documentation](https://www.ibm.com/docs/en/tap/3.6.1?topic=objects-object-migration-overview ) for more information on Object Migration

### Tririga API User Access

In order for AppConnect to be able to use Tririga APIs, it will need a user with certain permissions. These user's credentials will be configured in App Connect.

1. Create a new user by following the steps given in [Chapter 2](https://www.ibm.com/docs/en/SSFCZ3_11.2/pdf/pdf_tri_app_admin.pdf).

- Choose a security group or create a new group (refer to [the above OM Migration steps](#object-migration-access) if a new security group needs to be created) and add the newly created user to it.

- Add the permissions below for the new user's group:

  | Module        |     Business Object     |     Permissions    |
  |---------------|-------------------------|--------------------|
  | Location      |     triBuilding         |     Read           |
  | triAPIConnect |     triAPICTimestamp    | Read and Update    |
  
  
- Follow the steps given in [Chapter 1](https://www.ibm.com/docs/en/SSFCZ3_11.2/pdf/pdf_tri_app_admin.pdf) and add **any one of the licenses below** for the new user's group:<br>
  a. IBM TRIRIGA Portfolio Data Manager<br>
  b. IBM Facilities and Real Estate Management on Cloud Self Service<br>
  c. Any other license that grants access to the modules

Please refer to the [TRIRIGA Documentation on Security and Licenses](https://www.ibm.com/docs/en/tap/3.6.1?topic=platform-license-files) for additional information.
  
The user will now be able to interact with the proper Tririga Modules.

### Outbound Traffic from Tririga

For outbound traffic from Tririga, grant at least READ access on the Business Objects that will be used. The table below shows the various supported business Objects the API can pull from: 

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

### Inbound traffic to Tririga

For inbound traffic, Data Access needs to be enabled as well as Application Access permissions to the triAPIConnect Module or the individual Objects. To enable an API user to create a building, grant access to the triAPICBuilding Business object as shown below: 

<img width="1000" alt="Inbound Traffic Business Object" src="https://media.github.ibm.com/user/348712/files/e5633b80-3b58-11ed-842e-cea368ad86dd">

### Minimum requirements

For users to pull from these URLs, the minimum requirements are:

URL | Requirement
--|--
GET /oslc/spq/triAPICOutboundBuildingQC  | READ access to triBuilding Business object
GET /oslc/spq/triAPICTimeStampQC  | READ access to triAPICTimestamp Business Object 
POST /oslc/so/triAPICTimeStampRS/<ID> | Write access to triAPICTimestamp Business Object 

## Group Name Configuration


### Data Modeler

Go to Tools -> Builder Tools -> Data Modeler and using the Object Browser navigate to Location->triBuilding.

Revise the BO and add four fields: cstEnviziParentOneTX, cstEnviziParentTwoTX,
cstEnviziParentThreeTX and cstEnviziGroupNamePathTX

<img width="512" alt="Data Modeler Fields" src="https://media.github.ibm.com/user/348712/files/3ddd5780-09b5-11ed-9014-fbd85b21e6eb">

Name and Label should be the following:

Name | Label
--|--
cstEnviziParentOneTX | Envizi Group 1
cstEnviziParentTwoTX | Envizi Group 2
cstEnviziParentThreeTX | Envizi Group 3
cstEnviziGroupNamePathTX | Envizi Path

After entering these values, click 'Publish' to publish the BO

### Form Builder

Under Tools -> Builder Tools -> Form Builder, click on the Location module on the left side of the screen and then click on triBuilding.

Revise the triBuilding form by clicking 'Revise' in the top right corner of the pop-up

In the Navigation pane on the left side of the screen, click on triBuilding and then 'Add Tab'. Enter "cstEnvizi" as the 'Name' and 'Envizi' as the Label. Click Apply

Select this new tab and click on 'Add Section'

Enter "cstEnviziDetails" as the 'Name' and "Envizi Details" as the 'Label'. Click 'Apply'.

Now click on the newly created Section and select 'Add Field'. Select each of the four created business objects from the previous step as fields under "Envizi Details".

Select 'cstEnviziGroupNamePathTX' and modify 'Start Row' to 3 and 'Col Span' to 2 on the
properties window. Mark this field as "ReadOnly" and click 'Apply'.

The form should look like this:

<img width="1028" alt="Form Builder" src="https://media.github.ibm.com/user/348712/files/4f266400-09b5-11ed-9ca4-4253b2fa9baa">

Click on triBuilding on the left panel and then click on 'Sort Tab'. Move
the 'cstEnvizi' tab to the second position and click 'Apply'

<img width="256" alt="Form Builder Properties" src="https://media.github.ibm.com/user/348712/files/4f266400-09b5-11ed-8a73-f526a24a86db">

Publish the form

### Object Migration

Go to Import Migration and import package EnviziConfig.zip

To do that, click on New Import Package, and select the zip file and click Ok.

A new window will be displayed. If it is not displayed, just select the package from the list.

On this package, click on Validate and wait for the validation to complete. If no errors are displayed, import the package.

### Security Manager

Go to Tools -> Administration -> Security Manager

This application sets who can and cannot access this newly created tab. Click on the desired group, and navigate to the 'Access' tab

On this tab select Location -> triBuilding -> cstEnvizi

Choose the access level for the group and 'Save'

<img width="423" alt="Security Manager Access" src="https://media.github.ibm.com/user/348712/files/4fbefa80-09b5-11ed-99e5-b92fca943ddf">

### Workflow Builder

Go to Tools -> Builder Tools -> Workflow Builder. Select Location -> triBuilding.

Within the Location object, search for the existing Workflow "triBuilding - Synchronous - Permanent Save
Validation".

Revise this workflow and after Call Module Level Validation add a new WF
call to "triBuilding - Update Envizi fields with defined options" like displayed below:

<img width="736" alt="WF call in Update Envizi Path" src="https://media.github.ibm.com/user/348712/files/374fdf80-09b7-11ed-8f47-3c588b667a58">

It will be defined as the image below:
 
<img width="1062" alt="Call Envizi" src="https://media.github.ibm.com/user/348712/files/478e2400-4e19-11ed-81eb-e5462bbcdff1">

Click on the 'Start' task at the top and publish the workflow

### My Reports and OSLC

Go to My Reports and navigate to 'System Reports'. Add those four new fields to the existing query
"triAPICBuilding - OSLC -- Outbound" by clicking the 'Columns' tab and adding the four fields like below:

<img width="800" alt="System-Report-Columns" src="https://media.github.ibm.com/user/348712/files/f0d51a00-4e19-11ed-9e28-a658b12af215">

Save the report.

Open Tools -> System Setup -> Integration -> OSLC Resource manager
and search for "triAPICOutboundBuildingRS". On this resource, add the four
new fields either individually or using 'Import all Fields'

<img width="683" alt="OSLC Manager" src="https://media.github.ibm.com/user/348712/files/73387400-09ba-11ed-912f-569fce1dc420">

### Navigation Builder

Go to Tools->Builder Tools-> Navigation Builder and find TRIRIGA Global Menu(or the menu associated to the user that will need access to the app). Select and click Edit

<img width="700" alt="Navigation-builder-1" src="https://media.github.ibm.com/user/348712/files/27c34480-3a95-11ed-89ca-17ce43fdeb1d">

On navigation Items section, expand Landing Page –Tools -> Menu Group –System Setup. Select Menu group –Integration and expand Navigations Item Library

<img width="700" alt="Navigation-builder-2" src="https://media.github.ibm.com/user/348712/files/272aae00-3a95-11ed-85a2-1cb9a42d6900">

Search for Envizi, select the item and click on Add to Collection

<img width="414" alt="Navigation-builder-3" src="https://media.github.ibm.com/user/348712/files/272aae00-3a95-11ed-8777-57d9809f75dd">


Click Save. Logout and Login again to the system

## How to use

### Time Stamp Pre-requisite

The triAPICTimestamp is a tririga record needed to set the baseline for when API connect runs for the first time. 

To enable this functionality go to My Reports -> System Reports and search for Timestamp in the 'Name' section. Run the system report "triAPICTimestamp – Display – Manager Query" as shown below:

<img width="1792" alt="Time-Stamp-1" src="https://media.github.ibm.com/user/348712/files/e5fbd200-3b58-11ed-8cb3-21aef581d0d3">
 
Click Add, and create a new record without details, as shown below, and close it

<img width="800" alt="Time-Stamp-2" src="https://media.github.ibm.com/user/348712/files/6a495780-3b4f-11ed-940d-137135ba303d">

The default date and time the record gets automatically applied to the record, and consequent opening of the record shows the default date and time as shown below:

<img width="800" alt="Time-Stamp-3" src="https://media.github.ibm.com/user/348712/files/69b0c100-3b4f-11ed-95fc-1848213524cf">

### Using the Integration

This tool will allow user to make changes on this new Envizi group name fields. But we must consider the existing records too. If you want those records to be populated, there is a patch helper workflow that can handle that.

The first this that must be defined is which fields will be used to populate groups 1, 2 and 3 to be used on envizi. To access the envizi tool, go to Tools -> System Setup -> Integration -> Envizi Integation.

When you open the page, the fieldswill be displayed as group1/group2/group3. By default the values are World Region/Country/City. The field Envizi Hierarchy Path shows how the envizi groupnames will be configured according to the selected option. 

Enable Envizi checkbox is available too. The envizi tab will be displayed only when this checkbox is marked

One more item that must be configured is the Number of levels to be used on the envizi configuration. Envizihierarchy path will match this selection.

<img width="900" alt="How-to-use" src="https://media.github.ibm.com/user/348712/files/21cd6380-3a95-11ed-800d-aa0c22556421">

Also, notice that there is a section named “Active/Retire with missing data”and “Draft/Revision with Missing Data”. This section will list the buildings that don’t have data defined for envizi group 3, so it means that no envizi group will be populated on those buildings.


You can filter to change only the desired records by changing query “cst -triBuilding -Query -Get All Buildings for envizi”. The list of buildings displayed on this query will be the listof buildings that the patch helper will modify.

To use the tool, just select the desired envizi group names and click Save. On the moment Save is triggered, all buildings will be populated with the desired options. This process may take a few minutes depending on how many buildings you have on your system. After that envizi groups and path will be updated according to the selections made on Envizi Integration page.

<img width="900" alt="How-to-use-2" src="https://media.github.ibm.com/user/348712/files/27c34480-3a95-11ed-9dc8-b6347de21bee">

Also, every time a building is saved and there are changes on the defined fields, or a new building is created, the envizi groups and path will be modifiedaccording to the selected options. You can find the groups on tab Envizi on the building record
