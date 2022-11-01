# Tririga - Envizi integration

## Using the flows
Flows included in this integration:

- <a href="https://github.com/IBM/tririga-envizi-appconnect-flows/blob/v1.0.1/AppConnect%20Flows/TririgaBuildings_Always_On.yaml" target="_blank">TririgaBuildings_Always_On</a>
- <a href="https://github.com/IBM/tririga-envizi-appconnect-flows/blob/v1.0.1/AppConnect%20Flows/TririgaBuildings_On_Demand.yaml" target="_blank">TririgaBuildings_On_Demand</a>



This integration comes with two types of flows:

#### On-demand Flow

This flow is for the initial sync or to sync data that was added/updated between specific dates. This flow is meant to be executed just once whenever needed and then stopped.

1. The following parameters in the initial **Set variable** node need to be configured in order to use this flow:
![Set variable configuration](https://media.github.ibm.com/user/375131/files/06799700-0932-11ed-90c4-5842714c7033)
*Override Dates in Set Variable*

Parameter | Value
--|--
OverrideFromDate | The start timestamp of the window between which the data will be pulled from. e.g., `2022-06-26T23:09:30-07:00`
OverrideToDate | The end timestamp of the window between which the data will be pulled from. e.g., `2023-06-26T23:09:30-07:00`

<i>**These dates must be specified in ISO 8601 format**</i>


#### Always-On Flow
This flow is to keep syncing the data after the initial sync. This flow is meant to be kept running and will only sync the data that has been added or updated after its previous execution event. For example, if the flow executes at 2 PM and it's previous execution was at 1 PM, the flow will pull data that has been added or updated after 1 PM.

#### Configuring the Flow Parameters

1. Click on the initial **Set variable** node
![Set variable configuration](https://media.github.ibm.com/user/375131/files/06799700-0932-11ed-90c4-5842714c7033)
*Fields in the initial Set variable node*

- In **Variable -> config -> customer**, enter the value provided by Envizi
- In **Variable -> config -> triURL**, enter URL for the Tririga instance. (e.g., https://example.com:9080)


### Starting and Stopping the flow

1. Click on the kebab menu (three dots) on either the flow's tile or the specific flow page.
<img width="656" alt="Flow start 1" src="https://media.github.ibm.com/user/375131/files/5be4ac00-eb1b-11ec-85b0-b7b2d400a7e1">
*Start the flow from the dashboard*
<img width="959" alt="Flow start 2" src="https://media.github.ibm.com/user/375131/files/5be4ac00-eb1b-11ec-93f2-4031220e144f">
*Start the flow from the page*

- Click on **Start API** or **Stop API** depending on which action is desired.

_Note: If the flow is not running, App Connect will give Error 404 on the API call._

## Post-Setup Instructions

The Envizi implementation team will support the next steps of the integration by provisioning the file transfer details, enabling Envizi data connectors and validating the Envizi configuration ready to receive the data flow.

Reach out to the primary contact at Envizi who can help coordinate next steps.
