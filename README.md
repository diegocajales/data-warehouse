# data-warehouse

## Homework 3: Data Warehousing

### Question 1 - Count of records for the 2024 Yellow Taxi Data

First, we create the external table

```sql
CREATE OR REPLACE EXTERNAL TABLE `data-warehouse-486723.nytaxi.external_yellow_tripdata`
OPTIONS (
  format = 'PARQUET',
  uris = ['gs://diego-warehouse/yellow_tripdata_2024-*.parquet']
);
```

Then we use this query

```sql
-- Check yellow trip data
SELECT 
  COUNT(1)
FROM 
  nytaxi.external_yellow_tripdata;
```

```
20332093
```

### Question 2 - Estimated amount of data that will be read on the External Table and the Table

First, we create the table

```sql
CREATE OR REPLACE TABLE `data-warehouse-486723.nytaxi.yellow_tripdata` AS
SELECT * FROM `data-warehouse-486723.nytaxi.external_yellow_tripdata`;
```

And then we use the respectives queries

```sql
SELECT DISTINCT
  COUNT(eyt.PULocationID) 
FROM 
  nytaxi.external_yellow_tripdata AS eyt;


SELECT DISTINCT
  COUNT(yt.PULocationID) 
FROM 
  nytaxi.yellow_tripdata AS yt;
```

Reading 0 MB for the External Table and 155.12 MB for the Materialized Table

### Question 3 - Difference of Bytes between queries

When writting these two queries

```sql
SELECT
  yt.PULocationID 
FROM 
  nytaxi.yellow_tripdata AS yt;

SELECT
  yt.PULocationID,
  yt.DOLocationID
FROM 
  nytaxi.yellow_tripdata AS yt;
```

The second one, represents the double of the first one, because of BigQuery being a colunar database

### Question 4 - Records that have a fare_amount of 0

This query

```sql
SELECT
  COUNT(1)
FROM 
  nytaxi.yellow_tripdata AS yt
WHERE
  yt.fare_amount = 0;
```

Returns the an amount of 8333

### Question 5 - Best strategy to make an optimized table in Big Query if your query will always filter based on tpep_dropoff_datetime and order the results by VendorID

Partition by tpep_dropoff_datetime and Cluster on VendorID

### Question 6 - Difference between queries on normal and partitioned-clustered tabble

This query creates the mentioned table

```sql
CREATE OR REPLACE TABLE `data-warehouse-486723.nytaxi.yellow_tripdata_partioned_clustered`
PARTITION BY DATE(tpep_dropoff_datetime)
CLUSTER BY VendorID AS
SELECT * FROM `data-warehouse-486723.nytaxi.external_yellow_tripdata`;
```

Then we compare the amount of data processed with these queries

```sql
SELECT DISTINCT
  COUNT(yt.VendorID)
FROM 
  `data-warehouse-486723.nytaxi.yellow_tripdata` yt
WHERE
  tpep_dropoff_datetime between '2024-03-01' and '2024-03-15';

SELECT DISTINCT
  COUNT(ytpc.VendorID)
FROM 
  `data-warehouse-486723.nytaxi.yellow_tripdata_partioned_clustered` ytpc
WHERE
  tpep_dropoff_datetime between '2024-03-01' and '2024-03-15';
```

### Question 7 - Where is the data stored in the External Table created

Container Registry

### Question 8 - It is best practice in Big Query to always cluster your data

No. One basic example, is that it does not gain benefit in smaller tables
