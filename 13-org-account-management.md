# 13. Org & Account Management
Managing Snowflake organizations and accounts.

## Organizations
### Introduction
An organization is a first-class Snowflake object that links the accounts owned by your business entity. Organizations simplify account management and billing, Replication and Failover/Failback, Snowflake Secure Data Sharing, and other account administration tasks.

ORGADMIN can:
- Create an account in the organization. 
- View/show all accounts within the organization. 
- View/show a list of regions enabled for the organization. 
- View usage information for all accounts in the organization. 
- Enable database replication for an account in the organization.

### Getting Started
skipped

### Managing Accounts within Your Organization
skipped

### Connecting to Your Accounts
Snowflake supports multiple URL formats when connecting to a Snowflake account without a browser: 
- The account name format uses the name of the account and its organization to identify the account. `https://<orgname>-<account_name>.snowflakecomputing.com`
- The connection name format, which replaces the account name with the name of a connection, is required when using the Client Redirect feature. `https://<orgname>-<connectionname>.snowflakecomputing.com`
- The legacy account locator format is currently supported, but its use is discouraged. `https://<accountlocator>.<region>.<cloud>.snowflakecomputing.com`

## Accounts
### Account Identifiers
An account identifier uniquely identifies a Snowflake account within your organization, as well as throughout the global network of Snowflake-supported cloud platforms and cloud regions.

### Trial Accounts
skipped

### Parameter Management
All parameters have default values, which can be overridden at the account level. In addition, the default values for session and object parameters can be overridden at each level in the parameter hierarchy.  

### User Management
During the initial user creation, it is possible to set a weak password for the user that does not meet the minimum requirements described below (e.g. 'test12345'). This allows administrators to use generic passwords for the user during the creation process. If this pathway is chosen, Snowflake strongly recommends setting the MUST_CHANGE_PASSWORD property to TRUE to require users to change their password on their next login, including the initial login, to Snowflake.

Snowflake enforces the following password policy as a minimum requirement while using the ALTER USER command and the web interface:
- Must be at least 8 characters long.
- Must contain at least 1 digit.
- Must contain at least 1 uppercase letter and 1 lowercase letter.

### Behavior Change Release Management
Snowflake implements behavior changes monthly in bundles in regularly-scheduled releases. During the testing period and opt-out period for each behavior change bundle, you can enable/disable it. 

You can check whether a particular bundle is enabled in your account, and enable/disable it.

