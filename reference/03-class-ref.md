# Class Reference
## anomaly_detection
Anomaly detection allows you to detect outliers in your time series data by using a machine learning algorithm. You use `create snowflake.ml.anomaly_detection my_model_name (...)` to create and train the detection model, and then use the `my_model_name!detect_anomalies()` method to detect anomalies.

To train the model, the sql cmd takes the table name, the timestamp col in the table, the val col in the table, and an optional label column, etc.  

The method to detect anomalies takes similar arguments, and the same col names. 

## budget
Enable you to manage budgets in your account.

### Commands
create snowflake.core.budget ...

drop snowflake.core.budget ...

### Methods
activate(): You must activate the "account budget", in order to use the budgets feature. Not for custom budgets. 

add_resource(): Add an object to a "custom budget"; add by obj reference. 

get_config(): View the config properties for a budget.

get_linked_resources(): List the objects in a "custom budget".

get_measurement_table(): View the credit usage data collected by the budget maintenance task. 

get_notification_email()

get_notification_integration_name()

get_notification_mute_flag()

get_service_type_usage(): View the credit usage for a budget by service type.

get_spending_history(): View the spending history for a budget.

get_spending_limit(): Remove an object from a "custom budget". 

remove_resource()

set_email_notifications()

set_notification_mute_flag()

set_spending_limit()


## forecast
A forecast model produces a forecast, for a single time series, or for multiple time series.
