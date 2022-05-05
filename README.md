# mysql space analysis

## analysis：
### 1.search dbs:
``` select TABLE_SCHEMA, concat(truncate(sum(data_length)/1024/1024,2),' MB') as data_size,
concat(truncate(sum(index_length)/1024/1024,2),'MB') as index_size
from information_schema.tables
group by TABLE_SCHEMA
order by data_length desc;
```
### 2.search tables:
```
select TABLE_NAME, concat(truncate(data_length/1024/1024,2),' MB') as data_size,
concat(truncate(index_length/1024/1024,2),' MB') as index_size
from information_schema.tables where TABLE_SCHEMA = 'TestDB'
group by TABLE_NAME
order by data_length desc;
eg: TestDB
```
### 3.search binlog:
login mysql, execute: show binary logs; 
show log used：show master status;


## resolve：
### 1.binlog too large
#### del binlog
refer: https://blog.csdn.net/weixin_33895657/article/details/92248743
used log: mysql-bin.000005, which can not be del.
del binlog except of mysql-bin.000005, and release some space.
use:purge binary logs to 'mysql-bin.000005';
```
mysql> purge binary logs to 'mysql-bin.000005';
```
#### fix my.cnf
```
expire_logs_days=10  # days, default 0
```
restart mysql.
#### set expire_logs_days do not restart mysql
login mysql:
```
show variables like '%log%';
set global expire_logs_days = 10;
show variables like '%log%';
```

### 2.index && data file
use pt-online-schema-change instead of optimize table(table will be locked) refer: https://blog.csdn.net/song634/article/details/80411064
```
pt-online-schema-change --user=username --password=password --host=yourhost P=yourport,D=yourdb,t=yourtable --charset=utf8 --alter="ENGINE=InnoDB" --nocheck-replication-filters --alter-foreign-keys-method=auto --execute
```
restart mysql
