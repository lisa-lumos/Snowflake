# 7. Data Unloading
Snowflake supports bulk export (unloading) of data from a database table into flat, delimited text files. The process is the same as the loading process, except in reverse - "copy into", then "get" command. 

Snowflake supports specifying a SELECT statement (instead of a table) in the COPY INTO command, the query results are written into one or more files (SINGLE = FALSE is default, MAX_FILE_SIZE determines maximum file size, PARTITION BY for partitioning) in the specified loc.

(updated to https://docs.snowflake.com/en/user-guide/data-unload-considerations.html), will update this chapter later. 

Use case: after a lot of processing in snowflake, you obtain the curated data which can be directly used for analysis. If you want to use this data offline, you can copy it out of snowflake.  






































