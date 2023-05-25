# 13. Snowpark API
Snowpark libraries for three languages: 
- Java
- Python
- Scala

Snowpark operations are executed lazily on the server, meaning that you can use the library to delay running data transformation until as late in the pipeline as possible while batching up many operations into a single operation. This reduces the amount of data transferred between your client and the Snowflake database. It also improves performance.

You can create UDFs inline in a Snowpark app. Snowpark can push your code to the server, where the code can operate on the data at scale. Snowpark automatically pushes the custom code for UDFs to the Snowflake database. When you call the UDF in your client code, your custom code is executed on the server (where the data is). You don't need to transfer the data to your client in order to execute the function on the data.





























