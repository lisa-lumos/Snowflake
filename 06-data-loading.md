# 6. Data loading
## Overview
The location of data files in cloud storage called a stage. A named external stage is a database object that lives in a schema. 

Snowflake supports 3 types of internal stages: 
- User: belong to a single user, cannot be dropped, do not support file format options because it may need to handle any file format. User need insert privilege on target table. `@~`
- Table: belong to a table, cannot be dropped. Do not support transform while loading. Only table owner can use it. `@%table_name`
- Named:  a db object lives inside a schema. Most flexible. Privileges can be granted/revoked, ownership can be transferred. `@stage_name`

You can do bulk loading and continuous loading from any sort of stages to tables. You can directly load from S3 bucket without using an external stage.  

Bulk loading uses user-provided virtual warehouses, specified in the COPY statement. 

Data transformations supported in COPY INTO command:
- Column reordering & omission & casts
- Truncating text strings that exceed the target column length

Continuous loading using snowpipe:
- used to load small volumes of data incrementally
- loads data within minutes
- uses serverless compute resources, so automatically resized & scaled
- can be used in data pipelines with streams and tasks, for complex transformations

The Snowflake Connector for Kafka can connect to an Apache Kafka server, load data from topics into an internal stage, and load data files into tables (manual or pipe). Example: a Kafka topic has 3 partitions, and each partition loads files into their respective folder in a internal stage, then the REST API for snowpipe is called, and the 3 pipes would load data into one table concurrently, from their respective folder. 

For semi-structured data (Apache Parquet, Apache Avro, and ORC files), snowflake supports auto detection of schema, which retrieves the column definitions (names, data types, and ordering of columns in the files). Use function INFER_SCHEMA and GENERATE_COLUMN_DESCRIPTION. Create tables with the column definitions derived from a set of staged files using the CREATE TABLE ... USING TEMPLATE. 

Instead of loading data into sf table, you can query then directly in the stage also. Useful if you only want to query a subset of data. 

You can create external stages/tables on software and devices, on premises or in a private cloud, that is highly compliant with Amazon S3 API (Amazon S3-compatible Storage). 

Note:
- metadata for pipe and manual bulk load lives in different places, if pipe already loaded the files, and you run copy into command manually on these files, you will get duplicated records in the table. 

## Feature Summary
For files to load to SF, the default encoding character set is UTF-8. 

gzip is the default compression format for uncompressed files to be loaded to SF stage. For already compressed files, `gzip, bzip2, deflate, raw_deflate` are supported, and can be auto-detected. Auto-detection is not yet supported for `Brotli and Zstandard`, you need to specify it during staging and loading. 

The files sits in the SF stage are encrypted using `128-bit keys by default`, if the files came unencrypted at the beginning. If they are already encrypted, you will need the encryption key. 

## Considerations
### Preparing Your Data Files
The number of data files that are processed in parallel is determined by the amount of compute resources in a warehouse.

File sizing (for both bulk and pipe loading): 
- Data files >= 100-250 MB compressed. 
- Do not load large files (>= 100GB).
- Can use split command in Linux to partition large files

Semi-structured:
- Supported semi-structured data formats: JSON, Avro, ORC, Parquet, XML (preview)
- The VARIANT data type imposes a 16 MB size limit on individual rows. 
- Use the STRIP_OUTER_ARRAY file format option for the COPY INTO command to load the records into separate table rows. 
- Extract semi-structured data elements containing "null" values into relational columns before loading them; or, set the file format option STRIP_NULL_VALUES to TRUE when loading it.

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

This metadata is used to prevent data duplication during loading for the target table. It expires after 64 days for bulk loading. For files with unknown status, they are skipped by default. 

When loading staged data, narrow the path to the most granular level, such as using pattern options or use common prefix of the files. 

### Managing Regular Data Loads
After staged files are loaded, consider removing them (PURGE copy option or the REMOVE command). 

## Preparing to Load Data
If you regularly load similarly-formatted data, recommend named file formats. 

File format / copy options priority: COPY INTO statement > Stage definition > Table definition. 

File format options are `not cumulative`, while copy options `are cumulative`. 

COMPRESSION file format option describes how your data files are already compressed in the stage. Set it by 2 ways:
- As a file format option in COPY INTO.
- As a file format option inside a named file format or a stage, which can be referred in COPY INTO.

Copy options determine the error handling behavior, maximum data size etc. 

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

auto_ingest parameter should be set to true. 
### Automating for Amazon S3
skipped
### Automating for Google Cloud Storage
skipped
### Automating for Microsoft Azure
skipped
## Snowpipe REST Endpoints
Calling the public REST endpoints to load data. Calls to the public Snowpipe REST endpoints use key-based authentication, not the typical username/password authentication, because the ingestion service does not maintain client sessions.

Your client app / aws Lambda function calls a public REST endpoint with a list of data filenames and a referenced pipe name. If new data files matching the list exists, they are queued for loading. 

auto_ingest param should be set to false. 

Use case: if files are constantly being loaded into an internal stage (using Python or Java), and we want snowpipe to pick up these files and load in to tables in snowflake. The problem is, internal stage do not have event notification feature, to still be able to use snowpipe, we need to call its REST API from the code. 

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

ALTER PIPE ... REFRESH statement loads the files that are staged within the past 7 days, and checks the load history for both the target table and the pipe to ensure the same files are not loaded twice.


## Snowpipe Costs
Query either of the following:
- PIPE_USAGE_HISTORY table function (in the Snowflake Information Schema).
- PIPE_USAGE_HISTORY View

Managed service, with additional cost of 0.06 credit/1k files being loaded. 

## Snowpipe Streaming Overview (preview)
Calling the Streaming API from your `Java application code` to `insert` rows of data to Snowflake tables directly, without staged files in the middle. This architecture results in lower load latencies and costs, which makes it great for real-time data streams. Do not require and pipe objects. 

The API ingests rows via one or more channels. Each channel points to only one table, but a table can have multiple channels pointing to it. 

## Snowpipe Streaming Configuring
API calls rely on key pair authentication with JSON Web Token (JWT).

## Snowpipe Streaming Recommendations
Recommend calling the API with fewer clients that write more data per second.

## Snowpipe Streaming Costs
Similar to other serverless features. 

## Snowpipe Streaming Kafka Connector with Snowpipe Streaming
Optionally replace Snowpipe with Snowpipe Streaming in your data loading chain from Kafka. Low latency and low cost. 

## Amazon S3-compatible Storage (Preview)
The industry-standard Amazon S3 REST API enables programmatic access to storage buckets and objects. To access storage outside of the public cloud, you can create external stages in Snowflake that store the S3-compliant API endpoint, bucket name/path/credentials. Then you can load and unload data from and to the storage locations.

## Querying Data in Staged Files
sf supports using SQL to query data files in an internal stage or named external (S3 etc) stage. Useful for inspecting the file content before loading / after unloading.

## Querying Metadata for Staged Files
Snowflake automatically generates metadata for files in stages. This metadata is "stored" in virtual columns that can be:
- Queried using a standard SELECT statement.
- Loaded into a table, along with the regular data columns, using COPY INTO (can only insert new rows, cannot modify existing row).

The metadata columns are: 
- METADATA$FILENAME: current row's file path
- METADATA$FILE_ROW_NUMBER: row num for each row
- METADATA$FILE_CONTENT_KEY: checksum of file for cur row
- METADATA$FILE_LAST_MODIFIED: cur row's file's last modified timestamp in NTZ
- METADATA$START_SCAN_TIME: cur row's start to scan time in LTZ

## Transforming Data During Load
All file formats supported by COPY INTO can use transformations, except JSON has to be "Newline delimited JSON" standard format. 

The VALIDATION_MODE param does not support transforming data during a load.

## Continuous data pipelines Overview
It automates many manual steps in transforming and optimizing continuous data loads. Usually, the raw data files are first loaded temporarily into a staging table, then transformed using SQL statements, then inserted into the destination reporting tables. The efficient way is to transform only new/modified data. 

A stream object records the delta (inserts etc, DML) of CDC info for a table - allows querying/consuming the changes at the row level, between two transactional points of time.

A task object defines a recurring schedule for executing a SQL statement (and SP calls) - can be chained together for successive execution of complex periodic processing. Can use streams by calling SYSTEM$STREAM_HAS_DATA. Users can define a tree-like structure of tasks to process/move data.

## Continuous data pipelines - Streams
A stream itself does not contain any table data - it only stores source object's offset and returns CDC records, using the source's versioning history. The stream rely on both the offset and the change tracking metadata stored in the table (several hidden cols created when stream for the table was first created, or when ALTER TABLE ... CHANGE_TRACKING = TRUE is executed).

For streams on views, change tracking must be enabled explicitly for the view & underlying tables to add the hidden columns to these tables.

A stream provides the minimal set of changes from its current offset to the current version of the table. Multiple queries can independently consume the same change data from a stream without changing the offset. A stream advances the offset only when it is used in a DML transaction. 

When you query a stream within a transaction, it follows repeatable read isolation transaction isolation level. When DML transaction completes successfully, the stream offset advances. To ensure multiple statements access the same change records in the stream (lock the stream), surround them with an explicit transaction statement (BEGIN .. COMMIT). 

Stream differs from the read committed mode supported for tables, in which statements see any changes made by previous statements executed within the same transaction, even though those changes are not yet committed.

Stream has 3 additional cols:
- METADATA$ACTION: the DML operation INSERT/DELETE.
- METADATA$ISUPDATE: in the stream, updates are represented as a pair of DELETE and INSERT records with  METADATA$ISUPDATE = TRUE. Because streams record the differences between two offsets, so if a row is added and then updated, the delta change is a new row. The METADATA$ISUPDATE row records a FALSE value.
- METADATA$ROW_ID: the unique and immutable ID for the row.

3 stream types:
- Standard. tracks all DML changes (insert/update/delete). Support tables, directory tables, views. Do not work with geospatial data - recommend creating append-only streams on them.
- Append-only. tracks row inserts only. Support standard tables, directory tables, views. 
- Insert-only. tracks row inserts only. Support external tables only. Overwritten/appended files are handled as new files - the stream does not record the diff of the old and new file versions.

A stream (unconsumed records inside it) becomes stale (gone) when its offset is outside of the effective data retention period for its source table, or the underlying tables for a source view. To track new change records for the table, recreate the stream. If the data retention period for a table is < 14d, and a stream has not been consumed, Snowflake extends to 14d by default (MAX_DATA_EXTENSION_TIME_IN_DAYS param), aka effective data retention time for this stream, regardless of the Snowflake edition. This could incur storage costs. 

To view the current staleness status of a stream, execute DESCRIBE STREAM / SHOW STREAMS. 

Recreating an object drops its history, which makes any stream relies on it go stale. 

If a table and its stream are cloned, the cloned stream offset is when it was cloned. 

Renaming a source obj does not break its stream (references are used instead of obj names).

Recommend to create a separate stream for each DML consumer (task, script, ... that consumes the same change data) for an obj.

Streams works for local views and shared secure views, but NOT for materialized views. Underlying tables must be native tables with change tracking enabled (using ALTER TABLE ... CHANGE_TRACKING = TRUE), and the view may only have projections, filters, inner/cross joins, union all, system scalar functions (group by, qualify, limit, etc are not supported). 

The CHANGES clause in SELECT enables querying change tracking metadata between two points in time, without having a stream. Using a stream is sufficient for most use cases, CHANGES clause is useful in infrequent cases when you need to manage the offset for arbitrary periods of time. 

The main cost associated with a stream is the processing time used by a virtual warehouse to query the stream. These charges appear on your bill as familiar Snowflake credits.

### Managing streams



























### Examples of streams

## Continuous data pipelines - Tasks

### Enabling Error Notifications for tasks
Enabling Error Notifications using AWS SNS
Enabling Error Notifications using MS Azure Grid Events
Enabling Error Notifications for Tasks using Google Pub/Sub
Integrating Task Error Notifications with Tasks
Task Error Payload Example
### Troubleshooting tasks

## Continuous data pipelines - Examples








