# 15. Performance Optimization
Snowflake quick start series offer T1-T3 level queries for performance optimization. Should go through it. 

## Exploring Execution Times
Examine the past performance of queries and tasks. 

You can use Snowsight Query history, Warehouses, Task history, to gain visual insights into the performance of queries and tasks as well as the load of a warehouse.

The Query Profile allows you to examine which parts of a query are taking the longest to execute. It includes a Most Expensive Nodes pane, which identifies the operator nodes that are taking the longest to execute. You can drill down even further, by viewing what percentage of a node's execution time was spent in a particular category of query processing.

The Account Usage schema contains views related to the execution times of queries and tasks: 
- QUERY_HISTORY
- WAREHOUSE_LOAD_HISTORY
- TASK_HISTORY

Users without access to the ACCOUNT_USAGE schema can still return recent execution times and other query metadata, using the QUERY_HISTORY() table functions of the Information Schema.

## Optimizing Warehouses for Performance
Fine-tuning the compute resources provided by a warehouse can improve the performance of a query or set of queries.

### Reducing Queues
- For a regular warehouse (not a multi-cluster warehouse), consider creating additional warehouses, and then distribute the queries among them. If specific queries are causing usage spikes, focus on moving those queries.
- Consider converting a warehouse to a multi-cluster warehouse so the warehouse can elastically provision additional compute resources when demand spikes. Multi-cluster warehouses require the Enterprise Edition.
- If you are already using a multi-cluster warehouse, increase the maximum number of clusters.

### Resolving Memory Spillage
Performance degrades drastically when a warehouse runs out of memory while executing a query, because memory bytes must "spill" onto local disk storage. If the query requires even more memory, it spills onto remote cloud-provider storage, which results in even worse performance.

When memory spillage is the issue, you can convert your existing warehouse to a `Snowpark-optimized warehouse`, which provides `16x more memory per node` and `10x the local cache` compared to a standard warehouse. Though a larger warehouse also has more memory available, a query might not require its expanded compute resources.

### Increasing Warehouse Size
The larger a warehouse, the more compute resources are available to execute a query or set of queries. This makes increasing the size of a warehouse a straightforward strategy for improving query performance; simply upsize the warehouse, re-run the query, and if the increased performance does not justify the increased cost of running the query, return the warehouse to its original size.

Using a larger warehouse has the biggest impact on larger, more complex queries, and may not improve the performance of small, basic queries.

Best practice is to limit who can adjust the size of a warehouse. Allowing users to increase the size of a warehouse to meet the needs of an individual query can result in unexpected costs if they forget to return the warehouse to its original size once the query has been executed.

### Trying Query Acceleration (Enterprise Edition &+)
The query acceleration service offloads portions of query processing to serverless compute resources, which speeds up the processing of a query while reducing its demand on the warehouse's compute resources.

Examples of workloads that might benefit from the query acceleration service include ad hoc analytics, workloads with unpredictable data volume per query, and queries with large scans and selective filters.

Query acceleration service is enabled for an entire warehouse, but unlike upsizing a warehouse, it is only used for queries that benefit from increased compute power. 

You can use the warehouse's scale factor to help control the cost of the query acceleration service. This scale factor, which is a multiplier of the warehouse's credit consumption, sets a limit on how much serverless compute can be used by a warehouse. For example, if a warehouse has a scale factor of 5, the credit consumption rate of serverless compute resources cannot exceed the consumption rate of the warehouse by more than 5 times.

### Optimizing the Cache
A running warehouse maintains a cache of table data that can be accessed by queries running on the same warehouse. This can improve the performance of subsequent queries, if they are able to read from the cache instead of from tables.

The auto-suspension setting of the warehouse can have a direct impact on query performance, because the cache is dropped when the warehouse is suspended. If a warehouse is running frequent and similar queries, it might not make sense to suspend the warehouse in between queries, because the cache might be dropped before the next query is executed.

You can use the following general guidelines when setting that auto-suspension time limit:
- For tasks, Snowflake recommends immediate suspension.
- For DevOps, DataOps and Data Science use cases, Snowflake recommends setting auto-suspension to approximately 5 minutes, as the cache is not as important for ad-hoc and unique queries.
- For query warehouses, for example BI and SELECT use cases, Snowflake recommends setting auto-suspension to at least 10 minutes to maintain the cache for users.

Keep in mind that a running warehouse consumes credits, even if it is not processing queries. Be sure that your auto-suspension setting matches your workload. For example, if a warehouse executes a query every 30 minutes, it does not make sense to set the auto-suspension setting to 10 minutes. The warehouse will consume credits while sitting idle without gaining the benefits of a cache, because it will be dropped before the next query executes.

### Limiting Concurrent Queries
Queries running concurrently in a warehouse must share the warehouse's resources, meaning each query might be granted fewer resources. You can use the MAX_CONCURRENCY_LEVEL parameter to limit the number of concurrent queries running in a warehouse.

Lowering the concurrency level may boost performance for individual queries, especially large, complex, or multi-statement queries, but these adjustments should be thoroughly tested to ensure they have the desired effect.

Be aware that lowering the `MAX_CONCURRENCY_LEVEL` for a warehouse can lead to more queries being placed in a queue, which has a performance implication for those queries. Other strategies, such as using a dedicated warehouse, or using the Query Acceleration Service, can boost the performance of a large or complex query, without impacting the rest of the workload. 

Adjusting the `STATEMENT_QUEUED_TIMEOUT_IN_SECONDS` parameter can cancel queries, rather than let them remain in the queue for an extended period of time.

## Optimizing Storage for Performance
- clustering a table
- search optimization service - point lookup. You can enable the Search Optimization Service for an entire table or for specific columns. supports both structured and semi-structured data. 
- materialized views

Automatic Clustering makes sense when many queries filter, join, or aggregate the same few columns.

If more than one strategy could potentially improve the performance of a particular query, you might want to start with Automatic Clustering, or the Search Optimization Service, because other queries with similar access patterns could also be improved.

Automatic Clustering: 
- Biggest performance boost comes from a `WHERE clause that filters on a column of the cluster key`, but it can also improve the performance of other clauses and functions that act upon that same column (joins/aggregations).
- Ideal for range queries, or queries with an inequality filter. Also improves an equality filter, but the Search Optimization Service is usually faster for point lookup queries.
- Available in Standard Edition of Snowflake.
- There can be only one cluster key. If different queries against a table act upon different columns, consider using the Search Optimization Service or a materialized view instead.

Search Optimization Service: 
- Improves `point lookup` queries that return a small number of rows. If the query returns more than a few records, consider Automatic Clustering instead.
- Includes support for point lookup queries that:
  - Match substrings or regular expressions using predicates such as LIKE and RLIKE.
  - Search for specific fields in VARIANT, ARRAY, or OBJECT columns.
  - Use geospatial functions with GEOGRAPHY values.

Materialized view: 
- Improves intensive and frequent calculations, such as aggregation and analyzing semi-structured data (not just filtering).
- Usually focused on a specific query/subquery calculation.
- Improves queries against external tables.

The cost of maintaining materialized views or the Search Optimization Service can be significant, when Automatic Clustering is enabled for the underlying table. With Automatic Clustering, Snowflake is constantly reclustering its micro-partitions around the dimensions of the cluster key. Every time the base table is reclustered, Snowflake must use serverless compute resources to update the storage used by materialized views and the Search Optimization Service. As a result, Automatic Clustering activities on the base table can trigger maintenance costs for materialized views and the Search Optimization Service, beyond the cost of the DML commands on the base table.

You can run the `SYSTEM$ESTIMATE_SEARCH_OPTIMIZATION_COSTS` function to help estimate the cost of adding the Search Optimization Service to a column or entire table. The estimated costs are proportional to the number of columns that will be enabled, and how much the table has recently changed.

You might choose a cluster key for just one or two tables, and then assess the cost before choosing a key for other tables.


