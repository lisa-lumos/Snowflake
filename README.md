# Snowflake Summary
This document will be updated daily until complete. 

## 🏷  RBAC best practices
RBAC is a journey - as the company grows, existing RBAC might need modifications. 

`ACCOUNTADMIN` role is the most powerful role in the system, it alone is responsible for `configuring parameters at the account level`. It is intended for performing `initial setup tasks` in the system and `managing account-level objects and tasks` on a day-to-day basis. In the system role hierarchy, other admin roles are children of it:
- `USERADMIN`: has privileges to `create and manage users and roles` (assuming ownership of those roles or users has not been transferred to another role).
- `SECURITYADMIN`: has global MANAGE GRANTS privilege to `grant or revoke privileges on objects in the account`. The USERADMIN role is its child role in the default access control hierarchy.
- `SYSADMIN`: has privileges to `create warehouses, databases, and all database objects`.

By default, when your account is provisioned, the first user is assigned the ACCOUNTADMIN role. This user should then create users who are assigned the USERADMIN role. `All remaining users should be created by USERADMIN or another role that is granted the global CREATE USER privilege`.

**ACCOUNTADMIN best practices:**
- Assign it only to a `limited number of people`.
- All users with this role should use multi-factor authentication (`MFA`) for login.
- Assign it to `at least two users`. So avoids having to go through complex resetting procedures because they can reset each other’s passwords.
- Associate an `actual person’s email address` to its users, so Snowflake knows who to contact in an urgent situation.
- `Do not` make it the `default role` for any users
- `Do not` use it for `automated scripts`

Recommend: create a `hierarchy of roles` aligned with `business functions` in your organization and `ultimately` `assign then to SYSADMIN`. So all warehouse and database object operations can be performed using the SYSADMIN role or lower roles in the hierarchy. And creating or modifying users or roles performed by SECURITYADMIN role or another role with right privileges.

To access database objects, in addition to the privileges on the specific database objects, users must be granted the `USAGE` privilege on their `database and schema`.

When `a custom role is first created`, it exists in isolation. It must then be assigned to users who need to use the role. It must also be granted to roles that manages the objects created by it. By default, not even ACCOUNTADMIN can modify or drop objects created by it. It must be granted to the ACCOUNTADMIN role directly or indirectly as the parent.

In a role hierarchy, roles are granted to other roles to form an inheritance relationship. Permissions granted to roles at a lower level are inherited by roles at a higher level.

**RBAC creation best practices:**
- `Grant permissions` on database objects or account objects to `access roles`.
- Assign (Give) access roles to (->) `functional roles`. (Functional roles correspond to the business functions.)
- When appropriate, grant `lower-level functional roles` to `higher-level functional roles` in a parent-child relationship where the parent roles map to business functions.
- Grant the `highest-level functional roles` via a role hierarchy to the SYSADMIN.

There is no technical difference between an object `access role` and a `functional role` in Snowflake. The difference is `logical`. 

Refer to `https://docs.snowflake.com/en/user-guide/security-access-control-considerations.html`.

To further lock down object security, consider using managed access schemas. In a managed access schema, object owners lose the ability to make grant decisions. Only the schema owner (i.e. the role with the OWNERSHIP privilege on the schema) or a role with the MANAGE GRANTS privilege can grant privileges on objects in the schema, including future grants, centralizing privilege management.

For security reasons, only the user who executed a query can access the query results.

A cloned container object (a database or schema) retains any privileges granted on the objects inside the container. e.g., a cloned schema retains any privileges granted on the tables, views, UDFs, and other objects in this schema. 

## Data Encryption
End-to-end encryption (E2EE) is a method to secure data that prevents third parties from reading data while at-rest or in transit to and from Snowflake and to minimize the attack surface.

Client-side encryption means that a client encrypts data before copying it into a cloud storage staging area. It follows a specific protocol defined by the cloud storage service. The service SDK and third-party tools implement this protocol. The client-side encryption protocol works as follows:
1. The customer creates a secret master key, which is shared with Snowflake.
2. The client, which is provided by the cloud storage service, generates a random encryption key and encrypts the file before uploading it into cloud storage. The random encryption key, in turn, is encrypted with the customer’s master key.
3. Both the encrypted file and the encrypted random key are uploaded to the cloud storage service. The encrypted random key is stored with the file’s metadata.

When downloading data, the client downloads both the encrypted file and the encrypted random key. The client decrypts the encrypted random key using the customer’s master key. Next, the client decrypts the encrypted file using the now decrypted random key. This encryption and decryption happens on the client side. At no time does the cloud storage service or any other third party (such as an ISP) see the data in the clear. Customers may upload client-side encrypted data using any client or tool that supports client-side encryption.

### Encryption Key Management in Snowflake
Tri-Secret Secure (business critical feature): Snowflake manages data encryption keys to protect customer data. This management occurs automatically without any need for customer intervention. Customers can use the key management service in the cloud platform that hosts their Snowflake account to maintain their own additional encryption key. When enabled, the combination of `a Snowflake-maintained key` and `a customer-managed key` creates `a composite master key` to protect the Snowflake data. This dual-key encryption model, together with Snowflake’s built-in user authentication, enables the three levels of data protection offered by `Tri-Secret Secure`.

Snowflake uses strong `AES 256-bit encryption` with a `hierarchical key model` rooted in a hardware security module. A hierarchical key model provides a framework for Snowflake’s encryption key management. The hierarchy is composed of several layers of keys in which `each higher layer of keys (parent keys) encrypts the layer below (child keys`). Snowflake’s hierarchical key model consists of four levels of keys: The root key, Account master keys, Table master keys, File keys. 

<img src="images/hierarchical-key-model.png" style="width: 70%">

### Encryption Key Rotation
All Snowflake-managed keys are automatically rotated when they are more than 30 days old. When active, a key is used to encrypt data and is available for usage by the customer. When retired, the key is used solely to decrypt data and is only available for accessing the data. Regular key rotation limits the life cycle for the keys to a limited period of time.

### Periodic Rekeying (Enterprise edition feature)
While key rotation ensures that a key is transferred from its active state to a retired state, rekeying ensures that a key is transferred from its retired state to being destroyed. If periodic rekeying is enabled, then when the retired encryption key for a table is older than one year, Snowflake automatically creates a new encryption key and re-encrypts all data previously protected by the retired key using the new key. The new key is used to decrypt the table data going forward.

## 🏷  Governance
### Column-level Security （enterprise edition feature)
Allows for masking a column within a table or view. Contains two features:
- Dynamic Data Masking: Uses masking policies to selectively mask plain-text data in table and view columns at query time. The masking policy conditions determine whether unauthorized users see masked, partially masked, obfuscated, or tokenized data. 
- External Tokenization: tokenize data (remove sensitive data by replacing it with an undecipherable token) before loading it into Snowflake and detokenize the data at query runtime.

One masking policy can be applied to many tables and views. Masking policies support `segregation of duties (SoD)` through the role separation of policy administrators from object owners. Secure views do not have SoD, which is a profound limitation to their utility. With masking policy, object owners cannot unset masking policies, and cannot view the masked columns. 





### Row Access Policies


### Object Tagging


### Tag-based Masking Policies


### Data Classification


### Access History


### Object Dependencies



## Managing Cost

## Developing Apps










