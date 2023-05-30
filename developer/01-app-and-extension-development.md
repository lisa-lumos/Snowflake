# 1. App and Extension Development
You can write apps that extend Snowflake, act as a client/integrating-component.

## Query/Process Data with the Snowpark API
Using Snowpark APIs in Java/Python/Scala, you can build apps that process data in Snowflake, without moving data to the system where your application code runs.

With Snowpark, you can:
- Create apps/pipelines/processing-logic with Java/Python/Scala.
- Write data statements using libraries, which integrate natively with supported languages.
- During data processing, reduce the overhead of transferring data to/from Snowflake.
- Create UDFs with Java/Python/Scala. Snowflake generates the corresponding SQL code behind the scenes.

## Extend Snowflake with Procedures, UDFs, and External Functions
With a procedure, you can perform scheduled/on-demand operations, by executing code/SQL-statements. 

With a UDF, you can run logic to calculate/return data. UDFs are useful for batch processing, and integrating custom logic into SQL. 

With an external function, you can set up an integration between Snowflake and custom code running outside of Snowflake, then use the custom code like a UDF.

## Build Client App with Drivers/APIs
You can integrate Snowflake operations into a client app, using - Snowpark API
- Language/platform-specific drivers

### Drivers
Connect from your code/apps to Snowflake and perform operations, using languages such as C#, Go, and Python, etc. 

### RESTful API
Using the Snowflake RESTful SQL API, you can access/update data over HTTPS/REST. You can submit SQL statements, create/execute SPs, provision users, ...

In the SQL REST API, you submit a SQL statement inside a POST request. You then use GET requests to check execution status, and fetch results.

## Integrate with Other Systems
- Snowflake connector for Kafka
- Snowflake connector for Spark
- ...

## Supported languages
- Java: Snowpark, SP, UDF, JDBC driver, integrate with Spark
- JavaScript: SP, UDF, Node.js driver
- PHP: PHP PDO driver
- Python: Snowpark, SP, UDF, Python connector, integrate with spark
- Scala: Snowpark, SP, UDF, JDBC driver, integrate with Spark
- Snowflake Scripting: SP
- SQL: UDF
