# 4. Databases, Tables, & Views
All data in Snowflake is maintained in databases. 

Database can have one or more schemas (logical groupings of db objects, such as tables, views, ...). 

Snowflake do not limit the number of databases, schemas, or objects you can create.

## Table structures
All data in Snowflake is stored in database tables. Permanent tables have time trave period of 0 or 1 (default) day for standard edition, and 0 to 90 days of time travel for enterprise edition and above. Fail-safe is always 7 days. 

### Micro-Partitions & Data Clustering
Data in Snowflake tables is automatically divided into compressed micro-partitions, each contains 50-500 MB of data before compression. Each partition contains multiple rows of the table stored column by column (columnar), and metadata about these rows (value range of each cols, num of distinct vals, ...). 

Data in a table is partitioned by the data insertion/load order. 

Benefits: 
- done automatically, no need to manual definition
- small partitions allow for fast dml and fine-grained pruning
- partitions can overlap, prevents skew
- columnar format allows for efficient analytical queries
- each column in a partition is compressed

Clustering metadata for a table:
- num of partitions
- num of partitions with overlapping vals
- overlap depth of partitions (clustering depth)

Clustering depth can be use for:
- monitor cluster health of a large table
- decide whether a large table would benefit from a clustering key

To view clustering info for a col in a table, use these system functions:
- SYSTEM$CLUSTERING_DEPTH
- SYSTEM$CLUSTERING_INFORMATION (including clustering depth)

### Cluster Keys & Clustered Tables
A clustering key for a table is some columns (or expressions) that are used to co-locate the data in the same micro-partitions. Useful for large tables who do not have ideal ordering. Queries benefit from clustering when the queries filter/sort on the clustering key.

Enable clustering on a table/materialized view by specifying a clustering key for it. Snowflake then will automatically maintain it being clustered (by reclustering), which consumes serverless compute credits and storage costs (especially for time travel and fail-safe). 

Cost-effective for tables that are queried frequently and updated infrequently.

Clustering is best for tables that satisfies all of this: 
- contains TBs of data
- queries benefit from clustering
- high % of queries benefit from same clustering key. 

Recommend: 
- <= 4 columns per key
- prioritize cols most used in filters, then joins
- col with too high or too low cardinality usually is not a good candidate
- for multi col key, order them from low to high cardinality

Notes: 
- Clustering key cannot be defined on cols with these 4 data types: GEOGRAPHY, VARIANT (but path in it is okay), OBJECT, ARRAY. 
- When adding a clustering key to a table with data, not all expressions are allowed. Check it using SHOW FUNCTIONS: `show functions like 'function_name';`

### Automatic Clustering
You can suspend or resume this for a table. 

If a table is created by cloning from a table (or db/schema that contains this table) with cluster key, then its automatic clustering is suspended by default. 

To check whether automatic clustering is enabled: 
- SHOW TABLES command.
- TABLES view (in the Snowflake Information Schema).
- TABLES view (in the Account Usage shared database).

View cost by: 
- Snowsight: admin -> usage
- SQL: 
  - AUTOMATIC_CLUSTERING_HISTORY table function
  - AUTOMATIC_CLUSTERING_HISTORY View

## Temporary and Transient tables/stages
### Temporary tables
Lives in the session only. Not visible to other users or sessions, and when the session ends, it is gone forever. You pay for the storage for its duration, can manually drop when it is no longer needed to save cost. It cannot be changed to other table types. 

Have 0 or 1 (default) day of time travel as long as the session is alive, otherwise gone forever. Do not have fail-safe. 

Temporary table can have same name with a permanent table in the same location, but it is not recommended to do so. 

### Transient tables/databases/schemas
Lives until you manually drop it. Visible to all users with right privileges. You pay for the storage for its duration. 

Have 0 or 1 (default) day of time travel. Do not have fail-safe. 

All tables created in a transient schema, as well as all schemas created in a transient database, are transient by definition. It cannot be changed to other table types. 

## External Tables
The data in an external table is stored in files in an external stage. It stores metadata about the data files (filename, version identifier, ...), so you can query a file in an external stage as if it were inside a database. Can access data supported by COPY INTO statements.

This metadata can be manually refreshed (charged as cloud services), or configured to auto-refresh using event notifications from your cloud service (within snowpipe charges). AUTO_REFRESH_REGISTRATION_HISTORY table function, EXTERNAL_TABLES View, etc shows history of metadata and credits consumption. 

External tables are read-only. Views/materialized views can be created based on them.

All external tables include the following columns:
- VALUE - A VARIANT col of one row in the external file; select * returns this col
- METADATA$FILENAME - A pseudo col of path_in_stage/filename for each file
- METADATA$FILE_ROW_NUMBER - A pseudo col of row number for each row.

You can create additional virtual cols as expressions using the VALUE cols and/or the 2 pseudo cols. 

Recommend external tables to be partitioned to improve query performance. 

Similar to tables, the query results for external tables persist for 24 hours, as long as metadata is not refreshed. 

## Search Optimization service


## Views


## Secure views


## Materialized views


## Table design


## Cloning


## Data storage


































