# IBM TAS Connector for Envizi

## Overview

Enable on-demand and scheduled synchronization of location data between TRIRIGA and Envizi systems.

![Tririga-Envizi integration](https://media.github.ibm.com/user/375131/files/1a642100-01ed-11ed-8151-31e573db3664)

*TRIRIGA to Envizi Integration Diagram*

## Prerequisites

This integration requires running instances of TRIRIGA, App Connect, and Envizi. 

## TRIRIGA Configuration

Before App Connect flow is configured, the OM package needs to be installed on the TRIRIGA instance.

Follow the [documentation](./tririga-configuration.md) for detailed instructions.


## App Connect Configuration

*Note: IBM Cloud App Connect Professional or Enterprise is needed to run this flow.*

*Note: The names in the screenshots are generic, the elements in this integration will not have the same names during setup.*


### Adding Accounts

Before importing the flow to App Connect, add Accounts for "Amazon S3" and "HTTP" connectors. While adding the HTTP connector account, include credentials for the Tririga user which can consume the OSLC API.

1. Navigate to Catalog section of the App Connect instance
<img width="960" alt="Create Account 1" src="https://media.github.ibm.com/user/375131/files/eb8b5a00-eb1d-11ec-8401-35b47d561ce4">
*Create Account 1*

- In the **Search application** field, type name of the connector.

- If the App Connect instance does not have an account for the connector, click on **Connect** to create a new account
<img width="960" alt="Create Account 2a" src="https://media.github.ibm.com/user/375131/files/ec23f080-eb1d-11ec-9f62-af1addec3fd3">
*Create Account 2*

- Else, open the account selection drop down, and click on **Add a new account ...**
<img width="948" alt="Create Account 2b" src="https://media.github.ibm.com/user/375131/files/ecbc8700-eb1d-11ec-9693-4ce83ffda2e6">
*Add a New Account*

- Enter the necessary details for the connector<br>
    a. For Amazon S3, it will be the Secret Access Key and Access Key ID provided by Envizi.<br>
    b. For HTTP, it will be the Authentication Key or username and password needed for Tririga.
<img width="960" alt="Create Account 3" src="https://media.github.ibm.com/user/375131/files/ed551d80-eb1d-11ec-9894-350e87fbb2b2">
*Connect to S3/HTTP*

- Click on Connect
<img width="960" alt="Create Account 4" src="https://media.github.ibm.com/user/375131/files/ed551d80-eb1d-11ec-983c-c276d43e0654">
*Connect the Account*

- From the account selection drop down, select the newly created account. e.g., The default name will be `Account 2` if `Account 1` is already present
- Click on the kebab menu (three dots) and select **Rename Account**
<img width="960" alt="Create Account 5" src="https://media.github.ibm.com/user/375131/files/ededb400-eb1d-11ec-9a29-fd9fa65dbf8b">
*Rename the Account step 1*

- Enter an account name and click on **Rename Account**. This name can now be used by the connector in the flow.
<img width="958" alt="Create Account 6" src="https://media.github.ibm.com/user/375131/files/ee864a80-eb1d-11ec-96d1-fdc42c679896">
*Rename the Account step 2*

### Importing the flow

1. Open the Dashboard of the App Connect instance.
- Click on the **New** button and select **Import Flow**.
<img width="861" alt="Flow Import 0" src="https://media.github.ibm.com/user/375131/files/ea582e00-eb19-11ec-9912-0881d321f3c7">
*Import the flow*

- Browse to the flow's YAML file and click on **Import**.
<img width="960" alt="Flow Import 1" src="https://media.github.ibm.com/user/375131/files/ea582e00-eb19-11ec-88a7-abebff4e0534">
*Select the desired flow*

- The flow will now be imported and opened.
<img width="960" alt="Flow Import 2" src="https://media.github.ibm.com/user/375131/files/eaf0c480-eb19-11ec-885b-919da812499b">
*The main page for the flow*

### Configuring the flow to use the right accounts

When importing a flow, it is important to check if the flow is using the right accounts for the different connectors.

1. Click on **Edit Flow**

- See if the connectors are using the right accounts.

- To change the account for any connector, select the connector and click on the dropdown icon next to the Account's name
<img width="960" alt="Select Account 1" src="https://media.github.ibm.com/user/375131/files/11186380-eb1e-11ec-9c92-5f892df61a57">
*Change associated Account*

- Select the account name to use from the list
<img width="659" alt="Select Account 2" src="https://media.github.ibm.com/user/375131/files/11b0fa00-eb1e-11ec-88fd-8e4998f6304a">
*Select desired Account*

### Configuring the Scheduler

1. Click on the **Scheduler** node
<img width="659" alt="Scheduler node" src="https://media.github.ibm.com/user/375131/files/ea694d80-01ed-11ed-8852-16b625339675">
*The Scheduler node*

- Configure the schedule as needed

<img width="659" alt="Scheduler configuration 1" src="https://media.github.ibm.com/user/375131/files/d70ab200-01ee-11ed-9af7-4f66738aea7c">
*Options for configuration- Hourly*
<img width="659" alt="Scheduler configuration 2" src="https://media.github.ibm.com/user/375131/files/00c3d900-01ef-11ed-86aa-29008ff86450">
*Options for configuration- Daily*