# 4. External functions
UDFs that are developed/maintained/stored/executed outside of Snowflake.

Eliminates the need to export and reimport data when using 3rd-party services, significantly simplifying your data pipelines.

## Introduction to External Functions
An external function calls code (remote service) that is executed outside Snowflake.

Information sent to a remote service is usually relayed through a proxy service.

Snowflake stores external function info in an API integration.

client uses external function -> sf sends http POST request (includes JSON data, http header, authentication info) to Gateway -> Gateway sends http POST request to the remote service -> remote service sends HTTP response to Gateway -> Gateway sends http response to sf -> client receives function results.

If the remote service responds with an HTTP code to signal asynchronous processing, then Snowflake sends one or more HTTP GET requests to retrieve the result from the remote service.

Inside sf, the external function is stored as a db object (db_name.sc_name.function_name), which contains url of the proxy service, and name of its API integration. This relays information to/from the remote service.

The remote service must act like a function - it must return a scalar value. To be used by snowflake, it must accept & return JSON, and expose an HTTPS endpoint. 

Examples of a remote service:
- An AWS Lambda function.
- A Microsoft Azure Function.
- An HTTPS server (e.g. Node.js) running on an EC2 instance.

Snowflake does not call a remote service directly - it calls a proxy service, which relays the data to the remote service. The proxy service can increase security, by authenticating requests to the remote service.

Examples of proxy services:
- Amazon API Gateway.
- Microsoft Azure API Management service.

An API integration stores remote service info and authentication info, which are needed to work with a proxy service.

When a query has a large number of rows to send to a remote service, the rows are split into batches, which allow more parallelism and faster queries. A remote service returns 1 batch of rows for each batch received. For a scalar external function, the number of rows in the returned batch is equal to the number of rows in the received batch. Each batch has a unique batch ID, which is included in each request sent from Snowflake to the remote service. Retry operations (e.g. due to timeouts) are done at the batch level.

External functions have these advantages over other UDFs:
- Remote service can be written in languages that other UDFs cannot be written in
- Remote service can use functions and libraries that cannot be accessed by internal UDFs.
- Remote services can be called from Snowflake, and from other software.

Currently, external functions cannot be shared with data consumers via Secure Data Sharing.

The default value for a column cannot be an expression that calls an external function.

Using external functions incurs costs associated with:
- Snowflake warehouse usage.
- Data transfer.

## Data Formats





## Request and Response Translators





## Performance





## Best Practices





## Security













