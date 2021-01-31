# 常用宏

## LD_LIBRARY_PATH

### 为什么修改LD_LIBRARY_PATH呢

因为运行时动态库的搜索路径的先后顺序是：
1. 编译目标代码时指定的动态库搜索路径；
2. 环境变量`LD_LIBRARY_PATH`指定的动态库搜索路径；
3. 配置文件/etc/ld.so.conf中指定的动态库搜索路径；
4. 默认的动态库搜索路径/lib和/usr/lib；

这个顺序是 compile gcc 时写在程序内的，通常软件源代码自带的动态库不会太多，而我们的 /lib和 /usr/lib 只有 root 权限才可以修改，而且配置文件 /etc/ld.so.conf 也是root的事情，我们只好对`LD_LIBRARY_PATH`进行操作啦。