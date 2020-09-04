
## NYC taxi Datasets

### Fare data

```sql
DROP TABLE IF EXISTS fare_data;
CREATE EXTERNAL TABLE fare_data
(
medallion Numeric,
hack_license numeric, vendor_id String,
pickup_datetime string, payment_type String,
fare_amount numeric, surcharge numeric,  mta_tax Numeric,
tip_amount numeric, tolls_amount numeric, total_amount numeric
)
ROW FORMAT DELIMITED
FIELDS TERMINATED BY ','
STORED AS TEXTFILE
Location '/user/${env:USER}/taxi_fare/'
```

## Imdb datasets

### Imdb title basics

```sql
Drop table if exists imdb_title_basics;
create external table imdb_title_basics
(tconst string , titletype string , primarytitle  string, 
originaltitle string, isadult string ,  startyear INT , 
endyear INT ,  runtimeminutes INT , genres array<string> )
row format
delimited fields terminated BY '\t'
STORED AS TEXTFILE
location '/user/${env:USER}/imdb_title_basics/'
tblproperties("skip.header.line.count"="1");
```

### Imdb title_principals

```sql
Drop table if exists imdb_title_principals;
create external table imdb_title_principals
(tconst string, ordering numeric, nconst string, 
category string, job string , characters string)
row format
delimited fields terminated BY '\t'
STORED AS TEXTFILE
location '/user/${env:USER}/imdb_title_principals/'
tblproperties("skip.header.line.count"="1");
```

### Imdb title ratings

```sql
Drop table if exists imdb_title_ratings;
create external table imdb_title_ratings
(tconst string , averagerating double, numvotes int )
row format
delimited fields terminated BY '\t'
STORED AS TEXTFILE
location '/user/${env:USER}/imdb_title_ratings/';
tblproperties("skip.header.line.count"="1");
```

### Imdb title crew

```sql
Drop table if exists imdb_title_crew_csv;
create external table imdb_title_crew_csv
(tconst string , director array<string> , writers array<string> )
row format
delimited fields terminated BY '\t'
STORED AS TEXTFILE
location '/user/${env:USER}/imdb_title_crew/'
tblproperties("skip.header.line.count"="1");
```

### Imdb title akas

```sql
Drop table if exists imdb_title_akas;
create external table imdb_title_akas
(
titleId string , ordering numeric, title string, region string, 
language string, types  string, attributes string, isOriginalTitle numeric 
)
row format
delimited fields terminated BY '\t'
STORED AS TEXTFILE
location '/user/${env:USER}/imdb_title_akas/'
tblproperties("skip.header.line.count"="1");
```

### Imdb title episode

```sql
Drop table if exists imdb_title_episode;
create external table imdb_title_episode
(tconst string, parentTconst string, seasonNumber numeric,  episodeNumber numeric )
row format
delimited fields terminated BY '\t'
STORED AS TEXTFILE
location '/user/${env:USER}/imdb_title_episode/'
tblproperties("skip.header.line.count"="1");
```

### Imdb name basics

```sql
Drop table if exists imdb_name_basics;
create external table imdb_name_basics
(nconst string , primaryname  string, birthyear int
 , deathyear int, primaryprofession array<string>, knownfortitles  array<string> )
row format
delimited fields terminated BY '\t'
STORED AS TEXTFILE
location '/user/${env:USER}/imdb_name_basics/'
tblproperties("skip.header.line.count"="1");
```

## Wikipedia Datasets

```sql
Drop table if exists wikipedia_json;
Create external table wikipedia_json
(
id  STRING,
type STRING,
aliases map<string ,array<struct<language:STRING , value:STRING>>>,              
descriptions map<string, struct<language:STRING, value:STRING >>,             
labels map<string, struct<language:STRING, value:STRING>>,        
claims  map<string,array<struct<id:string,mainsnak:struct<snaktype:string,property:string,datatype:string,datavalue:struct<value:string,type:string>>,type:string,rank:string>>>, 

sitelinks map<string, struct<site:STRING, title:STRING, badges:STRING>>
)        
ROW FORMAT SERDE 'org.openx.data.jsonserde.JsonSerDe'
WITH SERDEPROPERTIES ( "ignore.malformed.json" = "true" , "skip.header.line.count"="1")
STORED AS TEXTFILE
location '/user/${env:USER}/wikipedia/';
```
