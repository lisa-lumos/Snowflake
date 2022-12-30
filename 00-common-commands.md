# Common Commands
```sql
describe user lisa; 


-- staging files ------------------------------------------------------------
create or replace stage my_stage -- create a internal named stage, associate stage with named file format
file_format = my_csv_format;

create or replace stage my_stage -- create a named stage, associate stage with anonymous file format
file_format = (type = 'CSV' field_delimiter = '|' skip_header = 1);

create stage my_stage url='s3://mybucket/US/California/san_diego/' credentials=(aws_key_id='1a2b3c' aws_secret_key='4x5y6z');

-- (Linux/Unix)
put file:///data/data.csv @~/staged; -- put local file in the /data dir into user stage in specified folder named "staged" 
put file:///data/data.csv @%mytable; -- ... into table stage
put file:///data/data.csv @my_stage; -- ... into named stage

-- (Windows OS), to do the same as above
put file://c:\data\data.csv @~/staged; 
put file://c:\data\data.csv @%mytable;
put file://c:\data\data.csv @my_stage;

list @~;        -- list files in user stage
list @%mytable; -- list files in table stage
list @my_stage; -- list files in named stage

-- bulk loading files ------------------------------------------------------------
-- this requires a warehouse
copy into mytable -- from files with filename that has mydata as prefix
from @%mytable/US/california/san_diego/2016/06/01/11/mydata;

copy into mytable -- from files specified in the path
from @%mytable/US/california/san_diego/2016/06/01/11/
files=('mydata1.csv', 'mydata1.csv');

copy into mytable -- from files with matched filenames in the path, delete files after copied successfully
from @%mytable/US/california/san_diego/2016/06/01/11/
pattern='.*mydata[^[0-9]{1,3}$$].csv'
purge = true;

copy into mytable -- from all files in specified path in user stage
from @~/staged 
file_format = (format_name = 'my_csv_format');

copy into mytable -- from all files from the table stage (from clause omitted)
file_format = (type = csv field_delimiter = '|' skip_header = 1);

copy into mytable -- from all files in named stage, using file format associated with this stage
from @my_stage;

copy into mytable -- use validation before loading
from @my_stage
validation_mode = 'RETURN_ALL_ERRORS';

copy into mytable -- ignore the space before opening quotes before values
from @%mytable
file_format = (type = csv trim_space=true field_optionally_enclosed_by = '0x22');

copy into mytable -- load each array elem into its own row
from @~/test1.json 
file_format = (type = 'JSON' strip_outer_array = true strip_null_values = true);

-- remove staged files
remove @mystage/path1/subpath2; -- rmv all files under this path







-- SnowSQL ------------------------------------------------------------
> snowsql
> !set prompt_format=[#FF0000][user].[role].[#00FF00][database].[schema].[#0000FF][warehouse]>
> !q


-- parameters ------------------------------------------------------------
-- account parameters (only can be set at account level)
alter account set min_data_retention_time_in_days = 5; -- some admins can set these

-- session parameters (can be set at account, user and session levels)
--   alter session > alter user > alter account)
alter user lisa set default_role = sysadmin; -- some admins and users(only for themselves) can both set these
alter user lisa set default_warehouse = compute_wh;
alter session set lock_timeout = 3600; -- users can use this to set session parameters within their sessions
alter session unset lock_timeout; -- set back to default

-- object parameters (can be set at both account and object level)
--   alter table/pipe... > alter schema > alter database > alter account; 
--   alter warehouse > alter account
alter account set data_retention_time_in_days = 5; -- for time travel
alter table t1 set data_retention_time_in_days = 5;

show parameters; -- show only session parameters
show parameters in database db1; -- show object parameters for this db
show parameters in warehouse wh1; -- show object parameters for this wh
show parameters in account; -- show all account, session and object parameters
show parameters like 'time%' in account; -- show all params start with "time"















```


































