
# BENCHMARK STUDY OF DIFFERENT FILE FORMATS: CSV, JSON, XML, PROTOCOL BUFFER, AVRO, PARQUET, ORC #

In data processing, there are different types of files formats to store your data sets. Each format has its own pros and cons depending upon the use cases and exists to serve a purpose. It is important to know and leverage their specificities when choosing a specific format type. 

Some formats are more relevant for certain usages or treatments, for example in Business Intelligence, network communication, web application, batch or stream processing. For instance, CSV format is very understandably and, while everyone critics its lack of formalism, it is still widely used. In case of web communication, JSON is privileged and cover most of the world communication even if, in some cases, XML might do the same work. The latest format benefit from a large ecosystem of tools and is widely used in the enterprise world. In terms of streaming processing, AVRO and Protocol buffer are a privileged formats. In addition, protocol buffer is the stand of gRPC (Remote procedure call) on which rely the Kubernetes ecosystem. Finally, column storage offered by ORC and Parquet provide a significant performance boost in data analytics.

## [**Consideration to be taken into account for choosing a file format**]()

The most popular and representative file formats are described with the various considerations to keep in mind when choosing one format over another one. But first, let's review their main characteristics in regards to storage, queries types, batch versus streaming usages, ...

### Row and column storage, OLTP versus OLAP

In row based storage, data is stored row by row, such that the first column of row will be next to the last column of the previous row. This architecture is appropriated to add data easily and quickly. It's also suitable in case where the entire row of data needs to be accessed or processed simultaneously. This is commonly used for Online Transactional Processing (OLTP). OLTP systems usually process CRUD queries (Read, Insert, Update and Delete) at a record level. OLTP transactions are usually very specific in the task they perform, and usually involve a single record or a small selection of records. The main emphasis for OLTP systems is focus on maintaining data integrity in multi-access environments and an effectiveness measured by number of transactions per second. 

Inversely, in column-based storage, data is stored such that each row of a column will be next to other rows from that same column. That is most useful when performing analytics queries which require only a subset of columns examined over very large data sets. This way of processing data is usually called OLAP (OnLine Analytical Processing) query. OLAP is an approach designed to quickly answer analytics queries involving multiple dimensions. This approach has played a critical role in business intelligence analytics, especially regarding to Big Data. Storing data in column avoid the excessive processing times of reading irrelevant information across a data set by ignoring all the data that doesn't apply to a particular query. It also provides a greater compression ratio by aligning data of a same type and optimizing `null` values of sparse columns.

Here is an example if you wanted to know the average population density in cities with more than a million people. With a row-oriented storage and, your query would access each record in the table (meaning all of its fields) to get the necessary information from the three columns `city name`, `density` and `population`. That would involve a lot of unnecessary disk seeks and disk reads, which impact performance. 
Columnar databases performs well the process by reducing the amount of data that needs to be read from disk.

[Example of row and column based storage]()
![2](https://user-images.githubusercontent.com/51121757/81609463-db00d680-93cf-11ea-9b11-786b494e0bd7.JPG)

*  __Splittable__

In a distributed file system like [Hadoop HDFS](https://hadoop.apache.org/docs/current/hadoop-project-dist/hadoop-hdfs/HdfsDesign.html), it is important to have a file that can be divided into several pieces. In Hadoop, the HDFS file system stores data in chunk and the data processing is initially distributed according to those chunks. To be able to begin reading data at any point within a file is helpful in order to take the full advantages of Hadoop’s distributed processing. When the data set is not splittable, it is the user responsibility to partitioned his data into several files to enable parallelism.

* __Compression (Bzip2, LZO, Sappy,...)__

A system is a slow as its slowest components and, most of the time, the slowest components are the disks. Using compression reduce the size of the data set being stored and thereby reduce the amount of read IO to perform. It also speeds up file transfers over the network. Column based storage brings higher compression ration and better performance compared to row based storage because similar pieces of data are stored together. As mention earlier, it is particularly effective on sparse columns which contains several `null` values.

In addition, compression mostly apply to text file formats unless the binary file format include compression in its definition. Attempting to compress a binary file often lead to heavier generated file compare to its original. Also, when comparing compression ratio, we must also put into perspective the original file size. It wouldn't be fare to compare the compression ratio of JSON versus XML if we don't take into consideration that the XML is originally larger than its JSON counterpart.

There are many kind of compression algorithms which differ according of the ratio of compression, speed, performance, splittable, ...

* __Batch and Streaming__

Batch processing read, analyses and transform multiple records at once. In opposition, stream processing works in real time and process records as they arrive. Sometimes, a same data set may be processed both in streaming and in batch. Let's consider financial transactions received inside a system. The data arrive in Streaming and could be processed instantly to detect fraud detection. Complementary, once the data is store, a job could run periodically to prepare some reports. In such case, we might rely on one or two file formats between the format of the data arriving over the wire and the format of the data being stored in our Data Warehouse or Data Lake.

* __Support Schema evolution, allowing us to change schema of file__

The schema will store the definition of each attribute and its types. Unless your data is guaranteed to never change (e.g. of archives), you’ll need to think about schema evolution in order to figure out if your data schema changes over time. In other word, schema evolution allows to update the schema used to write new data while maintaining backwards compatibility with the schema of your old data.

## [**File format description**]()

### 1. CSV 

CSV is a text format which use a newline as a record delimiter with an optional header. It is easy to edit manually and very understandably by human perspective. CSV doesn't support schema evolution even though sometimes the header is optionally considered as the schema of the data. It's not splittable because it doesn't contain a specific character where you could based on to split the text while remaining relevant. 
Generally in big data, CSV seems to be used for processing but strictly speaking this is [TSV](http://jkorpela.fi/TSV.html) format. Indeed, CSV format is not a fully standardized format. Its field delimiters could be for example, colon or semicolons. It become difficult to parse when the field itself may also contain these kind of quotes or even embedded line breaks. In addition, the field data could be enclosed by a quotation marks. These quotes raise sometimes irrelevant structure of data about the identification of the beginning or ending of sentences or even a null values. Unlike TSV, CSV embodies an escape character (`\`) into its data. Thus, all those various considerations make it complex for parsing. Whereas, TSV being generally delimited either by tabulation or comma, it doesn't support an escape character. This allows to distinguish easily each field and eventually help to parse easily than CSV. This is the reason why it is mainly used in big data case.
However, sometime it could be possible to write your own CSV code through [Ruby](https://www.rubyguides.com/2018/10/parse-csv-ruby/) even if it's not a well defined file-format.

   * __Advantage__
       * CSV is compressible 
       * CSV supports batch and streaming processing      
   * __Drawback__
       * It doesn't support a null value 
       * Parsed a CSV format requires a program in order to handle the escape character
       * CSV is not splittable and doesn't support schema evolution
       * It's not considered as a standard format because everyone has its own interpretations
   * __Ecosystems__
       * It is supported by a wide range of applications (Hadoop, [Spark](http://spark.apache.org/), [kafka](http://kafka.apache.org/),...)
       * Being generally less recommended (because it's not a standard format), CSV is one of the most popular formats used
       
~~~{r}
Player Name;Position;Nicknames;Years Active
Skippy Peterson;First Base;"""Blue Dog"", ""The Magician""";1908-1913
Bud Grimsby;Center Field;"""The Reaper"", ""Longneck""";1910-1917
Vic Crumb;Shortstop;"""Fat Vic"", ""Icy Hot""";1911-1912
~~~
[source](https://www.computerhope.com/issues/ch001356.htm)

### 2. JSON (JavaScript object notation) 

JSON is a text format containing a record which might be in any form (string, integer, booleans, array, object, key-value, nested data...). Being human and machine readable, JSON is relatively lightweight format and privileged in web application. It widely supported by many software and comparatively simple to implement in multiples languages. JSON natively supports serialization and deserialization process. 
Its major inconvenience is related to its lack of splittable. The alternative used format in this case is generally a JSON lines. This one contains several lines where each individual line is a valid JSON object, separated optionally by newline character `\n`. Its main difference compare to the standard format is related to an aesthetic advantage since JSON line separate objects by a line break. This structure make you sure that the next line is a new objects so it will be easy to be used as reference for splitting data.

* __Advantage__
    * JSON is a simple syntax. It can be opened by any text editor
    * There are many tools available to validate the schema 
    * JSON is compressible and used as data exchange format
    * It supports batch and streaming processing
    * It stores metadata with data and supports schema evolution
    * JSON is lightweight text-based format in comparison to XML 
* __Drawback__
    * JSON is not splittable and lack indexing as many text formats
* __Ecosystems__
    * It is privileged format for web applications
    * JSON is a widely used file format for NoSQL databases such as MongoDB, Couchbase and Azure Cosmos DB. For example,in [MongoDB](https://docs.mongodb.com/), BSON is used instead the standard format. This architecture of MongoDB is easier to modify than structured database as [Postgres](https://www.postgresql.org/docs/12/index.html) where you'd need to extract the whole document for changing. Indeed, BSON is the binary encoding of JSON-like documents. It's indexed and allows to be parsed much more quickly than standard JSON. However, this doesn’t mean you can’t think of MongoDB as a JSON database. 
    * JSON is also an used language in [GraphQL](https://graphql.org/) on which rely a data fully typed. GraphQL is a query language for an API. Instead of working with rigid server-defined endpoints, you can send queries to get exactly the data you are looking for in one request.
          
[Example of JSON](https://docs.microsoft.com/bs-latn-ba/azure/hdinsight/hadoop/using-json-in-hive)
~~~ {r}
      {
              "fruits": [
                  { "kiwis": 3,
                    "mangues": 4,
                    "pommes": null
                  },
                 { "panier": true }
              ],
              "legumes": {
                  "patates": "amandine",
                  "poireaux": false
               },
               "viandes": ["poisson","poulet","boeuf"]
          }
~~~ 
[Example of JSON lines](https://hackernoon.com/json-lines-format-76353b4e588d)
~~~{r}
{"id":1,"father":"Mark","mother":"Charlotte","children":["Tom"]}
{"id":2,"father":"John","mother":"Ann","children":["Jessika","Antony","Jack"]}
{"id":3,"father":"Bob","mother":"Monika","children":["Jerry","Karol"]}
~~~

### 3. XML 

XML stands for Extensible Markup Language and is a markup language used to describe data. XML is a [W3C](https://www.w3.org/) recommendation. This means it is an industry standard governed by a vendor-independent body. It offers a standardized way to represent textual data and easily to exchange it between application. Often the XML data is also referred to as an XML document. The XML data doesn’t perform anything on its own; to process that data, you need to use a piece of software called a parser. XML documents are self-describing and are both humans and machine readable. XML data is well structured and hierarchy. Like JSON, XML is also used to serialize and encapsulate data. Its syntax is verbose, especially for human readers, compare to other text-based formats.

* __Advantage__
    * XML supports batch/streaming processing.
    * Being a standard format, it's suitable for exchanging or sharing data
    * It stores meta data with data and supports the schema evolution.
    * Being a verbose format, It provides a good ratio of compression compare to JSON file or other text file format.
* __Drawback__
    * It is not splittable since XML has an opening tag at the beginning and a closing tag at the end. You cannot start processing at any point in the middle of those tags.
    * XML doesn't support an array type
    * The redundancy of its syntax causes higher storage and transportation cost when the volume of data is large.
    * The support of namespace can be difficult to correctly implement in an XML parser.
    * XML is difficult to be parsed due to its escape character which might be in different form (" " ' ' < < > > & &). The alternative used case is often HTML which only has a simple character (< < > >). In addition the `< ` quote is interpreted as an opening HTML.
* __Ecosystems__
    * XML is practically anymore used in any enterprise because it brings many disadvantage in term of storage or exchange data.
    
~~~ {r}
<?xml version="1.0" encoding="UTF-8"?>
<scientists>
  <!-- Some important physicists -->
  <scientists id="phys0">
    <first name="Galileo" name="Galilei"/>
    <dates of birth="1564-02-15" death="1642-01-08"/>
    <contributions>Relativity of movement &amp; heliocentric system</contributions>
  </scientists>
~~~ 
    
[source](https://www.alsacreations.com/article/lire/609-XML-en-quelques-mots.html)
       
### 4. AVRO

Avro is framework developed within Apache's Hadoop project. It is a row-based storage format which is widely used as a serialization process. AVRO stores its schema in JSON format making it easy to read and interpret by any program. The data itself is stored in binary format by doing it compact and efficient. A key feature of AVRO is related to the fact it handle easily schema evolution. It attach metadata into their data in each record. 

* __Advantage__
    * Avro is a data serialization system
    * It is splittable (AVRO has a sync marker to separate the block) and compressible
    * Avro is good file format for data exchange. It has a data storage which is very compact, fast and efficient for analytics
    * It highly supports schema evolution (at different time and independently)
    * It supports batch and mainly a streaming engine
    * Avro schemas defined in JSON facilitate implementation in the software that already have JSON libraries.
    * It doesn't need code generation. The data is always accompanied by schema, which allow full processing on the data.
* __Drawback__
    * Its data is not readable by human
    * Avro has a limited environment since all languages don't occur a encoded or decoded process at the same time.
* __Ecosystems__
    * It widely used in many application (Kafka, Spark, ... )
    * Avro is a remote procedure call ([RPC](https://www.tutorialspoint.com/remote-procedure-call-rpc))
    
    
   ![e.g. AVRO file](https://user-images.githubusercontent.com/51121757/80033375-8232d200-84e4-11ea-9531-076f72e30bea.JPG)

[Source](https://blog.clairvoyantsoft.com/big-data-file-formats-3fb659903271)
    
### 5. Protocol Buffer

Protocol buffers has been developed by Google and open sourced in 2008. It is language-neutral, an extensible way of serializing structured data. It's used in communications protocols, data storage, and more. It is easy to read and understandably by human. The schema is required for generating code and read the data. 

* __Advantage__
    * Its data is fully typed
    * Protocol buffer supports schema evolution and batch/streaming processing
    * It is mainly used to serialize data as AVRO
    * Having a predefined and larger set of data types, messages serialized on Protobuf can be automatically validated by the code that is responsible to exchange them.
* __Drawback__
    * Protocole buffer is not splittable and not compressible
    * It doesn't support Map reduce
    * It needed a reference file to be decoded 
    * Protocol Buffers are not designed to handle large messages. Since it doesn't support random access, you'll have to read the whole file, even if you only want to access a specific item.
* __Ecosystems__
    * There are multiples languages which can implemented a protocol buffer code: JavaScript, Java, PHP, C#, Ruby, Python, C++,...
    * Protobuf is the format support of gRPC in [kubernetes](https://kubernetes.io/docs/home/). 
    * ProfaneDB is a database for Protocol Buffer objects.
  
~~~{r}
     message Person {required string name = 1; required int32 id = 2; optional string email = 3;}                          
~~~

### 6. Parquet

Parquet is an open source file format for Hadoop ecosystem. Its design is a combined effort between Twitter and [Cloudera](https://docs.cloudera.com/) for an efficient data storage of analytics. Parquet stores data by column-oriented like ORC format. It provides efficient data compression and encoding schemes with enhanced performance to handle complex data in bulk. Parquet is a binary file containing metadata about their content. The column metadata is stored at the end of the file, which allows for fast, one-pass writing. Parquet is optimized for the paradigm of write Once read many (WORM).  

* __Advantage__
     * Parquet is splittable 
     * Organizing by column, It allows a better compression, as data is more homogeneous.
     * It is very efficient for OLAP query.
     * It supports schema evolution and only support batch processing
 * __Drawback__
     * Parquet is not human readable
     * It difficults to make it update unless you delete and recreate it again.
 *__Ecosystems__
     * It gives an efficient analysis in BI (Business Intelligence)
     * Parquet is very fast to read in Spark environment
     * Aside Spark and Impala; Arrow, Drill are part of the most compatibles platforms of parquet.
     
   ![parquet](https://user-images.githubusercontent.com/51121757/80372035-c3333980-888a-11ea-87af-97425e00c476.JPG)

[Source](https://blog.ippon.fr/2020/03/02/de-limportance-du-format-de-la-donnee-pratique-partie-2-2/) 

### 7. ORC (Optimized Row Columnar)

In February 2013, ORC format was announced by [Hortonworks](https://www.cloudera.com/downloads/hdp.html) in collaboration with Facebook. It is a column-oriented data storage format from the Apache Hadoop ecosystem.
ORC file contains groups of row data called stripes, along with auxiliary information in a file footer. At the end of the file a postscript holds compression parameters and the size of the compressed footer. The default stripe size is 250 MB. Large stripe sizes enable large efficient reads from HDFS.

  * __Advantage__
      * ORC is compressible
      * It reduces the size of the original data up to 75%
      * ORC is more efficient for OLAP than OLTP queries
      * It supports batch and streaming processing 
  * __Drawback__
      * ORC can’t be load data directly into ORCFILE
      * It does not support schema evolution
   * __Ecosystems__
      * It occurs an efficient analysis in case of Business Intelligence
      * ORC improves performance reading, writing, and processing in Hive
      * [Impala](https://docs.cloudera.com/documentation/enterprise/5-5-x/topics/impala.html) currently does not support ORC format
      * ORC is well supported on Hortonworks
      
   ![Screenshot from 2020-05-13 09-28-10](https://user-images.githubusercontent.com/51121757/81783778-244e4480-94fc-11ea-8ede-04d03f3b06eb.png)

[Source](https://cwiki.apache.org/confluence/display/Hive/LanguageManual+ORC) 

## [**Conclusion**]()

In general, reading or writing fast according to the type of storage is very related to the kind of query (use cases) tied to the property of each file format. Usually, row-oriented databases are well-suited for OLTP-like workloads whereas column-oriented systems is well suited of OLAP query. Since columnar format contains a uniform type of data in each column, this will be efficient to compression compare to a row based storage. 

In term of comparison, JSON, XML are less advisable in the processing of parallel job due to their lack of splittable. The alternative used case is generally JSON line which provides a great distinction between object by a line break. XMl being pratically no used in enterprise, JSON take over and is actually considered the privileged format of web applications. It is also well known to support schema evolution like AVRO. Being both splittable and compressible, AVRO is considered the privilege format of streaming processing. According all these advantages from AVRO, it could make it as an appropriated or based exchange formats in big data ecosystem. Even protocol buffer is efficient in streaming processing, it is not splittable and not compressible. However, being a language fully typed, it occurs better performance for compiler optimization. We can considered thus a generic usage pattern of Avro on Hadoop-specific formats and protocol buffers for non-Hadoop projects.

Finally, the last two formats (ORC and parquet) are optimized the cost storage by avoiding repeated data. They are very efficients for Business Intelligence analytics and are both compatibles in Hive ecosystems. Therefore, it is preferable to use Parquet in CDH (cloudera) since this one will have updated the languages optimized for hive related to parquet. Similarly, it is preferable to use ORC on HDP (Hortonworks) distribution.

In summary, the main consideration to bear in mind is that all these differences between theses files format depend really of the use cases. Even though each of them has its owns specificity, they sometime could be used by fitting in order to respond a specific purpose.
  
##  Overview

Types|CSV | JSON | XML| AVRO| Protocol Buffer | Parquet | ORC 
--- | --- | --- | --- | --- |--- |--- |---
[Format (text/binary)]()| text| text| text| text stored in JSON and data is binary format| text| binary|binary|
[Storage type]()| row| row| row|row| row| column|column|
[Splittable]()| depend on data types| :x: not the case of JSON line| :x:|:heavy_check_mark:| :x:| :heavy_check_mark:|:heavy_check_mark:|
[Compression]()| :heavy_check_mark:| :heavy_check_mark:| :heavy_check_mark:|:heavy_check_mark:| :heavy_check_mark:| :heavy_check_mark:|:heavy_check_mark:|
[Ecosystems]()| the most commonly used format| API and development web| practically no used in any environment|kafka,spark,... | Kubernetes,ProfaneDB, doesn't support MapReduce| Impala, HIve, BigQuery, Spark,...|Hive, It's not supported by Impala|
[Schema evolution]()| :x:| :heavy_check_mark:| :heavy_check_mark:|:heavy_check_mark:| :x:| :heavy_check_mark:|:x:|
[Suitable for OLAP or OLTP]()| OLTP| OLTP| OLTP|OLTP| :OLTP:| OLAP|OLAP|
[Batch]()|:heavy_check_mark:| :heavy_check_mark:| :heavy_check_mark:|:heavy_check_mark:| :heavy_check_mark:|:heavy_check_mark:|:heavy_check_mark:|
[Stream]()|:heavy_check_mark:| :heavy_check_mark:| :heavy_check_mark:|:heavy_check_mark: most efficient in this case | :heavy_check_mark: most efficient in this case|:x:|:x:|
[Typed data]()|:x:| :x:| :x:|:x:| :heavy_check_mark:|:x:|:x:|

[Documentation]()
[e.g. File format comparison](https://2s7gjr373w3x22jf92z99mgm5w-wpengine.netdna-ssl.com/wp-content/uploads/2018/05/Nexla-File-Format.png), [e.g. Comparison in terms of reading, space,storage, execution time,...](https://luminousmen.com/post/big-data-file-formats)


[Let's test all these differences practically](https://github.com/Sohou08/Hadoop-Spark/tree/master/file_format/pratice_test)...
