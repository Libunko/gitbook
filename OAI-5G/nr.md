# OAI NR

## 代码流程

```c
main()
	/*
     * 创建两个管道，一个用来执行命令，另一个用来获取执行命令的结果
     * fork 创建进程，子进程返回，父进程 while (1）循环接收管道命令
     */
	start_background_system()
    
    /* 加载配置模块 */
    load_configmodule()
    
    /* 注册信号回调函数 */
    set_softmodem_sighandler()
    
    /* 日志初始化 */
    logInit()
```

