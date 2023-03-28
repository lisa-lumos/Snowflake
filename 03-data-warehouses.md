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

## Multi-cluster

## Considerations

## Working with warehouses

## Query acceleration service

## Monitoring load

## Snowpark-optimized warehouses




























