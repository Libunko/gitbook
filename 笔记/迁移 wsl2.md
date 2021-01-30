# 迁移 wsl2 至新电脑

最近更换了使用了 5 年的电脑，老电脑 wsl2 里面装了很多重要工程及文件，要是全部打包手动导出费时费力，有没有什么办法整体迁移 wsl2 至新电脑呢，答案是有的，wsl2 提供了导出 `wsl export` 及导入 `wsl import` 命令可供迁移。

## 导出老电脑 wsl2

```
PS C:\Users\Libunko> wsl -l -v
  NAME            STATE           VERSION
* Ubuntu-20.04    Running         2
```

看到老电脑里面装的是 Ubuntu-20.04 ，于是执行命令导出

```
PS C:\Users\Libunko> wsl --export Ubuntu-20.04 Ubuntu-20.04.tar.gz
```

这条命令是将 Ubuntu-20.04 导出成文件 Ubuntu-20.04.ar.gz 到 \Users\Libunko 目录。

## 设置新电脑 wsl2 环境

命令执行比较久，节约时间就先设置新电脑 wsl2 环境吧。

参考 https://docs.microsoft.com/en-us/windows/wsl/install-win10 ，就不一一赘述了。

## 导入 wsl2 至新电脑

将老电脑导出的文件 Ubuntu-20.04.ar.gz 拷贝之新电脑，好了，下一步就可以导入了。

先看看 wsl 导入命令用法

```
PS C:\Users\Libunko> wsl -h
--import <分发版> <安装位置> <文件名> [选项]
```

于是开始导入

```
PS C:\Users\Libunko> wsl --import Ubuntu-20.04 D:\Libunko\wsl D:\Libunko\Desktop\Ubuntu-20.04.tar.gz
```

这里我将 wsl2 导出到 D:\Libunko\wsl 也就是 wsl 的安装目录，同样导入时用时也较久，慢等即可。

## 导入后的一些问题

导入新电脑后，发现 wsl 版本变成了 wsl1 ，并不是我导出时的 wsl2 ，what ? 手动切到 wsl2 吧

```
PS C:\Users\Libunko> wsl --set-version Ubuntu-20.04 2
```

又是漫长的等待，ok 后，`wsl -l -v` 就能看到版本变为 2 了。

进入 wsl 后发现 默认用户变成了 root 用户，嗯？怎么回事？看来 wsl 还有 很多小 bug 要修复呀，那我们就手动修复吧。

进入 wsl 后 `vim /etc/wsl.conf` ，加入默认用户：

```
[user]
default = user_name
```

`wsl --shutdown` 停止 wsl 后，再次进入 wsl 就 OK 了，所以的文件及配置都在，迁移成功。

## 迁移 wsl 安装目录

要是只是同电脑之前迁移目录怎么办呢？导入导出太麻烦，这里有快捷的操作方法，借助第三方工具，https://github.com/DDoSolitary/LxRunOffline , 用法如下：

```
.\LxRunOffline move -n 发行版名称 -d 目标目录
```

## Enjoy ~