# 11. Security
These topics are intended primarily for admins (ACCOUNTADMIN, SYSADMIN, or SECURITYADMIN).

## Authentication
### API Authentication & Secrets (Preview)
External API authentication let you authenticate to a service hosted outside of Snowflake. Snowflake supports these authentication methods for External API Authentication:
- Basic authentication (such as username and password).
- OAuth with code-grant/client-credentials flow.

In Snowflake, the authentication credentials are stored/accessed securely from an object called a `secret`, which is used with a connector to access a service outside of Snowflake. A secret is a schema-level object that stores sensitive information, limits access to the sensitive information using RBAC, and is encrypted using the master key in Snowflake key encryption hierarchy. 

If a user runs DESCRIBE SECRET on the secret, the password value stored in the secret is never exposed.

A `security integration for external API authentication` let Snowflake to connect to the service hosted outside of Snowflake with the OAuth flows.

### Federated Authentication and SSO
In a federated environment, user authentication is then passed to one or more services. So with one sign on, user can have access to Snowfake, Tableau, Gmail, etc. This env has two components:
- Service provider (SP): those who provide the service, such as Snowflake, Gmail, ...
- Identity provider (IdP): the one who maintains user credentials and authenticate them SSO to different services, such as Okta, MS Azure AD, ...

Federated authentication enables these SSO workflows:
- Logging into/out-of Snowflake.
- System timeout due to inactivity.

(configuration details skipped)

### Key Pair Authentication and Rotation
Snowflake supports key pair authentication as an alternative to basic authentication (username and password). You can generate the private-public key pair using OpenSSL. The public key is assigned to the user who uses any Snowflake client to connect/authenticate to sf. All sf clients support it. 

Snowflake supports rotating public keys, which allows compliance with more robust security and governance postures.

### Multi-Factor Authentication (MFA)
Snowflake supports multi-factor authentication (MFA) to provide increased login security. MFA support is an integrated Snowflake feature, powered by the Duo Security service, which is managed completely by Snowflake.

MFA is enabled on a per-user basis; however, users are not automatically enrolled in MFA - they must enroll themselves.

Snowflake strongly recommends that at least all users with the ACCOUNTADMIN role be required to use MFA.

### OAuth
OAuth is an open standard protocol that allows users to grant access to their protected resources on one website to another website/application without sharing their credentials. It is commonly used for authorization and secure access to APIs.OAuth uses tokens (access tokens & refresh tokens) to provide secure and limited access to the protected resources. The access token allows the client application to make authorized requests to the resource server on behalf of the user.

Snowflake supports the OAuth 2.0 protocol for authentication/authorization using one of these:
- Snowflake OAuth
- External OAuth

## Networking & Private connectivity
### Network Policies
For managing network configurations to Snowflake. A network policy allow you to create 
- an IP allowed list, 
- an IP blocked list, if desired.

A security admin or higher, or someone with global CREATE NETWORK POLICY privilege, can create a network policy to allow/deny access to a single IP address or a list of addresses.

Currently support only IPv4 addresses.

To activate a network policy, modify the account/user properties and assign the policy to the object. Only one network policy can be assigned to the account/one-user. Attaching a network policy to your account automatically replaces the existing network policy. 

Security admin, or a higher role, or someone with the global ATTACH POLICY privilege can activate a network policy.

A role that has been granted the global ATTACH POLICY privilege.

Snowflake supports specifying ranges of IP addresses using CIDR notation. 

It is possible to temporarily bypass a network policy for a set num of mins by configuring the user object property MINS_TO_BYPASS_NETWORK_POLICY. Contact Snowflake to set it. 

When a network policy includes values in both the allowed and blocked IP address lists, blocked IP address list takes precedence.

Do not add `0.0.0.0/0` to the blocked IP address list. This means "all IPv4 addresses on the local machine".

Your current IP address must be included in the allowed IP addresses in the policy.

Network policies can be managed through Snowsight, the Classic Console, or SQL. 

Snowflake supports replication and failover/failback for network policies. 

### AWS VPC Interface Endpoints for Internal Stages (business critical edition +)
Connect to Snowflake internal stages via AWS VPC Interface Endpoints.

### Azure Private Endpoints for Internal Stages (business critical edition +)
Connect to Snowflake internal stages via Microsoft Azure Private Endpoints.

### AWS PrivateLink (business critical edition +)
Configure AWS PrivateLink to directly connect Snowflake to one or more AWS VPCs. 

### Azure PrivateLink (business critical edition +)
Configure Azure Private Link to connect your Azure VNet to the Snowflake VNet in Azure.

### Google Cloud Private Service Connect (business critical edition +)
Configure Google Cloud Private Service Connect to connect your Google Cloud VPC network subnet to your Snowflake account hosted on GCP without traversing the public Internet.

## Administration & Authorization
### Sessions and Session Policies (Enterprise edition +)

### SCIM

### Access Control

### End-to-End Encryption

### Encryption Key Management





















