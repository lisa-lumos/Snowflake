# 3. Data Warehouses
A virtual warehouse is a cluster of compute resources in Snowflake. Provides CPU, memory, temporary storage to execute SQL select, DML queries, and data loading and unloading. 

Available in two types:
- Standard
- Snowpark-optimized

## Overview
Warehouse has 10 sizes: xs, s, m, l, xl, 2xl, ..., 6xl. xs warehouse consumes 1 credit per hour, and the credit usage doubles with each size up. 

For warehouses created using CREATE WAREHOUSE command, size is xs by default, as opposed to xl default for those created in the web UI. 

Snowflake utilizes per-second billing, with a 60-second minimum each time the warehouse starts. The total number of credits billed depends on how long the warehouse runs continuously.

Increasing the size of a warehouse does not always improve data loading performance - it is influenced more by the number and size of files being loaded.

The size of a warehouse impacts the queries runtime, particularly for larger, more complex queries. In general, query performance scales with warehouse size. Larger is not necessarily faster for small, basic queries.

Resizing a running warehouse do not impact running queries, but can be used for queued or new queries.

By default, a warehouse automatically resume or suspend based on activity. 

The number of queries runs concurrently in a warehouse is determined by the size and complexity of each query. As a query is submitted, the warehouse calculates and reserves the compute resources. If there is not enough resources for it, the query is queued until resources become available.

A default warehouse can be set for each user, which then be used by all sessions initiated by the user. The same can be set for SF clients such as SnowSQL, JDBC driver, ODBC driver, Python connector, etc. 

## Multi-cluster (starts from Enterprise Edition)
With multi-cluster warehouses, Snowflake supports allocating statically/dynamically additional clusters to allow better concurrency, in contrast, a single cluster warehouse will put queries into queues if there are not enough resources for them. Define a multi-cluster warehouse by specifying maximum and minimum number of clusters (up to 10). Num of clusters will not affect query performance, only concurrency. 

It can be created using sql or web UI. 

Multi-cluster warehouses can have two modes: 
- Maximized: max num of clusters = min num of clusters. Suitable for large numbers of concurrent user sessions/queries, and it not fluctuate much. 
- Auto-scale: max num of clusters != min num of clusters. Can set scaling policy to standard (default, start new cluster to avoid any query queue, close cluster when checked for ~3 min its load can move to other running clusters) or economy (start new cluster only if new query has to wait > ~6 min in queue; close cluster when checked for ~6 min its load can move to other running clusters). 

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
- Scale up by resizing a warehouse.
- Scale out by adding clusters to a multi-cluster warehouse

If you are using Snowflake Enterprise Edition or higher, all your wh should be configured as multi-cluster.

## Working with warehouses

## Query acceleration service

## Monitoring load

## Snowpark-optimized warehouses




























