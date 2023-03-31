# 4. Databases, Tables, & Views
All data in Snowflake is maintained in databases. 

Database can have one or more schemas (logical groupings of db objects, such as tables, views, ...). 

Snowflake do not limit the number of databases, schemas, or objects you can create.

## Table structures
All data in Snowflake is stored in database tables. 

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

## Temporary and Transient tables


## External Tables


## Search Optimization service


## Views


## Secure views


## Materialized views


## Table design


## Cloning


## Data storage


































