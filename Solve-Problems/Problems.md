## module_init 和 init_module 的区别

在看驱动时，发现其驱动的模块加载函数是 init_module()，但是看到大多数的驱动用的模块加载函数大多是 module_init() 函数。那么，module_init() 和 init_module() 这两个加载函数有什么区别吗？

1. init_module 是默认的模块的入口，如果你想指定其他的函数作为模块的入口就需要 module_init 函数来指定。
2. init_module() 是真正的入口，module_init 是宏，如果在模块中使用，最终还是要转换到 init_module() 上。如果不是在模块中使用，module_init 可以说没有什么作用。总之，使用 module_init 方便代码在模块和非模块间移植。

## linux 内核中的 likely() 和 unlikely() 宏的作用

- \#define likely(x) __builtin_expect(!!(x), 1)
- \#define unlikely(x) __builtin_expect(!!(x), 0)
- \_\_builtin_expect(long exp, long c)函数：

该函数用来引导gcc进行条件分支预测。在一条指令执行时，由于流水线的作用，CPU可以同时完成下一条指令的取指，这样可以提高CPU的利用率。在执行条件分支指令时，CPU也会预取下一条执行，但是如果条件分支的结果为跳转到了其他指令，那CPU预取的下一条指令就没用了，这样就降低了流水线的效率。
另外，跳转指令相对于顺序执行的指令会多消耗CPU时间，如果可以尽可能不执行跳转，也可以提高CPU性能。
使用__builtin_expect(long exp, long c)函数可以帮助gcc优化程序编译后的指令序列，使汇编指令尽可能的顺序执行，从而提高CPU预取指令的正确率和执行效率。

__builtin_expect(exp, c)接受两个long型的参数，用来告诉gcc：exp==c的可能性比较大。
例如，__builtin_expect(exp, 1) 表示程序执行过程中，exp取到1的可能性比较大。该函数的返回值为exp自身。
可见这里使用了gcc的内建函数__builtin_expect()。
likely(x)等价于x，即if(likely(x))等价于if(x)，但是它告诉gcc，x取1的可能性比较大。
unlikely(x)等价于x，即if(unlikely(x))等价于if(x)，但是它告诉gcc，x取0的可能性比较大。

## linux 3.0.51 移植overlay问题
### 环境描述
- Linux kernel: linux 3.0.51 （不支持overlay，官方3.18支持）
- lowerdir squashfs 只读文件系统，压缩率高
- upper jffs2 可读可写文件系统

### 问题描述：
删除只读 lowerdir 分区的文件会报错：
```
overlayfs: ERROR - failed to whiteout 'cat'
rm: can't remove 'cat': Operation not supported
```

### 问题分析
经代码分析出错位于：
```
fs/overlayfs/dir.c
    ovl_whiteout()
        vfs_setxattr()

fs/xattr.c
    vfs_setxattr()
        __vfs_setxattr_noperm()
            inode->i_op->setxattr 指针为 NULL
            
判断为 inode->i_op->setxattr 未赋值
查找代码发现
inode->i_op->setxattr 赋值被 CONFIG_JFFS2_FS_XATTR 宏控制，内核打开该宏问题解决。
        
```

## uboot envtools
uboot中一般来说只使用一个环境变量env分区，但是这种情况下env一旦损坏，保存在env中的重要值就会丢失，从而使用uboot代码写死的环境变量，那有没有什么办法可以使用两个分区互为备份来保存环境变量呢？答案是有的：

在板级头文件里定义
```
CONFIG_ENV_ADDR         /* env1 addr */
CONFIG_ENV_SIZE         /* env1 size */
CONFIG_ENV_ADDR_REDUND  /* env2 addr */
CONFIG_ENV_SIZE_REDUND  /* env2 size */
```

### 两个环境备份策略：
1. 以一个分区作为主分区A，在uboot或fw_setenv写入新环境变量时，只保存在该A分区；
2. 当下次写入新环境变量时，将之前的环境变量写入备分区B，在将新环境变量写入分区A，防止写入环境变量时两个env同时损坏。

### linux用户空间使用 uboot_envtools
uboot_envtools 提供 fw_printenv 和 fw_setenv 两个程序供linux用户空间操作uboot环境变量

注意：uboot选择双环境变量时，由于双环境变量和单环境变量头部大小不同，故uboot_envtools也要选择相同的双环境或者单环境变量。

uboot_envtools 在 fw_env.h 中提供两个方式选择环境个数：
```
#define CONFIG_FILE     "/etc/fw_env.config"

#define HAVE_REDUND /* For systems with 2 env sectors */
#define DEVICE1_NAME      "/dev/mtd1"
#define DEVICE2_NAME      "/dev/mtd2"
#define DEVICE1_OFFSET    0x0000
#define ENV1_SIZE         0x4000
#define DEVICE1_ESIZE     0x4000
#define DEVICE2_OFFSET    0x0000
#define ENV2_SIZE         0x4000
#define DEVICE2_ESIZE     0x4000

#define CONFIG_BAUDRATE		115200
#define CONFIG_BOOTDELAY	5	/* autoboot after 5 seconds	*/
#define CONFIG_BOOTCOMMAND							\
	"bootp; "								\
	"setenv bootargs root=/dev/nfs nfsroot=${serverip}:${rootpath} "	\
	"ip=${ipaddr}:${serverip}:${gatewayip}:${netmask}:${hostname}::off; "	\
	"bootm"

extern int   fw_printenv(int argc, char *argv[]);
extern char *fw_getenv  (char *name);
extern int fw_setenv  (int argc, char *argv[]);

extern unsigned	long  crc32	 (unsigned long, const unsigned char *, unsigned);
```
方法1. un-define CONFIG_FILE，定义宏 HAVE_REDUND 表示有双环境变量，分别定义
```
#define DEVICE1_NAME      "/dev/mtd1"
#define DEVICE2_NAME      "/dev/mtd2"
#define DEVICE1_OFFSET    0x0000
#define ENV1_SIZE         0x4000
#define DEVICE1_ESIZE     0x4000
#define DEVICE2_OFFSET    0x0000
#define ENV2_SIZE         0x4000
#define DEVICE2_ESIZE     0x4000
```
即可

方法2. 定义宏 CONFIG_FILE 表示fw程序将解析 /etc/fw_env.config 文件中环境变量分区，方法1失效
```
/etc/fw_env.config: 

# MTD device name	Device offset	Env. size	Flash sector size	Number of sectors
/dev/mtd1		0x0000		0x4000		0x4000
/dev/mtd2		0x0000		0x4000		0x4000
```

## busybox nfs 文件系统挂载
1. 内核、busybox开启nfs支持，开启nfs服务端
2. 修改bootargs：
root=/dev/nfs nfsroot=10.60.199.200:/mnt/nchp_root,v3
3. 启动时出现以下打印
```
VFS: Mounted root (nfs filesystem) on device 0:13.
Freeing init memory: 144K
```
即可认为挂载成功
4. 出现以下打印
```
Kernel panic - not syncing: Attempted to kill init!
```
可排查busybox /sbin/init /etc/init.d/rcS /etc/inittab 等busybox启动流程

**注意：nfs版本不同可能会导致挂载不上，故bootargs nfsroot后加,v3，可指定nfs版本**

## 解压openwrt ipk文件

`ar x *.ipk`

## kernel 内核 make menuconfig

1. 重选通用性功能。可能会造成之前内核编译出的 ko 文件在当前内核上不可用。（大坑）

## 终端代理
http:

export http_proxy=http://127.0.0.1:10809
export https_proxy=http://127.0.0.1:10809

socket:
export http_proxy=socks5://127.0.0.1:10809
export https_proxy=socks5://127.0.0.1:10809

all:
export all_proxy=socks5://127.0.0.1:10809

## ubuntu 添加用户并创建 home 目录并且加入 sudo 组

```
useradd -m libunko
usermod -aG sudo libunko

# 删除用户及其主目录和 mail spool
sudo deluser --remove-home libunko
```

## 在 oh-my-zsh 进入 包含 git 仓库目录时，会变的比平时慢/卡顿

原因是因为 oh-my-zsh 要获取 git 更新信息

- 解决办法：

设置 oh-my-zsh 不读取文件变化信息（在 git 项目目录执行下列命令）

$ git config --add oh-my-zsh.hide-dirty 1

如果你还觉得慢，可以再设置 oh-my-zsh 不读取任何 git 信息

$ git config --add oh-my-zsh.hide-status 1

okey 了，如果想恢复，设置成 0 就好


## ssh 登录老设备报错

```shell
sshpass -p root123 ssh 10.60.100.80 -lwl-admin

Unable to negotiate with 10.60.100.80 port 22: no matching key exchange method found. Their offer: diffie-hellman-group1-sha1,diffie-hellman-group14-sha1
```
### 解决办法

```
vim ~/.ssh/config 加入一下三句话

Host *
 KexAlgorithms +diffie-hellman-group1-sha1
 KexAlgorithms +diffie-hellman-group14-sha1
```