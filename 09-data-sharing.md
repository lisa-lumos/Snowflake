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
Using Snowsight, you can see data that was shared to you
- Privately shared listings, including data exchange listings
- Snowflake marketplace
- Direct shares

Then, you can create a database from a share, or filter the shares. 

Using Snowsight, you can also see data that was shared by your account, using a listing, a direct share, or as part of a data exchange.

Using Snowsight, you can see inbound/outbound requests from a data exchange.

If you have the Data Exchange Admin role, or Provider Profile Level Privileges, you can create and manage provider profiles for a data exchange. 

## Listings 
A listing is a way of Secure Data Sharing and uses the same provider and consumer model. As a provider, you can use a listing to share data in your Snowflake account with other Snowflake accounts privately, or by offering a listing on the Snowflake Marketplace. As a consumer, you can use a listing to access data shared by other Snowflake accounts on the Snowflake Marketplace, or privately with your account. 

Listings add capabilities to Secure Data Sharing:
- Share publicly on the Snowflake Marketplace.
- Charge consumers for the shared data.
- Monitor interest in your listing and usage of the shared data.
- Provide metadata about the share, such as a title, description, sample SQL queries, and info about the data provider.

When you create a listing, you choose one way to offer it:
- publicly on the Snowflake Marketplace (free/paid/personalized)
- privately to select customers (free/paid)

## Snowflake marketplace
Where you can explore/access 3rd party listings, and provide listings to consumers. Available globally to all non-VPS Snowflake accounts. 

As a provider, you can:
- Publish listings for free datasets to generate interest/opportunities.
- Publish listings for customizable datasets.
- Share live datasets in real-time.
- Avoid the costs of building/maintaining APIs/data pipelines to deliver data.

As a consumer, you can:
- Discover/test 3rd-party data sources.
- Easy access to raw data products from vendors.
- Combine new & existing data to derive new business insights.
- Have data available instantly and updated continually for users.
- Avoid the costs of building/maintaining APIs/data pipelines to load/update data.
- Use BI tools of your choice.

## Legal Requirements for Providers/Consumers of Listings
To use listings as a provider/consumer, Snowflake customers must agree to additional terms of service and abide by Snowflake policies. 

## Using listings as a provider
### Becoming a Provider of Listings (in preview)
Requirements to Become a Provider, you must:
- have a full Snowflake account. Reader accounts are not supported.
- have the ACCOUNTADMIN role, or have a role with provider privileges. 
- meet the Legal Requirements for Providers and Consumers of Listings. 

To offer specific types of listings, you must:
- create a provider profile & have it approved by Snowflake to offer listings on the Snowflake Marketplace
- you must set up a Stripe account to offer paid listings

### Accessing Provider Studio (in preview)
Snowsight -> Data -> Provider Studio

### Managing Your Provider Profile
read, skipped. 

### Preparing Data for a Listing
- To create/manage a list, you need ACCOUNTADMIN role, or a role with the global CREATE DATA EXCHANGE LISTING privilege.

A share's owner can attach it to a listing. They also need MODIFY LISTING privilege on that listing. 

You can only share protected health information (PHI) through a personalized listing, and both need to sign an agreement with Snowflake. 

while you can share personal data through free/personalized listing, you must have the applicable legal rights if the data is not publicly available.

You can add shares that are already shared with a consumer account, such as with a direct share, to a listing.

Not all countries can create paid listings. Your country is the one on your billing address. 

When you offer a paid listing on the Snowflake Marketplace, you must offer consumers the ability to trial the listing before they purchase it. Trials are optional for paid private listings. Use SYSTEM$IS_LISTING_PURCHASED function to control which data is visible to trial/paid consumers. 

Offering listings in other regions in Marketplace requires replicating data, you need to manually replicate the data or use auto-fulfillment.

For a private listing, you need to share your listing to the regions where your consumer's accounts are. Use auto-fulfillment to replicate. 

### Creating and Publishing a Listing
read, skipped. 

### Configuring Cross-Cloud Auto-fulfillment (in preview)
To automatically replicate the data product associated with your listing to other Snowflake regions.

### Managing Auto-fulfillment Costs
compute + storage + data transfer
### Configuring Listings
The fields that you can configure for a listing

### Paid Listings Pricing Models
Pricing Model Components: 
- Per-month charge: fixed fee per month
- Per-query charge: fixed fee per query, on top of per-month charge
- Maximum total charge per month: max monthly spend
- Number of free queries: num of free queries after the first query is run

### Managing Listings as a Provider
- Snowflake Marketplace paid listing, and private listings use cross-cloud auto-fulfillment. They are available to consumers immediately.
- Snowflake Marketplace free listings can use cross-cloud auto-fulfillment (available to consumers immediately), or be manually replicated (consumers must request these listings). 
- Personalized listings require manual data replication (consumers must request these listings).

### Modifying Published Listings
skipped

### Defining Listing Referral Links
You can send consumers a link to your Snowflake Marketplace listing. 

### Removing Listings as a Provider
When you delete a listing, you permanently remove the listing. A deleted listing cannot be recovered/republished. 

### Monitoring Usage of Your Listing
Use Provider Studio. 

Snowflake tracks many metrics for listings:
- Daily usage, such as the daily consumer query history.
- Consumers get/request events.
- Use of your listing, such as num of jobs run on the data.
- Access details, such as viewing the tables in your listing.
- The consumer account & org name, and consumers submitted info when requesting a personalized listing.
- ...

### Monetization Usage Views
Paid listings' historical usage data:
- MARKETPLACE_DISBURSEMENT_REPORT View
- MONETIZED_USAGE_DAILY View
- LISTING_EVENTS_DAILY View

## Using listings as a consumer
### Becoming a Consumer of Listings
To access private listings or Snowflake Marketplace listings. 

The ORGADMIN needs to accept the Snowflake Consumer Terms of Service for your org.

To access a listing, you must be ACCOUNTADMIN or a role with the CREATE DATABASE and IMPORT SHARE privileges.

The role with IMPORTED PRIVILEGES on the database created from the listing can use it. 

### Exploring Listings
Data dictionaries allow you to preview the contents of a free/paid Marketplace listing before installing it in your account. 

### Accessing and Installing Listings as a Consumer
skipped

### Paying for Listings
skipped

### Review Your Usage of Paid Listings
- MARKETPLACE_PAID_USAGE_DAILY View
- MARKETPLACE_PAID_USAGE_DAILY View

## Data sharing for providers
### Getting Started
How to create a share. You can provide a share to consumers using Direct Shares, Listings, or Data Exchanges.

Must need account admin role. 

Two ways to share database objects:
1. Grant "database roles" to a share. (In preview)
   - create many "database roles" in a database.
   - grant privileges on different of objects to each db role, then grant all these db roles to the share
   - consumers can grant these db roles to their own roles in their account
   - db roles live in one db
2. Grant privileges on objects directly to a share. 
   - consumer get a single privilege - IMPORTED PRIVILEGES. This allow their users to access all dbs and db objects in the share. 
   - on consumer side, do not allow different user groups to access a subset of the shared objects - All or nothing. 
   - can include objects from many dbs in a single share.

You can blend both options. 

Notify any data consumers if the name of the database role was changed.

### Working with Shares
VPS does not support Secure Data Sharing due to the current limitations against sharing data across regions.

New and modified rows in tables in a share are available immediately to consumers.

Create a separate schema for each table you wish to share.

A new object created in a database is not automatically available to consumers - you need to explicitly grant it to the share. 

If you decide to filter the data in table(s) based on certain conditions, you need to create secure views on the table(s).

SIMULATED_DATA_SHARING_CONSUMER session parameter can be used to validate whether objects are configured correctly to display only what you want to. 

Consumers can create streams in their own dbs on the shared tables/views. For consumers to create streams on shared tables/views, provider must enable change tracking on the shared tables or the underlying tables for a view.

You remove an account from a share by setting a new list of accounts for it and leaving the desired account off the list. After removing an account from a share, you can add it back again to the share; however, they lose the database they created earlier from the share. They must create a new database from the share. This can have a significant impact on the business operations of the account. Similar thing happens if you drop a share. 

### Sharing Data from Multiple Databases
Providers can share data that in different databases by using secure views. A secure view can reference objects such as schemas/tables/views from other databases.

In addition to performing all the standard steps to share data, you must grant the REFERENCE_USAGE privilege on each database referenced by the secure view you want to share.

### Replicating Shares Across Regions and Cloud Platforms (using account replication)
Account replication enables the objects replication from a source account to many target accounts in the same organization. Replicated objects are secondary objects and are replicas of the primary objects.

A replication group is a collection of source objects that are replicated as a unit to many target accounts. Replication groups provide read-only access for the replicated objects. Replication groups provide point-in-time consistency for the objects.

This is enabled by the Account Replication feature. The ORGADMIN role must enable replication for accounts in your organization. 

When a secondary replication group is created in the target account, an initial refresh is automatically executed.

In the target account, people who executes the `alter replication group ... refresh` command need a role with the REPLICATE privilege on the replication group. 

Recommend to schedule automatic refreshes using the REPLICATION_SCHEDULE parameter. 

### Sharing Data Across Regions and Cloud Platforms (using database replication)
Database replication is now a part of Account Replication. 

When sharing a view that references objects in many databases, each of these other databases must be replicated.

skipped. 

### Using Secure Objects to Control Data Access
Strongly recommend sharing secure views/UDFs instead of sharing tables.

For optimal performance, especially with extremely large tables, recommend defining clustering keys on the base table(s).

You can create a share with one of them:
- the ACCOUNTADMIN role
- OWNERSHIP on the shared database
- USAGE privilege on the database WITH GRANT OPTION. 

## Data sharing for consumers
### Consuming Data Shared with You
You must use the ACCOUNTADMIN role, or a role granted the IMPORT SHARE global privilege.

Limitations for consumers:
- Shared databases are read-only. Users can view/query data, but cannot insert/update data, nor create any objects in the database.
- Not allowed: 
  - Creating a clone of a shared object.
  - Time Travel for a shared object.
  - Editing the comments for a shared database.
- Shared objects on consumer side cannot be re-shared with other accounts.
- Shared databases cannot be replicated.

## General data sharing tasks
### Managing Reader Accounts
Reader accounts enable providers to share data with consumers who are not Snowflake customers and no not prefer to be. 

Provider need ACCOUNTADMIN role, or the CREATE ACCOUNT global privilege to proceed. 

The reader account is created/owned/managed/paid by the provider account. 

Recommend to setup a resource monitor for the warehouse.

In a reader account, you cannot:
- Upload/modify data. (insert/update/delete/merge/copy into table)
- Unload data using a storage integration. You can use the COPY INTO loc to unload into cloud storage. 
- create masking policy/pipe/row access policy/share/stage
- show procedures

You can do anything else, including creating materialized views. 

The reader account name(locator) is generated by Snowflake during account creation. 

The Snowflake Edition & region of reader account are the same as the provider's.

By default, you can create 20 reader accounts. If you reach the limit and need more, contact Snowflake Support.

### Configuring Reader Accounts
A newly-created reader account contains one user, and is the admin for the account. This admin must create additional objects in the account, including 
- custom roles (if needed)
- users, and grant roles to them
- virtual warehouses, and grant to your roles
- resource monitors
- one/more databases from the share.

Each reader account comes with the standard system-defined roles (SYSADMIN, SECURITYADMIN, PUBLIC). You can create additional custom roles.

Depends on how the share is created by provider, you can use grant database roles to your roles, or, grant imported privileges directly to your roles. 

The easiest way to invite users is to use the ALTER USER to reset the password for each user. This generates a unique URL for each user, which you then send/give to them. They use the URL to change their password and log into the account.

Note that operating on any object in a schema also requires the USAGE privilege on both the parent database and schema.

### Enabling Non-Admins to Perform Sharing Tasks
Ownership of a share, and the objects in it, can be obtained via a direct grant to the role, or inherited from a lower-level role in the role hierarchy.

### Granting Privileges to Other Roles
By default, only ACCOUNTADMIN in the Data Exchange admin account can manage a Data Exchange with these tasks:
- Add/remove members
- Approve/deny listing approval requests
- Approve/deny provider profile approval requests
- Show categories
To delegate these tasks to other users, use IMPORTED PRIVILEGES. 

### Sharing from Business Critical to Non-Business Critical
By default, Snowflake does not allow sharing data from a Business Critical to a non-Business Critical account. Snowflake provides the OVERRIDE SHARE RESTRICTIONS global privilege to the ACCOUNTADMIN role. When the parameter is disabled, a Business Critical provider account can add a regular consumer account to a share.

## Data exchange
### About

### Admin and Membership

### Accessing a Data Exchange

### Becoming a Data Provider

### Managing Data Listings

### Configuring and Using Data Exchanges

### Requesting a New Data Exchange


































