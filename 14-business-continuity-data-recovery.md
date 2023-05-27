# 14. Business continuity & Data recovery
A replication group allows customers to specify what to replicate, where to replicate to, and how often. This means specifying which objects to replicate, to which regions or cloud platforms, at customizable scheduled intervals. 

A failover group enables the replication and failover of the account objects in a group.

Account migration is the one-time process of migrating (or transferring) the Snowflake objects and your stored data to an account in another region or on a different cloud platform. Typical reasons for migrating your account include a closer proximity to your user base, or a preference for a different cloud platform based on your corporate strategy or co-location with other cloud assets (e.g. a data lake).

## Replication
Available to all account editions. `Database & share replication are available on all editions. Replication of all other objects are only available for Business Critical Edition (or higher)`. 

Replication of objects from a source account to one or more target accounts, in the same organization.

With asynchronous replication, secondary replicas typically lag behind the primary objects based on the replication schedule you configure. 

### Introduction
Replicated account level objects:
- databases
- integrations
- network policies
- parameters (account level)
- resource monitors
- roles
- shares
- users
- warehouses

Replicated database objects:
- tables (permanent/transient) and table constraints 
- sequences
- views (regular/materialized/secure)
- file formats
- SPs
- streams
- tasks
- UDFs
- policies (masking/row-access/tag-based-masking/session/password)
- tags

NOT replicated:
- temporary tables
- external tables
- Event tables
- stages (will be in the future)
- pipes (will be in the future)

### Considerations
skipped

### Configuration
skipped

### Security Integrations and Network Policy Replication
skipped

### Understanding Cost
Charges based on replication are divided into two categories: data transfer and compute resources - all are billed on the target account. 

The data transfer rate is determined by the location of the source account. Replication operations use Snowflake-provided compute resources to copy data between accounts across regions.

In general, monthly billing for replication is proportional to:
- amount of data got replicated
- frequency of replication refresh

## Failover (Business Critical &+)
### Account Failover
skipped. 

## Client redirect
### Overview
Client Redirect provides a connection URL that can be used by Snowflake clients to connect to a different Snowflake account. `organization_name-connection_name.snowflakecomputing.com`

## Data recovery
### Time Travel
Snowflake Time Travel enables accessing historical data at any point within a defined period. It can be used for:
- restore data-related objects
- backing up data
- analyze data usage

Using Time Travel, you can also:
- Query data in the past.
- Create clones of objects at or before specific points in the past.

Once the defined period of time has elapsed, the data is moved into Snowflake Fail-safe, and these actions can no longer be performed.

The standard data retention period is 1 day, and is automatically enabled for all Snowflake accounts. 

For Snowflake Standard Edition, the retention period can be set to 0 or 1, at the account and object level (databases/schemas/tables).

For Snowflake Enterprise Edition (and higher): For transient/temporary databases/schemas/tables, the retention period can be set to 0 or 1. 

For permanent databases/schemas/tables, the retention period can be set to any value from 0 - 90 days.

A retention period of 0 days for an object effectively disables Time Travel for the object.

- DATA_RETENTION_TIME_IN_DAYS can be used for account/db/schema/table level
- MIN_DATA_RETENTION_TIME_IN_DAYS can be set for account level, takes precedence over other settings for an object. 

Changing the retention period for your account or individual objects changes the value for all lower-level objects that do not have a retention period explicitly set.

Do not recommend changing the retention period to 0 at the account level.

Currently, when a db/schema is dropped, the data retention period for its child objects, if explicitly set to be different from the retention of the database, is not honored. The child schemas or tables are retained for the same period of time as the database. To honor the data retention period for these child objects, drop them explicitly before you drop the database or schema.

### Fail-safe
Fail-safe ensures historical data is protected, in the event of a system failure or other event (e.g. a security breach).

Fail-safe provides a non-configurable 7-day period, during which historical data may be recoverable by Snowflake. This period starts immediately after the Time Travel retention period ends. 

Fail-safe is a data recovery service that is provided on a best effort basis, and is intended only for use when all other recovery options have been attempted.

Fail-safe is not for accessing historical data after the Time Travel retention period has ended. It is only for Snowflake to recover data that are lost/damaged due to extreme operational failures.

Data recovery through Fail-safe may take from several hours to several days to complete.

In the web interface, account administrators can view the total data storage for their account, including historical data in Fail-safe.

### Storage Costs for Time Travel and Fail-safe
Storage fees are incurred for maintaining historical data, during both the Time Travel and Fail-safe periods.

Snowflake minimizes the amount of storage required for historical data, by maintaining only the info required to restore the individual table rows that were updated/deleted. As a result, storage usage is calculated as a % of the table that changed. 

Full copies of tables are only maintained when tables are dropped/truncated.

Transient/temporary tables have no Fail-safe period.
