# Tririga Configuration for Envizi Integration

## Table of Contents

<!--ts-->
  * [Import OM Package](#import-om-package)
  * [Data Modeler](#data-modeler)
  * [Form Builder](#form-builder)
  * [Security Manager](#security-manager)
  * [Workflow Builder](#workflow-builder)
    * [Optional: Patch Helper](#optional-step)
  * [My Reports/OSLC](#my-reports-and-oslc)
  * [How to Use](#how-to-use)


This solution adds four fields that hold the group name values and the path for the integration. It also makes sure that the payload is sent when there is an update.

### Import OM Package

Import the most recent [OM Package](https://github.com/IBM/tririga-api/tree/main/docs/ompackages) into the Tririga instance. Go to Tools -> Administration -> Object Migration and select 'New Import Package' to begin the import process.

Here are the manual changes that need to be done:

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

In the Navigation pane on the left side of the screen, click on 'Add Tab' and enter "cstEnvizi" as the 'Name'. Click Apply

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

### Security Manager

Go to Tools -> Administration -> Security Manager

This application sets who can and cannot access this newly created tab. Click on the desired group, and navigate to the 'Access' tab

On this tab select Location -> triBuilding -> cstEnvizi

Choose the access level for the group and 'Save'

<img width="423" alt="Security Manager Access" src="https://media.github.ibm.com/user/348712/files/4fbefa80-09b5-11ed-99e5-b92fca943ddf">

### Workflow Builder

Three Workflows will need to be edited.

**1. Go to Tools -> Builder Tools -> Workflow Builder. Select Location -> triBuilding.**

 Create a new Workflow name "cst - triBuilding - Update Envizi path"

   This Workflow should look like this:

   <img width="564" alt="Workflow" src="https://media.github.ibm.com/user/348712/files/4f266400-09b5-11ed-9919-1f2fb10fd860">

   Add to the workflow with the following steps:

   - Create the first 'Switch Condition' and enter the following condition:

   <img width="484" alt="1st Switch Expression" src="https://media.github.ibm.com/user/348712/files/4fbefa80-09b5-11ed-8738-62e0ccb761f9">

   - Create two 'Modify Tasks' and place them on either side of the switch. The 'Modify Parent One' task should be on the left and have this mapping:

   <img width="569" alt="Modify Parent One Mapping" src="https://media.github.ibm.com/user/348712/files/4fbefa80-09b5-11ed-9306-f120dffa38e8">

   - The 'Set Blank' task is the other condition of the first switch and it should have this mapping:

   <img width="680" alt="Set Blank Task" src="https://media.github.ibm.com/user/348712/files/50579100-09b5-11ed-80b3-0bc45efda653">

   - The second switch should flow from the 'Modify Parent One' task and have the following condition:

   <img width="473" alt="2nd Switch Expression" src="https://media.github.ibm.com/user/348712/files/4fbefa80-09b5-11ed-8db9-31cb358f4b60">

   - Create the 'Modify Parent Two' task and give it this mapping:

   <img width="629" alt="Modify Parent Two Mapping" src="https://media.github.ibm.com/user/348712/files/361eb280-09b7-11ed-9e3f-00b94068ebd6">

   <img width="587" alt="Parent Two Inputs and Formula" src="https://media.github.ibm.com/user/348712/files/36b74900-09b7-11ed-875c-e60f87afefcf">

   - The third switch should flow from 'Modify Parent Two' and have the following condition:

   <img width="474" alt="3rd Switch Expression" src="https://media.github.ibm.com/user/348712/files/36b74900-09b7-11ed-89c5-71da43fe6e50">

   - The 'Modify Parent Three' task should have this mapping:

   <img width="487" alt="Modify Parent Three Mapping" src="https://media.github.ibm.com/user/348712/files/36b74900-09b7-11ed-8058-03b7ed9d8bc6">

   <img width="589" alt="Modify Parent Three Mapping-2" src="https://media.github.ibm.com/user/348712/files/374fdf80-09b7-11ed-80d9-3911aa5f031e">

   Publish "cst - triBuilding - Update Envizi path" workflow

**2. Within the Location object, search for the existing Workflow "triBuilding - Synchronous - Permanent Save
Validation".**

Revise this workflow and after Call Module Level Validation add a new WF
call to "cst - triBuilding - Update Envizi path" like displayed below:

<img width="736" alt="WF call in Update Envizi Path" src="https://media.github.ibm.com/user/348712/files/374fdf80-09b7-11ed-8f47-3c588b667a58">

Publish the workflow

**3. Navigate to Data Utilities -> cstEnviziUtility on the left panel and revise
Workflow cst - cstEnviziUtility - Synchronous - Update Buildings with
Group Names**

This workflow has some mapping missing. Update the three Modify Tasks:

- Open Modify Group Name1 modify task and enter the following

<img width="759" alt="Group Name1 Modify Task" src="https://media.github.ibm.com/user/348712/files/36b74900-09b7-11ed-8722-b1dbeba27e23">

- Open Modify Group Name2 modify task and enter the following

<img width="768" alt="Group Name2 Modify Task" src="https://media.github.ibm.com/user/348712/files/374fdf80-09b7-11ed-96b8-9634174db7a1">

<img width="608" alt="Group Name2 Modify Task-2" src="https://media.github.ibm.com/user/348712/files/361eb280-09b7-11ed-8044-c0861e85aa4d">

- Open Modify Group Name3 modify task and enter the following

<img width="638" alt="Group Name3 Modify Task" src="https://media.github.ibm.com/user/348712/files/35861c00-09b7-11ed-8a4b-8f4ed004675e">

<img width="600" alt="Group Name3 Modify Task-2" src="https://media.github.ibm.com/user/348712/files/361eb280-09b7-11ed-84be-438ae45f0d7f">

Publish this workflow

### Optional Step

*This is an optional step if there are old records that need to be updated with the patch helper.*

Modify the Workflow "cst - triPatchHelper -
Update All Envizi location fields", located on triHelper ->
triPatchHelper

Revise the Workflow and modify the following tasks:

- Modify Parent One task(if a custom value is desired, set it on
these mappings):

<img width="529" alt="Patch Helper Parent One" src="https://media.github.ibm.com/user/348712/files/73d10a80-09ba-11ed-958f-6c3df5f47ad3">

- Modify Parent Two task(if a custom value is desired, set it on
these mappings):

<img width="619" alt="Patch Helper Parent Two" src="https://media.github.ibm.com/user/348712/files/73d10a80-09ba-11ed-9bca-cdf074e7cf37">

- Modify Parent TwoPath task:

<img width="629" alt="Modify Parent Two Mapping" src="https://media.github.ibm.com/user/348712/files/361eb280-09b7-11ed-9e3f-00b94068ebd6">

<img width="587" alt="Parent Two Inputs and Formula" src="https://media.github.ibm.com/user/348712/files/36b74900-09b7-11ed-875c-e60f87afefcf">

- Modify Parent Three task(if a custom value is desired, set it on
these mappings):

<img width="592" alt="Patch Helper Parent Three" src="https://media.github.ibm.com/user/348712/files/73387400-09ba-11ed-8a8a-9f412982a9ce">

- Modify Parent Three Path task:

<img width="487" alt="Modify Parent Three Mapping" src="https://media.github.ibm.com/user/348712/files/36b74900-09b7-11ed-8058-03b7ed9d8bc6">

<img width="589" alt="Modify Parent Three Mapping-2" src="https://media.github.ibm.com/user/348712/files/374fdf80-09b7-11ed-80d9-3911aa5f031e">

- SetBlank:

<img width="683" alt="Patch Helper Set Blank 1" src="https://media.github.ibm.com/user/348712/files/73387400-09ba-11ed-81fa-ab59763a2de0">

- Set Parent Two and Three Blank:

<img width="679" alt="Patch Helper Set Blank 2" src="https://media.github.ibm.com/user/348712/files/729fdd80-09ba-11ed-95df-381836697eed">

- Set Parent Three Blank:

<img width="674" alt="Patch Helper Set Blank 3" src="https://media.github.ibm.com/user/348712/files/73387400-09ba-11ed-9aea-16908b7c9861">

Save and publish the workflow

### My Reports and OSLC

Go to My Reports and navigate to 'System Reports'. Add those four new fields to the existing query
"triAPICBuilding - OSLC -- Outbound" and save the report.

Open Tools -> System Setup -> Integration -> OSLC Resource manager
and search for "triAPICOutboundBuildingRS". On this resource, add the four
new fields either individually or using 'Import all Fields'

<img width="683" alt="OSLC Manager" src="https://media.github.ibm.com/user/348712/files/73387400-09ba-11ed-912f-569fce1dc420">

### How to use

This tool will allow user to make changes on this new Envizi group name fields, but existing records must be considered too. To populate those records, there is a patch helper workflow that can handle that.

This workflow by default adds the groupnames with the value from Hierarchy Path. So for example, if the HierarchyPath is "Location/NorthAmerica/USA/Texas/Dallas", then the group names will be:

Name | Value
--|--
Envizi Group 1 | Dallas
Envizi Group 2 | Texas
Envizi Group 3 | USA
Envizi Path | USA/Texas/Dallas

You can customize this values by changing workflow "cst - triPatchHelper - Update All Envizi location fields" like described before

Also, you can filter to change only the desired records by changing
query "cst - triBuilding - Query - Get All Buildings for envizi". The
list of buildings displayed on this query will be the list of buildings
that the patch helper will modify

To run this patch helper, go To Administration -> Data
Integration to run patch helper. Upload a file using triNameTx = Envizi

<img width="682" alt="Data Integrator" src="https://media.github.ibm.com/user/348712/files/7469a100-09ba-11ed-9111-c30331bb797b">

TXT file should have this content

<img width="82" alt="TXT file" src="https://media.github.ibm.com/user/348712/files/73d10a80-09ba-11ed-97e9-d1655dfec1f8">

After that the records will be populated with the initial values
defined. Please note that Patch helpers can be executed only once. To run this patch helper one more time, Envizi will need to be removed from the list that can be found using the query `"triPatchHelper - Display - All"


There are two possible scenarios to use this solution:

1.  Modify only one record

    a.  Open the building to change and click on tab Envizi.
        Modify the values and save the record

2.  Modify multiple records

    a.  Go to My Reports, and run report "cst -
        cstEnviziUtility - QUERY - Update Envizi GroupNames"

    b.  Click on 'Add'

    c.  Define the values for each groupName and filter the buildings
        to apply on the Buildings section

    d.  Select the buildings and click on Create.

    e.  Click on Mass Update
