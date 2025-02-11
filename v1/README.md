# Integrating Turbo with Envizi via webMethods for Green IT data

This blog explains about the step-by-step instructions to pull green IT data from Turbonomic into Envizi via webMethods Integration.




#### Authors
 [Jeya Gandhi Rajan M](https://community.ibm.com/community/user/envirintel/people/jeya-gandhi-rajan-m1) <br />
 [Madhukrishna Parike]() <br />
 [JYOTI RANI]() <br />
 [INDIRA KALAGARA]()

## Contents

- [1. Prerequisite](#1-Prerequisite)
- [2. Architecture](#2-Architecture)
- [3. webMethods Accounts Workflow Configuration](#3-webMethods-Accounts-Workflow-Configuration)
- [4. Validate Workflow Execution](#4-Validate-Workflow-Execution)
- [5. Schedule Workflow Execution](#5-Schedule-Workflow-Execution)

## 1. Prerequisite

### 1.1 Environment

- Turbonomic v8.14.3 or higher 
- Envizi Saas instance access (Click [here](https://techzone.ibm.com/collection/aiapps-environmental-intelligencewith-envizi/environments) to get access). 
- webMethods SaaS (Click [here](https://signup.softwareag.cloud/#/basic-b) to signup for Trial).

### 1.2 Turbonomic Pre-Configuration

1. Create an user with `Observer` role in Turbonomic. Refer [here](#user-content-1-create-user-in-turbonomoic) to create the user.


### 1.3 Envizi Pre Configuration

#### 1.3.1 Organization, Account Style and Location

We would be retreiving Datacenters and its energy consumption (electricity) data from Turbonomic through the Turbonomic APIs.

For each Datacenter in Turbonomic there will be an `Account` created in Envizi under a pre-configured `Location`.

The entire organization hierarchy with Groups, Locations and Accounts in Envizi may look like this. Look at the location and account highlighted in the image below.
- Location : IN Bank - IBMC-WDC07-Ashburn VA 
- Account  : IN Bank - IBMC-WDC07-Electricity. 

<img src="images/GSI_Demo_WM_Envizi_Org_1.png">


1. Get the values for the below fields from Envizi
  - Organization (Organization name)
  - Organization Link (Organization reference id)
  - Account Style Link (Reference id for the account style `S2 - Electricity - kWh`)
  - Location Name (The location name under which the accounts to be created)

#### 1.3.2 Envizi S3 Bucket

Envizi S3 bucket details are needed to push the data into Envizi. If the S3 is not created in Envizi, refer Steps 1 and 2 [here](https://developer.ibm.com/tutorials/awb-sending-udc-excel-to-s3/) to create it in Envizi.

1. From Envizi S3 bucket screen, get the values for the below fields.
  - Bucket
  - Folder
  - Username
  - Access Key
  - Secret Access Key


## 2. Architecture

Here is the architecture  that describes about this Turbo and Envizi integration via webMethods.

webMethods Integration flow pulls the list of Cloud Regions and On-prem Data Centers from Turbo and sends it to Envizi's S3 bucket in a CSV file. This CSV file will be further processed by the Envizi internally.

<img src="images/arch.png">

## 3. webMethods Accounts Workflow Configuration

In this workflow, we will invoke Turbonomic APIs to fetch Energy consumption for each DataCenter locations and transform the JSON API response into the CSV template loaded by Envizi.

<details><summary>CLICK me for detailed instructions</summary>

### 3.1. Login to webMethods Integration

- Login to your instance of webMethods integration with the respective credentials.

### 3.2. Create a new Project

- Name Project Name as `Turbo_wM_Envizi` and Leave `Source Control - Git server/account` as Default, ignore if it is not shown . Note choose the project name as you desired.

<img src="images/wMAccNewProject-02.png">


### 3.3. Import the Workflows

- Download the Workflow archive file here [Accounts](./files/webMethods-archives/Accounts).
- Click on the `Import` and select the Workflow location that is downloaded in the above step.

<img src="images/wMAccImport-03.png">

- Provide the `Workflow name` as `Sustainability Solution - Accounts` and `Workflow description`. Please name `Workflow name` and `Workflow description` as per your need.
- Click on + symbol on the `Connect to Hypertext Transfer Protocol (HTTP)` and add the `https://[TurbonomicInstance-URL]/api/v3/entities/stats` as URL and leave other fileds as it is.
- Click on + symbol on the `Connect to Amazon Web Services` and add the `Access Key ID`, `Secret Access Key` and `Default Region` as per Envizi.
- Parameters custom `key-value pairs` used inside the Workflow.

#### Parameters
| Name       | Value                   | Comments             |
| ---------- | ----------------------- | --------------------
| TurboLoginAPI| https://[TurbonomicInstance-URL]/api/v3/login | Turbonomic Login API|
| TurboAccountStatsAPI| https://[TurbonomicInstance-URL]/api/v3/entities/ | Retrieves the Data Centres statistics such as electricity consumption|
| TurboUserName|changeme|Replace the `changeme` username created in 2nd bullet point under 1.1 step|
| S3BucketName| | S3 Bucket name as per your Envizi instance|
| EnviziTemplateFileName |  | S3 Folder name and File name as per Envizi instance. Example: client_7e87560fc4e648/Account_Setup_and_Data_Load_IBMCloud_electricity.csv|
| TurboDataCentresAPI|https://[TurbonomicInstance-URL]/api/v3/search|  Fetches the data centres locations from Turbomic instance.|
| statsFilter| {"data":{ "startDate":"2024-01-01 00:00:05", "endDate": "2024-12-31 23:59:59","statistics": [ { "name": "Energy", "filters": [ { "type": "relation", "value": "sold" }]}]}}| Please update statDate and endDate to retrieve the electricity consumption for the period.|
| DCNames | "IBMCloud" | Envizi provides the Data Centre names to be retrieved. More data centres can be added with &#124; symbol for example: "IBMCLoud&#124;Vc01dc01" |
| TurboPassword | changeme| Replace the `changeme` password created in 2nd bullet point under 1.1 step|

- For the `Connect to Hypertext Transfer Protocol (HTTP)` configuration details, please click on `+` symbol and provide URL as `https://[TurbonomicInstance-URL]/api/v3/entities/stats` under `URL`. Leave other fields as it is.
- For the `Connect to Amazon Web Services` configuration details, please click on `+` symbol
- Configure the `Add Account` AWS page with `Account Name`, `Access Key ID`, `Secret Access Key` and `Default Region`. Leave other fields as it is.
- Click on `Import` button

<img src="images/wMAccWorkflow-01.png">

#### Add Reference Data

- Reference data is a file which is a Envizi template expects as a final output.  Please download the Reference data [ReferenceData](./files/webMethods-archives/Reference/) which needs to be added after importing the Workflow in a project.
- Please update the column values as Envizi recommends before adding it into Workflow.

#### Reference Data Columns

Get the values for the below 3 fields from `Account Setup and Data Load` Report from Envizi. 
- Organization Link
- Organization
- Account Style Link


|Name                     |  Value               |Comments                  |
|-------------------------|----------------------|--------------------------|
|Organization Link|17000252| The refernce id for the Envizi Organization. Get it from Pre-requisite|
|Organization|GSI Demos	| The name of the Organization. Get it from Pre-requisite|
|Location|IBMCloud| The name of location where the account exists/to be created. It will be updated by webmethods|
|Location Ref| | Leave it empty|
|Account Style Link|14445| The refernce id for the `S2 - Electricity - kWh` account style. Get it from Pre-requisite|
|Account Style Caption|S2 - Electricity - kWh| The account style of this account.  It will be updated by webmethods|
|Account Subtype|Default| Leave it as it is.|
|Account Number|vc01dc01-electricity| The account name. It will be updated by webmethods |
|Account Reference|| Leave it empty|
|Account Supplier|| Leave it empty|
|Account Reader|| Leave it empty|
|Record Start YYYY-MM-DD|02-10-2024| It will be updated by webmethods|
|Record End YYYY-MM-DD|30-12-2024| It will be updated by webmethods |
|Record Data Quality|Actual| Leave it as it is. |
|Record Billing Type|Standard| Leave it as it is. |
|Record Subtype|Default| Leave it as it is. |
|Record Entry Method|Overwrite| Leave it as it is. s|
|Record Reference|| Leave it empty|
|Record Invoice Number|| Leave it empty|
|Total Electricity (kWh)|883.799| Electricity consumption value. It will be updated by webmethods |
|Green Power (kWh)||Leave it empty |
|Total Cost|| Leave it empty|

- Under the project created in step 3.2, Click on `Configurations -> Flow service -> Reference data -> Add Reference Data`
- `Save As` EnviziTemplate and `Reference Data File` Browse file and select the `EnviziTemplate.txt` and Click on `Next`, `Next` and `Done`

<img src="images/wMAccRefdata.png">

- Click on `Edit` by moving mouse over the Workflow imported above.

### 3.5. Workflow nodes

- Nodes used in the Workflow.

<img src="images/wMAccWorkflow-02.png">

#### About Nodes

- `Turbonomic API Login` :  This makes an API call to Turbonomic instance login API which returns `set-cookie` and used to authrize the subsequent API calls.
- `DataCentre Retrieve` : It invokes an API call to Turbonomic instance which returns array list of DataCentre’s.
- `JSON Parse` : It formats statsFilter raw JSON data.
- `Query JSON` : It retrieve JSON data from previous node.
- `Query JSON` : It queries the responseObject JSON data from `DataCentre Retrieve`.
- `DCTest` :  It is a flow-service which invokes the Turbonomic stats API to retrieve the electricity consumption and perform the data transformations as needed by Envizi.
- `JSON to CSV` : This converts JSON data from flowservice into a CSV file.
- `S3 Upload File` :  This node uploads the CSV file from previous node into S3 bucket from which Envizi loads into dashboard.

### 3.6. Activate the Workflow

- Toggle `ON` to activate the Workflow

<img src="images/wMAccAct-13.png">

### 3.7. Run the Workflow

- Run the Workflow to push the DataCentre electricity consumption stats to Envizi

<img src="images/wMAccRun-14.png">


</details>

## 4. Validate Workflow Execution

<details><summary>CLICK me for detailed instructions</summary>

#### 4.1. Data in S3

- The flows will pull the data from the Turbo and push it to S3. You can see the Data flow status in S3 like this.

<img src="images/image-11.png">

#### 4.2. Sample Data from S3

- The sample data is available [here](./files/data/).

#### 4.3. Processing S3 files in Envizi

- Envizi automatically pull the data from S3 and process it. The accounts and account summary page looks like this now.

<img src="images/image-15.png">


<img src="images/image-16.png">
<img src="images/image-17.png">

</details>

## 5. Schedule Workflow Execution

Locations and Accounts workflow can be scheduled for execution. Follow the steps below to define the schedule for workflow execution.

<details><summary>CLICK me for detailed instructions</summary>


- Mouse over the `Trigger` node in the workflow and click on `Settings`

<img src="images/sec8-trigger.png">

- From the Trigger window, search and select `Clock` and `Next`

<img src="images/sec8-clock1.png">

- Change the settings to define the schedule for flow execution and click `Done`

<img src="images/sec8-clocksettings.png">

- Save the workflow and it will execute automatically as per the defined schedule.

</details>

## Appendix

### 1. Create User in Turbonomoic

Let us create a local user in Turbonomic with the `Observer` role.

<details><summary>CLICK me for detailed instructions</summary>

1. Create a new Local user in Turbonomoic by choosing the below menu option.

`Home > SETTINGS > Local User >  New Local User`

<img src="images/image-1-usr11.png">

2. User name could be `demo_observer`, give some password and choose role as `Observer`

3. Click `Save` button

<img src="images/image-1-usr12.png">

4. User gets created.

<img src="images/image-1-usr13.png">

</details>

### 2. Reference

- Turbonomic - Envizi Integration https://ibm.github.io/IBM-Sustainability-Software-Portfolio-Connectors/turbonomic-envizi/

- Turbonomic - Envizi Integration https://github.com/IBM/turbonomic-envizi-appconnect-flows

- Creating Envizi S3 bucket (Refer Steps 1 and 2 [here](https://developer.ibm.com/tutorials/awb-sending-udc-excel-to-s3/) to create the bucket)

- Getting started with the Turbonomic REST API : https://www.ibm.com/docs/en/tarm/8.13.6?topic=reference-getting-started-turbonomic-rest-api

- IBM Envizi ESG Suite https://www.ibm.com/docs/en/envizi-esg-suite

- Integrate your ESG Data into Envizi using Integration Hub	https://developer.ibm.com/tutorials/awb-envizi-integration-hub/

- Sign up for webMethods SaaS Trial https://signup.softwareag.cloud/#/basic-b


#### Tags
#envizi
#Sustainability
#turbonomic

#ESG Data and Environmental Intelligence
#sustainability-highlights-home