# 7. Data Unloading
## Overview
Snowflake supports bulk unload of data from a database table into flat, delimited text files.

COPY INTO stage -> GET

You can also use SELECT statement instead of a table in the COPY INTO stage command. The results of the query are written to one or more files as specified in the command. SELECT queries in COPY statements support the full syntax and semantics of Snowflake SQL queries, including JOIN clauses, which enables downloading data from multiple tables.

The COPY INTO stage command provides SINGLE copy option for unloading data into single/multiple files. The default is SINGLE = FALSE (unload into multiple files).

Snowflake assigns each file a unique name. The location path in the command can contain a filename prefix for all the data files. If a prefix is not specified, Snowflake prefixes the generated filenames with `data_`. Snowflake appends a suffix that ensures each file name is unique across parallel execution threads; e.g. `data_stats_0_1_0`.

When unloading into multiple files, use the MAX_FILE_SIZE (default is 16MB) copy option to specify the maximum size of each file created. Max supported size for external stage is 5GB. 

The COPY INTO command includes a PARTITION BY copy option, which accepts an expression by which the unload operation partitions table rows into separate files to the stage. This has many use cases, such as using Snowflake to transform data for output to a data lake. Also, partitioning unloaded data into a directory structure in cloud storage can increase the efficiency with which third-party tools consume the data. 

Use case: after a lot of processing in snowflake, you obtain the curated data which can be directly used for analysis. If you want to use this data offline, you can copy it out of snowflake.  

## Features
Supported file format: delimited files (csv, tsv, ...), JSON, Parquet. 

Supported file encoding: utf-8 only. 

Supported compression: gzip (default), bzip2, Brotli, Zstandard. 

Encryption:
- internal stage: 128-bit (default) or 256-bit key
- external stage: user-supplied key

## Considerations
In CSV files, a NULL value is typically represented by two successive delimiters (,,) to indicate that the field contains no data. An empty string is typically represented by a quoted empty string ('') to indicate that the string contains zero characters. 

`FIELD_OPTIONALLY_ENCLOSED_BY` (default is NONE) allow you to differentiate between empty strings and NULL values. In this way, sql null values are just `,,`, and empty strings are enclosed by what you specify, such as `''`. 

Also works but not recommended: The COPY INTO command can unload empty strings no enclosing quotes, with the EMPTY_FIELD_AS_NULL = FALSE (default is TRUE, which causes null and empty string in csv indistinguishable), and specifying NULL_IF. Snowflake converts SQL NULL values to the first value in the list of `NULL_IF = ('null')` (default is nothing). In this way, sql null values are converted to what you specify, such as `,'null',`, and empty strings are just `,,` in output files. 

You can use the OBJECT_CONSTRUCT function inside COPY command to convert the rows in a relational table to a single VARIANT column and unload the rows into a file.

You can unload data in a relational table to a multi-column Parquet file by using a SELECT statement inside the COPY statement. The SELECT statement specifies the column data in the relational table to include in the unloaded file. Use the `HEADER = TRUE` copy option to include the column headers in the output files.

When floating-point number columns are unloaded to CSV/JSON files, Snowflake truncates the values to approximately (15,9). The values are not truncated when unloading floating-point number columns to Parquet files.

## Preparing to Unload Data
read, skipped

## Unloading into a Snowflake Stage
read, skipped

## Unloading into Amazon S3
skipped

## Unloading into Google Cloud Storage
skipped

## Unloading into Microsoft Azure
skipped

