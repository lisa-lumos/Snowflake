# System functions
Snowflake provides the following types of system functions:
- Control functions that allow you to execute actions in the system (e.g. aborting a query).
- Information functions that return information about the system (e.g. calculating the clustering depth of a table).
- Information functions that return information about queries (e.g. information about EXPLAIN plans).

Many of these system functions have the prefix SYSTEM$, such as `select system$typeof('a')`;

## system$clustering_information()
Returns clustering information for a table, including average clustering depth, based on one or more columns.

You can use this argument to return clustering information for any columns in the table, regardless of whether a clustering key is defined for the table. In other words, you can use this to help you decide what clustering to use in the future.

```sql
select system$clustering_information('test2', '(col1, col3)');
-- +--------------------------------------------------------------+
-- | SYSTEM$CLUSTERING_INFORMATION('TEST2', '(COL1, COL3)')       |
-- |--------------------------------------------------------------|
-- | {                                                            |
-- |   "cluster_by_keys" : "LINEAR(COL1, COL3)",                  |
-- |   "total_partition_count" : 1156,                            |
-- |   "total_constant_partition_count" : 0,                      |
-- |   "average_overlaps" : 117.5484,                             |
-- |   "average_depth" : 64.0701,                                 |
-- |   "partition_depth_histogram" : {                            |
-- |     "00000" : 0,                                             |
-- |     "00001" : 0,                                             |
-- |     "00002" : 3,                                             |
-- |     "00003" : 3,                                             |
-- |     "00004" : 4,                                             |
-- |     "00005" : 6,                                             |
-- |     "00006" : 3,                                             |
-- |     "00007" : 5,                                             |
-- |     "00008" : 10,                                            |
-- |     "00009" : 5,                                             |
-- |     "00010" : 7,                                             |
-- |     "00011" : 6,                                             |
-- |     "00012" : 8,                                             |
-- |     "00013" : 8,                                             |
-- |     "00014" : 9,                                             |
-- |     "00015" : 8,                                             |
-- |     "00016" : 6,                                             |
-- |     "00032" : 98,                                            |
-- |     "00064" : 269,                                           |
-- |     "00128" : 698                                            |
-- |   }                                                          |
-- | }                                                            |
-- +--------------------------------------------------------------+
```

## system$clustering_depth()
Computes the average depth of the table according to the specified columns (or the clustering key defined for the table). The average depth of a populated table (i.e. a table containing data) is always 1 or more. The smaller the average depth, the better clustered the table is with regards to the specified columns.

```sql
select system$clustering_depth('tpch_orders');
-- +----------------------------------------+
-- | SYSTEM$CLUSTERING_DEPTH('TPCH_ORDERS') |
-- |----------------------------------------+
-- | 2.4865                                 |
-- +----------------------------------------+

select system$clustering_depth('tpch_orders', '(c2, c9)');
-- +----------------------------------------------------+
-- | SYSTEM$CLUSTERING_DEPTH('TPCH_ORDERS', '(C2, C9)') |
-- +----------------------------------------------------+
-- | 23.1351                                            |
-- +----------------------------------------------------+

select system$clustering_depth('tpch_orders', '(c2, c9)', 'c2 = 25');
-- +----------------------------------------------------+
-- | SYSTEM$CLUSTERING_DEPTH('TPCH_ORDERS', '(C2, C9)') |
-- +----------------------------------------------------+
-- | 11.2452                                            |
-- +----------------------------------------------------+
```


















