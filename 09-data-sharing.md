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
### Exploring Listings
### Accessing and Installing Listings as a Consumer
### Paying for Listings
### Review Your Usage of Paid Listings

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


































