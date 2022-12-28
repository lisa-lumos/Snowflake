# Common Commands
```sql
describe user lisa; 
alter user lisa set default_role = sysadmin;
alter user lisa set default_warehouse = compute_wh;

-- loading data
create stage my_stage url='s3://mybucket/US/California/san_diego/' credentials=(aws_key_id='1a2b3c' aws_secret_key='4x5y6z');

copy into t1 -- copy files with filename that has mydata as prefix in the path into table
from @%t1/US/california/san_diego/2016/06/01/11/mydata;

copy into t1 -- copy files with specified filenames in the path into table
from @%t1/US/california/san_diego/2016/06/01/11/
files=('mydata1.csv', 'mydata1.csv');

copy into t1 -- copy files with matched filenames in the path into table, delete files after copied successfully
from @%t1/US/california/san_diego/2016/06/01/11/
pattern='.*mydata[^[0-9]{1,3}$$].csv'
purge = true;

copy into mytable -- ignore the space before opening quotes before values
from @%mytable
file_format = (type = csv trim_space=true field_optionally_enclosed_by = '0x22');

copy into table1 from @~/test1.json -- load each array elem into its own row
file_format = (type = 'JSON' strip_outer_array = true strip_null_values = true);

-- remove staged files
remove @mystage/path1/subpath2; -- rmv all files under this path







-- SnowSQL
> snowsql
> !set prompt_format=[#FF0000][user].[role].[#00FF00][database].[schema].[#0000FF][warehouse]>
> !q
```


































