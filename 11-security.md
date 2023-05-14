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

### AWS VPC Interface Endpoints for Internal Stages

### Azure Private Endpoints for Internal Stages

### AWS PrivateLink

### Azure PrivateLink

### Google Cloud Private Service Connect

## Administration & Authorization
### Sessions and Session Policies

### SCIM

### Access Control

### End-to-End Encryption

### Encryption Key Management































