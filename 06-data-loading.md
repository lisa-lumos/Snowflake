# 6. Data loading
## Overview
The location of data files in cloud storage called a stage. A named external stage is a database object that lives in a schema. 

Snowflake supports 3 types of internal stages: 
- User: belong to a single user, cannot be dropped. `@~`
- Table: belong to a table, cannot be dropped. `@%table_name`
- Named:  a db object lives inside a schema. Most flexible. `@stage_name`

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
After staged files are loaded, consider removing them (PURGE copy option or the REMOVE command). 

## Preparing to Load Data
If you regularly load similarly-formatted data, recommend named file formats. 

File format priority: COPY INTO TABLE statement > Stage definition > Table definition. 

File format options are not cumulative, while copy options are. 

## Loading Using the Web Interface (Limited)
The Web Interface (classic console) for loading data (combines put and copy command into one operation) is only intended for loading small numbers of files of limited size (up to 50 MB). 

## Bulk loading from Local File System
PUT -> COPY INTO. 

Bulk loads are always performed in a single transaction.
### Choosing an Internal Stage
checked. 
### Staging Files
checked. 
### Copying Data from a Local Filesystem
Before loading data, you can validate that the data in the uploaded files will load correctly by using the VALIDATION_MODE parameter in COPY INTO command. 

The ON_ERROR copy option for the COPY INTO command indicates what action to perform if errors are encountered in a file during loading.

## Bulk loading from Amazon S3
### Allowing the Virtual Private Cloud IDs
Your sf account has a unique VPC id, you can create an Amazon S3 policy allowing this id, then provide an IAM role for sf to access the specific S3 bucket. 
### Configuring Secure Access
You can use storage integrations to allow Snowflake to read/write data from/to an Amazon S3 bucket referenced in a stage. This avoids the need for passing explicit cloud provider credentials like secret keys or access tokens. It stores an AWS IAM user ID. An admin in your org grants the IAM user permissions to the AWS account.

Different stage objs can refer different buckets and paths and use the same storage integration for authentication. SF provisions a single IAM user for your entire sf account. All S3 storage integrations use that IAM user.
### AWS Data File Encryption
Snowflake supports either client-side encryption (Requires a MASTER_KEY of 128/256-bit and base64 encoded) or server-side encryption. 
### Creating an S3 Stage
skipped
### Copying Data from an S3 Stage
skipped
## Bulk loading from Google Cloud Storage
skipped
## Bulk loading from Microsoft Azure
skipped
## Bulk loading Troubleshooting
Data load failures:
- view the copy history for the table
- use validation_mode in COPY INTO statement: `VALIDATION_MODE=RETURN_ALL_ERRORS`

CURRENT_TIMESTAMP is evaluated when the load statement is compiled, rather than when the record is inserted into the table. 
## Snowpipe 
2 mechanisms for detecting the staged files:
- using cloud messaging
- calling Snowpipe REST endpoints
## Snowpipe Overview
Cross-cloud support is currently only available to accounts hosted on AWS.
## Snowpipe Auto Ingest
Event notifications received while a pipe is paused are retained for only a limited period of time (14 days). 
### Automating for Amazon S3
skipped
### Automating for Google Cloud Storage
skipped
### Automating for Microsoft Azure
skipped
## Snowpipe REST Endpoints
Calling the public REST endpoints to load data. Calls to the public Snowpipe REST endpoints use key-based authentication, not the typical username/password authentication, because the ingestion service does not maintain client sessions.

Your client app / aws Lambda function calls a public REST endpoint with a list of data filenames and a referenced pipe name. If new data files matching the list exists, they are queued for loading. 

### Preparing to Load Data
The Snowpipe service requires the Java/Python(3.6+) SDK. 

The Snowpipe REST endpoints require key pair authentication with JSON Web Token (JWT). 
### Load Data Using REST API
skipped
### Load Data Using AWS Lambda
skipped
### Snowpipe REST API
skipped
## Snowpipe Error Notifications
Snowpipe can push error notifications to a cloud messaging service when it encounters errors while loading data. 

Snowpipe error notifications only work when the ON_ERROR copy option is set to SKIP_FILE (the default). Use the NOTIFICATION_HISTORY table function to query the history of notifications sent through Snowpipe. 

### Enabling for Amazon SNS
skipped
### Enabling for Microsoft Azure Event Grid
skipped
### Enabling for Google Pub/Sub
skipped
## Snowpipe Troubleshooting
1. check the pipe status
2. view the copy history of the table
3. validate the data files

Snowflake uses file loading metadata (which lives in the pipes) to prevent reloading the same files in one table. Snowpipe prevents loading files with the same name even if they were later modified. Staged files with the same name as files that were already loaded are ignored, even if they have been modified, e.g. if new rows were added or errors in the file were corrected. If the files are modified and staged again after 14 days, Snowpipe loads the data again, potentially resulting in duplicate records in the target table.

## Snowpipe Managing
Pipe does not support the PURGE copy option. To remove staged files that you no longer need, recommend periodically executing the REMOVE command to delete the files. Alternatively, configure any lifecycle management features provided by your cloud storage service provider.

ALTER PIPE … REFRESH statement checks the load history for both the target table and the pipe to ensure the same files are not loaded twice.

## Snowpipe Costs
Query either of the following:
- PIPE_USAGE_HISTORY table function (in the Snowflake Information Schema).
- PIPE_USAGE_HISTORY View

## Snowpipe Streaming Overview (preview)
Calling the Snowpipe Streaming API prompts low-latency loads of streaming data rows using the Snowflake Ingest SDK and your application code. The API writes rows of data to Snowflake tables directly, without staged files in the middle. This architecture results in lower load latencies and lower costs, which makes it a powerful tool for handling real-time data streams.

## Snowpipe Streaming Configuring

## Snowpipe Streaming Recommendations

## Snowpipe Streaming Costs

## Snowpipe Streaming Kafka Connector with Snowpipe Streaming

## Amazon S3-compatible Storage

## Querying Data in Staged Files

## Querying Metadata for Staged Files

## Transforming Data During Load

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








