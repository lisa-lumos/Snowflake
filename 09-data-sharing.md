# 9. Data Sharing & Collaboration
Share data from your Snowflake account with users in other Snowflake accounts. Data provider can manage access to the data, data consumer could easily join shared data with their own data. 

Sharing options (first 2 support reader accounts):
1. Listing. Can: 
   - share data across region/clouds automatically, with no manual replication work. 
   - provide additional metadata to consumer, & view consumer usage info. 
   - optionally charge for data. 
   - offer data publicly/privately. 
2. Direct share. share data to other accounts within the same sf region. 
3. Data Exchange. share data with a group of accounts that you selected, if private listing is not an option for you. Can see usage metrics. 

## Intro
You can share the following Snowflake database objects:
- Tables
- External tables
- Secure views
- Secure materialized views
- Secure UDFs

All database objects shared to you are read-only. 

With Secure Data Sharing, no actual data is copied. Sharing uses Snowflake's services layer and metadata store. Shared data takes no storage in consumer accounts - they only pay for the compute to query them. 

The provider creates a share of existing database(s) grants access to some objects in the dbs. One or more accounts are then added to the share, which can include your own accounts. On the consumer side, a read-only database is created from the share. Access to this db is controlled by RBAC.

New objects added to a share become immediately available to all consumers. Access to a share, or objects in a share can be revoked at any time.

You can create as many shares as you want, and add as many accounts to a share as you want. If you want to provide a share to many accounts, you can use a listing or a data exchange.

You can consume as many shares as you want from data providers, but you can only create one database per inbound share.

Reader accounts: to share data with a consumer who does not already have a Snowflake account. Reader accounts belong to the provider account that created it. As a provider, you use shares to share databases with reader accounts; a reader account can only consume the shared data - they cannot do DML like data loading.

## Share data via Snowsight

















## Listings and Snowflake marketplace

## Data sharing for providers
### Getting Started

### Working with Shares

### Sharing Data from Multiple Databases

### Replicating Shares Across Regions and Cloud Platforms

### Sharing Data Across Regions and Cloud Platforms

### Using Secure Objects to Control Data Access

## Data sharing for consumers
### Consuming Data Shared with You

## General data sharing tasks
### Managing Reader Accounts
### Configuring Reader Accounts
### Enabling Non-Admins to Perform Sharing Tasks
### Granting Privileges to Other Roles
### Sharing from Business Critical to Non-Business Critical

## Data exchange
### About

### Admin and Membership

### Accessing a Data Exchange

### Becoming a Data Provider

### Managing Data Listings

### Configuring and Using Data Exchanges

### Requesting a New Data Exchange


































