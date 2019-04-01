---
title: 网易云音乐需要管理员权限运行及Pantheon无法弹出托盘图标
date: 2019/4/01
tag:
  - Linux
urlname: netease_music_error_in_elementaryos
---

网易云音乐的客户端在跨平台上领先国内其他音乐服务提供商，所以我一直都是在频繁使用网易云，也不知不觉网易云用户等级来到了最高等级。但是在Linux上使用网易云，我一直都是有一个心结。当使用网易云音乐v1.1.0版本在Arch或者Ubuntu运行时，必须使用 `sudo` 来启动网易云音乐客户端，这个启动方式和我平时能不使用就不使用root权限的习惯不符。所以这个小问题一直困扰着我，直到最近无意在知乎上看到了 [@Fancy](https://www.zhihu.com/people/fancyz) 的[回答](https://www.zhihu.com/question/277330447/answer/478510195)，终于解决了我的心病。

<!--more-->

### 问题的分析

当启动网易云音乐客户端时，必须使用 `sudo`，且不使用 `sudo` 启动时，在终端也没有对文件或目录权限的使用错误信息，我一直以为是网易云的有意为之。但在知乎回答中提到,

```bash
sudo -u <username> netease-cloud-music
```

上面的命令可以正常启动网易云音乐，证明与权限并没有关系。与权限无关，那当使用 `sudo` 启动时，还有变化的就是切换环境变量。其中影响到网易云运行的是变量 `SESSION_MANAGER`。


`SESSION_MANAGER` 变量是XSM协议里的, 用来指明session manager的socket位置. 桌面程序可以和session manager沟通, 保存当前状态. 以后登录就能直接回到之前的桌面状态. 网易云音乐可能用了这个变量, 但是沟通时出现了问题。知道问题出在哪里了，就比较好解决了，只要网易云音乐读到的`SESSION_MANAGER` 变量值和使用root权限时读到的值一样就可以了。 当你使用普通用户并非root登录桌面时，桌面环境的session manager使用的是以普通用户在运行，而root用户当前并没有运行任何桌面环境，也就是root用户下的 `SESSION_MANAGER` 变量值为空。 也可以使用下面的命令查看。

```bash
sudo env | grep SESSION_MANAGER
# 结果为空
env | grep SESSION_MANAGER
# 当前用户正常的值
```

### 终端解决方案

```bash
unset SESSION_MANAGER && netease-cloud-music
# 或者使用
SESSION_MANAGER='' && netease-cloud-music
```

### 图标点击的解决方案：

要使得图标点击也能正常打开，找到 `/usr/share/applications/netease-cloud-music.desktop` 文件，

```bash
sudo vim /usr/share/applications/netease-cloud-music.desktop
```

找到下面行

```
Exec= netease-cloud-music %U
```

并修改为

```
Exec=sh -c "unset SESSION_MANAGER && netease-cloud-music %U"
```

如果是使用的ElementaryOS，在Pantheon桌面环境下，网易云音乐的托盘图标不能正常使用，无法点击托盘图标弹出菜单，此时也需要在 `Exec=` 后面添加上 `XDG_CURRENT_DESKTOP=Unity`。

```
Exec=sh -c "unset SESSION_MANAGER && XDG_CURRENT_DESKTOP=Unity && netease-cloud-music %U"
```

PS：在ElementaryOS 5 Juno中，第三方应用的托盘图标不能显示的[解决方案](../display_system_tray_icons_in_elementaryos)