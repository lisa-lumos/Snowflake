# Class Reference
## anomaly_detection
Anomaly detection allows you to detect outliers in your time series data by using a machine learning algorithm. You use `create snowflake.ml.anomaly_detection my_model_name (...)` to create and train the detection model, and then use the `my_model_name!detect_anomalies()` method to detect anomalies.

To train the model, the sql cmd takes the table name, the timestamp col in the table, the val col in the table, and an optional label column, etc.  

The method to detect anomalies takes similar arguments, and the same col names. 

## budget
Enable you to manage budgets in your account.

### Commands
CREATE BUDGET

DROP BUDGET


### Methods
activate

add_resource

get_config

get_linked_resources

get_measurement_table

get_notification_email

get_notification_integration_name

get_notification_mute_flag

get_service_type_usage

get_spending_history

get_spending_limit

remove_resource

set_email_notifications

set_notification_mute_flag

set_spending_limit


## forecast






















