
## Convert trip data to ORC

```sql
-- Create a table
Drop table if exists trip_orc_uncom;
CREATE external table trip_orc_uncom
(
  medallion Numeric, hack_license Numeric,
  vendor_id STRING,  rate_code Numeric,
  store_and_fwd_flag Numeric,  pickup_datetime STRING,
  dropoff_datetime STRING,  passenger_count Numeric,
  trip_time_in_secs Numeric, trip_distance Numeric,
  pickup_longitude Numeric, pickup_latitude Numeric ,
  dropoff_longitude Numeric, dropoff_latitude Numeric  
)
stored as ORC
Location '/user/${env:USER}/taxi_trip/trip_orc_uncom'
TBLPROPERTIES ("orc.compress"="NONE");
--load data into trip_orc_uncom
INSERT into trip_orc_uncom
SELECT * from trip_data_10_13;
```

NOTE: It's important to make sure that compressed parameter in `TBLPROPERTIES` is `NONE` cause the default compressed of ORC is ZLIB. Otherwise, the table will be compressed and the performance evaluation can be biased.

In compression case, you have:

```sql
Drop table if exists trip_orc_com;
Create external table trip_orc_com
(
  -- all fields described below
)
stored as ORC
Location '/user/${env:USER}/taxi_trip/trip_orc_com'
TBLPROPERTIES ("orc.compress"="ZLIB");
-- load data into trip_orc_com
insert into trip_orc_com
select * from trip_data_10_13;
```

## Convert trip data to Parquet 

```sql
-- Uncompressed case
DROP table if exists trip_par_uncom;
CREATE external table trip_par_uncom
(
  -- all fields described below 
)
STORED AS Parquet
Location '/user/${env:USER}/taxi_trip/trip_par_uncom'
TBLPROPERTIES ("parquet.compress"="NONE");

SET hive.exec.compress.output = false;
INSERT into trip_par_uncom
SELECT * from trip_data_10_13;

-- Compressed case

DROP table if exists trip_par_com;
Create external table trip_par_com
(
  -- all fields described below
)
STORED AS Parquet
Location '/user/${env:USER}/taxi_trip/trip_par_com'
TBLPROPERTIES ("parquet.compress"="SNAPPY");
SET hive.exec.compress.output=true;
SET parquet.compression=SNAPPY; 
INSERT into trip_par_com
SELECT * from trip_data_10_13;
```

## Convert trip data to CSV 

```sql
-- Uncompressed case
DROP table if exists trip_csv_uncom;
CREATE external table trip_csv_uncom
(
  -- all fields described below
)
ROW FORMAT SERDE 'org.apache.hadoop.hive.serde2.OpenCSVSerde'
STORED AS TEXTFILE
Location '/user/${env:USER}/taxi_trip/trip_csv_uncom';

SET hive.exec.compress.output = false;
INSERT into trip_csv_uncom
SELECT * from trip_data_10_13;

-- Compressed case

DROP table if exists trip_csv_com;
Create external table trip_csv_com
(
  -- all fields described below
)
ROW FORMAT SERDE 'org.apache.hadoop.hive.serde2.OpenCSVSerde'
STORED AS TEXTFILE
Location '/user/${env:USER}/taxi_trip/trip_csv_com';
SET hive.exec.compress.output=true;
SET mapred.output.compression.codec=org.apache.hadoop.io.compress.GzipCodec;
INSERT into trip_csv_com
SELECT * from trip_data_10_13;
```

## Convert trip data to JSON 

```sql
-- Uncompressed case
DROP table if exists trip_json_uncom;
CREATE external table trip_json_uncom
(
  -- all fields described below  
)
ROW FORMAT SERDE 'org.apache.hive.hcatalog.data.JsonSerDe'
STORED AS TEXTFILE
Location '/user/${env:USER}/taxi_trip/trip_json_uncom';

SET hive.exec.compress.output = false;
INSERT into trip_json_uncom
SELECT * from trip_data_10_13;

-- Compressed case

DROP table if exists trip_json_com;
Create external table trip_json_com
(
  -- all fields described below
)
ROW FORMAT SERDE 'org.apache.hive.hcatalog.data.JsonSerDe'
STORED AS TEXTFILE
Location '/user/${env:USER}/taxi_trip/trip_json_com';
SET hive.exec.compress.output=true;
SET mapred.output.compression.codec=org.apache.hadoop.io.compress.GzipCodec;
INSERT into trip_json_com
SELECT * from trip_data_10_13;
```

## Convert trip data to AVRO 

```sql
-- Uncompressed case
DROP table if exists trip_avro_uncom;
CREATE external table trip_avro_uncom
(
  -- all fields described below
)
row format serde 'org.apache.hadoop.hive.serde2.avro.AvroSerDe'
STORED as AVRO
Location '/user/${env:USER}/taxi_trip/trip_avro_uncom';

SET hive.exec.compress.output = false;
INSERT into trip_avro_uncom
SELECT * from trip_data_10_13;

-- Compressed case

DROP table if exists trip_avro_com;
Create external table trip_avro_com
(
  -- all fields described below
)
row format serde 'org.apache.hadoop.hive.serde2.avro.AvroSerDe'
STORED as AVRO
Location '/user/${env:USER}/taxi_trip/trip_avro_com';
SET hive.exec.compress.output=true;
SET mapred.output.compression.codec=org.apache.hadoop.io.compress.GzipCodec;
INSERT into trip_avro_com
SELECT * from trip_data_10_13;
```

## Processing time and storage results from each conversion

<div style="width:100%; position:relative; overflow:auto; background:#ccc; padding: 0 1rem; margin-bottom:1rem;">

Dataset| format type| Compressed/no compressed| raw size in HDFS (GB) |Duration time (s) | CPU Time (s) 
--- |  --- | --- | --- |  --- |  --- 
Trip data | CSV |`raw data`|`86.8GB`| |
Trip data | CSV |no|76.1  | 282.457| 7753.900
Trip data | CSV |yes|9.9 |328.436|9791.830 
Trip data| JSON|no|222.0 |   268.737 |	6686.680 
Trip data |JSON|yes| 11.5 | 301.896| 8793.250
Trip data  |AVRO|no |55.2   | 241.246| 7060.400
Trip data |AVRO |yes | 9.2 | 294.635	| 8892.580
Trip data | ORC|no|19.8  |  210.732	|6239.120 
Trip data |ORC|yes |6.7  |221.852|6558.470
Trip data |Parquet|no|58.1 | 227.115| 6823.870 
Trip data |Parquet |yes|58.1 | 407.041	|9585.66
</div>
