# Xây Dựng Hệ Thống MySQL Master- Master replication
- B1: Cấu hình node1 làm master
- B2: Gán quyền replication trên node2 cho node1 cho user replication
- B3: Cấu hình node2 làm slave
- B4: Kiểm tra trạng thái trên node2 – slave
- B5: Tại máy node1
- B6: node2 thêm các dòng sau
- B7: Gán quyền replication trên node2 cho node1 cho user replication
- B8: Node1 thêm các dòng sau:
- B9: Khởi động mysql trên 2 node
- B10: Kiểm tra

# Xây Dựng Hệ Thống MySQL Master- Master replication

```
Node1: master/slave: 192.168.1.11
Node2: master/slave: 192.168.1.12
```

## B1: Cấu hình node1 làm master
Thêm các dòng sau vào trong file /etc/my.cnf

```
[mysqld]

log-bin
binlog-do-db=nhansu                    # database sẽ được replicated
binlog-ignore-db=mysql                 # database không  replication
binlog-ignore-db=test
server-id=1
```

##  B2: Gán quyền replication trên node2 cho node1 cho user replication
```
# mysql
grant replication slave on *.* to 'replication'@192.168.1.12 
identified by 'slave';
```

## B3: Cấu hình node2 làm slave
Thêm các dòng sau vào trong file /etc/my.cnf

```
[mysqld]
server-id=2
master-host= 192.168.1.11
master-user=replication
master-password=slave
master-port=3306
```

Trên 2 máy khởi động lại dịch vụ mysql

```
# service mysqld restart
```

## B4: Kiểm tra trạng thái trên node2 – slave

```
[root@node2 ~]# mysql

mysql> start slave;
mysql> show slave status\G;
*************************** 1. row ***************************
             Slave_IO_State: Waiting for master to send event
                Master_Host: node1.mysql01.com
                Master_User: replication
                Master_Port: 3306
              Connect_Retry: 60
            Master_Log_File: mysqld-bin.000009
        Read_Master_Log_Pos: 98
             Relay_Log_File: mysqld-relay-bin.000018
              Relay_Log_Pos: 236
      Relay_Master_Log_File: mysqld-bin.000009
           Slave_IO_Running: Yes
          Slave_SQL_Running: Yes
            …….
      Seconds_Behind_Master: 0
1 row in set (0.00 sec)
ERROR:
No query specified
mysql>

```

## B5: Tại máy node1
Quan sát thông tin cấu trúc master-slave

```
[root@node1 ~]# mysql
mysql> show master status;
+------------------------+----------+-------------------+----------------------------+
 | File                          | Position | Binlog_Do_DB | Binlog_Ignore_DB       |
+------------------------+----------+-------------------+----------------------------+
 | mysqld-bin.000009 |          98 | nhansu             | mysql,test                     |
+------------------------+----------+-------------------+----------------------------+
1 row in set (0.00 sec)
```
Tiếp tục thực hiện cho cấu trúc slave master


## B6: node2 thêm các dòng sau
```
[mysqld]

log-bin                     #node 2 trở thành master 
binlog-do-db=nhansu
```


## B7: Gán quyền replication trên node2 cho node1 cho user replication
```
# mysql
mysql> grant replication slave on *.* to 'replication'@192.168.1.11 identified by 'slave2';

```

## B8: Node1 thêm các dòng sau:
```
[mysqld]
#node1 thành slave.
master-host=192.168.1.12
master-user= replication
master-password=slave2
master-port=3306
```

## B9: Khởi động mysql trên 2 node
```
# service mysqld restart
```


### Trên node1:

```
# mysql

mysql> start slave;
```

### Trên node2:
    

```
mysql> show master status;

+-----------------------+----------+----------------------+--------------------------+
| File                          | Position | Binlog_Do_DB    | Binlog_Ignore_DB    |
+-----------------------+----------+----------------------+--------------------------+
| mysqld-bin.000012 |          98 | nhansu                       |                                    |
+-----------------------+----------+----------------------+--------------------------+
1 row in set (0.01 sec)
```


### Trên node1:
    
```
mysql> show slave status\G;

*************************** 1. row ***************************
             Slave_IO_State: Waiting for master to send event
                Master_Host: node2.mysql01.com
                Master_User: replication
                Master_Port: 3306
              Connect_Retry: 60
            Master_Log_File: mysqld-bin.000012
        Read_Master_Log_Pos: 98
             Relay_Log_File: mysqld-relay-bin.000022
              Relay_Log_Pos: 236
      Relay_Master_Log_File: mysqld-bin.000012
           Slave_IO_Running: Yes
          Slave_SQL_Running: Yes
…………………..
    Replicate_Wild_Do_Table:
Replicate_Wild_Ignore_Table:
                 Last_Errno: 0
      Seconds_Behind_Master: 0
1 row in set (0.00 sec)
ERROR:
No query specified
```


## B10: Kiểm tra
### Trên node1:

```
# mysql
create database nhansu ;
CREATE TABLE example (
         id INT,
         data VARCHAR(100)
       );
mysql> use nhansu
Reading table information for completion of table and column names
You can turn off this feature to get a quicker startup with -A
Database changed
mysql> show tables;
+----------------+
| Tables_in_idc1 |
+----------------+
| example        |
+----------------+
1 row in set (0.00 sec)
mysql>

```

### Trên node2:

Kiểm tra thấy database nhansu và table example đã được đồng bộ


```
# mysql

mysql> use nhansu
Reading table information for completion of table and column names
You can turn off this feature to get a quicker startup with -A
Database changed
mysql> show tables;
+----------------+
| Tables_in_nhansu|
+----------------+
| example        |
+----------------+
1 row in set (0.00 sec)
mysql>
```


Thêm 2 dòng sau vào bảng example

```
mysql> insert into example (id,data) values("10","sale");
Query OK, 1 row affected (0.01 sec)
mysql> insert into example (id,data) values("20","kythuat");
Query OK, 1 row affected (0.00 sec)
```


Trên node1 thực hiện truy vấn:
```
mysql> select * from example;
+------+------------+
| id      | data          |
+------+------------+
|   10   | sale|
|   20   | kythuat      |
+------+------------+
2 rows in set (0.00 sec)
```


> Như vậy, qua bài viết này mình đã hướng các bạn xây dựng hệ thống dự phòng Master Master.