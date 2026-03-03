# IBM MAS Connector for Workday

The **IBM MAS Connector for Workday** is released in MAS under the name *"IBM Maximo Connector for Workday"*.
This connector integrates the Workday Suite and Maximo Manage using Maximo Manage integration components as well as common integration components in Workday Suite. <br>

The integrations and related flows are grouped into three sections:

- Financials: This integration facilitates a bi-directional exchange of financial data, such that
Maximo’s cost collection transactions (i.e. labor & materials for a work order) can be reported
to Workday’s General Ledger.
- Purchasing: This integration facilitates the support of a single purchasing scenario that is
commonly seen with Maximo and ERP systems. This scenario is where Purchasing is done in
Workday (ERP). This purchasing scenario allows Maximo to drive the demand for goods (spare
parts) and services that are related to Maintenance activities (work orders).
- Inventory: This integration solution facilitates the support of a single Inventory scenario that is
commonly seen with Maximo and ERP systems. This scenario is where the Item Master is
managed in Workday (ERP) and Inventory Management (storerooms, balances, transactions)
are managed in Maximo. This inventory scenario allows Maximo to manage all inventory
activities independent of the ERP (Workday) system.

*** 

## Connector architecture
To support a full supply chain cycle, the integration provides App Connect flows and Maximo Integration Framework (MIF) artifacts.  These work in conjunction with Workday-provided public APIs to transfer data between applications.

On the Maximo side, the solution includes all of the necessary integration artifacts that define the External System, Endpoints, Enterprise Services, Publish Channels, and associated integration controls, database changes, and business rules.  A number of new cron tasks are also provided to schedule the data synchronization.

App Connect layer provides message transformation and communication with Workday SOAP APIs.  Workday provides numerous SOAP APIs to import and fetch data from their system.


![MAS-Workday Architecture](Images/MAS-Workday/Whitepaper%20Architecture.jpg)

As part of the normal cycle of maintaining assets, resources such as spare parts, labor, tools, and materials are required.  While IBM Manage has built-in capabilities to source and manage these resources, many customers prefer to use their ERP system to perform these business processes.  

They may have implemented complex procurement business processes, rules, and departmental limits that they would like to enforce for all Material/Repair/Operational (MRO) logistics and prefer not to reproduce the same rules in other systems.  

The adapter supports Supply Chain master data (such as Suppliers, Purchase Items, and Purchase Contracts), as well as transactional data (Requisitions, Orders, Receipts and Invoices).

![MAS-Workday Purchasing Cycle Image](Images/MAS-Workday/MAS-Workday%20Purchasing%20Cycle%20Image.png)

***

## Inventory: Master Data

The following sections detail the integrations for the Master Data that is loaded into Maximo from Workday.

### Suppliers

Workday is the System of Record for Suppliers.  The integration uses Workday API to fetch Suppliers and creates Companies in Maximo.  There are built-in capabilities for an initial synchronization (to load all suppliers during initial implementation) as well as incremental delta loads.  

Suppliers are brought over as “Companies” of type “V” (vendors) to Maximo.  Additionally, the Workday primary key (WID) is also established as the external reference for each supplier.


![Supplier Integration](Images/MAS-Workday/Supplier%20Integration.png)


### Purchase Items

Purchase items are maintained by Workday.  Like Suppliers, the integration fetches items using a Workday API and creates corresponding records in Maximo.  These can then be used during procurement on Purchase Requisitions.  Initial and delta loads are supported.

![PI Integration](Images/MAS-Workday/PI%20Integration.png)


### Purchase Contracts

Workday is the System of Record for Purchase Contracts.  The integration fetches contracts using a filter (contract type) and creates corresponding contracts in Maximo.  Contract revisions are also supported, as well as initial and delta loads.

![PC Integration](Images/MAS-Workday/PC%20Integration.png)


***

## Purchasing Cycle

The following sections detail the transactional data in the integrations for the Purchasing Cycle.

### Purchase Requisitions

The supply chain cycle begins in Maximo.  Asset management drives the demand for spare parts, materials and services.  Maximo can use the suppliers, items and contracts previously synchronized from Workday to generate purchase requisitions.  The integration then sends these requisitions to Workday for fulfillment.  

In addition to common purchasing data like item, supplier, quantity, etc., the integration also supports several pre-defined worktags.  Workday uses worktags to simplify accounting transactions and they play a central role in many business processes.  Worktags are stored as General Ledger (GL) components in Maximo, and they are synchronized using financial flows discussed later in this paper.  

When a purchase requisition is created in Maximo, a user can select values for the debit/credit accounts.  These values are presented in a dialog box and are sourced from Workday worktags (such as spend category, cost center, fund, account and project task).  We also provide an example of transferring a custom organization into a GL component.  This allows for extra flexibility for customers who need more worktags in Maximo.

![Requisition Integration](Images/MAS-Workday/Requisition%20Integration.png)


Purchase Requisitions in Maximo can be generated for inventory replenishment as well as for ongoing work order needs.  By using data from Workday, Maximo users can request fulfillment of inventory or direct issue items/materials and/or services from Workday.  Approval of the requisition is what initially triggers the integration but that is also a configurable element.

In Maximo, a purchase requisition line can be “coded” for things like items, materials (non-stocked items), and services.  Workday similarly supports these types of lines, and the integration populates the Workday API request with data for each of these, enabling capability to simultaneously procure items and services.

Finally, the Purchase Requisition is the document that links what you’re buying with the Work Order or Storeroom that you’re buying it for.  These details are used for subsequent transactions like the PO that will be linked to the PR and inherit the work order or inventory storeroom data.


### Purchase Orders

Purchase Orders are created in Workday and synchronized back to Maximo.  By default, the integration pulls all records from Workday that have been created or updated since the last successful pull.  Additionally, there is some filtering criteria provided to pull POs based on their status.

Maximo does not allow for Purchase Orders that have lines from multiple sites (combining requisitions from multiple sites).  Therefore, the integration enforces this business rule by checking the worktag that holds the site identifier from Maximo.  All of the site identifiers on all of the lines must match or an error condition is set.

Purchase Order lines can be created for materials and/or services.  Both are supported by the integration and are mapped back to Maximo.  Additionally, several worktags are also supported, and they are mapped back to a Maximo GL account.

Purchase Order revisions are also supported by the adapter.  These revisions are fetched using a different Workday SOAP API but mapped using the same flow.


![PO Integration](Images/MAS-Workday/PO%20Integration.png)


### Receipts

Receipts are created in Maximo and sent to Workday.  Returns and cancellations are also supported and sent via the same integration back to Workday.  

Receipts are sent to Workday using a Maximo Integration Framework Publish Channel that invokes an App Connect flow.  The flow receives a copy of the receipt data from Maximo and transforms that message into a Workday SOAP request.

The integration flow inspects the Maximo “issuetype” to determine if the transaction is supposed to submit a receipt, void it, or return it and it calls the appropriate Workday API to perform the corresponding action.


![Receipt Integration](Images/MAS-Workday/Receipt%20Integration.png)


### Invoices

Invoices are entered, approved, and paid in Workday.  After payment is complete, the invoice is brought back to Maximo and should be considered “read-only”.  The main purpose of bringing the invoice back is to close the procurement cycle.  The invoice transaction can update the Maximo work order if there are any variances, such as currency variances or amounts that were within tolerance of the invoice.

On the Workday side, the integration uses the “Get_Supplier_Invoices” API, which does not currently support delta extraction of supplier invoices based on dates.  Therefore, a complex solution was developed for this integration that extract a page of invoices based on a sliding number of days.  By default, this is set to 90 days.

Once pulled, the primary keys for these invoices are then used to extract a list of the same keys out of Maximo.  Any missing keys in Maximo indicate a missing invoice.  These missing invoices are then sent to Maximo so that the next invoice pull will exclude them.  This approach minimizes the amount of data and duplicates that are sent to Maximo.

It is important to note that this integration was designed as a “one time only” extraction of the invoice and does not support ongoing updates to it.  Since Maximo does not have an Accounts Payable module, it is expecting that the ERP system pay the supplier and then provide the final details to Maximo.


![Invoice Integration](Images/MAS-Workday/Invoice%20Integration.png)


***

## Financials

The following sections detail the integrations for the financial data that is loaded into Maximo from Workday.

### Worktags (GL Components)

Worktags are used in Workday to simplify accounting entries.  The integration supports some pre-defined worktags and can be configured to extract a list of values for each tag.  These values are stored as GL components in Maximo.  The primary function of these worktags is to enable outbound (Maximo à WD) transactions to use these values to report costs to Workday.

The diagram below represents a synchronization of Cost Centers from Workday to Maximo.  There are numerous worktags supported in addition to Cost Centers, such as Spend Categories, Fund Codes, Custom Organizations, Account Sets, and Project Tasks.  The architecture for these is essentially the same as for Cost Centers:


![Worktag Integration](Images/MAS-Workday/Worktag%20Integration.png)


### Work Order

The integration also provides an outbound worktag for Work Orders.  This worktag can be used to create a Custom Organization in a hierarchy (“All Maximo Workorders”) in Workday.  Once created, the worktag can be used to create Purchase Order lines and set the worktag on the line so the integration “knows” which Work Order to apply the purchase line to.  This step is not necessary if the line is sourced from a requisition that originated in Maximo, since the integration will automatically add this worktag on the requisition line AND the Purchase Order in Workday inherits this value automatically.  It is provided to support charging Maximo work orders for transactions that do not originate in Maximo.

![Work Order Integration](Images/MAS-Workday/Work%20Order%20Integration.png)


### Journal Entries

Journal entries for maintenance-related activities can be sent to Workday (for example, journals for labor reporting, material issues, tools, etc.).  As part of the normal supply chain cycle, some journals will be generated in Maximo, and some in Workday.  To prevent “double dipping” on journals, each of the Maximo journal types can be individually controlled to send or not.  This is done using an integration control and can be configured by implementation team.

Journals fully support the pre-defined worktags previously mentioned (Cost Center, Spend Category, Fund, Account, Project Task, and Custom Organization).  The integration includes code that uses a new Maximo domain to map the individual segments of a GL chart of accounts entry to the individual worktags in Workday.  So even though the integration ships with a pre-defined list of Worktags, the implementation team can configure the order of the segments and even add new ones as a customization.


![Journal Integration](Images/MAS-Workday/Journal%20Integration.png)


***

## HCM

The following sections detail the integrations for the Human Capital Management Data that is entered in Maximo and sent to Workday.

### Workers

Workers are entered and maintained in Workday.  The integration for workers is complex because it creates several different objects in Maximo.  Architecturally, this integration is also different from the others.  In all other flows, a single flow extracts data from Workday, performs the necessary transformations, and then posts that data to Maximo.

For the Worker integration, a separate flow is used to perform the initial fetch from Workday.  This “sub flow” is used by a second “mapping flow” that performs the transformations and posting to Maximo.  The reason for breaking this process into two flows is that the “Worker” schema from the Workday API is extremely large and it slows down the UI on App Connect and makes it difficult to perform the mapping work.

By separating the “Get Worker” fetch with the “Map Worker” workload, the fetching flow can be used as any other API and since that flow returns a much smaller payload to the mapping flow, it will not slow down the App Connect UI.

The integration fetches worker data from Workday and creates People, Labor, Craft, and Skill data in Maximo.  It also synchronizes pay rates (for hourly workers only) in case the customer wants to use these rates for applying costs to the Maximo workorder.  A complex JSONata function was developed to manipulate the payload from Workday and reduce its size.  The result of that function is a smaller JSON payload that the mapping flow uses to produce the Maximo payload.

![Worker Integration](Images/MAS-Workday/Worker%20Integration.png)




### Time Reporting

Labor reporting is performed in Maximo and time blocks are sent to Workday.  After labor is reported against a work order in Maximo, the integration sends this to Workday as a time block.  Currently, this integration point does not support worktags, as it was one of the original integration points released prior to worktag support.  

![Time Reporting Integration](Images/MAS-Workday/Time%20Reporting%20Integration.png)




***