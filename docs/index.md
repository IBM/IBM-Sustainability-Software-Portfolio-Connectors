# IBM Sustainability Software Portfolio Connectors

## Connectors Overview

IBM TRIRIGA Application Suite (TAS) and IBM Maximo Application Suite (MAS) customers have access to a collection of predefined integrations between TAS, MAS, and IBM Envizi ESG Suite. Those integrations can be used in a variety of combinations to enable many usecases and they can each be configured for additional flexibility. 

**1. The TRIRIGA - Maximo Connector** is released in TAS under the name *"TAS Connector for Maximo"* and in MAS under the name *"MAS Connector for TRIRIGA"*. 
This connector supports the following capabilities:  
- Bi-directional loading, and continuous synchronization of portfolio data such as People, Locations and Assets.
- Bi-directinoal routing of service requests between TRIRIGA and Maximo
- Automatic shadowing of TRIRIGA Work Tasks with Maximo Work Orders to enable Facility teams to do planing, budgeting and project management in TRIRIGA while work tasks get reflected in Maximo as Work Orders for execution.

**2. TAS Connector for Envizi** automatically sync space and occupancy data from TRIRIGA with Envizi to enable energy usage calculations across entire facility portfolio with advanced analytics by location, by SQF, and by occupant.

**3. MAS Connector for Envizi** automatically synchronizes asset location and meter readings from MAS to Envizi to automate tracking of energy usage and calculating its related scope 1 and 2 emissions of Electricity, Natural Gas, and Water.


## App Connect

**IBM App Connect** is a middlware that provides a flexible environment for integration solutions to transform, enrich, route, and process business messages and data. It is a pre-requisite for all the Portfolio Connectors and it is made available as a supporting program through TAS 11.3 and MAS 8.9.

**App Connect Flows** enable specific integration use cases by connecting to predefined APIs to route and transform data. They can be applied in different combinations to support a variety of use cases, they handle the required data mapping, data validation and they can be modified to support customizations. 


### Installing App Connect

- **App Connect SaaS** is available in the [IBM Cloud catalog ](https://cloud.ibm.com/catalog/services/app-connect). 



- **App Connect Enterprise on-prem** can be provisioned within your RedHat OpenShift Cluster directly from the [OpenShift Operator](https://www.ibm.com/docs/en/mas-cd/continuous-delivery?topic=ons-app-connect) 

- If you are using a MAS or TAS Managed Service provided by IBM SRE you can request Dev and Test instances of App Connect. Please reach out to your SRE contact.

