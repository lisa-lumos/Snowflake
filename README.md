# Snowflake Summary
This document will be updated daily until complete. 

## 🏷  RBAC best practices
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
- Create object `access roles` with different permissions on objects and assign them to `functional roles`
- `Grant permissions` on database objects or account objects to `access roles`.
- `Functional roles` correspond to the business functions.
- When appropriate, grant `lower-level functional roles` to `higher-level functional roles` in a parent-child relationship where the parent roles map to business functions.
- Grant the `highest-level functional roles` via a role hierarchy to the SYSADMIN.

There is no technical difference between an object `access role` and a `functional role` in Snowflake. The difference is `logical`. 


















