---
title: MySQL
date: 2018/03/08
tag:
  - MySQL
urlname: mysql-server-config
---

MySQL 5.7开放下载已经几年了，比 MySQL 5.6 快 3 倍，同时还提高了可用性，可管理性和安全性。并增加了一些重要的增强功能： 

1. 性能和可扩展性：改进 InnoDB 的可扩展性和临时表的性能，从而实现更快的网络和大数据加载等操作。
2. JSON支持：使用 MySQL 的 JSON 功能，你可以结合 NoSQL 的灵活和关系数据库的强大。
3. 改进复制 以提高可用性的性能。包括多源复制，多从线程增强，在线 GTIDs，和增强的半同步复制。 
4. 性能模式 提供更好的视角。我们增加了许多新的监控功能，以减少空间和过载，使用新的 SYS 模式显著提高易用性。
5. 安全: 我们贯彻“安全第一”的要求，许多 MySQL 5.7 新功能帮助用户保证他们数据库的安全。
6. 优化: 我们重写了大部分解析器，优化器和成本模型。这提高了可维护性，可扩展性和性能。
7. GIS: MySQL 5.7 全新的功能，包括 InnoDB 空间索引，使用 Boost.Geometry，同时提高完整性和标准符合性。

还有一点就是在MySQL 5.6中最大索引长度为767字节，使用utf8mb4编码时，最多只有191个字符。而PHP 的 Laravel 5.4以后默认使用utf8mb4编码。MySQL 5.7增加到了3072字节。

<!--more-->

#### MySQL 安装

##### Ubuntu server

如果你使用的Ubuntu系列的服务器，截至现在2018年3月，Ubuntu 16.04已经提供了MySQL5.7版本。可直接安装。

```bash
sudo apt update
sudo apt install mysql-server
```

##### Debian server

而到目前为止（2018年3月），Debian 9（stretch）只 APT提供了MySQL 5.5.9999 版本。安装需要自行去添加源或下载高版本源中的打包文件安装。
我使用的是去下载打包好的文件deb文件。自行下载对应平台的文件：

1. [mysql-server](https://packages.debian.org/sid/mysql-server)
2. [mysql-server-core-5.7](https://packages.debian.org/sid/mysql-server-core-5.7)
3. [mysql-common](https://packages.debian.org/sid/mysql-common)
4. [mysql-client](https://packages.debian.org/sid/mysql-client)
5. [mysql-client-core-5.7](https://packages.debian.org/sid/mysql-client-core-5.7)
6. [libevent-core-2.1-6](https://packages.debian.org/sid/libevent-core-2.1-6)
7. [libmecab2](https://packages.debian.org/sid/libmecab2)

> 注意： 这只符合但前（2018年3月），以后可能有更新，视以后具体版本而定

```bash
libevent-core-2.1-6_2.1.8-stable-4_armhf.deb
libmecab2_0.996-5_armhf.deb
mysql-client-5.7_5.7.21-1_armhf.deb
mysql-client-core-5.7_5.7.21-1_armhf.deb
mysql-common_5.8+1.0.4_all.deb
mysql-server-5.7_5.7.21-1_armhf.deb
mysql-server-core-5.7_5.7.21-1_armhf.deb
```

```bash
sudo dpkg -i *.deb
```

##### MySQL 配置

当使用deb文件安装完成后，没有常规的安装过程中的初始化密码的步骤。需要在启动MySQL服务后查找默认密码，或者重置密码。

```bash
sudo /etc/init.d/mysql start
# or
#sudo service mysql start
```

在网上中其他教程中，有说默认密码会在 `/var/log/mysqld.log` 日志中记录初始化的密码，不知是否是使用的Debian系列的关系，教程中的是使用CentOS或者RedHat，我的MySQL的日志文件只有 `/var/log/mysql/error.log` 。 并没有说的 `/var/log/mysqld.log` 文件。
如果你使用的 CentOS 或者 RedHat 可以尝试下在 `/var/log/mysqld.log` 查找下密码。

```bash
sudo grep 'temporary password' /var/log/mysqld.log
```

因为没有`/var/log/mysqld.log`，我只能是重置密码，重置密码也有多种方式。

1. 运行 `mysql_secure_installation` 按提示来重置密码
2. MySQL使用安全模式启动，安全模式下不需要密码，在登录MySQL更新密码。

```bash
sudo servcie mysql stop
sudo mysqld_safe --skip-grant-tables --skip-networking &
```
使用文件安装的MySQL这一步可能有提示 `/var/run/mysqld` 不存在，只需创建这个文件夹，并把所有者更改为mysql即可

```bash
mkdir -p /var/run/mysqld
sudo mkdir -p /var/run/mysqld
sudo chown mysql:mysql /var/run/mysqld
```

然后直接登录MySQL更新密码。

```bash
mysql -u root

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.
mysql> FLUSH PRIVILEGES;
# 数据库重新加载授权表。
mysql> ALTER USER 'root'@'localhost' IDENTIFIED BY 'new_password';
```

完成后关闭安全模式的MySQL和启动MySQL服务器。

```
sudo cat /var/run/mysqld/mysqld.pid 
xxxxx
sudo kill xxxxx
sudo service mysql start
```

##### MySQL 优化
在小内存的机器上安装上MySQL5.7，会出现占用内存过多的现象。
performance_schema 这个选项式分析用的,小内存机器可以关了，能省好多内存

```conf
[mysqld]
performance_schema     = 0
tabel_definition_cache = 400
tabel_open_cache       = 256
```
![htop.png](config/htop.png)