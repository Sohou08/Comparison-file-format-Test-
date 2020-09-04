
# COMPRESSION DATA #

Compressing big data help by reducing the amount of storage and bandwidth required for data sets, remove irrelevant or redundant data, making analysis and processing easier and faster. Although compression adds CPU load, for most cases this is more than offset by the savings in I/O. In case of distributed environment, not all compression formats supported are not splittable. For example in MapReduce framework splits data for input to multiple tasks, having a nonsplittable compression format is an impediment to efficient processing. If files cannot be split, that means the entire file needs to be passed to a single MapReduce task, eliminating the advantages of parallelism and data locality that Hadoop provides. For this reason, splittability is a major consideration in choosing a compression format as well as file format. We’ll discuss the various compression formats available for Hadoop, and some considerations in choosing between them.

Let's describe some types of compressions.

## 1. Gzip (short for GNU zip)

 Gzip generates compressed files that have a .gz extension. Gunzip command can be used to decompress files that were created by a number of compression utilities, including Gzip.
   * __Advantage__
      * provides very good compression performance (on average, about 2.5 times the compression that’d be offered by Snappy)
      * being not splittable it should be suitable as a container format
      * bring a better performance using a small block
   * __Advantage__
      * speed performance is not as good as Snappy’s (on average, it’s about half of Snappy’s). It slower than snappy because Gzip compressed files take up fewer blocks, so fewer tasks are required for processing the same data
      * not splittable
      
## 2. Bzip2

From a usability standpoint, Bzip2 and Gzip are similar. 
* __Advantage__
   * provides excellent compression performance
   * It generates a better compression ratio than does Gzip. It will normally compress around 9% better than GZip, in terms of storage space
   * splittable
* __Drawback__
   * all the available compression codecs in Hadoop, Bzip2 is by far the slowest.
   * can be significantly slower than other compression codecs such as Snappy in terms of processing performance.
   
## 3. Snappy

The Snappy codec from Google provides modest compression ratios, but a fast compression and decompression speeds. (In fact, it has the fastest decompression speeds, which makes it highly desirable for data sets that are likely to be queried often.)

* __Advantage__
   * default compression parquet in Impala
   * combination of fast compression and decompression 
* __Drawback__
   * 
  
## 4. LZO (Lempel-Ziv-Oberhumer)
LZO is a block compression algorithm - it compresses and decompresses a block of data. The size of the blocks must be the same when compressing and decompressing.
Being splittable, LZO needs to create an index when it compresses a file, because with variable-length compression blocks, an index is required to tell the mapper where it can safely split the compressed file. LZO is mainly really desirable if you need to compress text files.
* __Advantage__
   * simple and very fast decompression, requiring no more memory than the input and output buffers
   * fast compression, comparable in speed to the Deflate algorithm
   * provides a modest compression ratios
   * is splittable
* __Drawback__
   * 

## 4. Zstd (Zstandard)
Zstd is a real-time compression algorithm offering a tradeoff between speed and ratio of compression. 
* __Advantage__
   * 
   * 
   * 
   * 
* __Drawback__
   * 
