#mysql占用空间过大解决方案

##排查：
###1.查询所有数据库占用磁盘空间大小的SQL语句：
select TABLE_SCHEMA, concat(truncate(sum(data_length)/1024/1024,2),' MB') as data_size,
concat(truncate(sum(index_length)/1024/1024,2),'MB') as index_size
from information_schema.tables
group by TABLE_SCHEMA
order by data_length desc;
###2.查询单个库中所有表磁盘占用大小的SQL语句：
select TABLE_NAME, concat(truncate(data_length/1024/1024,2),' MB') as data_size,
concat(truncate(index_length/1024/1024,2),' MB') as index_size
from information_schema.tables where TABLE_SCHEMA = 'TestDB'
group by TABLE_NAME
order by data_length desc;（注意替换以上的TestDB为具体的数据库名）
###3.查询binlog大小
登陆进入mysql，并使用 show binary logs; 查看日志文件。
查看正在使用的日志文件：show master status;


##问题原因及解决方案：
###1.binlog过大
参考：https://blog.csdn.net/weixin_33895657/article/details/92248743
    当前正在使用的日志文件是mysql-bin.000005，那么删除日志文件的时候应该排除掉该文件。
删除日志文件的命令：purge binary logs to ‘mysql-bin.000005’;
mysql> purge binary logs to 'mysql-bin.000005';
 
删除除mysql-bin.000005以外的日志文件。
删除后就能释放大部分空间。
在my.cnf中，添加或修改expire_logs_days的值 (这里设置的自动
删除时间为10天, 默认为0不自动删除)
expire_logs_days=10
修改后，重启mysql就会生效。
但是，在生产环境中，重启mysql数据库往往会付出很高的代价。
于是，可以在不重启mysql的情况下，修改expire_logs_days值
登陆到mysql，并输入一下命令。 如下：
show variables like '%log%';
set global expire_logs_days = 10;v设置完后，可以通过 show variables like ‘%log%’; 
    看到expire_logs_days的值已被修改成10。
注意：通过这种方式设置expire_logs_days虽然不需要重启mysql
即可生效，但是该方式在重启mysql之后，值会被恢复。
于是，建议通过mysql命令设置expire_logs_days的同时，
也修改/etc/my.cnf下的expire_logs_days=10配置，这样在
下次重启mysql的时候，expire_logs_days也一样是10；

###2.index和data文件过大
使用pt-online-schema-change，代替optimize table，不会锁表，安装参考https://blog.csdn.net/song634/article/details/80411064
pt-online-schema-change --user=mombaby --password=098f6bcd4621d373cade4e832627b4f6 --host=10.10.75.209 P=3308,D=gzh,t=wx_user --charset=utf8 --alter="ENGINE=InnoDB" --nocheck-replication-filters --alter-foreign-keys-method=auto --execute
重启mysql，回收index
