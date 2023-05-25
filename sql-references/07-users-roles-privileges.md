# Users, roles, privileges

## grant privileges
```sql
grant
    globalprivileges         | all on account
  | accountobjectprivileges  | all on user | resource monitor | warehouse | database | integration <object_name>
  | schemaprivileges         | all on schema <schema_name> | all schemas in database <db_name>
  | schemaprivileges         | all on future schemas in database <db_name>
  | schemaobjectprivileges   | all on <object_type> <object_name> | all <object_type_plural> 
    in database <db_name> | schema <schema_name>
  | schemaobjectprivileges   | all on future <object_type_plural> in database <db_name> | schema <schema_name>
to [role] <role_name> 
[with grant option]
```

Multiple privileges can be specified for the same object type in a single GRANT statement. 

Future grants cannot be defined on objects of the following types:
- External function
- Policy objects: Masking/Row-access/Session policy
- Tag

```sql
grant select on future tables in database d1 to role r1;
grant insert,delete on future tables in schema d1.s1 to role r2;
grant operate on warehouse report_wh to role analyst;
grant operate on warehouse report_wh to role analyst with grant option; -- let this role grant this privi to other roles
grant select on all tables in schema mydb.myschema to role analyst;
grant all privileges on function mydb.myschema.add5(number) to role analyst;
grant all privileges on function mydb.myschema.add5(string) to role analyst; -- you can overload an udf with diff signatures
grant usage on procedure mydb.myschema.myprocedure(number) to role analyst;
grant create materialized view on schema mydb.myschema to role myrole;
grant select,insert on future tables in schema mydb.myschema to role role1;
use role accountadmin;
grant usage on future schemas in database mydb to role role1;
-- below grant privileges to db roles
grant select on all tables in schema mydb.myschema to database role mydb.dr1;
grant all privileges on function mydb.myschema.add5(number) to database role mydb.dr1;
grant all privileges on function mydb.myschema.add5(string) to database role mydb.dr1;
grant usage on procedure mydb.myschema.myprocedure(number) to database role mydb.dr1;
grant create materialized view on schema mydb.myschema to database role mydb.dr1;
grant select,insert on future tables in schema mydb.myschema to database role mydb.dr1;
use role accountadmin;
grant usage on future schemas in database mydb to database role mydb.dr1;
```




























