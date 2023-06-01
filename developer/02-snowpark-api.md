# 2. Snowpark API

## Intro
Snowflake currently provides Snowpark libraries for three languages: Java, Python, and Scala.

Compared with Snowflake Connector for Spark, Snowpark supports:
- Interacting  with Snowflake data using libraries/patterns built for different languages, without compromising on performance/functionality.
- Running Snowpark code using local tools such as Jupyter, VS Code, or IntelliJ.
- Pushdown for all operations, including Snowflake UDFs. This means Snowpark pushes down all heavy data transformation to Snowflake, enabling you to efficiently work with data of any size.
- All of the computations are done within Snowflake. 

The Snowpark API provides `programming language constructs` for building SQL statements. For example, the API provides a select method that you can use to specify the column names to return, rather than writing 'select column_name' as a string. Although you can still use a string to specify the SQL statement to execute, you benefit from features like intelligent code completion, and type checking, when you use the native language constructs provided by Snowpark.

Snowpark operations are executed `lazily` on the server, meaning that you can use the library to delay running data transformation, until as late as possible, while batching up many operations into a single operation. This reduces the amount of data transferred between your client and the Snowflake database. It also improves performance. The data isn't retrieved when you construct the DataFrame object. Instead, when you are ready to retrieve the data, you can perform an action, such as `df.collect()`, that evaluates the DataFrame objects, and sends the corresponding SQL statements to the Snowflake database for execution.

You can create UDFs inline in a Snowpark app. Snowpark can push your code to the server, where the code can operate on the data at scale.

## Java
(skipped)

## Python
You can write Snowpark Python code in:
- A local development environment, for client app
- A Python worksheet in Snowsight, for writing SPs

Snowpark Python can: 
- Query/process data with a DataFrame object
- Convert custom lambdas/functions to UDFs, that you can call to process data
- Write a UDTF, that processes data, and returns data
- Write a SP to process data, or automate tasks
- Perform machine learning tasks like training models with SP, and deploy them with udfs. 

The Snowpark API requires Python 3.8.

The Snowpark API provides methods for writing data to/from Pandas DataFrames.

Install the Snowpark Python package into the Python 3.8 venv by `pip install snowflake-snowpark-python`.

By writing code in Python worksheets, you can perform your development/testing in Snowflake, without needing to install dependent libraries.

## Scala
(skipped)

