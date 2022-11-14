---
title       : "Linux系统权限控制"
author      : "mingo"
date        : "2021-09-15 13:36:40"

layout      : post
category    : unix
tag         : ["shell","原创"]
blog        : true
star        : false
---

## 概述

> TODO

### 1. 查看指定用户信息

```shell
$ id mingo

uid=1000(mingo) gid=1000(mingo) 组=1000(mingo),0(root)
```

### 2. 查看当前系统所有用户

`/etc/passwd` 文件保存了所有用户信息

> TODO: 文件格式说明

> TODO: 密码存储位置, 加密方式

### 3. 添加用户

```shell
$ useradd boba  

# 创建用户时指定更多信息
# -g 指定主组, -G 指定附加组
$ useradd bobb -d /home/bobb -s /bin/zsh -g group -G wheel 
```

### 4. 删除用户

```shell
$ userdel boba
```

### 5. 查看所有group信息

`/etc/group` 包含了所有分组信息

```shell
$ groupmems -g tx -l    # 查看tx分组下所有成员列表

$ groupmems -h
用法：groupmems [选项] [动作]

选项：
  -g, --group groupname         更改组 groupname，而不是用户的组(只 root)
  -R, --root CHROOT_DIR         chroot 到的目录

动作：
  -a, --add username            将用户 username 添加到组成员中
  -d, --delete username         从组的成员中删除用户 username
  -h, --help                    显示此帮助信息并推出
  -p, --purge                   从组中移除所有成员
  -l, --list                    列出组中的所有成员
```

### 6. 创建group

```shell
$ groupadd tx
```

### 7. 删除group

```shell
$ groupdel tx
```

### 8. 把用户添加到组里

```shell
$ usermod boba -g tx

$ groupmems -g tx -a boba
```

### 9. 把用户从组里剔除

```shell
$ groupmems -g tx -d boba
```

### 10. 修改用户密码

```shell
$ passwd boba     # 给boba添加密码

$ passwd -d boba  # 删除boba密码
```

### 11. 给用户添加sudo权限

有时用户需要sudo权限去执行一些危险的命令, 可以给用户添加sudo权限, 主要是由 `/etc/sudoers`

此文件默认是`660`权限, 需要在编译前添加`可写`权限

```shell
$ ls -lh /etc/sudoers

-r--r----- 1 root root 3.9K 9月  14 15:41 /etc/sudoers
```

改完之后再改回`660`权限, 比较麻烦; 所以系统提供了一个`visudo`的工具, 可以直接操作

```shell
$ visudo
```

里面有很注释进行说明如何进行添加, 要想用户能有`sudo`权限有两种方式

```shell
boba ALL=(ALL)  ALL
```
> TODO 具体的格式, 权限控制

另外从`sudoers`文件里看到, 如果用户在wheel组里, 就可以拥有全部权限

```shell
## Allows people in group wheel to run all commands
%wheel  ALL=(ALL)   ALL
```

### 12. sudo免密码

把`sudoer`里的配置添加`NOPASSWD: `即可

```shell
boba ALL=(ALL)  NOPASSWD: ALL
```

```shell
%wheel  ALL=(ALL)   NOPASSWD: ALL
```
