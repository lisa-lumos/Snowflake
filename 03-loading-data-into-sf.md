# 3. Loading data into SF

Snowflake supports following data transformations while loading it into a table:
- Column reordering
- Column omission
- Casts
- Truncating text strings that exceed the target column length

For semi-structured data (Apache Parquet, Apache Avro, and ORC files), snowflake supports auto detection of schema. 

For files to load to SF, the default encoding character set is UTF-8. gzip is the default compression format for uncompressed files to be loaded to SF stage. Compression format can be auto detected for certain formats, if the files are  compressed from the beginning. The files sits in the SF stage encrypted using 128-bit keys by default, if the files came unencrypted at the beginning. 

You can include paths in a stage definition; you can also use put command to put the file into a certain folder (path) in the stage. 

File format priority: COPY INTO TABLE statement > Stage definition > Table definition. File format options are not cumulative, while copy options are. 

## Best practices for data loading
- Data files >= 100-250 MB compressed. 
- Do not load large files (>= 100GB).
- Dedicate separate warehouses for loading and querying (to optimize performance for each). 
- Unless bulk loading >= 100s or 1000s of files concurrently, a smaller warehouse (Small, Medium, Large) is generally sufficient.
- Partition the data into logical paths that with identifiers such as geographical location and date.
- When loading staged data, narrow the path to the most granular level, such as using pattern options or use common prefix of the files
- After staged files are loaded, consider removing them
- If you regularly load similarly-formatted data, recommend named file formats. 

Semi-structured:
- Supported semi-structured data formats: JSON, Avro, ORC, Parquet, XML
- The VARIANT data type imposes a 16 MB size limit on individual rows. 
- Use the STRIP_OUTER_ARRAY file format option for the COPY INTO command to load the records into separate table rows. 
- Extract semi-structured data elements containing “null” values into relational columns before loading them; or, set the file format option STRIP_NULL_VALUES to TRUE when loading it.
- Usually, tables that store semi-structured data consist of a single VARIANT column. Semi-structured data can be loaded into tables with multiple columns only if it is stored as a field in a structured file, then it is loaded into that field.

Snowpipe:
- Snowpipe is designed to load new data within a minute after a file notification is sent. 
- If it takes >= 1min to accumulate MBs of data in your source app, recommend to create 1 file/min.


































