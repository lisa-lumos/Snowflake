# 9. Integrations
## create api integration
An external function calls code that is executed outside Snowflake. The remotely executed code is known as a remote service. Information sent to a remote service is usually relayed through a proxy service. Snowflake stores security-related external function information in an API integration.

An API integration object stores info about an HTTPS proxy service:
- The cloud platform provider (e.g. Amazon AWS).
- The type of proxy service.
- The identifier and access credentials for a cloud platform role that has sufficient privileges to use the proxy service. 
- specifies allowed/blocked endpoints and resources on those proxy services.

Only the ACCOUNTADMIN role can create api integration by default. The privilege can be granted to additional roles as needed.

You can create more than one instance of an HTTPS proxy service in a cloud provider account, and you can use the same API integration to authenticate to multiple proxy services in that account.


```sql
create or replace api integration 
  demonstration_external_api_integration_01
api_provider=aws_api_gateway
api_aws_role_arn='arn:aws:iam::123456789012:role/my_cloud_account_role'
api_allowed_prefixes=('https://xyz.execute-api.us-west-2.amazonaws.com/production')
enabled=true;

create or replace external function local_echo(string_col VARCHAR)
returns variant
api_integration = demonstration_external_api_integration_01
as 'https://xyz.execute-api.us-west-2.amazonaws.com/production/remote_echo';
```

## create external function
An external function is a type of UDF. Unlike other UDFs, an external function does not contain its own code; instead, the external function calls code that is stored and executed outside Snowflake.

The external function can be made secure. 

Snowflake supports scalar external functions; the remote service must return exactly one row for each row received.

To be called by the Snowflake external function, the remote service must:
- Accept JSON inputs, and return JSON outputs. 
- Expose an HTTPS endpoint.

Currently, external functions cannot be shared with data consumers

Using external functions incurs normal costs associated with:
- Snowflake warehouse usage.
- Data transfer.

Data sent via Amazon API Gateway Private Endpoints incurs AWS PrivateLink charges for both ingress and egress.

Example see above. 


























