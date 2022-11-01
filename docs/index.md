# IBM Sustainability Software Portfolio Connectors

## Overview

IBM TRIRIGA Application Suite (TAS) and IBM Maximo Application Suite (MAS) customers have access to a collection of predefined integrations between TAS, MAS, and IBM Envizi ESG Suite. Those integrations can be used in a variety of combinations to enable many usecases and they can each be configured for additional flexibility. 

**The TRIRIGA - Maximo Connector** is released under two names: TAS Connector for Maximo and MAS Connector for TRIRIGA depending on which Applications Suite you are using to access it. This connector supports the following use cases:  
- Bi-directional loading, and continuous synchronization of portfolio data such as People, Locations and Assets.
- Bi-directinoal routing of service requests between TRIRIGA and Maximo
- Automatic shadowing of TRIRIGA Work Tasks with Maximo Work Orders to enable Facility teams to do planing, budgeting and project management in TRIRIGA while work tasks get reflected in Maximo as Work Orders for execution.

**TAS Connector for Envizi** automatically sync space and occupancy data from TRIRIGA with Envizi to enable energy usage calculations across entire facility portfolio with advanced analytics by location, by SQF, and by occupant.

**MAS Connector for Envizi** automatically sync asset location and meter readings from MAS to Envizi to automate tracking of energy usage and calculating related scope 1 and 2 emissions of Electricity, Natural Gas, and Water.


## App Connect

**IBM App Connect** is a middlware that provides a flexible environment for integration solutions to transform, enrich, route, and process business messages and data. It is a pre-requisite for all the Portfolio Connectors and it is made available as a supporting program through TAS 11.3 and MAS 8.9.

**App Connect Flows** enable specific integration use cases by connecting to predefined APIs to route and transform data. They can be applied in different combinations to support a variety of use cases, they handle the required data mapping, data validation and they can be modified to suite individual requirements. Soe flows invoked based on events or on a schedule.


## Installing App Connect

**App Connect Pro SaaS** is available in the IBM Cloud catalog [here](https://cloud.ibm.com/catalog/services/app-connect). 
>Please Note: if you are using a Managed Service provided by IBM SRE you may be entitled to a discounted instance of App Connect. Please reach out to your SRE contact.

**App Connect Enterprise on-prem** can also be provisioned within your RedHat OpenShift Cluster directly from the Open Shift Operator. Please follow the docs [here](https://www.ibm.com/docs/en/mas-cd/continuous-delivery?topic=ons-app-connect) about installing the operator.