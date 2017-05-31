---
title: Linux 笔记
date: 2017/5/31
tag:
  - Linux
---

##### Linux 终端代理

之前用上了Shadowsocks来代理上网，在终端下需要http代理，不能能使用socks5代理，需要将socks5代理转为http代理。

```shell
$ sudo apt install polipo
```
接着修改polipo的配置文件/etc/polipo/config：
```ini
logSyslog = true
logFile = /var/log/polipo/polipo.log
proxyAddress = "0.0.0.0"
proxyPort = 8123
socksParentProxy = "127.0.0.1:1080"
socksProxyType = socks5
chunkHighMark = 50331648
objectHighMark = 16384
serverMaxSlots = 64
serverSlots = 16
serverSlots1 = 32
```
重启polipo服务并为终端配置http代理：
```shell
$ sudo /etc/init.d/polipo restart
```


##### Ubuntu apt-get 代理

虽然各大发行版有国内的镜像，但是我用的elementary OS暂时还没有国内的镜像。而直接下载的速度只有几KB，需要对apt-get进行代理访问。

可以修改apt-get的配置文件/etc/apt/apt.conf：
```ini
Acquire::http::Proxy = "http://127.0.0.1:8123"
Acquire::ftp::proxy = "ftp://127.0.0.1:8123"
Acquire::https::proxy = "https://127.0.0.1:8123"
```
也可以在使用的时候进行代理，这样比较符合日常使用，普通就用国内的镜像加速，必须使用国外镜像下载就加上代理。
```shell
$ sudo apt -o Acquire::http::Proxy="http://127.0.0.1:8123" update
```