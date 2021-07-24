# DATA ENGINEERING: ENTREPRENEURSHIP & REAL ESATE 
### Data Engineering Capstone Project

#### Project Summary
This project is to build a data pipeline for data scientists to study the impact of entrepreneurship on real estate market, such as house value.

The project follows the follow steps:
* Step 1: Scope the Project and Gather Data
* Step 2: Explore and Assess the Data
* Step 3: Define the Data Model
* Step 4: Run ETL to Model the Data
* Step 5: Complete Project Write Up

### Step 1: Scope the Project and Gather Data

#### Scope 
##### This project is to build a data pipeline solution through transforming data in Spark then loading data to NoSQL Cassandra database to study the impact of entrepreneurship on real estate market over time. Data is extracted through two different data sources:
* Property Assessment Data provided by the City of Edmonton (API): https://data.edmonton.ca/City-Administration/Property-Assessment-Data-Historical-/qi6a-xuwt
* Business Licenses Data in data/business_licenses_edmonton.csv

#### Describe and Gather Data 
Describe the data sets you're using. Where did it come from? What type of information is included? 
* Property Assessment Data provided by the City of Edmonton (API) coming from https://data.edmonton.ca/City-Administration/Property-Assessment-Data-Historical-/qi6a-xuwt
    * This dataset has 3.38 million rows, please see property_assessment_historical.csv and the documentation here: https://data.edmonton.ca/City-Administration/Property-Assessment-Data-Historical-/qi6a-xuwt
    * Property Assessment Data includes the granular information of all the properties in Edmonton, such as its account number, assessed value, assessment_year, whether it has a garage or not, the property's house_number, its latitude, legal description, longitude, lot_size, neighbourhood_name, street_name, suite, year_built, and zoning.
* Business Licenses Data in data/business_licenses_edmonton.csv coming from https://data.edmonton.ca/Sustainable-Development/City-of-Edmonton-Business-Licenses/qhi4-bdpu.
    * This dataset includes information regarding to a business license registration, such as its category, trade name, address, licence number, licence status (issue or renewal), issue date, neighbourhood, latitude, and longitude.

##### The final solution should display the number of business licenses issued each year along with the average house value in Edmonton.
### Step 2: Explore and Assess the Data
#### Explore the Data 
* The property assessment data has a lot of missing values in columns such as point_location, year_built. Since we are not interested in those information for now, we can just exclude them. 
* The business licenses data has missing values in the address column. Since we are not interested in this information, we could also exclude this column. Additionally, this data set captures of renewed business licenses and new issued. Since we want to know how entrepreneurship impacts real estate market, we will filter out this data set by License Status = "ISSUED" in the next step.

### Step 3: Define the Data Model
#### 3.1 Conceptual Data Model
This project utilizes an ETL data pipeline for a NoSQL database (Apache Cassandra). 
* ETL is used because we are still at the exploration stage to understand the impact of entrepreneurship on real estate market. We want to reduce the volume of data that is stored to preserve storage, bandwidth, and computation resources, which in turn saves cost.
* NoSQL database was chosen because we need to process a large amount of data (over 1 million rows). 
* With Apache Cassandra you model the database tables on the queries you want to run, which is beneficial to us because we are still exploring data and different metrics to understand the impact of entrepreneurship on real estate market in Edmonton.
* In the output directory, there are houses.parquet and businesses.parquet which are two entities used to study the impact of entrepreneurship on house value (see the diagram below)
    * Because Apache Cassandra (NoSQL database) is our final analytical storage, physical database schema was used which lays out how data is stored physically on a storage system in terms of files and indices (parquet folders) to provide flexibility & optimize performance (we are processing big data - over 1 million rows).
 #### 3.2 Mapping Out Data Pipelines
* Extract: Using Pandas dataframe, data was extracted from a third-party provided API and a csv file, and read as pyspark dataframes for further cleaning and processing. 
* Transform: Leveraging PySpark, data transformation includes processing missing data, redefining the data type for each column, then doing an aggregation on the datasets to calculate the number of business licenses issued and average house value by years.
* Load: After data is transformed, it is exported as parquet files, so we could load the datasets to Cassandra table afterwards. Cassandra table has "year" as the primary key along with two corresponding values such as "avg_house_price" and "total_licenses".

### Step 4: Run Pipelines to Model the Data 
#### 4.1 Create the data model
Build the data pipelines to create the data model.

#### 4.2 Data Quality Checks
Check 1: Check if the table is loaded data (e.g. count is not zero). 

Check 2: Checking for duplicate data. 
#### 4.3 Data dictionary 
###### Final Summary Table
| Field Name | Data Type | Field Size for display | Description | Example |
| :- | :-:  | :-: | :-: | :-: |
| year | Integer | 4 | The Year of Each Metric (Primary Key) | 2019 |
| avg_house_prices | Float | 8 | The Average House Prices for A Particular Year in Edmonton | 400000.20 |
| total_licenses | Integer | 4 | The Total New Business Licenses Issued for A Particular Year in Edmonton | 30 |


###### Houses Table (output/houses.parquet)
| Field Name | Data Type | Field Size for display | Description | Example |
| :- | :-:  | :-: | :-: | :-: |
| account_number | String | 7 | Account Number Associated with Each Property | 1001007 |
| assessed_value | Integer | 10 | The Assessed Price for A Particular House in Edmonton | 534000 |
| assessment_year | Integer | 4 | The Year the Property Was Assessed in Value | 2018 |
| garage | String | 1 | Whether the Property has a Garage (N = No, Y = Yes) | N |
| house_number | Integer | 5 | The House Number of the Associated Property | 130 |
| mill_class_1 | String | 16 | The Assessment Class #1 for the property | RESIDENTIAL |
| neighborhood_name | String | 32 | The neighborhood the house is located | RIVERVIEW AREA |
| street_name | String | 32 | The Street Name the house is located at | 184 STREET |
| zoning | String | 4 | The Zone The House is Located At | AG |


###### Busineses Table (output/businesses.parquet)
| Field Name | Data Type | Field Size for display | Description | Example |
| :- | :-:  | :-: | :-: | :-: |
| category | String | 32 | General classification of business | General Contractor |
| tradeName | String | 32 | Common Name for the business | TECHSPERT SYSTEMS |
| address | String | 32 | The Municipal Address where the business is located | 9803 - 33 AVENUE NW |
| licenceNumber | String | 16 | Unique identifier for a business permit | 106807972-002 |
| licenceStatus | String | 8 | Current Status of Licence - Issued, Expired | ISSUED |
| issueDate | Date | 32 | Date Business Licence Issued | June 09, 2020 |
| neighbourhoodId | Integer | 4 | Unique number identifier for the Neighbourhood | 6570 |
| neighbourhood | String | 32 | The name of the Neighbourhood | Ozerna |
| latitude | Float | 9 | The latitude value for the geo location of the business. | 53.465356 |
| longitude | Float | 9 | The longitude value for the geo location of the business. | 54.467213 |


#### Step 5: Complete Project Write Up
* The project uses PySpark to do data transformation, Python Pandas dataframe to extract data from CSV file and API endpoint, and finally loads data to Cassandra NoSQL database. These technologies are leveraged because at an exploratory stage, data scientists could share their findings on Jupyter Notebook at a minimal to no cost. Then after the data scientists have more in depth discussion with the policymakers and the business requirements are clear, we then could migrate the data sources and analytical data store to AWS for scalability.
* Data should be updated yearly to understand the impact of entrepreneurship on real estate market so the policymakers could come up with a budget to stimulate the economy amidst economic recovery.
* Write a description of how you would approach the problem differently under the following scenarios:
 * The data was increased by 100x: NoSQL database (e.g. Cassandra) is great for handling big data, so there will be no change to the analytical data store. However, datasets stored in csv files need to be migrated to S3 data storage on AWS to save costs and optimize performance.
 * The data populates a dashboard that must be updated on a daily basis by 7am every day: The automated data pipeline in Apache Airflow needs to be configured, so at 6.30 AM the DAG is triggered to populate data in the dashboard.
 * The database needed to be accessed by 100+ people: NoSQL is good enough to handle that many concurrent traffics. However, NoSQL database should now be hosted on cloud to optimize resources.
