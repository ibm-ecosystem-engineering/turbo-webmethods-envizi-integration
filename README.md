# Integrating Turbo with Envizi via webMethods for Green IT data

This blog explains about the step-by-step instructions to pull green IT data from Turbonomic into Envizi via webMethods Integration.

#### Authors
 [Jeya Gandhi Rajan M](https://community.ibm.com/community/user/envirintel/people/jeya-gandhi-rajan-m1)

## Contents

- [1. Prerequisite](#1-Prerequisite)
- [2. Architecture](#2-Architecture)
- [3. Turbonomic Configuration ](#3-Turbonomic-Configuration)
- [4. Envizi's S3 bucket ](#4-envizis-s3-bucket)
- [5. webMethods Configuration](#5-webMethods-Configuration)
- [6. webMethods flow Execution](#6-webMethods-flow-Execution)

## 1. Prerequisite

- Turbonomic v8.14.3 or higher 
- webMethods SaaS or on-prem
- Envizi's S3 bucket 

## 2. Architecture

Here is the architecture  that describes about this Turbo and Envizi integration.

webMethods Integration flow pulls the list of Cloud Regions and On-prem Data Centers from Turbo and sends it to Envizi's S3 bucket in a CSV file. This CSV file will be further processed by the Envizi internally.

<img src="images/arch.png">

## 3. Turbonomic Configuration

### Mandatory Configuration

- Create a user with `Observer` role in Turbo. webMethods needs this user to fetch data from Turbo. 

### Optional Configuration 

- Add the following Tag in vCenter and add their values as tags to the Data Centers for accurate emission calculations from Envizi:
    - `Country`: Name of Country
    - `Latitude`: Latitude in Decimal Degrees format
    - `Longitude`: Longitude in Decimal Degrees format
    <img src="images/turbo-tag.png">

- By default, Envizi will use the Data Center name configured in Turbonomic/vCenter. To change this, Tag Category `EnviziAlternateName` can be added with the desired display name as its value.
- Envizi Locations (Data Centers in this case) need unique display names. If there are any Data Centers with same names, they should be changed from vCenter or Tag Category `EnviziAlternateName` should be added to the Data Center(s) with different name(s)

**Note:** Tags sync from vCenter to Turbonomic might take upto 20 minutes.

## 4. Envizi's S3 bucket 

Envizi product team would have created and shared S3 bucket. This S3 bucket details to be feed into the webMethods flow.

The webMethods flow pulls the data from Turbo and sends it to S3 bucket in a CSV file format. Envizi will further process this CSV file.

## 5. webMethods Configuration

### 5.1. Create Connectors

Need to create Amazon S3 connector and 2 HTTP connectors.

#### 5.1.1. Create Amazon S3 Connector

Need to create `Amazon S3` connector. 

1. Click on `Catalog > Amazon S3 > Add a New Account` 
<img src="images/01connector1.png">

2. Fill in the S3 details as below. (Credentials should have been given by Envizi)

```
Secret access key : Moxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
Access key ID : AKXXXXXXXXXXXXXXXXXX
```
<img src="images/01-connector-2.png">

3. The new connector account (ex: Account 2) might have been created.

<img src="images/01-connector-3.png">

#### 5.1.2. Create Http Connector 1

1. Type `http` in the filter

2. Click on `Http Connector > Add a New Account` 

<img src="images/01-connector-4.png">

3. Fill in details as below. 

- User Name :  `username` (as it is)
- API Key : Enter the Turbonomic credentials in this format `USERNAME&password=PASSWORD`.  **Example:** If the username is `test-user` and the password is `test-password`, enter `test-user&password=test-password`
- API Key Location :  `body URL encoded`

<img src="images/01-connector-5.png">
<img src="images/01-connector-6.png">

4. Click on `Connect` button.
 
5. The new connector account (ex: Account 3) might have been created.

<img src="images/01-connector-7-1-1.png">


#### 5.1.3. Create Http Connector 2

1. Click on `Http Connector > Add a New Account` 

<img src="images/01-connector-7-1-2.png">

2. Leave all the fields as it is. 

<img src="images/01-connector-7-2.png">
<img src="images/01-connector-7-3.png">

3. Click on `Connect` button.
 
4. The new connector account (ex: Account 4) might have been created.

<img src="images/01-connector-7-4.png">

### 5.2. Import the Flows - Locations

Need to import the flow and configure connector, variables and schedule.

#### 5.2.1. Import the flow

1. Click on `Dashboard > Import Flow` 

<img src="images/02-flow-1.png">

2. Download the flow file [TurbonomicLocations.yaml](./files/webMethod-flows/TurbonomicLocations.yaml). 

3. Drag and Drop the same file into the given place.

4. Click on `Import`

<img src="images/02-flow-2.png">

5. The flow should have been created.

<img src="images/flow1.png">

<img src="images/02-flow-3.png">

#### 5.2.2 Set Http Connector 1

1. Click on `HTTP` node with the label `Invoke Method`. The details are displayed in the bottom. 

<img src="images/03-flow-http1.png">

2. Select the first http account (Account 3) created above,

<img src="images/03-flow-http2.png">

#### 5.2.3 Set Http Connector 2

1. Click on `HTTP` node with the label `Invoke Method 2` and the details are displayed in the bottom. 

<img src="images/03-flow-http3-1.png">

2. Select the second http account (Account 4).

<img src="images/03-flow-http3-2.png">

#### 5.2.4. Set Http Connector 3

1. Click on `HTTP` node with the label `Invoke Method 3` and select the second http account (Account 4) for this also. 

<img src="images/03-flow-http3-3.png">

#### 5.2.5. Set S3 Connector

1. Here is the sample S3 bucket name called `envizi-data-load` created and available. This should have been given by Envizi.

<img src="images/04-flow-s3-1.png">

2. Click on `Amazon S3` node with the label `Create Object` and the details are displayed in the bottom. 

<img src="images/04-flow-s3-2.png">

3. Enter the S3 bucket name (ex: envizi-data-load).

<img src="images/04-flow-s3-3.png">

#### 5.2.6. Set Variables

1. Click on `Set Variable` node and the details are displayed in the bottom. 

- URL : The Turbonomic url
- Customer : Here 'My-Turbo-Locations' is given as an example.

<img src="images/05-flow-variable.png">

#### 5.2.7. Set Scheduler

1. Click on `Scheduler` node and the details are displayed in the bottom. 

2. The flow can be configured to run as per our need. Here it is configured to run every day at 00:05 hours.

3. The checkbox `Also run the flow, when it's first switched on` to be on, if you want to run the flow immediately after you start the flow.

<img src="images/06-flow-schedule.png">

#### 5.2.8. Dashboard

The flow is created and available.

<img src="images/07-flow-dashboard.png">

## 5.3. Import the Flows - Accounts

#### 5.3.1. Import the flow

1. Click on `Dashboard > Import Flow` 

2. Download the flow file [TurbonomicAccounts.yaml](./files/webMethod-flows/TurbonomicAccounts.yaml). 

3. Drag and Drop the same file into the given place.

4. Click on `Import`

<img src="images/10-flow-1.png">

5. The flow should have been created.

<img src="images/flow2.png">

#### 5.3.2. Set Http Connector 1

1. Click on `HTTP` node with the label `Invoke Method` and the details are displayed in the bottom. 

<img src="images/11-flow-http1.png">

2. Select the first http account (Account 3) created above.

#### 5.3.3 Set Http Connector 2, 3 and 4

1. Click on `HTTP` nodes with the labels `Invoke Method 3` , `Invoke Method 2` and `Invoke Method 5`  and the details are displayed in the bottom

2. Select the Second http account (Account 4) created above.

<img src="images/11-flow-http2.png">
<img src="images/11-flow-http3.png">
<img src="images/11-flow-http5.png">

#### 5.3.4. Set S3 connector

1. Click on `Amazon S3` node with the label `Create Object` and the details are displayed in the bottom. 

2. Enter the S3 bucket name (ex: envizi-data-load).

<img src="images/12-flow-s3-1.png">

#### 5.3.5. Set Variables

1. Click on `Set Variable` node and the details are displayed in the bottom. 

- URL : The Turbonomic url
- Customer : Here 'My-Turbo-Accounts' is given as an example.
- OverrideStartDate : Start date of the period for which the details are required from Turbo
- OverrideEndDate : End date of the period for which the details are required from Turbo. This could be the current date.

<img src="images/13-flow-variable.png">

#### 5.3.6. Set Scheduler

1. Click on `Scheduler` node and the details are displayed in the bottom. 

2. The flow can be configured to run as per our need. Here it is configured to run every day at 00:15 hours.

3. The checkbox `Also run the flow, when it's first switched on` to be on, if you want to run the flow immediately after you start the flow.

<img src="images/14-flow-schedule.png">

#### 5.3.7. Dashboard

The flow is created and available.

<img src="images/15-flow-dashboard.png">

## 6. webMethods flow Execution - Location Feed

#### 6.1. Login to webmethods io with username and password

1. Login page.

<img src="images/wMAccLogin-01.png">

#### 6.2. Create a new Project

<img src="images/wMAccNewProject-02.png">


#### 6.3. Import the project

Click on the Import and select the project location to be imported.

<img src="images/wMAccImport-03.png">

The sample data is available here.  [Accounts](./files/data/accounts/),  [Locations](./files/data/locations/).

#### 6.4. Supply Workflow name, Workflow description, AWS service

Provide the Workflow name, Workflow description and select/click on "+" to provide the AWS service and click on Import

<img src="images/wMAccWorkflow-04.png">

#### 6.5. Activate the Workflow

Toggle ON the workflow to activate

<img src="images/wMAccToggleON-05.png">

#### 6.6. Run the Workflow

Click on the run the workflow to generate the location feed and push the feed to AWS S3 bucket.

<img src="images/wMAccRun-06.png">

#### 6.7. Use another "HTTP Request" connector for DataCentre details

1. DataCentre API

Provide the API details

<img src="images/wMAccDataCentre-08.png">

2. DataCentre Headers

Provide the header details

<img src="images/wMAccDataCentreHeaders-07.png">

#### 6.8.  User "Query JSON" connector for querying JSON response object

<img src="images/wMLocQeuryJson-16.png">

#### 6.9. Use Flowservice to map and generate custom output

<img src="images/wMLocFlorServiceMap-17.png">

#### 6.10. Use "JSON to CSV" connector to make CSV format

<img src="images/wMLocJsonToCSV-18.png">

#### 6.11. Use custom connector to convert CSV into XLSX format
1. CSV to XLSX format

<img src="images/wMLocCSVToXLS-19.png">

2. Transform base64

<img src="images/wMLocCSVToXLS-20.png">

#### 6.12. Upload the XLSX file onto AWS S3 Upload file

<img src="images/wMLocS3UploadFile-21.png">


## Reference

Turbonomic - Envizi Integration https://ibm.github.io/IBM-Sustainability-Software-Portfolio-Connectors/turbonomic-envizi/

Turbonomic - Envizi Integration https://github.com/IBM/turbonomic-envizi-appconnect-flows


## Appendix

Refer the following.

#### Achieve Green IT targets by integrating Turbonomic with Envizi

This document covers how Organizations with a sustainability initiative can have an instant impact in their IT Operations by leveraging IBM Turbonomic with Envizi. 

[../01-green-it-turbo-envizi](../01-green-it-turbo-envizi/)


#### Envizi - Turbonomic Performance Dashboard 

This document describes about the Turbonomic Performance Dashboard available in Envizi.

[../03-turbonomic-performance-dashaboard](../03-turbonomic-performance-dashaboard/)

 
#envizi
#Sustainability
#turbonomic

#ESG Data and Environmental Intelligence
#sustainability-highlights-home