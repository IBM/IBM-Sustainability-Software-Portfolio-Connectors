# Turbonomic - Envizi Integration

## Table of contents

<!--ts-->
  * [Use Case Example](#use-case-example)
  * [Connector Architecture](#connector-architecture)
  * [Data Mapping](#data-mapping)
  * [App Connect Flows](#app-connect-flows)
  * [Installation & Configuration Guide](#installation--configuration-guide)
  * [Before you begin you will need:](#-before-you-begin-you-will-need)
  * [Installation Steps Overview](#installation-steps-overview)
  * [Troubleshooting](#troubleshooting)

  <!--te-->
***

## Use Case Example
Sustainability Manager can view and compare the energy usage of all data centres within Envizi. They can understand energy efficiency among their data centres by number of active hosts, by number of VMs. They can understand the density of VMs to Hosts in each data centre to identify opportunities to optimize efficiency.

***


## Connector Architecture

![Turbonomic Connector Architecture](https://media.github.ibm.com/user/375131/files/95555e06-3639-47bb-83ab-03bb152fadd6)

***

## Data Mapping

The image below illustrates the type of data that is being sent by the API and App Connect Flows.

![Turbonomic Data Mapping](https://media.github.ibm.com/user/375131/files/25557ce0-3cd5-435f-8228-407f1de5f1d7)

***

## App Connect Flows

- Included with this connector are two flows that export locations and accounts, along with all the required fields they contain. 
- The table below shows the naming convention for these flows and the current integration use case.

File | Flow | Destination | Operation
-- | -- |--|--
TurbonomicLocations.yaml | Locations | Turbonomics to Envizi  | Depending on configuration Changes only or Bulk initial load
TurbonomicAccounts.yaml  | Accounts | Turbonomics to Envizi  | Depending on configuration Changes only or Bulk initial load

***
  
## Installation & Configuration Guide
## &nbsp; Before you begin you will need:
   - Turbonomic version 8.6.6 or higher
   - An instance of App Connect with the Designer component.
## Installation Steps Overview
1. App Connect Configuration<br>
   a. Add Accounts ([reference document](https://www.ibm.com/docs/en/app-connect/containers_cd?topic=scratch-connecting-accounts))<br>
   b. Importing Flow in App Connect ([reference document](https://www.ibm.com/docs/en/app-connect/containers_cd?topic=designer-exporting-importing-flows#importingflow))<br>

2. Turbonomics Configuration<br>
   a. Create user <br>
   b. Configure tags <br>
   
## Part 1. App Connect Configuration
 
   _**Note**: When configuring App Connect you will have the option to set up a secure connection with a self-signed certificates, a private network, or both._

  1. Add Account for "Amazon S3" connector
     - Credentials will be given by Envizi
  2. Add an Account for "HTTP" connector
     - This account will be used for Authentication
        - **API Key**: Enter the Turbonomic credentials in this format `<USERNAME>&password=<PASSWORD>`
            - e.g., If the username is `johndoe` and the password is `TheCakeIsALie`, enter `johndoe&password=TheCakeIsALie`
        - **API Key Location**: Select "body URL encoded"
        - **API Key Name**: Enter the text `username` and **not** your actual username.
     ![Account Configuration example](https://media.github.ibm.com/user/375131/files/c26d7c80-5142-11ed-87b2-d32c7d2798b8)

  3. Add another Account for "HTTP" connector
     - This account will be used for other API calls
     - Leave all authentication related fields empty
  4. Import the flows
        - Navigate to the flows below and use the link to their raw files for importing in App Connect:
        - [TurbonomicLocations](./AppConnect%20Flows/TurbonomicLocations.yaml)
        - [TurbonomicAccounts](./AppConnect%20Flows/TurbonomicAccounts.yaml)
  5. Configure the flow to use the right accounts
        - For all Amazon S3 nodes in the flow, select the account created in Step 1 
        - For the first HTTP node in the flow, select the account created in Step 2
        - For all other HTTP nodes in the flow, select the account created in Step 3
  6. Configure the scheduler
        - Click on the "Scheduler" node

            <img width="659" alt="Scheduler node" src="https://media.github.ibm.com/user/375131/files/ea694d80-01ed-11ed-8852-16b625339675">

        - Configure the schedule as needed

            <img width="659" alt="Scheduler configuration 2" src="https://media.github.ibm.com/user/375131/files/00c3d900-01ef-11ed-86aa-29008ff86450">

     - The TurbonomicAccounts flow must be executed daily, preferably at UTC midnight between 00:05 and 00:30
     - The TurbonomicLocations flow can be configured to run once a month or even On-demand as described [here](#on-demand-data-pull)
     - **Note:** By default, the flow will run when it is started. To change this behavior, untick the below checkbox in the "Scheduler" node. It is recommended to keep this checked for On-demand data pull
    
       ![Run on Start checkbox](https://media.github.ibm.com/user/375131/files/f010f080-5157-11ed-8325-66c9c74f6243)
   
  7. Configure the flow parameters
        - Click on the "Set variable" node
     ![Set variable configuration](https://media.github.ibm.com/user/375131/files/09576580-50a2-11ed-8f46-b3beb969f206)

        - In Variable > config > customer, enter the value provided by Envizi
        - In Variable > config > url, enter URL for the Turbonomic instance
             - e.g., https://example.com:9080

        - Scroll to the Amazon S3 node and click on it to open the configuration
         ![S3 Configuration](https://media.github.ibm.com/user/375131/files/56c06800-4bb4-11ed-9aa6-c6758d61300a)

        - From the bucket dropdown, select the bucket name provided by Envizi
        - Perform these actions on all Amazon S3 nodes in the flow
  8. Start the flow
        - Once the flow it started, it will run as configured in scheduler. However, the flow will only collect data from the day it has been started.
        - In order to pull any historical data, additional configurations described in [On-demand Data Pull](#on-demand-data-pull) need to be done.
## Part 2. Turbonomic Configuration

- Turbonomic v8.6.6 or higher is required for this integration to work
- Create a user in Turbonomic with "Observer" role. The App Connect will be configured to use this user's credentials to fetch the necessary data.
- For accurate emission calculations from Envizi, add the following Tag Categories in vCenter and add their values as tags to the Data Centers:
    - `Country`: Name of Country
    - `Latitude`: Latitude in Decimal Degrees format
    - `Longitude`: Longitude in Decimal Degrees format

    ![vCenter Tags](https://media.github.ibm.com/user/375131/files/0eb7bd00-5141-11ed-9b2c-4bde7c361f33)
- By default, Envizi will use the Data Center name configured in Turbonomic/vCenter. To change this, Tag Category `EnviziAlternateName` can be added with the desired display name as its value.
- Envizi Locations (Data Centers in this case) need unique display names. If there are any Data Centers with same names, they should be changed from vCenter or Tag Category `EnviziAlternateName` should be added to the Data Center(s) with different name(s)

_**Note:** Tags sync from vCenter to Turbonomic might take upto 20 minutes._

## Operating the connector
### On-demand Data Pull

Use the on-demand data pull when there is historical energy related data that needs to be loaded from Turbonomic to Envizi.

- This method is for the initial sync or to sync data that was added/updated between specific dates.
- This method is meant to be executed just once whenever needed and then stopped.
- App Connect logs will show when On-demand load has been completed successfully or that it can be stopped.

- For TurbonomicLocations, no additional configuration is needed.

- To use this method in flows other than TurbonomicLocations, the following parameters in "Set variable" node need to be configured:
  - Variable > config > OverrideStartDate:
    - The start timestamp of the window between which the data will be pulled from.
    - The date must be specified in yyyy-mm-dd format.
    - e.g., `2022-06-26`
  - Variable > config > OverrideEndDate:
    - The end timestamp of the window between which the data will be pulled from.
    - The date must be specified in yyyy-mm-dd format.
    - e.g., `2022-06-27`
- **Note:** The dates configured here will be considered as UTC

    ![Set variable configuration](https://media.github.ibm.com/user/375131/files/09576580-50a2-11ed-8f46-b3beb969f206)

## Part 3. Testing and Verification

1. Set the OverrideStartDate and OverrideEndDate in the flow to a date that you know the data exists for.
    - Preferably keep it a single date.
    - Refer to the [On-demand Data Pull](#on-demand-data-pull) section for more details
2. Make sure "Also run the flow when it's switched on" is selected in the "Scheduler" node
3. Start the flow
4. Open App Connect logs from the left side bar
5. Keep refreshing the logs until you see a message that says "Flow <FlowName> completed successfully".
6. If you get an error message, refer to the [Troubleshooting](#troubleshooting) section

## Troubleshooting

The below errors are found in the App Connect logs.

 Error | Cause | Resolution
 -- |-- |--
 The HTTP request returned with an error: 400 "Bad Request"|No configuration for first HTTP Node|Please configure the HTTP node with the exact same configurations as mentioned in the Step 5 of [App Connect Configuration](#part-1-appconnect-configuration)
 The HTTP request returned with an error: 400 "Bad Request"|Invalid API Key name|Enter the text `username` and **not** your actual username.
 The HTTP request returned with an error: 400 "Bad Request"|Invalid API Key location|Select "body URL encoded" in API Key Location
 The HTTP request returned with an error: 401 "Unauthorized"|Invalid API Key in HTTP Account|Enter the correct Turbonomic credentials in this format `<USERNAME>&password=<PASSWORD>`
 The HTTP request returned with an error: 500 "Internal Server Error"|Invalid URL in Set Variable|Configure the correct URL of the Turbonomic instance
 The HTTP request returned with an error: 404 "Not Found”|If / is appended at the end of the valid URL|Configure the correct URL of the Turbonomic instance and do not append / at the end of the URL
 Your Amazon S3 account doesn't have the required permissions to complete the isObjectPresent action on the object object: 403 "Forbidden"|Invalid customer|Configure the customer name in set variable exactly as provided by Envizi