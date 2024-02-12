# DataEngineerZoomCamp2024Week3

[Course Website](https://dezoomcamp.streamlit.app/Module%203%20Data%20Warehouse%20and%20BigQuery)

![CleanShot 2024-02-11 at 21 22 29](https://github.com/alangan17/DataEngineerZoomCamp2024Week3/assets/14330702/3cbadc0f-7211-4b64-bba5-0eeebaf1eeef)

[Slides](https://docs.google.com/presentation/d/1a3ZoBAXFk8-EhUsd7rAZd-5p_HpltkzSeujjRGB2TAI/edit#slide=id.p)

[BigQuery Basic SQL](https://github.com/DataTalksClub/data-engineering-zoomcamp/blob/main/03-data-warehouse/big_query.sql)

# Lesson Learned
## Data Warehouse
### OLAP vs OLTP

### What is data warehouse
- OLAP solution
- Used for reporting and data analysis

### BigQuery
- No servers to manage or database software to install
- Separates compute engines and storage

## Partitioning vs Clustering
### Partitioning
- Time-unit column
- Max. limit: 4000
- Known cost saved (how much data will be scanned) upfront
- Not much improvement with table size < 1GB

### Clustering
- Queries commonly use filters or aggregation against multiple columns
- Cardinality of the number of values in a column or group of columns is large
- Unknown cost saved (how much data will be scanned) upfront
- Not much improvement with table size < 1GB
- BigQuery performs auto re-clustering in the background

## BigQuery Best Practices

## BigQuery Internals

## BigQuery Machine Learning

## BigQuery Machine Learning Deployment

# Homework
ATTENTION: At the end of the submission form, you will be required to include a link to your GitHub repository or other public code-hosting site. This repository should contain your code for solving the homework. If your solution includes code that is not in file format (such as SQL queries or shell commands), please include these directly in the README file of your repository.

<b><u>Important Note:</b></u> <p> For this homework we will be using the 2022 Green Taxi Trip Record Parquet Files from the New York
City Taxi Data found here: </br> https://www.nyc.gov/site/tlc/about/tlc-trip-record-data.page </br>
If you are using orchestration such as Mage, Airflow or Prefect do not load the data into Big Query using the orchestrator.</br> 
Stop with loading the files into a bucket. </br></br>
<u>NOTE:</u> You will need to use the PARQUET option files when creating an External Table</br>

<b>SETUP:</b></br>
Create an external table using the Green Taxi Trip Records Data for 2022. </br>

Steps:
1. Go to Google Cloud Storage, open a new bucket (Note: Make sure the bucket is in the same location as BigQuery)
2. Go to Google Cloud Console (https://console.cloud.google.com)
3. Open the editor, paste the bash code and save it
```bash
#!/bin/bash

# Set the bucket name where you want to upload the files
BUCKET_NAME=dez2024-wk3-ny-green-taxi
# Base URL for the files
BASE_URL=https://d37ci6vzurychx.cloudfront.net/trip-data

# Loop through all the months
for MONTH in {01..12}; do
  # Form the file name based on the year and month
  FILE_NAME=green_tripdata_2022-$MONTH.parquet
  # Download the file
  echo "Downloading $FILE_NAME..."
  curl -O "$BASE_URL/$FILE_NAME"
  # Upload the file to GCS
  echo "Uploading $FILE_NAME to GCS..."
  gsutil cp $FILE_NAME gs://$BUCKET_NAME/$FILE_NAME
  # Optionally, remove the file after upload to save space
  rm $FILE_NAME
done

echo "All files have been downloaded and uploaded to GCS."
```
3. Go to the terminal, run the bash file
4. You'll see 12 files downloaded from the data source and uploaded to the GCP

5. Go to BigQuery
```sql
-- Creating external table referring to gcs path
CREATE OR REPLACE EXTERNAL TABLE `dez2024-413305.nytaxi.external_green_tripdata_2022`
OPTIONS (
  format = 'PARQUET',
  uris = ['gs://dez2024-wk3-ny-green-taxi/green_tripdata_2022-*.parquet']
);
```
6. Test the external table
```sql
SELECT *
FROM dez2024-413305.nytaxi.external_green_tripdata_2022
LIMIT 10;
```


> Create a table in BQ using the Green Taxi Trip Records for 2022 (do not partition or cluster this table). </br>
7. Create table from the external table
```sql
-- Create a non partitioned table from external table
CREATE OR REPLACE TABLE dez2024-413305.nytaxi.green_tripdata_non_partitoned AS
SELECT * FROM dez2024-413305.nytaxi.external_green_tripdata_2022;
```
</p>

## Question 1:
Question 1: What is count of records for the 2022 Green Taxi Data??
```sql
SELECT COUNT(*) FROM dez2024-413305.nytaxi.green_tripdata_non_partitoned;
```

- 65,623,481
- -> 840,402
- 1,936,423
- 253,647

## Question 2:
Write a query to count the distinct number of PULocationIDs for the entire dataset on both the tables.</br> 
What is the estimated amount of data that will be read when this query is executed on the External Table and the Table?
![CleanShot 2024-02-11 at 22 26 49](https://github.com/alangan17/DataEngineerZoomCamp2024Week3/assets/14330702/28e17108-29bc-44ab-8570-361b857bb97c)
![CleanShot 2024-02-11 at 22 27 47](https://github.com/alangan17/DataEngineerZoomCamp2024Week3/assets/14330702/9d3ef327-2a24-4fb4-9101-d4c2317286c8)

Note: both queries billed 10 MB after execution

- -> 0 MB for the External Table and 6.41MB for the Materialized Table
- 18.82 MB for the External Table and 47.60 MB for the Materialized Table
- 0 MB for the External Table and 0MB for the Materialized Table
- 2.14 MB for the External Table and 0MB for the Materialized Table


## Question 3:
How many records have a fare_amount of 0?
```sql
SELECT count(*)
FROM dez2024-413305.nytaxi.green_tripdata_non_partitoned
WHERE fare_amount = 0
;
```
![CleanShot 2024-02-11 at 22 32 30](https://github.com/alangan17/DataEngineerZoomCamp2024Week3/assets/14330702/16f2ff15-a043-4811-8891-c5bf5cf7515e)

- 12,488
- 128,219
- 112
- -> 1,622

## Question 4:
What is the best strategy to make an optimized table in Big Query if your query will always order the results by PUlocationID and filter based on lpep_pickup_datetime? (Create a new table with this strategy)
- Cluster on lpep_pickup_datetime Partition by PUlocationID
- Partition by lpep_pickup_datetime  Cluster on PUlocationID
- Partition by lpep_pickup_datetime and Partition by PUlocationID
- Cluster on by lpep_pickup_datetime and Cluster on PUlocationID

## Question 5:
Write a query to retrieve the distinct PULocationID between lpep_pickup_datetime
06/01/2022 and 06/30/2022 (inclusive)</br>

Use the materialized table you created earlier in your from clause and note the estimated bytes. Now change the table in the from clause to the partitioned table you created for question 4 and note the estimated bytes processed. What are these values? </br>

Choose the answer which most closely matches.</br> 

- 22.82 MB for non-partitioned table and 647.87 MB for the partitioned table
- 12.82 MB for non-partitioned table and 1.12 MB for the partitioned table
- 5.63 MB for non-partitioned table and 0 MB for the partitioned table
- 10.31 MB for non-partitioned table and 10.31 MB for the partitioned table


## Question 6: 
Where is the data stored in the External Table you created?

- Big Query
- GCP Bucket
- Big Table
- Container Registry


## Question 7:
It is best practice in Big Query to always cluster your data:
- True
- False


## (Bonus: Not worth points) Question 8:
No Points: Write a `SELECT count(*)` query FROM the materialized table you created. How many bytes does it estimate will be read? Why?
