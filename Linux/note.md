---
title: Linux 笔记
date: 2017/5/31
tag:
  - Linux
---


#### 代理设置

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


#### Cherry Trail平板安装 Ubuntu

System: cube 

product: SurfTab twin 11.6 

设备正常使用：

- [x] CPU : Intel Atom x5-Z8350  
- [x] GPU : Intel® HD Graphics 400
- [x] eMMC : 某寨品牌eMMC
    > 早期的内核对eMMC支持不完善，在内核4.10以前都有可能遇到瞬间读写太大，导致卡死，我在使用Ubuntu16.04.2和基于Ubuntu16.04.2也遇到了这个问题。建议使用新内核的Ubuntu17.04，或者使用Debian。能避开这个问题
- [x] Wifi : rtl8723bs
    > github 有开源驱动 [https://github.com/hadess/rtl8723bs.git](https://github.com/hadess/rtl8723bs.git)， 现在已经加入在 kernel 4.12-rc1中了。可直接更新内核和更新linux-firmware，直接驱动。
- [x] Battery 
- [x] Bluetooth 

设备暂时没法使用

- [ ] Audio : RealTek ALC5642
    > 内核中无驱动， 在kernel 4.12-rc1开始将没法识别的定义为 `bytcht_nocodec`[http://elixir.free-electrons.com/linux/v4.12-rc1/source/sound/soc/intel/atom/sst/sst_acpi.c#L520](http://elixir.free-electrons.com/linux/v4.12-rc1/source/sound/soc/intel/atom/sst/sst_acpi.c#L520)，这样不能被驱动，打上[https://github.com/plbossart/UCM](https://github.com/plbossart/UCM)中的`bytcht_nocodec`能够驱动，但是配置不对。导出DSDT，使用iasl转换为刻度模式。
    ```
      Name (_HID, "10EC5640" /* Realtek I2S Audio Codec */)  // _HID: Hardware ID
                Name (_CID, "10EC5640" /* Realtek I2S Audio Codec */)  // _CID: Compatible ID
                Name (_DDN, "ALC5642")  // _DDN: DOS Device Name
    ```
    根据CID可知 ALC5642 和 ALC5640 基本一样，这里需要在kernel源码中加上一行`{"10EC5640", "bytcr_rt5642", "intel/fw_sst_22a8.bin", "bytcr_rt5642", cht_quirk, &chv_platform_data },`，并重新编译打包。（留着有空再做）
- [ ] Touch screen 
- [ ] GPS
