# 1. Introduction to Snowflake

## Key Concepts & Architecture
Snowflake is a `Saas` data platform that provides all of the functionality of an `enterprise analytic database`, along with many additional special features and unique capabilities.

Snowflake is a true SaaS offering. More specifically:
- There is no hardware to select, install, configure, or manage.
- There is no software to install, configure, or manage.
- Ongoing maintenance, management, upgrades, and tuning are handled by Snowflake.

Snowflake’s architecture is a `hybrid` of both:
- `traditional shared-disk architecture` - central data repository for persisted data that is accessible from all compute nodes; Have data management simplicity.  
- `shared-nothing database architecture` - processes queries using massively parallel processing (MPP) compute clusters where each node stores a portion of the entire data set locally. Have performance and scale-out benefits. 

Snowflake’s unique architecture consists of `three key layers`:
- Database Storage - internal optimized, compressed, columnar format. Snowflake manages all aspects of how this data is stored. 
- Query Processing - performs query execution. Snowflake processes queries using “virtual warehouses” (MPP compute cluster composed of multiple compute nodes allocated by Snowflake from a cloud provider)
- Cloud Services - a collection of services that coordinate activities across Snowflake. Also runs on compute instances provisioned by Snowflake from the cloud provider. (Authentication, Infrastructure management, Metadata management, Query parsing and optimization, Access control)

Snowflake supports multiple `ways of connecting to the service`:
- A web-based UI.
- Command line clients (e.g. SnowSQL).
- ODBC and JDBC drivers that can be used by other applications (e.g. Tableau) to access SF.
- Native connectors (e.g. Python, Spark) that can be used to develop applications for connecting to Snowflake.
- Third-party connectors that can be used to connect applications such as ETL tools (e.g. Informatica) and BI tools (e.g. ThoughtSpot) to Snowflake.

## Supported Cloud Platforms
A Snowflake account can be hosted on any of the following `cloud platforms`:
- Amazon Web Services (AWS)
- Google Cloud Platform (GCP)
- Microsoft Azure (Azure)

Pricing: Differences in unit costs for credits and data storage are calculated by region on each cloud platform. 

Data Loading: Snowflake supports loading data from files staged in Internal stages, Amazon S3, Azure blob, Google Cloud Storage, regardless of the cloud platform for your Snowflake account. Supports both `bulk data loading` and `continuous data loading (Snowpipe)`. Supports unloading data from tables into any of the above staging locations.

HITRUST CSF Certification: Enhances Snowflake’s security posture in regulatory compliance and risk management. Applicable to Snowflake editions that are Business Critical (or higher). 

## Supported Cloud Regions
Each Snowflake account is hosted in a single region. If you wish to use Snowflake across multiple regions, you must maintain a Snowflake account in each of the desired regions. There are `differences` in unit costs for credits and data storage between regions.

If `latency` is a concern, you should choose the available region with the closest geographic proximity to your end users; however, this may have cost implications.

If you are a government agency or a commercial organization that must comply with specific privacy and security requirements of the US government, you can choose between two `dedicated government regions` provided by Snowflake.

Snowflake supports two formats to use as the account identifier in your hostname:
- Account name (preferred)
- Account locator

## Snowflake Editions
Each successive edition builds on the previous edition, adding edition-specific features and/or higher levels of service. Changing editions is easy.

The Snowflake Edition that your organization chooses affects the unit costs for credits and storage. 

Features for different editions: (`https://docs.snowflake.com/en/user-guide/intro-editions.html`)

## Snowflake Releases
The new release deployments happen weekly and transparently in the background.

## Overview of Key Features
`https://docs.snowflake.com/en/user-guide/intro-supported-features.html`

## Overview of the Data Lifecycle






















