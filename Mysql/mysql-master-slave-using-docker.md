---
title: 使用Docker配置Mysql主从复制
date: 2019/12/4
tag:
  - Docker
  - MySQL
urlname: mysql-master-slave-using-docker
---


随着Docker的广泛应用，原来需要多台物理机或者虚拟机才能实现的任务，现在使用Docker就可以轻松完成。本篇就简单介绍使用Docker构建MySQL的主从设置，当然Docker不仅能完成构建主从复制，复杂的主主复制也可以构建。

我们使用`docker-compose`来创建和管理Docker容器，这将大大减少我们的工作量。所以我们首先使用docker-compose创建两个docker容器。

<!--more-->

#### `docker-compose.yml`

```yml
version: '3.7'
services:

  msyqlmaster:
    container_name: mysqlmaster
    image: mariadb
    ports:
      - '3307:3306/tcp'
    environment: 
      MYSQL_ROOT_PASSWORD: 'root'
    volumes:
      - './data/mysql-master:/var/lib/mysql'
      - './config/mysql-master:/etc/mysql/conf.d'
    restart: always
  
  mysqlslave0:
    container_name: mysqlslave0
    image: mariadb
    ports:
      - '3308:3306/tcp'
    environment:
      MYSQL_ROOT_PASSWORD: 'root'
    volumes:
      - './data/mysql-slave-0:/var/lib/mysql'
      - './config/mysql-slave-0:/etc/mysql/conf.d'
    restart: always
```

接下来我们分别创建Master和Slave的`my.cnf`

#### `config/mysql-master/my.cnf`

```ini
[mysqld]
# 服务器标识 1 ~ 4294967295
server-id = 1
# 启用二进制日志记录
log-bin    = mysql-bin
```

#### `config/mysql-slave-0/my.cnf`

```ini
[mysqld]
server-id = 2
log-bin    = mysql-slave-bin
relay_log  = edu-mysql-relay-bin
```



对于 master 我们只需要配置`server-id`和`log-bin`，由于我们使用docker-compose一同创建了两个容器，Master和Slave处于同一局域网内，`server-id`的值必须唯一。

配置完成后，我们创建容器并自动启动服务。

```bash
docker-compose up -d
```

下一步我们在master数据库中创建数据同步用户，并授予用户`REPLICATION SLAVE`权限。

#### `Master Server`

```sql
-- 创建用户 slave
create user 'slave'@'%' identified by 'root';                                                     

-- 授予 REPLICATION SLAVE权限
GRANT REPLICATION SLAVE ON *.* TO 'slave'@'%';                                 
```

要将Slave配置为在正确的位置启动复制过程，需要Master的二进制日志中的当前坐标。使用 [`SHOW MASTER STATUS`](http://www.searchdoc.cn/rdbms/mysql/dev.mysql.com/doc/refman/5.7/en/show-master-status.com.coder114.cn.html)语句来确定Master当前二进制日志文件的名称和位置：

#### `Master Server`

```
mysql > SHOW MASTER STATUS;
+------------------+----------+--------------+------------------+
| File             | Position | Binlog_Do_DB | Binlog_Ignore_DB |
+------------------+----------+--------------+------------------+
| mysql-bin.000001 | 328      |              |                  |
+------------------+----------+--------------+------------------+
```

该`File`列显示日志文件的名称，列显示文件`Position`中的位置。在这个例子中，二进制日志文件是`mysql-bin.000001`和位置是328。记录这些值。你以后需要它们，当你设置Slave。它们表示从站应该开始处理来自主站的新更新的复制坐标。

#### `Slave Server`

```sql
CHANGE MASTER TO MASTER_HOST='172.22.0.2',MASTER_USER='slave', MASTER_PASSWORD='root', MASTER_PORT=3306, MASTER_LOG_FILE='mysql-bin.000001', MASTER_LOG_POS=328;
```

>  **MASTER_HOST**   容器的ip地址，可以使用`docker inspect mysqlmaster | grep 'IPAddress'`查询ip地址

到这里已经配置完成，只需要进行最后的测试。

#### `Master Server`

```sql
CREATE DATABASE test_replication;
```

#### `Slave Server`

```sql
SHOW DATABASES;
```

最后一条命令产生以下输出，表明复制工作正常

```
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
| performance_schema |
| sys                |
| test_replication   |
+--------------------+
```

