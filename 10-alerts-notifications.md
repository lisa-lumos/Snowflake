# 10. Alerts & Notifications
Use Snowflake alerts & email notifications to send notifications and perform actions automatically.

## Snowflake Alerts (Preview)
Use cases - you want to receive a notification when:
- The warehouse credit usage increases by ?% of your current quota.
- The resource consumption for your pipelines/tasks/materialized views/etc increases beyond a specified amount.
- A data access request is received from an unauthorized user.
- Your data fails to comply with a business rule that you set up.

A Snowflake alert is a schema-level object that specifies:
- A condition that triggers the alert (e.g. the presence of queries that take > 1s to complete).
- The action to perform when the condition is met (e.g. send an email notification, capture some data in a table).
- Frequency/schedule to evaluate the condition (e.g. every 24 hrs).

To create an alert, you need following privileges: 
- EXECUTE ALERT on the account, which can only be granted by ACCOUNTADMIN.
- USAGE and CREATE ALERT on the schema of the to-be-created alert.
- USAGE on the database containing the schema.
- USAGE on the warehouse used to execute the alert.

When you create an alert, it is suspended by default. You must resume the newly created alert for the alert to execute.

To get the timestamps of the current schedule alert and the last alert that was successfully evaluated, use:
- snowflake.alert.scheduled_time() returns the timestamp of when the current alert was scheduled.
- snowflake.alert.last_successful_scheduled_time() returns the timestamp of when the last successfully evaluated alert was scheduled.


To monitor the execution of the alerts:
- Check the results of the action of the alert. eg, if the action inserted rows into a table, then check the table for new rows.
- View the history of alert executions -
  - your_db.information_schema.alert_history()
  - snowflake.account_usage.alert_history view 

The queries executed by alerts only appear in the alert history, not in the query history. 

## Email Notifications (Preview)
Sending an email can be the action of an alert. 

A single account can define a max of 10 email integrations and enable one or more simultaneously.

Email notifications can only be sent to users in the same Snowflake account.

To send an email notification:
1. The users (recipients) need to verify their email addresses.
2. Create a notification integration.
3. Grant the privilege to use the notification integration.
4. Call the SYSTEM$SEND_EMAIL() stored procedure to send an email notification.

The email notification message is sent from no-reply@snowflake.net.

The recipient emails in the SYSTEM$SEND_EMAIL() sp must exist in the list of ALLOWED_RECIPIENTS in the notification integration. 

### SYSTEM$SEND_EMAIL
```sql
CALL SYSTEM$SEND_EMAIL(
    '<integration_name>',
    '<email_address_1> [ , ... <email_address_N> ]',
    '<email_subject>', -- cannot be empty
    '<email_content>'  -- cannot be empty
);
```
