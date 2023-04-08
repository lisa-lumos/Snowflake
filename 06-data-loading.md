# 6. Data loading
## Overview
The location of data files in cloud storage called a stage. A named external stage is a database object that lives in a schema. 

Snowflake supports 3 types of internal stages: 
- User: belong to a single user, cannot be dropped
- Table: belong to a table, cannot be dropped
- Named:  a db object lives inside a schema. Most flexible. 

You can do bulk loading and continuous loading. 

Bulk loading uses user-provided virtual warehouses, specified in the COPY statement. 

Data transformations supported in COPY INTO command:
- Column reordering & omission
- Casts
- Truncating text strings that exceed the target column length

Continuous loading using snowpipe:
- used to load small volumes of data incrementally
- loads data within minutes
- uses serverless compute resources, so automatically resized & scaled
- can be used in data pipelines with streams and tasks, for complex transformations

The Snowflake Connector for Kafka let users to connect to an Apache Kafka server, read data from topics, and load data into Snowflake tables.

For semi-structured data (Apache Parquet, Apache Avro, and ORC files), snowflake supports auto detection of schema, which retrieves the column definitions (names, data types, and ordering of columns in the files). Use function INFER_SCHEMA and GENERATE_COLUMN_DESCRIPTION. Create tables with the column definitions derived from a set of staged files using the CREATE TABLE ... USING TEMPLATE. 

Instead of loading data into sf table, you can query then directly in the stage also. Useful if you only want to query a subset of data. 

You can create external stages/tables on software and devices, on premises or in a private cloud, that is highly compliant with Amazon S3 API (Amazon S3-compatible Storage). 

## Feature Summary
For files to load to SF, the default encoding character set is UTF-8. 

gzip is the default compression format for uncompressed files to be loaded to SF stage. For already compressed files, gzip, bzip2, deflate, raw_deflate are supported, and can be auto-detected. Auto-detection is not yet supported for Brotli, you need to specify it during staging and loading. 

The files sits in the SF stage are encrypted using 128-bit keys by default, if the files came unencrypted at the beginning. If they are already encrypted, you will need the encryption key. 

## Considerations
### Preparing Your Data Files
The number of data files that are processed in parallel is determined by the amount of compute resources in a warehouse.

File sizing (for both bulk and pipe loading): 
- Data files >= 100-250 MB compressed. 
- Do not load large files (>= 100GB).
- Can use split command in Linux to partition large files

Semi-structured:
- Supported semi-structured data formats: JSON, Avro, ORC, Parquet, XML
- The VARIANT data type imposes a 16 MB size limit on individual rows. 
- Use the STRIP_OUTER_ARRAY file format option for the COPY INTO command to load the records into separate table rows. 
- Extract semi-structured data elements containing “null” values into relational columns before loading them; or, set the file format option STRIP_NULL_VALUES to TRUE when loading it.


Snowpipe:
- Snowpipe is designed to load new data within a minute after a file notification is sent. 
- If it takes >= 1min to accumulate MBs of data in your source app, recommend to create 1 file/min, using Amazon Kinesis Firehose etc to batch data files.

csv files:
- UTF-8 encoding is the default, but can specify other types
- field vals that contain delimiter should be single/double-quoted
- if data contains single/double quotes, they need to be escaped
- carriage returns should be single/double-quoted
- num of cols in each row should be same

### Planning a Data Load
Dedicate separate warehouses for loading and querying (to optimize performance for each). 

Unless bulk loading >= 100s or 1000s of files concurrently, a smaller warehouse (Small, Medium, Large) is generally sufficient.

### Staging Data
You can include paths in a stage definition; you can also use put command to put the file into a certain folder (path) in the stage. 

Partition the data into logical paths such as geographical location with date. 

### Loading Data
COPY INTO command supports you to choose files by:
- path/prefix
- specifying a list of file names
- use regex pattern matching

Snowflake maintains data loading metadata for each table, this metadata lives inside the table, including:
- Name & size & ETag of the files loaded into this table
- Num of rows parsed in the file
- Last load timestamp for each file
- Loading error info for the file

This metadata is used to prevent data duplication during loading for the target table. It expires after 64 days. For files with unknown status, they are skipped by default. 

When loading staged data, narrow the path to the most granular level, such as using pattern options or use common prefix of the files. 

### Managing Regular Data Loads
After staged files are loaded, consider removing them (PURGE copy option or the remove command). 

## Preparing to Load Data
If you regularly load similarly-formatted data, recommend named file formats. 

File format priority: COPY INTO TABLE statement > Stage definition > Table definition. 

File format options are not cumulative, while copy options are. 

## Loading Using the Web Interface (Limited)
The Web Interface (classic console) for loading data (combines put and copy command into one operation) is only intended for loading small numbers of files of limited size (up to 50 MB). 

## Bulk loading from Local File System
PUT -> COPY INTO. 



Choosing an Internal Stage
Staging Files
Copying Data from a Local Filesystem
## Bulk loading from Amazon S3
Allowing the Virtual Private Cloud IDs
Configuring Secure Access
AWS Data File Encryption
Creating an S3 Stage
Copying Data from an S3 Stage
## Bulk loading from Google Cloud Storage
## Bulk loading from Microsoft Azure
## Bulk loading Troubleshooting
## Snowpipe 
## Snowpipe Overview
## Snowpipe Automation
Automating for Amazon S3
Automating for Google Cloud Storage
Automating for Microsoft Azure
## Snowpipe REST Endpoints
Preparing to Load Data
Load Data Using REST API
Load Data Using AWS Lambda
Snowpipe REST API
## Snowpipe Error Notifications
Enabling for Amazon SNS
Enabling for Microsoft Azure Event Grid
Enabling for Google Pub/Sub
## Snowpipe Troubleshooting
## Snowpipe Managing
## Snowpipe Costs
 
## Snowpipe Streaming Overview
## Snowpipe Streaming Configuring
## Snowpipe Streaming Recommendations
## Snowpipe Streaming Costs
## Snowpipe Streaming Kafka Connector with Snowpipe Streaming
## Amazon S3-compatible Storage
## Querying Data in Staged Files
## Querying Metadata for Staged Files
## Transforming Data During Load
## 
## Continuous data pipelines Overview
## Continuous data pipelines Streams
Managing
Examples
## Continuous data pipelines Tasks
Enabling Error Notifications
Enabling Error Notifications using AWS SNS
Enabling Error Notifications using MS Azure Grid Events
Enabling Error Notifications for Tasks using Google Pub/Sub
Integrating Task Error Notifications with Tasks
Task Error Payload Example
Troubleshooting
## Continuous data pipelines Examples





Bulk loads are always performed in a single transaction.


