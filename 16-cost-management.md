# 16. Cost management
Effective Snowflake cost management is divided into three parts: visibility, control, and optimization.

3 types of compute resources that consume credits:
- Virtual Warehouse Compute
- Serverless Compute
- Cloud Services Compute (only charge for the credits that exceeds 10% of the daily warehouse usage credits.)

Storage Resources: The monthly cost for storing data in Snowflake is based on a flat rate per TB. Storage is calculated monthly based on the average number of on-disk bytes stored each day in your Snowflake account.

Snowflake does not charge data ingress fees to bring data into your account, but does charge for data egress. Snowflake charges a per-byte fee when you transfer data from a Snowflake account into a different region on the same cloud platform or into a completely different cloud platform. 

### Understanding Compute Cost
Warehouses are only billed for credit usage while running. When a warehouse is suspended, it does not use any credits. credits are billed per-second, with a 1-minute minimum. 

Charges for serverless features are calculated based on total usage of snowflake-managed compute resources measured in compute-hours. Compute-Hours are calculated on a per second basis, rounded up to the nearest whole second. The number of credits consumed per compute hour varies depending on the serverless feature. 

Serverless Features: 
- Automatic Clustering
- external table metadata automated refreshing
- Materialized Views
- Query Acceleration Service
- Replication
- Search Optimization Service - Automated background maintenance of the search access paths
- Snowpipe
- Snowpipe Streaming
- Tasks - Execution of SQL code

### Understanding Storage Cost
Storage cost contains:
- Files staged for bulk data loading/unloading
- Database tables, including those in Time Travel & Fail-safe
- Clones of database tables that reference data deleted in the table that owns the clones.

The monthly costs for storing data in Snowflake is based on a flat rate per TB.

In addition, clones can be cloned, with no limitations on the number or iterations of clones that can be created. 

### Understanding Data Transfer Cost
Snowflake Features that Incur Transfer Costs:
- Unloading data - Unloading data from Snowflake to Amazon, Google Cloud Storage, or Microsoft Azure.
- Replicating Data
- Writing External Functions - Use of external functions to transfer data from your Snowflake account to AWS, Microsoft Azure, or Google Public cloud. 

## Exploring Cost
```sql
select -- total usage costs for the organization, broken down by account
  account_name,
  round(sum(usage_in_currency), 2) as usage_in_currency
from snowflake.organization_usage.usage_in_currency_daily
where usage_date > dateadd(month,-1,current_timestamp())
group by 1
order by 2 desc;
```
### Exploring Compute Cost
WAREHOUSE_METERING represents credits consumed by warehouses while PIPE represents credits consumed by the serverless Snowpipe feature.

### Exploring Storage Cost
skipped

### Exploring Data Transfer Cost
skipped

## Monitoring Cost
A resource monitor allows you to monitor credit usage by user-managed virtual warehouses and the cloud services layer of the Snowflake architecture.

### Working with Resource Monitors
A resource monitor is a first-class object in Snowflake, consisting of:
- Credit Quota: num of credits allocated to the monitor, by all warehouses assigned to the monitor, for the specified frequency interval. Accounts for credits consumed by both `user-managed virtual warehouses` and `virtual warehouses used by cloud services`. 
- Monitor Level: whether monitor the credit usage for your entire Account, or a specific set of individual warehouses. If not set, then it monitors nothing. 
- Schedule: When to set credits to 0. Frequency: daily/weekly/monthly/yearly/never, at the start of that day. Start: immediately/later. End: timestamp to suspend warehouses and this resource monitor (not commonly used).  If you specify a frequency, you must also specify a start datetime. 
- Actions: action to perform at a certain threshold. (A resource monitor must have at least one action defined)
  - Notify & Suspend
  - Notify & Suspend Immediately
  - Notify

A single monitor can be set at the account level to control credit usage for all warehouses in your account. In addition, a monitor can be assigned to one or more warehouses, thereby controlling the credit usage for each assigned warehouse. Note, however, that `a warehouse can be assigned to only a single resource monitor below the account level`.

An account-level resource monitor does not control credit usage by the Snowflake-provided Compute resources for serverless features (for example, Snowpipe, automatic reclustering, and materialized views).

If a monitor has a Suspend or Suspend Immediately action defined and its used credits reach the threshold for the action, any warehouses assigned to the monitor are suspended and cannot be resumed until one of the following conditions is met:
- The next interval starts, as dictated by the start date for the monitor.
- The credit quota for the monitor is increased.
- The credit threshold for the suspend action is increased.
- The warehouses are no longer assigned to the monitor.
- The monitor is dropped.

Notifications can be received by account administrators through the Classic Console and/or email; however, by default, notifications are not enabled:
- To receive notifications, each account administrator must explicitly enable notifications through their preferences in the Classic Console. See Enabling Receipt of Notifications for Account Administrators.
- In addition, if an account administrator chooses to receive email notifications, they must provide a valid email address (and verify the address) before they will receive any emails.

By default, resource monitors can only be created by account administrators and, therefore, can only be viewed and maintained by them. However, roles that have been granted the following privileges on specific resource monitors can view and modify the resource monitor as needed using SQL:
- MONITOR
- MODIFY

You must assign at least one warehouse to a resource monitor, or set the monitor at the account level, for it to begin monitoring/tracking credit usage. 

default schedule: starts monitoring immediately and resets on the first day of each calendar month. 

```sql
use role accountadmin;
create or replace resource monitor limit1 
with 
  credit_quota=1000
  triggers on 100 percent do suspend; -- allow running queries to continue
alter warehouse wh1 set resource_monitor = limit1;

use role accountadmin;
create or replace resource monitor limit1 
with 
  credit_quota=1000
  triggers 
    on 90 percent do suspend -- allow running queries to continue, suspend other warehouses
    on 100 percent do suspend_immediate; -- abort running queries
alter warehouse wh1 set resource_monitor = limit1;

use role accountadmin;
create or replace resource monitor limit1 
with 
  credit_quota=1000
  triggers 
    on 50 percent do notify
    on 75 percent do notify
    on 100 percent do suspend
    on 110 percent do suspend_immediate;
alter warehouse wh1 set resource_monitor = limit1;

use role accountadmin;
create or replace resource monitor limit1 
with 
  credit_quota=1000
  frequency = monthly
  start_timestamp = immediately
  triggers on 100 percent do suspend;
alter warehouse wh1 set resource_monitor = limit1;

use role accountadmin;
create or replace resource monitor limit1 
with 
  credit_quota=2000
  frequency = weekly
  start_timestamp = '2019-03-04 00:00 pst'
  triggers 
    on 80 percent do suspend
    on 100 percent do suspend_immediate;
alter warehouse wh1 set resource_monitor = limit1;
alter warehouse wh2 set resource_monitor = limit1;

alter resource monitor limit1 set credit_quota=3000;

-- set an account level resource monitor
use role accountadmin;
create resource monitor accountmax 
with 
  credit_quota=10000
  triggers on 100 percent do suspend;
alter account set resource_monitor = accountmax;
```

To view whether a resource monitor is set for your account, use the web interface or the SHOW RESOURCE MONITORS command. The `LEVEL` column for a resource monitor displays whether it is set for your account or individual warehouses.

A warehouse-level resource monitor can monitor, but cannot suspend, credit usage by Cloud Services.

## Attributing Cost
Snowflake provides the following attribution strategies:
- Object tagging: can provide granular attribution that allows you to assign the cost of using individual resources like a warehouse or database to a specific unit within the organization.
- Executing queries: can attribute warehouse usage by role, user, or query, which is particularly helpful when multiple cost centers share the same warehouse.

Using tags: an administrator creates a tag (e.g. cost_center), then defines a list of possible values of the tag (e.g. sales, finance). In this example, each cost center gets a unique tag value. The tag/value combination is then assigned to resources used by a cost center. As these resources consume credits, you can run reports grouped by the tag value. Because this tag value corresponds directly to a particular grouping within the organization, costs can be attributed accurately.

## Controlling Cost
Carefully defining who can work with warehouses and what they can do with those warehouses helps control cost. 

Centralizing the responsibility of creating and scaling warehouses to just a few members of your team is considered a best practice. You can create a dedicated role with permissions to create and modify all warehouses, and then grant that role to a limited number of users. This allows you to control your warehouse policies and prevent accidental cost overruns resulting from warehouses being created or upsized unexpectedly.

To avoid the excess cost associated with a runaway query, you can set the STATEMENT_TIMEOUT_IN_SECONDS parameter to define the maximum amount of time a SQL statement can run before it is cancelled. The STATEMENT_TIMEOUT_IN_SECONDS parameter can be set for an entire account, a user, a session, or a specific warehouse so that you can carefully set time limits that match the expected run times for various workloads. 

The parameter that controls the amount of time that a SQL statement stays in the queue is STATEMENT_QUEUED_TIMEOUT_IN_SECONDS. This parameter can be set for an entire account, a user, a session, or a specific warehouse. This parameter is set at the account level by default.

