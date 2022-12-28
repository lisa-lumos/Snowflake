# Common Commands
```sql
describe user lisa; 
alter user lisa set default_role = sysadmin;
alter user lisa set default_warehouse = compute_wh;

-- loading data
create stage my_stage url='s3://mybucket/United_States/California/Los_Angeles/' credentials=(aws_key_id='1a2b3c' aws_secret_key='4x5y6z');

copy into t1 from @%t1/united_states/california/los_angeles/2016/06/01/11/mydata;

copy into t1 from @%t1/united_states/california/los_angeles/2016/06/01/11/
  files=('mydata1.csv', 'mydata1.csv');

copy into t1 from @%t1/united_states/california/los_angeles/2016/06/01/11/
  pattern='.*mydata[^[0-9]{1,3}$$].csv';

copy into <table> 
from @~/<file>.json 
file_format = (type = 'JSON' strip_outer_array = true);









-- SnowSQL
> snowsql
> !set prompt_format=[#FF0000][user].[role].[#00FF00][database].[schema].[#0000FF][warehouse]>
> !q
```


































