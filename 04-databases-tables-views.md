# 4. Databases, Tables, & Views
All data in Snowflake is maintained in databases. 

Database can have one or more schemas (logical groupings of db objects, such as tables, views, ...). 

Snowflake do not limit the number of databases, schemas, or objects you can create.

## Table structures
All data in Snowflake is stored in database tables. Permanent tables have time trave period of `0 or 1 (default) day for standard edition`, and `0 to 90 days of time travel for enterprise edition and above`. Fail-safe is always 7 days. 

### Micro-Partitions & Data Clustering
Data in Snowflake tables is automatically divided into compressed micro-partitions, each contains `50-500 MB` of data before compression. Each partition contains multiple rows of the table stored column by column (columnar), and metadata about these rows (value range of each cols, num of distinct vals, ...). 

Data in a table is partitioned by the data insertion/load order. 

Snowflake stores these metadata about a micro-partition:
1. val range for each col
2. num of distinct values
3. other properties

When a column in a table is dropped, the micro-partitions that contain the data for the dropped column are not re-written. The data in the dropped column remains in storage.

Benefits: 
- done automatically, no need manual definition
- small partitions allow for fast dml and fine-grained pruning
- partitions can overlap, prevents skew
- columnar format allows for efficient analytical queries
- each column in a partition is compressed

Clustering metadata for a table contains:
- num of partitions
- num of partitions with overlapping vals
- overlap depth of partitions (clustering depth, starts from 1)

Clustering depth can be use for:
- monitor cluster health of a large table
- decide whether a large table would benefit from a clustering key

To view clustering info for a col in a table, use these system functions:
- `system$clustering_depth()`
- `system$clustering_information()` (including clustering depth)

### Cluster Keys & Clustered Tables
A clustering key for a table is some columns/expressions that are used to co-locate the data in the same micro-partitions. Useful for large tables who do not have ideal ordering. Queries benefit from clustering when the queries filter/sort on the clustering key.

Enable clustering on a table/materialized view by specifying a clustering key for it. Snowflake then will automatically maintain it being clustered (by reclustering), which consumes serverless compute credits and storage costs (especially for time travel and fail-safe). 

Cost-effective for tables that are queried frequently, and updated infrequently.

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
  - AUTOMATIC_CLUSTERING_HISTORY() table function
  - AUTOMATIC_CLUSTERING_HISTORY View

## Temporary and Transient tables/stages
Permanent table cannot be changed to other table types. 
### Temporary tables
Lives in the session only. Not visible to other users/sessions, and when the session ends, it is gone forever. You pay for the storage for its duration, can manually drop when it is no longer needed to save cost. It cannot be changed to other table types. 

Have 0 or 1 (default) day of time travel as long as the session is alive, otherwise gone forever. Do not have fail-safe. 

Temporary table can have same name with a permanent table in the same location, but it is not recommended to do so. 

### Transient tables/databases/schemas
Lives until you manually drop it. Visible to all users with right privileges. You pay for the storage for its duration. 

Have 0 or 1 (default) day of time travel. Do not have fail-safe. 

All tables created in a transient schema, as well as all schemas created in a transient database, are transient by definition. It cannot be changed to other table types. 

## External Tables
The data in an external table is stored in files in an external stage. It stores metadata about the data files (filename, version identifier, ...), so you can query a file in an external stage as if it were inside a database. Can access data supported by COPY INTO statements.

This metadata can be manually refreshed (charged as cloud services), or configured to auto-refresh using event notifications (cloud messaging) from your cloud service (within snowpipe charges). auto_refresh_registration_history() table function, EXTERNAL_TABLES View, etc shows history of metadata and credits consumption. 

`External tables are read-only. Views/materialized views can be created based on them.`

All external tables include the following columns:
- VALUE - A variant col represents each row in the external file; select * returns this col
- METADATA$FILENAME - A pseudo col of path_in_stage/filename for each file
- METADATA$FILE_ROW_NUMBER - A pseudo col of row number for each row.

You can create additional virtual cols as expressions using the VALUE cols and/or the 2 pseudo cols. 

Performance is slower compared with regular tables, but saves cost. Recommend external tables to be partitioned to improve query performance. 

Partition example: Suppose you have order data from different years S3, with each file contain a particular year of data, and the external table points to all these files. Without partition, if you only need the orders data from a particular year, the query needs to scan all files (of all years), then do the filtering to return only data for that year. With partitioning, you will have one data file in each folder named by year, and the query only retrieves the data in that file. You declare partition by year in the external table definition, and select year as extracting year from METADATA$FILENAME. 

Similar to tables, the query results for external tables persist for 24 hours, as long as metadata is not refreshed. 

Use case: in a S3 bucket, you have weather data of each city in a country that is uploaded daily into the bucket. But your customer has a manufacture business in city A, so they are only interested in visualizing the weather data of this city (only a subset of your massive data). 

External table supports Delta lake. 

The Hive metastore connector for Snowflake can integrate Apache Hive metastores with Snowflake, using external tables.

## Iceberg Tables
Snowflake supports Iceberg tables that use the Apache Parquet file format.

Iceberg tables for Snowflake combine the performance and query semantics of regular Snowflake tables with external cloud storage that you manage. They are ideal for existing data lakes that you cannot, or choose not to, store in Snowflake. 

## Hybrid tables
A hybrid table is a Snowflake table type, that is optimized for hybrid transactional/operational workloads, that require low latency and high throughput on small random point reads/writes. 

A hybrid table supports unique and referential integrity constraint enforcement that is critical for transactional workloads.

While you should expect Snowflake standard tables to offer better performance on large analytical queries, hybrid tables allows for faster results on short-running operational queries. The following types of queries are most likely to benefit from hybrid tables:
- High concurrency random point reads, versus large range reads.
- High concurrency random writes, versus large sequential writes (for example, bulk loading).
- Retrieval of a small number of entire records (for example, customer object) versus narrow projections with analytical functions (for example, aggregations or group by).

## Search Optimization service (Enterprise edition and higher)
`Applies to a whole table or columns in a table.` Can significantly improve the performance of certain types of queries that
- returns small num of distinct rows (with highly selective filters like equal to)
- does substr and regex searches (like, etc)
- uses variant, object, array fields with filters
- uses geospatial functions with geography vals

A background maintenance service creates and maintains the "search access path". Has storage and compute cost. 

The search optimization can improve the performance of views (including secure views).

The search optimization can improve query performance for tables with masking policies and row access policies.

The search optimization service does not support:
- External tables.
- Materialized views.
- Columns defined with a COLLATE clause.
- Column concatenation.
- Analytical expressions.
- Casts on table columns (except for fixed-point numbers cast to strings).

If you clone a table, schema, or database, the SEARCH OPTIMIZATION property and search access paths of each table are also cloned. 

If a table in the primary database has the SEARCH OPTIMIZATION property enabled, the property is replicated to the corresponding table in the secondary database. Search access paths in the secondary database are not replicated but are instead rebuilt automatically. 

If a table has a high churn rate, enabling automatic clustering and configuring search optimization for the table can result in higher maintenance costs than if the table is just configured for search optimization.

To estimate the cost of search optimization, use the `SYSTEM$ESTIMATE_SEARCH_OPTIMIZATION_COSTS()` function.

Search optimization and query acceleration can work together to optimize query performance. 

## Views
A regular view is a named definition of a query, and can be recursive. Views can be used for combining, segregating, and protecting data (by granting privileges on a view, instead of on the underlying tables). Can be used for writing modular code and increase code re-use. 

User who have access to standard view can see its definition in show view / query profile. 

## Secure views
Both non-materialized and materialized views can be secure. Secure views have improved data privacy, can be used for data sharing (standard view cannot be shared); however, they have performance impacts.

Secure view is for data privacy: 
- will not use internal optimizations (so will not expose underlying data by re-ordering predicates, etc)
- will not expose view definition

Views can be set as secure after creation. 

Secure view's definition is visible to its owner role. In Query Profile, the internals of a secure view are not exposed even for the owner, because non-owners might have access to an owner's Query Profile.

View security can be integrated with Snowflake users and roles using the CURRENT_ROLE and CURRENT_USER context functions.

When using secure views with Secure Data Sharing, use the CURRENT_ACCOUNT function to authorize users from an account.

## Materialized views (Enterprise edition and plus)
Goal is to improve performance. A materialized view's results are stored, but requires storage and compute cost to maintain it. 

Materialized views support clustering, so you can create multiple materialized views on the same table, with each view clustered on a different column, so that different queries can run on the view best for that query. You don't need to specify a materialized view in a query in order for the view to be used - the query optimizer can automatically decide to use it.

Materialized views are particularly useful when:
- Query results contain a small num of rows
- Query require significant processing, and used often
- The query is on an external table
- The base table does not change frequently.

Limitations:
- can query only one table
- cannot have join, having, order by, limit, nor group by cols not in the view. 

MATERIALIZED_VIEW_REFRESH_HISTORY table function and view shows costs. 

You cannot use Time Travel to query historical data for materialized views.

Recommend batching DML operations on the base table to minimize costs. 

You can suspend a materialized view to defer maintenance costs. 

`You can create a materialized view on data shared to you.` You pay for its charges. 

You can share a materialized view. 

You can create a materialized view on external tables. 

## Table design
When defining cols of dates/timestamps, recommend to use date/timestamp data type rather than VARCHAR. This improves query performance.

Referential integrity constraints in SF are informational (not enforced), except for NOT NULL. 

There is no storage/performance difference between a col with a max length declaration like VARCHAR(16777216), and a smaller precision. But using appropriate col length is recommended for easy debugging and convenience for 3rd party tools. 

If unsure what operations to perform on semi-structured data, recommend storing it in a VARIANT column for now. For data that is mostly regular and uses only native types (strings and integers), the storage requirements and query performance for operations on relational data and data in a VARIANT column is very similar.

## Cloning
When a db or schema got cloned, privileges of all its child objects are copied, but the container itself's privileges do not get copied. Use "copy grants" clause to copy container's privileges (except ownership), or grant explicitly. 

Cloned objects get object parameters that were set on the source object.

Automatic clustering is suspended for the new table by default. 

Individual external named stages can be cloned. Internal stages cannot be cloned. 

When a container is cloned, any pipes inside that uses an internal stage are not cloned, while those that uses external stages are cloned. A cloned pipe is paused by default.

When a container is cloned, the cloned streams inside becomes empty.

When a container is cloned, the cloned tasks inside are suspended by default.

Cloning a schema results in the cloning of all policies within the schema.

A explicitly cloned table maps to the same policies as the source table.

You can use Time Travel to clone objects at a specific time point in the past. 

Clones can be cloned. 

Cloning is fast, but not instantaneous, particularly for large tables. 


## Data storage considerations
Storage is charged for data in the Active, Time Travel, and Fail-safe state. 

TABLE_STORAGE_METRICS view includes a breakdown of the physical storage (in bytes) for table data in the 3 states of the CDP life-cycle. 

Data files staged in Snowflake internal stages have no Time Travel and Fail-safe cost, but they have standard data storage costs. 

Zero-copy clone is useful for creating instant backups that do not have any additional costs, until changes are made to the cloned object.

High-churn dimension tables can be identified by calculating the ratio of FAILSAFE_BYTES divided by ACTIVE_BYTES in the TABLE_STORAGE_METRICS view. Any table with a large ratio is a high-churn table. 

For large, high-churn dimension tables that incur overly-excessive CDP costs, the solution is to create these tables as transient with 0 Time Travel retention, and then copy these tables periodically (at least once a day) into a permanent table. This creates a full backup of these tables. When a new backup is created, the old one can be deleted and protected by CDP.


