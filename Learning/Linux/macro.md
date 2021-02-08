# Linux 常用宏

## LD_LIBRARY_PATH

### 为什么修改LD_LIBRARY_PATH呢

因为运行时动态库的搜索路径的先后顺序是：
1. 编译目标代码时指定的动态库搜索路径；
2. 环境变量`LD_LIBRARY_PATH`指定的动态库搜索路径；
3. 配置文件 /etc/ld.so.conf 中指定的动态库搜索路径；
4. 默认的动态库搜索路径 /lib 和 /usr/lib；

这个顺序是 compile gcc 时写在程序内的，通常软件源代码自带的动态库不会太多，而我们的 /lib 和 /usr/lib 只有 root 权限才可以修改，而且配置文件 /etc/ld.so.conf 也是 root 的事情，我们只好对`LD_LIBRARY_PATH`进行操作啦。

## LD_PRELOAD

### LD_PRELOAD 是什么

在 Linux 中，LD_PRELOAD 是一个环境变量用来指定预先装载的一些共享库或目标文件，且无论程序是否依赖这些共享库或者文件，LD_PRELOAD 指定的这些文件都会被装载，其优先级比 LD_LIBRARY_PATH 自定义的进程的共享库查找路径的执行还要早。

### LD_PRELOAD 主要作用

LD_PRELOAD 指定的共享库或者目标文件的装载顺序十分靠前，几乎是程序运行最先装载的，所以其中的全局符号如果和后面的库中的全局符号重名的话，就会覆盖后面装载的共享库或者目标文件中的全局符号。通过这个环境变量，我们可以在主程序和其动态链接库的中间加载别的动态链接库，甚至覆盖正常的函数库。一方面，我们可以以此功能来使用自己的或是更好的函数（无需别人的源码），而另一方面，我们也可以以向别人的程序注入恶意程序，从而达到那不可告人的罪恶的目的。

### LD_PRELOAD 安全性问题

既然 LD_PRELOAD 有如此强大的功能，那我们如何避免 LD_PRELOAD 造成安全性问题呢？

1. 随时警惕 LD_PRELOAD
2. 让 LD_PRELOAD 这个宏失效
    1. 前面说 LD_PRELOAD 原理是通过加载共享库来实现的，那么我们将程序静态链接就没这个问题了
    2. 设置执行文件的 setgid/setuid 标志。在有SUID权限的执行文件，系统会忽略 LD_PRELOAD 环境变量。也就是说，如果你有以 root 方式运行的程序，最好设置上SUID权限

### 参考资料

[0]. https://blog.csdn.net/aganlengzi/article/details/21824553?utm_source=tuicool&utm_medium=referral

[1]. https://ixyzero.com/blog/archives/3137.html

