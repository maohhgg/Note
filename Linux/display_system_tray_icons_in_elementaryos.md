---
title: ElementaryOS 5.0 Juno 显示托盘图标
date: 2019/3/31
tag:
  - Linux
urlname: display_system_tray_icons_in_elementaryos
---

ElementaryOS当从Loki升级到Juno，会遇到第三方应用的系统托盘图标不显示的问题。因为在发行版Juno中，ElementaryOS 取消了对 Ayatana Indicators 的支持。

在旧版本中，操作系统中 Ayatana AppIndicators 由两个软件包负责显示，`indicator-application` 和 `wingpanel-indicator-ayatana`。而在新的发行本Juno中，系统默认自带了 `indicator-application`，`wingpanel-indicator-ayatana` 在ElementaryOS 5.0 Juno中已被遗弃。

<!--more-->

### 调整 indicator-application 包

`indicator-application` 虽然系统自带，但还是需要一些调整才能配合Panthon运行，需要将桌面的自启动的配置文件 `/etc/xdg/autostart/indicator-application.desktop` 中的

```
OnlyShowIn=Unity;GNOME;
```

修改为

```
OnlyShowIn=Unity;GNOME;Pantheon;
```

如果只希望影响当前用户，并且每次更新 `indicator-application` 包后不被重置自启动配置文件。需要将 `/etc/xdg/autostart/indicator-application.desktop` 文件复制到 `~/.config/autostart` 。

```bash
mkdir -p ~/.config/autostart

cp /etc/xdg/autostart/indicator-application.desktop ~/.config/autostart/

sed -i 's/^OnlyShowIn.*/OnlyShowIn=Unity;GNOME;Pantheon;/' ~/.config/autostart/indicator-application.desktop
```

或者希望给全部用户启用 `Ayatana Indicators`

```bash
sudo sed -i 's/^OnlyShowIn.*/OnlyShowIn=Unity;GNOME;Pantheon;/' /etc/xdg/autostart/indicator-application.desktop
```

### 安装 wingpanel-indicator-ayatana

虽然 `wingpanel-indicator-ayatana` 在Juno中不可用，但可以从此处[下载](http://ppa.launchpad.net/elementary-os/stable/ubuntu/pool/main/w/wingpanel-indicator-ayatana/wingpanel-indicator-ayatana_2.0.3+r27+pkg17~ubuntu0.4.1.1_amd64.deb
)旧版本的DEB包 （名称中没有 -dbg 的包）并将其安装在系统中。

```bash
wget http://ppa.launchpad.net/elementary-os/stable/ubuntu/pool/main/w/wingpanel-indicator-ayatana/wingpanel-indicator-ayatana_2.0.3+r27+pkg17~ubuntu0.4.1.1_amd64.deb

sudo dpkg -i wingpanel-indicator-ayatana_2.0.3+r27+pkg17~ubuntu0.4.1.1_amd64.deb
```

安装完成后，注销然后登录到基本桌面后，Ayatana AppIndicators应该可以工作，系统托盘中第三方应用就可以正常显示了。