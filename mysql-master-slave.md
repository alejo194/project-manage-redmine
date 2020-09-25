
https://blog.csdn.net/doStruggle/article/details/94738922


#### mysql 主：
```bash
mysql> CREATE USER 'slave'@'%' IDENTIFIED BY 'm.....n';
mysql> GRANT REPLICATION SLAVE, REPLICATION CLIENT ON *.* TO 'slave'@'%';
mysql> flush privileges;
mysql> show master status;
+------------------+----------+--------------+------------------+-------------------+
| File             | Position | Binlog_Do_DB | Binlog_Ignore_DB | Executed_Gtid_Set |
+------------------+----------+--------------+------------------+-------------------+
| mysql-bin.000004 |     9833 |              |                  |                   |
+------------------+----------+--------------+------------------+-------------------+
# mysqldump --all-databases --master-data=2 -u root -p > /tmp/dbdump.db
```

#### mysql 从：
```bash
docker exec -it mysqlslave1 bash
#  head -30 /tmp/dbdump.db
....
-- CHANGE MASTER TO MASTER_LOG_FILE='mysql-bin.000004', MASTER_LOG_POS=9833;
....
# mysql -u root -p  < /tmp/dbdump.db
# mysql -u root -p
mysql> CHANGE MASTER TO
    ->  MASTER_HOST='主ip',
    -> MASTER_USER='slave',
    -> MASTER_PASSWORD='m.....n',
    -> MASTER_LOG_FILE='mysql-bin.000004',
    -> MASTER_LOG_POS=9833;
mysql> start slave;
mysql> show slave status\G
.....
Slave_IO_Running: Yes
Slave_SQL_Running: Yes
.....
```
