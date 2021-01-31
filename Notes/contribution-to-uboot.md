# 如何向 uboot 社区贡献代码



## 修改测试你的代码

1.  fork uboot master 分支

```
git clone https://gitlab.denx.de/u-boot/u-boot.git
```

2.  修改代码...
3.  尽可能测试你修改的代码，然后 commit 至本地

## 生成path

生成最近一次 commit 的 patch，如果只有一次提交的话，多次提交参考 git format-patch 用法

```
git format-patch -1 commit_hash
```

此时会生产一个文件：xxxxxx.patch

## 配置 git send-email

1.  安装 git-email

```
# ubuntu
sudo apt install git-email
```

2.  配置 stmp 邮箱（QQ邮箱发信会被退回，提示所属域名不存在，邮件无法送达。No Mx Record Found），尽量选择其他邮箱，如Gmail邮箱：

在 ~/.gitconfig 中加入：

```
[sendemail]
        smtpServer = smtp.gmail.com
        smtpServerPort = 587
        smtpEncryption = tls
        smtpuser = xxxxxxxx@gmail.com
        stmppass = xxxxxx
```

## 发送 patch 至 uboot 社区邮箱

1.  首先需要订阅一下，地址在此 https://lists.denx.de/listinfo/u-boot，使邮箱地址对应有一个成员名称，才能向uboot社区发送补丁，否则会收到`Post by non-member to a members-only list`

2.  发送补丁至 u-boot@lists.denx.de，如果 gmail 发送失败提示 `Username and Password not accepted.`，请在谷歌账户中开启安全性较低的应用的访问权限。

```
git send-email xxxxx.patch --to u-boot@lists.denx.de
```

3.  发送完之后即可在 https://patchwork.ozlabs.org/project/uboot/list/ 看到了（首次贡献的话发送完补丁后需要等待至少12个小时才会将发送的补丁显示在patchwork 上）。

## 参考资料：

1.  https://www.cnblogs.com/dakewei/p/11419233.html
2.  https://www.denx.de/wiki/U-Boot/Patches

