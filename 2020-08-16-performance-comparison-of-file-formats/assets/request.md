
## Browsing the entire dataset

### taxi driver datasets

```sql
Select count (*) 
from trip_data_10_13;
```

Dataset| format type |Duration time | CPU Time
--- | --- | --- | ---
Trip | ORC | 19.169|484.27
Trip |Parquet| 114.853|2489
Trip  | CSV|248.836|5568.79
Trip  | JSON|228.254| 6129.01
Trip |AVRO|225.494|5076.48

### Imdb datasets

```sql
select count(*)
from imdb_title_principals;
```

Dataset| format type |Duration time | CPU Time
--- | --- | --- | ---
Imdb| ORC |8.063 |22.11
Imdb|Parquet|13.68 |239.41
Imdb | CSV|26.239| 588.48
Imdb | JSON|19.862| 499.85
Imdb |AVRO|18.038|479.07

### Wikipedia datasets

```sql
select count(*)
from wikipedia ;
```

Dataset| format type |Duration time | CPU Time
--- | --- | --- | ---
wikipedia| ORC |  10.287 |  63.35
wikipedia|Parquet|30.049  | 645.01
wikipedia| JSON| 969.659 |  1081.94   
wikipedia|AVRO| 54.82 |  1452.15

## Fetch the max values

### Taxi driver datasets

```sql

select Max ( trip_distance)
from trip_data_10_13;
```

Dataset| format type |Duration time | CPU Time
--- | --- | --- | ---
trip | ORC |27.87 |623.41
trip |Parquet|33.496|809.24
trip | CSV|251.336| 5640.22
trip | JSON|224.076|6196.48
trip |AVRO|240.708|5129.01

### Imdb datasets

```sql
select Max (ordering)
from imdb_title_principals;
```

Dataset| format type |Duration time | CPU Time
--- | --- | --- | ---
Imdb| ORC | 1.55|11.64
Imdb|Parquet|11.104|249.62
IMDb| CSV|28.228|642.08
Imdb| JSON|21.723|528.77
Imdb|AVRO|20.752|529.73

### Wikipedia datasets

```sql
select MAX(claims['p1245'].mainsnak.datavalue.`value`)
from wikipedia;
```

Dataset| format type |Duration time | CPU Time
--- | --- | --- | ---
wikipedia| ORC | 18.747  | 239.02
wikipedia|Parquet|20.798  |  514.15
wikipedia| JSON| 1020.102 |  1162.68
wikipedia|AVRO| 51.557 |   1436.68

## Grouping data

### Taxi driver datasets

```sql
select AVG( trip_distance), vendor_id
from trip_data_10_13
Group by vendor_id;
```

Dataset| format type |Duration time | CPU Time
--- | --- | --- | ---
trip | ORC |21.926 |520.44
trip |Parquet|32.277 |903.7
trip | CSV|52.792|7119.06
trip | JSON|266.453|7495.17
trip |AVRO|210.14|5982.36

### Imdb datasets

```sql
SELECT category, COUNT(*)
FROM  imdb_title_principals
Group BY category;
```

Dataset| format type |Duration time | CPU Time
--- | --- | --- | ---
Imdb| ORC |7.585 |  24.8
Imdb |Parquet| 16.182|  349.4
Imdb | CSV|36.363 | 866.91
Imdb | JSON| 31.42| 708.37
Imdb |AVRO|26.521 | 676.56

### Wikipedia datasets

```sql
select count(id),type
from wikipedia
Group by type;
```

Dataset| format type |Duration time | CPU Time
--- | --- | --- | ---
wikipedia| ORC | 8.481 |  112.62
wikipedia|Parquet| 19.416 | 505.97
wikipedia| JSON|  1021.993|  1280.96
wikipedia|AVRO| 61.842 | 1677.71


## JOIN Request 

### Taxi driver datasets

```sql
Select
f.tolls_amount   
From
fare_data_10_13 as f
Join trip_data_10_13 as t
 ON f.medallion  = t.medallion
Where
  f.surcharge = 10.5;
```

Dataset| format type |Duration time | CPU Time
--- | --- | --- | ---
trip | ORC |101.758 |2563.52
trip |Parquet|110.151|2745.96
trip | CSV|689.666|14901.87
trip | JSON|586.174|14937.04
trip |AVRO|532.149|11598.1

### Imdb datasets

```sql
Select
t.titletype 
From
imdb_title_principals as r
Join imbd_title_basics as t
 ON t.tconst = r.tconst
Where
  r.tconst LIKE "tt0000001";
```

Dataset| format type |Duration time | CPU Time
--- | --- | --- | ---
Imdb| ORC |5.274|24.5
Imdb|Parquet|25.414  | 347.67
Imdb | CSV| 39.951| 767.21
Imdb | JSON|39.242|872.17
Imdb |AVRO| 43.481|754.33

## Data lookup 

### Taxi drivers datasets

```sql
select  vendor_id
from trip_data_10_13
Where trip_distance = 49.72;
```

Dataset| format type |Duration time | CPU Time
--- | --- | --- | ---
trip | ORC | 16.847|411.610 
trip |Parquet|28.845| 862.090 
trip | CSV| 185.360 |5732.320  
trip | JSON|197.593|6272.470
trip |AVRO| 169.256|5136.140 

### Imdb datasets

```sql
select category
from imdb_title_principals
Where nconst  LIKE "nm1588970";
```

Dataset| format type |Duration time | CPU Time
--- | --- | --- | ---
Imdb | ORC | NULL|NULL
Imdb|Parquet|  10.632|  267.040 
Imdb| CSV| 22.801|  680.270
Imdb| JSON|19.743 | 570.860
Imdb |AVRO|19.069 |  548.800

### Wikipedia datasets

- Lookup a field based on the primary key

```sql
select type
from wikimedia
where id = "Q19660479";
```

Dataset| format type |Duration time | CPU Time
--- | --- | --- | ---
wikipedia | ORC |5.087 | 80.280
wikipedia |Parquet|10.830  |  308.920 
wikipedia | JSON| 818.087 | 979.310 
wikipedia |AVRO| 44.770 |  1409.740

```sql
select *
from  wikipedia
where id = "Q19660479";
```

Dataset| format type |Duration time | CPU Time
--- | --- | --- | ---
wikipedia | ORC | 92.51 |  134.910
wikipedia |Parquet|  18.999 |  441.300 
wikipedia | JSON| 747.412 |   914.630 
wikipedia |AVRO|  49.344 |    1427.160  

- Lookup fields based on non primary-key

```sql
select *
from wikipedia_orc
where array_contains (claims['p1245'].mainsnak.datavalue.`value`, "7956");
```

Dataset| format type |Duration time | CPU Time
--- | --- | --- | ---
wikipedia | ORC |   40.396 | 398.010  
wikipedia |Parquet| 23.502 |   709.480 
wikipedia | JSON|   827.408| 966.120 
wikipedia |AVRO|48.092  |    1487.540


### Word Count map

```sql
Drop table if exists test;
CREATE TABLE test AS 
with subquery AS (
select  alias_key, d.language as lan, d.value as val from (
select explode(aliases) as (alias_key, detail) 
from wikipedia
) t lateral view explode(detail) detailexploded as d),
subquery2 as (select concat_ws ("," , alias_key,lan, val ) as col
from subquery ) 
select word , count (*) 
from subquery2
lateral view explode(split(col, ',')) lTable as word
GROUP BY word;
```

Dataset| format type |Duration time | CPU Time
--- | --- | --- | ---
wikipedia | ORC | 46.748    | 785.54 
wikipedia |Parquet| 45.536  |1257.35    
wikipedia | JSON| 1052.275  |  3282.1
wikipedia |AVRO|85.905 |  2484.94    
