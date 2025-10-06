# 3. Virtual Warehouses
A virtual warehouse is a cluster of compute resources in Snowflake. Provides CPU, memory, temporary storage to execute SQL select, DML queries, and data loading and unloading. 

Available in two types:
- Standard
- Snowpark-optimized

## Overview
Warehouse has 10 sizes: xs, s, m, l, xl, 2xl, ..., 6xl. xs warehouse consumes 1 credit/hr, and the credit usage doubles with each size up. 

For warehouses created using CREATE WAREHOUSE command, size is `xs by default`, as opposed to xl default for those created in the legacy web UI. 

Snowflake utilizes per-second billing, with a 60-second minimum each time the warehouse starts. The total number of credits billed depends on how long the warehouse runs continuously.

Increasing the size of a warehouse does not always improve data loading performance - it is influenced more by the number and size of files being loaded.

The size of a warehouse impacts the queries runtime, particularly for larger, more complex queries. In general, query performance scales with warehouse size. Larger is not necessarily faster for small, basic queries.

Resizing a running warehouse do not impact running queries, but can be used for queued or new queries.

By default, a warehouse automatically resume/suspend based on activity. 

The number of queries runs concurrently in a warehouse is determined by the size and complexity of each query. As a query is submitted, the warehouse calculates and reserves the compute resources. If there is not enough resources for it, the query is queued until resources become available.

A default warehouse can be set for each user, which then be used by all sessions initiated by the user. The same can be set for SF clients such as SnowSQL, JDBC driver, ODBC driver, Python connector, etc. 

Default wh precedence: ad hoc command > default wh set for the client driver > default wh set for the user.  

## Multi-cluster (starts from Enterprise Edition)
With multi-cluster warehouses, Snowflake supports allocating statically/dynamically additional clusters to allow better concurrency, in contrast, a single cluster warehouse will put queries into queues if there are not enough resources for them. Define a multi-cluster warehouse by specifying maximum and minimum number of clusters (up to 10). Num of clusters will not affect query performance, only concurrency. 

It can be created using sql or web UI. 

Multi-cluster warehouses can have two modes: 
- Maximized: `max num of clusters = min num of clusters`. Suitable for large numbers of concurrent user sessions/queries, and it not fluctuate much. The whole cluster resume and suspend together, not one by one. 
- Auto-scale: `max num of clusters != min num of clusters`. Can set scaling policy to `standard` (default, start new cluster to avoid any query queue, close cluster when checked for ~3 min its load can move to other running clusters) or `economy` (start new cluster only if new query has to wait > ~6 min in queue; close cluster when checked for ~6 min its load can move to other running clusters). 

To set max/min num of cluster, start with auto-scale, and start with small nums, then adjust with loads over time.

If a multi-cluster warehouse is resized, the new size applies to all the clusters, including running ones. 

You can increase or decrease the number of clusters for a warehouse at any time, even while running. 

## Considerations
Query performance factors:
- The size of the tables has more impact than the num of rows.
- filtering, joins, num of tables
- cache of running wh can be reused for latter queries. The larger the wh, the larger the cache. Cache is dropped when wh is suspended. Consider the trade-off between saving credits by [suspending a wh] versus [maintaining the cache from prev queries to improve performance].

warehouse size:
- for data loading, size should match number and size of files
- smaller wh for testing, larger wh for production

If you enable auto-suspend, recommend setting it to a low value (5 or 10 min or less) to save cost.

Consider disabling auto-suspend for a warehouse if you:
- have a heavy, steady workload.
- require it to be available with no delay. Provision new wh costs a few secs, but may be longer.  
 
To disable auto-suspend, explicitly select Never in the web UI, or specify 0 or NULL in SQL.

If wish to control costs or user access, disable auto-resume and manually resume the wh only when needed. 

Scaling Up vs Scaling Out
- Scale up by resizing a warehouse
- Scale out by adding clusters to a multi-cluster warehouse

If you are using Snowflake Enterprise Edition or higher, all your wh should be configured as multi-cluster.

## Working with warehouses
A Snowflake session can only have one current warehouse at a time, which can be specified/changed at any time with the USE WAREHOUSE command.

## Query acceleration service (starts from Enterprise Edition)
`Can be enabled for a warehouse.` Can accelerate query or its sub-queries, by offloading parts of the work in parallel to serverless compute resources provided by the service. Reduces the impact of outlier queries. 

Types of workloads that might benefit from the query acceleration service:
- Ad hoc analytics.
- Workloads with unpredictable data volume per query.
- Queries with large scans and selective filters.

To identify the queries that might benefit from this service: 
- use the system$estimate_query_acceleration() function to check a specific query
- query the QUERY_ACCELERATION_ELIGIBLE View. It identifies the queries/warehouses that might benefit the most from the service. 

Enable the service by specifying ENABLE_QUERY_ACCELERATION = TRUE for a warehouse. 

supports the following SQL commands:
- SELECT
- INSERT when it contains a SELECT
- CREATE TABLE AS SELECT

The service may increase the cost rate of a warehouse - limited by the maximum scale factor of the wh, it can be set (default is 8, setting it to 0 removes the upper bound limit). E.g.: set the scale factor to 5 for a  warehouse means it can lease compute resources up to 5 times the size/cost of itself. The amount it actually uses depends on how much it needs.  

The is billed by the second only when the service is in use. These credits are billed separately from warehouse usage.

This service usage can be seen in query profile, under Query Acceleration section. Can also been seen in the query_history view:
- QUERY_ACCELERATION_BYTES_SCANNED
- QUERY_ACCELERATION_PARTITIONS_SCANNED
- QUERY_ACCELERATION_UPPER_LIMIT_SCALE_FACTOR

This service's cost is billed as a serverless feature. Can be seen in 
- the UI for the warehouse, 
- the query_acceleration_history view, 
- query_acceleration_history function 

## Monitoring load
Query load chart from the UI shows num of concurrent queries of a warehouse in the past 2 weeks. Viewer needs MONITOR privilege on the wh. 

In the query_history view, you can see wh performance metrics for different queries.

## Snowpark-optimized warehouses (aka, memory-optimized)
They have 16x ram/node compared to standard ones. Can be used for snowpark workloads with:
- large memory requirements (ML with a SP on a single wh node). 
- UDF or UDTF

They do not support query acceleration.

To maximize CPU and memory resources of a wh, set its MAX_CONCURRENCY_LEVEL = 1. 

They have sizes m, l, xl, 2xl, ..., 6xl; their cost of credit/hr starts from 6, and doubles with each size up. 

# Interactive tables and warehouses
They deliver low-latency query performance for high-concurrency, interactive workloads. Together, they enable use cases such as real-time dashboards, data-powered APIs, and serving high-concurrency workloads. 

Interactive warehouse is optimized to run continuously, serving high volumes of concurrent queries.

Interactive tables have different methods for data ingestion and support a more limited set of SQL statements and query operators than standard Snowflake tables. They are optimized for fast, simple queries when you require consistent low-latency responses. Interactive warehouses provide the compute resources required to serve these queries efficiently. 

The simple queries that work best with interactive tables are SELECT statements with selective WHERE clauses, optionally including a GROUP BY clause on a few dimensions. Avoid queries involving large joins and large subqueries. The performance of queries that use other features, such as window functions, is highly dependent on the data shapes that you are querying.

Limitations:
- wh do not support long running queries
- interactive wh do not auto suspend
- limited table DML, such as no update/delete. 
- table do not support materialized view, etc. 




















