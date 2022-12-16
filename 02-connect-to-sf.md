# Connecting to Snowflake
- 3rd-party tools
- CLI
- connectors, drivers

Snowflake limits the size of query text submitted through Snowflake clients to 1 MB per statement, after compression. To load data that exceeds the limit, load from data files. 

Snowflake supports preparation for the following SQL statement types:
- SELECT
- DML Commands
- SHOW \<objects\>

## SnowSQL
SnowSQL is the command line client for connecting to Snowflake to execute SQL queries and perform all DDL and DML operations, including loading data into and unloading data out of database tables.

config file location: `<HOME_DIR>/.snowsql/config`

Configure the behavior of SnowSQL by adding settings in the `[options]` section of the config file:
```
[options]
<option_name> = <option_value>
```
In the [variables] section of the config file, you can store values as variables for reuse. This enables you to use user-defined and database values in queries.




















