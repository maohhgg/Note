---
title: Android 微信聊天记录导出
date: 2018/3/29
tag:
  - sqlite
urlname: decrypted-sqlite-database
---

最近在写一个关于微信聊天记录统计和可视化的应用，我使用的Android操作系统，微信的所有的数据都以SQLite数据库的形式保存在手机的 `/data` 目录下。我只需复制文件就可以来读取所有需要的信息，注意Android系统的根目录需要root权限才能访问，如果没有root权限请先获取root权限。

<!--more-->

### 提取信息

我们需要的文件叫 `EnMicroMsg.db`，保存的目录是在

```
/data/data/com.tencent.mm/MicroMsg/{hash}/EnMicroMsg.db
```

这里的hash值为32位字符串，如果曾经登录多个微信帐号，这里会有多个hash值的文件夹，请根据聊天多少来判断 EnMicroMsg.db 的大小来区别。

获取加密的PRAGMA KEY
微信的数据使用了加密来保护消息，不然就太不安全了。现在我们就需要得到这个加密的密码。
加密的密码是 = MD5（手机IMEI + 用户UIN）
手机IMEI使用拨号输入 `×#06#` 可以查看，也可以在 设置>关于手机>状态信息>IMEI信息 中查看。
用户UIN：

```
/data/data/com.tencent.mm/shared_prefs/system_config_prefs.xml
```

上面路径文件中的 `<int name='default_uin' value='-125125546'>` 这段保存下了UIN，value中的值即为用户的实际UIN，具体值为字符串，可能包含-。

而将IMEI和UIN计算md5值，md5值的前7位即是我们需要的密码。

```php
<?php
    $imei = '89455883981245';
    $uin = '-125125546';
    echo substr(md5($imei.$uin),0,7)
?>
```

### 读取SQLite

因为是加密的SQLite文件，常规的数据库软件无法完成输入密码的步骤。这里是使用了sqlcipher。

windows下使用 sqlcipher.exe 图形界面，会提示输入密码。

我这里使用Ubuntu，可以使用apt直接安装也可以自行下载源码编译。

```bash
$ sqlcipher --version 
3.15.2 2016-11-28 19:13:37 bbd85d235f7037c6a033a9690534391ffeacecc8
```

打开加密数据

```bash
$ sqlcipher EnMicroMsg.db 
sqlite>  PRAGMA key = 'xxxxxxx';
sqlite>
```

删除密码
删除密码就直接导出一个新的数据库即可

```sql
sqlite> PRAGMA cipher_migrate; ATTACH DATABASE "decrypted_database.db" AS decrypted_database KEY "";SELECT sqlcipher_export("decrypted_database");DETACH DATABASE decrypted_database;
```

导出的decrypted_database.db即为没有密码的原来数据库了