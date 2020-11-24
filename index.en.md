---
title: Performance comparison of different file formats
permalink: performance-comparison-of-file-formats
description: Compare the performance of different file formats (ORC, Parquet, CSV, JSON, AVRO) in Hive engine
lang: en
author: aida
date: 2020-08-14
image: feature/file-format-storage.png
categories:
  - data-engineering
  - data-science
tags:
  - big-data
  - apache-avro
  - apache-hdfs
  - apache-hive
  - apache-orc
  - apache-parquet
  - data-lake
  - file-format
  - json
  - ?metrics
  - ?data-structures
  - ?data-warehouse
  - ?apache-hadoop
  - ?automation
  - ?batch
  - ?apache-mapreduce
---

Choosing an appropriate file formats is essential, whether your data transits on the wire or is stored at rest. Each file format comes with its own advantages and disadvantages. We covered them in a precedent article presenting and comparing the [most popular file formats in Big data](/en/2020/07/23/benchmark-study-of-different-file-format/). In a follow up article, we will compare their performance according to multiple scenarios. The compression used for a given format greatly impact the query performances. This article will prepare the tables needed for this follow up article and takes the opportunity to compare the compression algorithms in term of storage spaces and generation time. It will also helps us to select the most appropriate compression algorithms for each format.

## Database Choice
  
Apache Hive is a Data Warehouse software commonly used with Hadoop to give users the ability to perform SQL queries. We choose it for the following reasons. It supports both an OLAP and OLTP types of queries. It can take fully advantage of the distributed data processing. It supports several types of [file formats](https://cwiki.apache.org/confluence/display/Hive/FileFormats). In some circumstances, using an optimized language like HiveQL present the advantage of preventing the code execution from human mistakes and enables potential engine optimizations. HiveQL being based on SQL, it is an [declarative language](https://en.wikipedia.org/wiki/Declarative_programming) which makes the model definition and the query declaration easy to express and read in this article.

## Characteristics of the cluster and datasets

Our environment is running the HDP distribution 3.1 from Hortonwork and now distributed by [Cloudera](https://www.cloudera.com/). In order to better assess the raw performance of file format, it is relevant to work on a relatively large data set. At least 10 GB is to be considered the minimum, a few hundred of GBs would be much more relevant. 

We selected 3 datasets with different characteristics to evaluate the behaviors of the selected file formats on different data structures. Several types of requests were created to illustrate several use-cases such as records picking, data crunching, large joins queries, ...

* [NYC taxi](https://uofi.app.box.com/v/NYCtaxidata/folder/2332218797) is a time series data. The datasets contain trip and fare data represented on multiples months and years in CSV format. We created a dataset that combines data from several years (from 2010 to 2013). The raw data of trip have about 86.8 GB and fare takes up 49.9 GB.
    
* [IMDB](https://datasets.imdbws.com/) is a relational data. It provides seven tables in TSV format: `imdb.name.basics`, `imdb.title.basics`, `imdb.title.crew`, `imdb.title.akas`, `imdb.title.ratings`, `imdb.title.episode` and `imdb.title.principals`. Their total size in uncompressed format is 4.7 GB. 

* [wikipedia](https://archive.org/details/wikimediadownloads?and%5B%5D=%22Wikidata%20entity%20dumps%22) is a semi-structured data. It takes up 43.8 GB of uncompressed data in JSON format. The data contains a nested data structures.
  
## Load data in HDFS

If your data is present on your local machine, you can push it directly to HDFS. This implies to have your local machine set up with a configured HDFS client and Kerberos. You can also use the HTTP REST API of HDFS. If Kerberos is activated on your targeted cluster, the REST queries are not so trivial.

If you have access to a configured edge node, you can execute the `hdfs` command from there. Do not move your data to an edge node with SSH or SCP and upload it later into HDFS. Edges nodes are not sized to store large amount of data. Instead, pipe the data with the `hdfs` command. In our case, the data is available over HTTP, we use the `curl` command to fetch the URL and redirect its output in HDFS:

```bash
# Use curl command to load data
source='http://...'
hdfs dfs -put <(curl -sS -L $source) /user/${env:USER}/dataset
```

## Schema declaration in Hive

The NYC taxi datasets is made of 2 tables:

```sql
-- trip data
CREATE EXTERNAL TABLE trip_data
( 
  medallion NUMERIC, hack_license NUMERIC, vendor_id STRING, rate_code NUMERIC,
  store_and_fwd_flag NUMERIC, pickup_datetime STRING, dropoff_datetime STRING, 
  passenger_count NUMERIC, trip_time_in_secs NUMERIC, trip_distance NUMERIC, 
  pickup_longitude NUMERIC, pickup_latitude NUMERIC, dropoff_longitude NUMERIC, 
  dropoff_latitude NUMERIC 
)
ROW FORMAT DELIMITED
FIELDS TERMINATED BY ','
STORED AS TEXTFILE
LOCATION '/user/${env:USER}/trip_data/';

-- fare data 
CREATE EXTERNAL TABLE fare_data
(
  medallion NUMERIC, hack_license NUMERIC, vendor_id STRING, pickup_datetime STRING,
  payment_type STRING, fare_amount NUMERIC, surcharge NUMERIC, mta_tax NUMERIC,
  tip_amount NUMERIC, tolls_amount NUMERIC, total_amount NUMERIC
)
ROW FORMAT DELIMITED
FIELDS TERMINATED BY ','
STORED AS TEXTFILE
LOCATION '/user/${env:USER}/fare_data/';
```

The IMDB dataset is made of seven tables:

```sql
-- imdb_title_basics
CREATE EXTERNAL TABLE imdb_title_basics
(
  tconst STRING, titletype STRING, primarytitle STRING, originaltitle STRING, 
  isadult STRING, startyear INT, endyear INT, runtimeminutes INT, genres ARRAY<string>
)
ROW FORMAT DELIMITED
FIELDS TERMINATED BY '\t'
STORED AS TEXTFILE
LOCATION '/user/${env:USER}/imdb_title_basics/';

-- imdb_name_basics
CREATE EXTERNAL TABLE imdb_name_basics
( 
  nconst STRING, primaryname STRING, birthyear INT, deathyear INT, 
  primaryprofession ARRAY<string>, knownfortitles ARRAY<string>
)
ROW FORMAT DELIMITED
FIELDS TERMINATED BY '\t'
STORED AS TEXTFILE
LOCATION '/user/${env:USER}/imdb_name_basics/';

--imdb_title_principal
CREATE EXTERNAL TABLE imdb_title_principal
(
  tconst STRING, ordering NUMERIC, nconst STRING, category STRING, 
  job ARRAY<string>, characters ARRAY<string> 
)
ROW FORMAT DELIMITED
FIELDS TERMINATED BY '\t'
STORED AS TEXTFILE
LOCATION '/user/${env:USER}/imdb_title_principal/';

-- imdb_title_akas
CREATE EXTERNAL TABLE imdb_title_akas
( 
  titleId STRING, ordering NUMERIC, title STRING, region STRING, 
  language STRING, types STRING, attributes STRING, isOriginalTitle NUMERIC 
)
ROW FORMAT DELIMITED
FIELDS TERMINATED BY '\t'
LOCATION '/user/${env:USER}/imdb_title_akas/';

-- imdb_title_ratings
CREATE EXTERNAL TABLE imdb_title_ratings
(
  tconst STRING, averagerating DOUBLE, numvotes INT
)
ROW FORMAT DELIMITED
FIELDS TERMINATED BY '\t'
STORED AS TEXTFILE
LOCATION '/user/${env:USER}/imdb_title_ratings/';

--imdb_title_Crew
CREATE EXTERNAL TABLE imdb_title_crew
(
  tconst STRING, director ARRAY<string>, writers ARRAY<string> 
)
ROW FORMAT DELIMITED
FIELDS TERMINATED BY '\t'
STORED AS TEXTFILE
LOCATION '/user/${env:USER}/imdb_title_crew/';

--imdb_title_episode
CREATE EXTERNAL TABLE imdb_title_episode
(
  tconst STRING, parentTconst STRING, seasonNumber NUMERIC, episodeNumber NUMERIC
)
ROW FORMAT DELIMITED
FIELDS TERMINATED BY '\t'
STORED AS TEXTFILE
LOCATION '/user/${env:USER}/imdb_title_episode/';
```

The Wikiperia dataset is made of one table:

```sql
CREATE EXTERNAL TABLE wikipedia_json
(
  id STRING, `type` STRING, 
  aliases MAP<string, array<struct<language:string, value:string>>>,              
  descriptions MAP<string, struct<language:string, value:string>>,
  labels MAP<string, struct<language:string, value:string>>,        
  claims MAP<string, array<struct<id:string, mainsnak:struct<snaktype:string, property:string,
  datatype:string, datavalue:struct<value:string, type:string>>, type:string, `rank`:string>>>, 
  sitelinks MAP<string, struct<site:string, title:string, badges:string>>
)        
ROW FORMAT SERDE 'org.openx.data.jsonserde.JsonSerDe'
WITH SERDEPROPERTIES ( "ignore.malformed.json"="true", "skip.header.line.count"="1")
STORED AS TEXTFILE
LOCATION '/user/${env:USER}/wikipedia/';
```

Our table are declared in Hive as `external`. This allows Hive to define a table which is not managed by Hive and whose content is placed anywhere on HDFS.

Here are two simple queries to ensure our table are declared and our data are readable:

```sql
DESCRIBE FORMATTED trip_data;
SELECT * FROM file_name LIMIT 10;
```

## Converting data into another file format

Each conversion is tested with and without compression. This is done in ORC, Parquet, CSV, JSON and AVRO formats.

In Hive, the CSV format can be declared in different manner, either using `CSV Serde` or `CSV simplified format` from TEXTFILE formats. The first includes all the characteristics related to CSV format such that the separator, quote and escape characters. However, it contains a limitation by treating every columns as a string type. The second was used instead. While the CSV implementation is more naive, it preserves the native data type of each columns.  

File formats can support all or a selected list of compression algorithms called codecs. For example, ORC and Parquet respectively default to ZLIB and GZIP and both additionnaly support Snappy. Text file formats have no restriction on the compression being applied. The available codecs are limited by what the data warehouse supports. In our case, there are many [different types of compression](https://docs.cloudera.com/documentation/enterprise/6/6.3/topics/introduction_compression.html) supported in Hive. In addition to Snappy and gzip, we choose to test bzip2 and lz4. About AVRO, it supports by default in Hive Deflate and Snappy where both were tested. Nevertheless, it can be additionally compressed by [other codec](https://avro.apache.org/docs/current/spec.html). 

In order to automate this process, we wrote a bash script which creates and retrieves the metrics before deleting the table.  

```bash
# Create table and retrieve the generation time of conversion
username=`whoami`
for i in /home/${username}/requests_file/*;
do
metrics_file="$(echo "$i" | awk -F"/" '{print $NF}')"."csv"
sh <<EOF
beeline -u "jdbc:hive2://zoo-1.au.adaltas.cloud:2181,zoo-2.au.adaltas.cloud:2181,zoo-3.au.adaltas.cloud:2181/dsti;serviceDiscoveryMode=zooKeeper;zooKeeper$"
 -f "$i" 2>&1 | awk '/^[INFO].*VERTICES.*$/||/^[INFO].*Map .*$/||/^[INFO].*Reducer .*$/' >> "$metrics_file"
EOF 
# Retrieve the storage space
hdfs dfs -du -s -h /user/${username}/tables/* >>"$metrics_file"
# Delete table 
hdfs dfs -rm -r -skipTrash /user/${username}/tables/*
done
```` 

An example of the `$i` variable could be:

```sql
--Convert trip data to ORC 
CREATE EXTERNAL TABLE trip_data_orc
(
   -- all fields described above
)
STORED AS ORC
LOCATION '/user/${env:USER}/trip_data/trip_data_orc'
TBLPROPERTIES ("orc.compress"="ZLIB");
SET hive.exec.compress.output=true;
insert into trip_data_orc
select * from trip_data;
```

**NOTE**: 

- Make sure that the file name from `/home/${username}/requests_file/` is well described 
- The command to retrieve the storage space generates two columns. The first column provides the raw size of the files in HDFS directories. The second column indicates the actual space consumed by these files in HDFS. The second column provides a value much higher than the first due to [replication factor](https://hadoop.apache.org/docs/r1.2.1/hdfs_design.html) configured in HDFS.
- We deliberately choose to not convert the Wikipedia dataset into CSV because of its structured which does not perfectly fit the CSV constrains.
- For more reliability, these tests were done on three repetitions. The final result described in the next section is represented by the mean value. 

## Results

Dataset | Format-codec | Compressed | Raw size (GB) | Storage ratio | Duration time (s) | CPU Time (s) 
--- |  --- | --- | --- |  --- | --- 
Trip data | CSV            |no  | 86.8  | 39%  |      |
Trip data | CSV-Lz4        |yes | 17.2  | 8%   | 140  | 4002
Trip data | CSV-Snappy     |yes | 16.5  | 7%   | 147  | 4007
Trip data | CSV-Gzip       |yes | 9.9   | 4%   | 251  | 7134 
Trip data | CSV-Bzip       |yes | 6.0   | 3%   | 459  | 13730 
Trip data | JSON           |no  | 222.0 | 100% | 282  | 6848
Trip data | JSON-Lz4       |yes | 22.3  | 10%  | 281  | 6402
Trip data | JSON-Snappy    |yes | 26.7  | 12%  | 292  | 7222
Trip data | JSON-Gzip      |yes | 11.5  | 5%   | 324  | 9196
Trip data | JSON-Bzip      |yes | 6.5   | 3%   | 2020 | 60201
Trip data | AVRO           |no  | 55.2  | 25%  | 273  | 7459
Trip data | AVRO-Snappy    |yes | 9.2   | 4%   | 321	| 9613
Trip data | AVRO-Deflate   |yes | 9.2   | 4%   | 417	| 9808
Trip data | ORC            |no  | 19.8  | 9%   | 222	| 6481
Trip data | ORC-Snappy     |yes | 10.0  | 5%   | 227  | 6746
Trip data | ORC-Zlib       |yes | 6.7   | 3%   | 244  | 7046
Trip data | Parquet        |no  | 58.1  | 26%  | 253  | 7073
Trip data | Parquet-Gzip   |yes | 58.1  | 26%  | 262  | 7337
Trip data | Parquet-Snappy |yes | 58.1  | 26%  | 299	| 8014


Dataset | Format-codec | Compressed | Raw size (GB) | Storage ratio | Duration time (s) | CPU Time (s) 
--- |  --- | --- | --- |  --- |  --- 
Wikipedia | JSON           |no  | 43.8 | 100% |      |	
Wikipedia | JSON-Lz4       |yes | 5.9  | 21%  | 1978 | 2743
Wikipedia | JSON-Snappy    |yes | 6.5  | 15%  | 1928 | 2567
Wikipedia | JSON-Gzip      |yes | 3.6  | 8%   | 2620 | 2824
Wikipedia | JSON-Bzip      |yes | 2.4  | 5%   | 7610 | 8007
Wikipedia | AVRO           |no  | 17.4 | 40%  | 1969 | 2341
Wikipedia | AVRO-Snappy    |yes | 3.7  | 8%   | 2412 | 2891
Wikipedia | AVRO-Deflate   |yes | 3.7  | 8%   | 2850 | 3121
Wikipedia | ORC            |no  | 7.0  | 16%  | 2221 | 2475 
Wikipedia | ORC-Snappy     |yes | 4.1  | 9%   | 2215 | 2398
Wikipedia | ORC-Zlib       |yes | 2.8  | 6%   | 2229 | 2487
Wikipedia | Parquet        |no  | 10.3 | 24%  | 1827 | 2222 
Wikipedia | Parquet-Snappy |yes | 10.3 | 24%  | 1778 | 2190 
Wikipedia | Parquet-Gzip   |yes | 10.3 | 24%  | 1957 | 2365

## Performance comparison 

Let's analyze the results. The storage ratios are based on the uncompressed JSON file format used as a reference.

### Text based formats without compression

In both datasets, JSON is the format taking the most space in its uncompressed form. It is not a surprise. JSON and CSV are the only text formats. As such, they are the one using the most space. By separating values by a delimiter such as a comma, CSV has a minimalist syntax and is smaller than JSON. The taxi driver contains a lot small numeric values which creates a lot of syntax, thus increasing the difference between JSON and CSV. This difference would have been lighter in the Wikipedia dataset where the values are made of long strings. For the same reasons, XML would have yield to a much higher storage usage than JSON. 

### Text based formats with compression

In terms of space, the best option of compression is bzip by achieving for example in NYC taxi a ratio of storage equal to 3% in CSV and JSON. Then, we have gzip whose its gap with bzip2 is not very significant. However in terms of generation time, the latter always provides twice as long as gzip. This means that promotes gzip in case where the speed of compression has an importance will be more efficient than bzip2. For example in archive case, gzip could be a relevant choice since it will process faster the compression and provides a storage space not too far from bzip2.

Unlike JSON format, the ratio of storage from lz4 in CSV is slightly greater than snappy. This mean that depending to the file formats, the appropriate codec must be chosen properly. Therefore, lz4 generates less time of compression than snappy in case of NYC taxi data. 

### Binary based formats without compression

ORC takes up less space than parquet and AVRO. Being the default format of Hive, [ORC](https://orc.apache.org/docs/) provides a very efficient way to store Hive data. Plus, It was designed to overcome limitations of the other Hive file formats. Its design can explain the efficiency of storage space it occurs. In NYC taxi, the difference of ratio of storage between AVRO and parquet is quite insignificant. While in Wikipedia, parquet optimizes significantly the space than AVRO. This important gap could be explained to how the datasets is handled by each of them. Both support nested data, but parquet is built from the ground up with complex nested data structures. Thus, Parquet could store more efficiently nested data than AVRO with Wikipedia data. 

### Binary based formats with compression

Let's look at each of binary file formats.

- ORC 

ORC with zlib produces respectively in both datasets an efficient ratio of storage to 3% and 6% than snappy to 5% and 9%. The latter is slightly better in terms of generation time than zlib. Since, their difference is not to important, promotes zlib in ORC will be more advantageous.

- Parquet 

The compression of parquet doesn't provide an effect on the size of data even in terms of bytes. Normally, as it supports two codecs in Hive, we have expected at least get a little difference between them. But in reality this is explained to the file formats itself. Parquet is probably compressed but you won't see it directly. Because, the compression is baked in into the format. Instead of the whole file being compressed, individual segments are compressed using the specified algorithm. In addition to that, it has the ability to efficiently encode columns in which the number of unique values is fairly small. This can therefore lead to a significant compression within the format. 

According to this result, it will be difficult to choice an appropriate codec for parquet in Hive. Nevertheless, the choice could be based on snappy as it provides at least an efficient speed of compression.

- AVRO

The global size between codecs looks the same. But in terms of bytes, the difference between snappy AVRO and deflate AVRO is 300 bytes in NYC taxi datasets. While Wikipedia, the difference between them is 3 bytes. In both cases, snappy has a higher byte value than deflate but a faster compression speed. 

## Conclusion

In general, store data in columnar format offers an optimized storage than row file formats. ORC is the best choice than parquet both in compression and without compression. Between row file formats, the choice depends to the data structure and compression case. Without compression, AVRO is an appropriate format for both datasets. With compression, specially in NYC taxi, CSV with Bzip2 is more compressible respectively than AVRO and JSON. The latter looks not suitable for storing even in terms of generation time. In that case, it is preferable to avoid it as much as possible. 

In terms of generation time from codecs, the speed seems to be correlated to the storage space. The better the gain of storage, the longer it takes to perform the compression task. 

Through this description, we got an insight of how each file formats behaves in terms of storage and compression. Let's take a look in the [next article]() how they perform read requests based on the decompression and compression case. For the latter, we will take the codecs types which optimize better the storage of each file formats. 

## Additional information

- Add data into HDFS

Sometimes when you try to add data into HDFS, some error could be raised as following:

```bash
[a.ngom-dsti@edge-1 ~]$ hdfs dfs -put <(curl -sS -L $source) /user/${env:USER}/dataset
20/06/25 16:15:52 INFO hdfs.DataStreamer: Exception in createBlockOutputStream blk_1101247789_27508043
java.io.EOFException: Unexpected EOF while trying to read response from server
        at org.apache.hadoop.hdfs.protocolPB.PBHelperClient.vintPrefixed(PBHelperClient.java:549)
        at org.apache.hadoop.hdfs.DataStreamer.createBlockOutputStream(DataStreamer.java:1762)
        at org.apache.hadoop.hdfs.DataStreamer.nextBlockOutputStream(DataStreamer.java:1679)
        at org.apache.hadoop.hdfs.DataStreamer.run(DataStreamer.java:716)
20/06/25 16:15:52 WARN hdfs.DataStreamer: Abandoning BP-188581868-10.0.0.17-1551214078993:blk_1101247789_27508043
20/06/25 16:15:52 WARN hdfs.DataStreamer: Excluding datanode DatanodeInfoWithStorage[10.0.0.49:1019,DS-8ff05913-5f16-4fa1-aca4-435303d3055a,DISK]
```

This error is caused by saturated storage of file systems. It can be solved it by reducing space. Additionally, you can check the health of the file system (number of block, missing files, over replicated, corrupted blocks,...) with the following command:

```bash
hdfs fsck /user/${env:USER}/trip_data
```

For more information about your HDFS capacity, this command below may helps too.

```bash
[a.ngom-dsti@edge-1 ~]$ hdfs dfsadmin -report
Configured Capacity: 3809922252800 (3.47 TB)
Present Capacity: 3805824406191 (3.46 TB)
DFS Remaining: 1962398300142 (1.78 TB)
DFS Used: 1843426106049 (1.68 TB)
DFS Used%: 48.44%
Replicated Blocks:
	Under replicated blocks: 4
	Blocks with corrupt replicas: 0
	Missing blocks: 0
	Missing blocks (with replication factor 1): 0
	Low redundancy blocks with highest priority to recover: 0
	Pending deletion blocks: 0
Erasure Coded Block Groups: 
	Low redundancy block groups: 0
	Block groups with corrupt internal blocks: 0
	Missing block groups: 0
	Low redundancy blocks with highest priority to recover: 0
	Pending deletion blocks: 0
```
