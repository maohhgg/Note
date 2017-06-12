---
title: Linux挂载smaba目录
date: 2017/5/30
tag:
  - Android
urlname: Linux-mount-smaba
---


### 使用mount挂载

本地Linux版本是elementary OS 0.4.1 Loki，是基于 "Ubuntu 16.04.2 LTS" 构建的Linux发行版，相当于Ubuntu 16.04。

先要安装cifs-utils
```shell
$ sudo apt install cifs-utils
```

#### mount命令

查看 mount 命令了解如何使用，具体属性：
```shell
$ man mount
```

我们需要使用-t -o 两个参数
```

 -o, --options opts
              Use the specified mount options.  The opts argument is a comma-separated list.  For example:

                     mount LABEL=mydisk -o noatime,nodev,nosuid

              For more details, see the FILESYSTEM-INDEPENDENT MOUNT OPTIONS and FILESYSTEM-SPECIFIC MOUNT OPTIONS sections.

-t, --types fstype
              The argument following the -t is used to indicate the filesystem type.  The filesystem types which are currently supported  depend  on  the  running  kernel.   See
              /proc/filesystems  and  /lib/modules/$(uname  -r)/kernel/fs for a complete list of the filesystems.  The most common are ext2, ext3, ext4, xfs, btrfs, vfat, sysfs,
              proc, nfs and cifs.

              The programs mount and umount support filesystem subtypes.  The subtype is defined by a '.subtype' suffix.  For example  'fuse.sshfs'.   It's  recommended  to  use
              subtype notation rather than add any prefix to the mount source (for example 'sshfs#example.com' is deprecated).

              If  no  -t  option  is given, or if the auto type is specified, mount will try to guess the desired type.  Mount uses the blkid library for guessing the filesystem
              type; if that does not turn up anything that looks familiar, mount will try to read the file /etc/filesystems, or, if that does not exist, /proc/filesystems.   All
              of  the  filesystem  types  listed there will be tried, except for those that are labeled "nodev" (e.g., devpts, proc and nfs).  If /etc/filesystems ends in a line
              with a single *, mount will read /proc/filesystems afterwards.  While trying, all filesystem types will be mounted with the mount option silent.

              The auto type may be useful for user-mounted floppies.  Creating a file /etc/filesystems can be useful to change the probe order (e.g., to try vfat before msdos or
              ext3 before ext2) or if you use a kernel module autoloader.

              More  than one type may be specified in a comma-separated list, for option -t as well as in an /etc/fstab entry.  The list of filesystem types for option -t can be
              prefixed with no to specify the filesystem types on which no action should be taken.  The prefix no has no effect when specified in an /etc/fstab entry.

              The prefix no can be meaningful with the -a option.  For example, the command

                     mount -a -t nomsdos,smbfs

              mounts all filesystems except those of type msdos and smbfs.

              For most types all the mount program has to do is issue a simple mount(2) system call, and no detailed knowledge of the filesystem type is  required.   For  a  few
              types  however  (like nfs, nfs4, cifs, smbfs, ncpfs) an ad hoc code is necessary.  The nfs, nfs4, cifs, smbfs, and ncpfs filesystems have a separate mount program.
              In order to make it possible to treat all types in a uniform way, mount will execute the program /sbin/mount.type (if that exists)  when  called  with  type  type.
              Since different versions of the smbmount program have different calling conventions, /sbin/mount.smbfs may have to be a shell script that sets up the desired call.


```

#### 附加的参数（-o --options）

在连接smaba服务时，需要使用用户名和密码，所以参数加上用户名和密码，也可以加上所在域
```
-o username=name,password=passwd,domain=WORKGROUP
```
也可以写作
```
-o username=name@WORKGROUP,password=passwd
```
如果smaba允许匿名访问，参数也可以使用匿名。
```
-o guest
```
我们还可以指定访问的权限，例如获取读写的全部权限，这需要被smaba共享目录的权限大于等于访问权限。
```
-o rw,file_mode=0777,dir_mode=0777
```
有时需要在Terminal操作，为了方便，不用每次都使用`sudo`,我们也可以指定访问的用户。比如共享目录的所有者为`pi`
```
-o uid=pi
```
我这里本地机的普通用户是`pi`，samba服务器的用户也为`pi`，共享目录文件的所有者也为`pi`。也可以实现上面的作用。
也可以将贡献目录的所有者和所在组改为本地机相同就可以了，比如本地机组和用户为`pi:pi`。在samba服务器上对目标文件执行。
```shell
$ sudo chown pi:pi -R dir/
```
当服务器和客户端为不同的操作系统，Windows和Linux，需要指定访问的编码。服务器为Linux设定为utf8，Windows设定为cp936。
```
-t iocharset=utf8
```

#### 挂载的类型（-t --types）

在Ubuntu 12.10以后使用cifs-utils，所以挂载类型为
```
-t cifs
```

#### 示例

比如我的samba访问地址为smb://222.222.222.200/www，想要挂载到/mnt/www下。

```shell
$ cd /mnt
$ sudo mkidr www
$ sudo mount t cifs //222.222.222.200/www /mnt/www -o username=pi@WORKGROUP,password=passwd,iocharset=utf8,rw,file_mode=0777,dir_mode=0777,uid=pi

$ cd /mnt/www
$ touch hello.world
```