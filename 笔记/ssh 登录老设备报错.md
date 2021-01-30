# ssh 登录老设备报错

```shell
sshpass -p root123 ssh 10.60.100.80 -lwl-admin

Unable to negotiate with 10.60.100.80 port 22: no matching key exchange method found. Their offer: diffie-hellman-group1-sha1,diffie-hellman-group14-sha1
```
## 解决办法

```
vim ~/.ssh/config 加入一下三句话

Host *
 KexAlgorithms +diffie-hellman-group1-sha1
 KexAlgorithms +diffie-hellman-group14-sha1
```