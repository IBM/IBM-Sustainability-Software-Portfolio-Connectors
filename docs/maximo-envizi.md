# Maximo - Envizi integration

## Using the flows

Flow included in this integration:

- <a href="https://github.com/IBM/maximo-envizi-appconnect-flows/blob/main/AppConnect%20Flows/PLUSZMXTOSFTP.yml" target="_blank">PLUSZMXTOSFTP</a>

This integration uses the built in Maximo cron tasks.

#### Cron Tasks

For every Object Structure that gets pulled out of Maximo and pushed to Envizi, there are two types of Cron jobs:

##### Always-on Cron (Always-Cron)
This is for pulling the live or recent data, which is supposed to run frequently. This will pull the data that has come into the system after the last time it ran.

Based on its running frequency, the system will automatically decide the window of dates/timestamps to pull the data from.

These Cron Task Instances will use the suffix `_ALWAYS_ON`

<img width="360" alt="Cron task instance - always on" src="https://media.github.ibm.com/user/375131/files/27732000-f7aa-11ec-82f3-0b1e39ea5c02">


##### On-demand Cron (Cron-demand)
Even though this is a Cron, this is supposed to run just once, on-demand, when the customer or the user wants to pull data between fixed dates/timestamps. Once the first cron event executes, it should be stopped in order to prevent the same data getting pulled again and again. _This is a stop-gap for now as we dont have another way from Maximo to call a preconfigured API on-demand._

This might be needed during initial setup of the system, as the Always-on Cron will start pulling data that has arrived after it has been started. This can also be used anytime in the future if there is any need to pull any historical data on-demand.

The start date (`OVERRIDESTARTDATE`) and end date (`OVERRIDEENDDATE`) have to be configured before the cron is started. 

These Cron Task Instances will use the suffix `ON_DEMAND`

<img width="356" alt="Cron task instance - on demand" src="https://media.github.ibm.com/user/375131/files/6e611580-f7aa-11ec-80ac-5378b76aa2a9">


##### Basic Parameters

- `MXURL`: The base URL of the Maximo instance. This would include the protocol, domain name and the port number. Do not put a trailing `/` at the end. In the absense of port number, the default port for the protocol will be used _(80 for http, 443 for https)_.
    - e.g. http://example.maximo.com:9080

- `CUSTOMER`: Name of the customer. This will be used to name the CSV files. To be supplied by Envizi.

- `OVERRIDESTARTDATE`: _(Only for On-demand Cron)_ The start timestamp of the window between which the data will be pulled from.
    - The date must be specified in ISO 8601 format.
    - e.g. `2022-06-26T23:09:30-07:00` where `-07:00` represents timezone offset of UTC-07:00 hours
- `OVERRIDEENDDATE`: _(Only for On-demand Cron)_ The end timestamp of the window between which the data will be pulled from.
    - The date must be specified in ISO 8601 format.
    - e.g. `2022-06-26T23:09:30-07:00` where `-07:00` represents timezone offset of UTC-07:00 hours


##### Advanced Parameters

These will be set to the right values by default. DO NOT change these parameters without consulting with Envizi.

- `SELECT`: Names of the fields from the Object Structure
- `FROM`: Name of the Object Structure
- `WHERE`: _(Optional)._ OSLC Condition for data selection
- `ORDERBY`: _(Optional)._ Sequence by which the data should be pulled
- `TARGETDATECOLUMN`: Name of the DateTime Column in the Object structure that will be used to filter records between a date-time window.
- `SAVEDQUERY`
- `ENDPOINTNAME` - Name of the Maximo End Point configured to connect to the AppConnect flow.
- `PAGESIZE` - Maximum number of records pulled from Maximo in one API call. This will also be the maximum number of records in the CSV file.

##### Configuring Cron Task

- In the Maximo menu, click on System Configuration > Platform Configuration > Cron Task Setup
<img width="638" alt="Cron task - Menu" src="https://media.github.ibm.com/user/375131/files/204c1200-f7aa-11ec-902b-56a6cacbbbb1">

- Under the "Cron Task" column, type "PLUSZ" and hit enter to search
<img width="754" alt="Cron task - Search" src="https://media.github.ibm.com/user/375131/files/217d3f00-f7aa-11ec-8d77-1339ac049a2d">

- Click on the "PLUSZEXPORT" Cron Task
- In the Cron Task Instances, click on the Schedule/Calendar icon to set the schedule for the Cron Task Instance
- Once it is configured, click on "OK"
- Select the Cron Task Instance to configure Parameters for
- Scroll down to the "Cron Task Parameters" section and configure the "Value" column as needed. Ideally, only the `CUSTOMER` and `MXURL` parameters will need to be edited for all instances. For the "On-demand" Crons, the `OVERRIDESTARTDATE` and `OVERRIDEENDDATE` parameters will need to be edited.
- Click on Save

##### Start/Stop the Cron Task Instance
- Navigate inside the Cron Task as mentioned above
- Enable/Disable the checkbox in the column "Active?" in front of the desired Cron Task Instance
- Click on Save

### Starting and Stopping the flow

- If using AppConnect Dashboard, Click on the kebab menu (three dots) on the flow's tile.
<img width="656" alt="Flow start 1" src="https://media.github.ibm.com/user/375131/files/5be4ac00-eb1b-11ec-85b0-b7b2d400a7e1">

- If inside the flow, Click on the kebab menu (three dots) on top right of the screen.
<img width="959" alt="Flow start 2" src="https://media.github.ibm.com/user/375131/files/5be4ac00-eb1b-11ec-93f2-4031220e144f">

- Click on "Start API" or "Stop API" depending on what action you want to perform.

_Note: If the flow is not running, AppConnect will give Error 404 on the API call._

<img width="893" alt="PLUSZMXTOSFTP_P1" src="https://media.github.ibm.com/user/375131/files/4383bb00-f7c4-11ec-9d13-c85b6f8ce6ee">
<img width="884" alt="PLUSZMXTOSFTP_P2" src="https://media.github.ibm.com/user/375131/files/441c5180-f7c4-11ec-9cbd-229152cebafd">
<img width="902" alt="PLUSZMXTOSFTP_P3" src="https://media.github.ibm.com/user/375131/files/467eab80-f7c4-11ec-9558-276e2708db02">


#### If
Checks for the mandatory fields that are required for the flow to function. If any of the fields are missing, the flow returns Error 400.

#### If 3
Checks the presence of `OVERRIDESTARTDATE` and `OVERRIDEENDDATE` in the Request. It also checks if they are valid dates. If all checks are passed, `dateQuery` with these override dates is generated.
If the override date checks fail, it checks if `LASTSTARTDATE` is present. If it is, `dateQuery` with it is generated.
If all checks fail, the flow returns Error 400

#### Set variable 5

- `where`: The final `oslc.where` query created using `WHERE` from Request and `dateQuery` from the output of "If 3"
- `batchTimestamp`: Current Epoc timestamp in milliseconds. This will be included in every filename to indicate that they are from the same batch of exported data.

#### Set variable

- `query`: The query to be sent to OSLC API to fetch the required data
- `pagesize`: The total number of records that will be fetched in one page. If it is not specified in the request, the flow will use default value of `100`.

#### HTTP Invoke Method
Does an HTTP GET to the Maximo's OSLC API to fetch the total number of records that match the query. `countonlyparams` from "Set variable" will be used here.

#### Set variable 2
`totalCount`: Total Count received in the response of the "HTTP Invoke Method"

#### Set variable 3
`totalPages`: Total number of pages to be fetched from Maximo OSLC API that match the query. This is calculated using the `totalCount` from "Set variable 2" and `pagesize` from "Set variable".

#### For each: page
- Summary: Fetches data from Maximo OSLC API page by page and writes CSV file to the SFTP server.
- Input: An array of numbers from 1 to `totalPages` from "Set variable 3"
- Output: `fileWritten` - Array of string containing names of files written to the SFTP server.

#### HTTP Invoke method 2
Does an HTTP GET call to Maximo OSLC API to fetch the data that matches `queryparams` from "Set variable" for the current page from "For each: page"

#### Set variable 4
`filename`: File name as per format specified by Envizi team using `CUSTOMER` and `FROM` from the Request, `batchTimestamp` from "Set variable 5" and current page from "For each: page".

eg. `DemoCorporation_PLUSZLOCATIONS_1654856600431_1.csv`

#### SFTP Create file
Writes Response body from "HTTP Invoke method 2" to the configured SFTP Server with the `filename` from "Set variable 4" at the preconfigured file path.

#### Response
Returns HTTP Status 200 with JSON body containing:
- `filesWritten`: Array of file names written to the SFTP server, obtained from the output of "For each: page"
- `queryparams`: `query` from "Set variable"

The JSON in response body is not used or stored by Maximo.
