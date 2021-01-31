# Linux铁三角之 - 进程、线程和调度

1. 进程控制块 PCB task_struct

   ```
   task_struct
   {
   	pid
   	*mm
   	*fs
   	*files
   	*signel
   	...
   }
   ```

1. pid 的数量是有限的

   ```
   cat /proc/sys/kernel/pid_max
   ```

2. task_struct 的管理方式

   - 链表

   - 树

   - 哈希

     之所以用不同的数据结构来管理 task_struct ，是为了满足不同的需要，例如遍历进程、快速找到某个进程等。

3. 进程的生命周期

   - 就绪
   - 执行
   - 睡眠
     - 浅度睡眠：可被资源和信号唤醒
     - 深度睡眠：只能被资源唤醒
   - 僵死
   - 暂停

4. 僵尸是什么：子进程死掉还没被父进程 wait 的时候

   资源已经释放，不存在内存泄露，task_struct 还在，父进程可查询到子进程死因

5. 内存泄漏到底是什么

   不是：进程死了，内存没释放（进程死了除了 task_struct 没释放，其他资源都灰飞烟灭了）

   而是：进程活了，运行越久，耗费内存越多

6. 暂停态是什么状态

   - 被动式的，例如 gdb 调试或者 ctrl-z

   - ctrl+z 可暂停程序，fg 前台继续执行，bg 后台继续执行，jobs 可查看当前有多少命令在后台运行

   - cpulimit 就是利用暂停来实现控制 cpu 占用率的
     - 用法 cpulimit -l 占用率 -p 程序pid

7. fork

   - 返回值 = -1 ：失败

   - 返回值 = 0 ：子进程

   - 返回值 = 其他：父进程，且含返回值是子进程的 pid

   - 子进程死亡父进程清场

   - 所有资源对拷一份，之后再分裂

     - 写时复制（Copy on Write）

       fork 之后内核把 mm 资源置为 RD_ONLY，当父进程或者子进程要写入 mm 时（谁先写谁先发生写时复制），内核会发生 page_fault，内核重新申请物理内存，虚拟内存在指向该内存（因此无 mmu 不能执行 fork），在去把之前要写入 mm 的值重新写入。（领导一句话，小弟跑断腿）

8. vfork

   - mm 资源共享一份，其他资源对拷
   - 使用 vfork：父进程阻塞直到子进程 exit 或者 exec

9. pthread_create（轻量级进程）

   - 共享所有资源
   - 线程也拥有 task_struct

10. clone

    - 可根据标志位共享部分资源，对拷另一部分资源

11. PID & TGID

    POSIX 要求线程 pid 一致，于是 linux 在内核做了一个障眼法，在所有线程中 getpid() 时获取的都是主线程的 pid，用 gettid() 才能获取线程 task_struct 中真正的 pid，top 命令以进程视角查看，top -H 以线程视角查看

12. init & SUBREAPER 进程

    - 进程可通过 prctl() 函数声明该函数为 SUBREAPER
    - 当一个子进程的父进程死亡时，子进程变成了孤儿，他会在进程树上寻找就近的 SUBREAPER 进程，如果找不到就托孤到 init 进程

13. 进程 0 和 1

    进程 0 时进程 1 (init) 的父进程，在 fork 进程 1 后，进程 0 退化为 IDLE 进程，调度优先级最低，只有其他所有进程睡眠后，IDLE 进程才被调度，cpu 实现低功耗
    
15. 调度算法

    1. 吞吐 vs 响应

       响应：某个任务的响应时间

       吞吐：把时间花在有用功上，无用功：上下文切换时间和 cache miss

       内核编译选项 Kernel Features: Preemption Model 可选择内核抢占模型

    2. I/O 消耗型 vs CPU 消耗型

       CPU 消耗性：把时间都花在 cpu 上

       I/O 消耗性：时间都花在 IO 上，cpu 执行时间很少 （linux 照顾 I/O 消耗性）

       arm 为什么要采用 big.LITTLE 架构：

       - 大核性能强，运算能力强，适合 CPU 消耗型任务
       - 小核功耗低，适合 I/O 消耗型任务

    3. 实时进程调度

       RT: 0 - 99 位图优先级，数字越小，优先级越高，用户空间数字相反

       调度策略：高优先级不睡，低优先级没机会，同等优先级：

       - SCHED_FIFO：先入先出，先调度的睡了后面才能用 cpu
       - SCHED_RR (Round Robin)：轮换调度

       RT 调度占用的最大的时间比例：

       `/proc/sys/kernel/sched_rt_runtime_us`

       `/proc/sys/kernel/sched_rt_period_us`

    4. 普通进程调度：

       nice 值：-20 ~ 19 对应 100 ~ 139 位图优先级

       CFS 完全公平调度（红黑树数据结构）：同时考虑了 CPU、I/O 和 nice

       virtual runtime = phytime / 权重 * 系数

       权重 sched_prio_to_weight ：由 nice 值决定，nice 0 权重 1024

    5. 工具 chrt 和 renice

       1. 设置 nice 值：`renice -n -5 -g -p pid`

          用法：
          
          - -n nice	
          - -g 进程的所有线程
          - -p 线程

       2. 设置线程为 RT 线程：`chrt -f -a -p 50 pid`

          用法：

          - -f SCHED_FIFO
          - -a 所有线程
          - -p 优先级

       3. 设置进程的 nice 值：`nice -n 值 a.out`

          用法：

          - -n nice 值：-20 ~ 19

    6. 调度相关的系统调用

       - nice()
       - sched_setscheduler()
       - sched_getscheduler()
       - sched_setparam()
       - sched_getparam()
       - sched_get_priority_max()
       - sched_get_priority_min()
       - sched_get_rr_get_interval()
       - sched_setaffinity()
       - sched_getaffinity()
       - sched_yield()  // 暂时让出 cpu
       - pthread_setschedparam()
    
16. 负载均衡

    - RT 进程：N 个优先级最高的 RT 分布到 N 个核

      - pull_rt_task()
      - push_rt_task()

    - 普通进程

      - 周期性负载均衡
      - IDLE 时负载均衡
      - fork 和 exec 时负载均衡

    - CPU task affinity（亲和力）

      - 设置 affinity

        - pthread_attr_setaffinity()
        - pthread_attr_getaffinity()
        - sched_setaffinity()
        - sched_getaffinity()

      - taskset: taskset -a -p cpu_id掩码 pid

        - -a 所有线程
        - -p cpu_id 掩码

      - IRQ affinity

        - 分配 IRQ 到某个 CPU
          - echo 01 > /proc/irq/145/smp_affinity	// IRQ 145 分配到 01 cpu
        - 多核间的 soft IRQ scaling
          - RPS  将包处理负载均衡到多个 CPU
            - echo fffe > /sys/class/net/eth1/queues/rx-0/rps_cpus	// 软中断分配到除 cpu0 以外的其他 15 个核
            - watch -d "cat /proc/softirqs | grep NET_RX"

      - cgroup

        **cgrpup 间也按照 CFS 算法调度，cgrpup 内也按照 CFS 算法调度**

        - 创建 cgroup A:  mkdir -p /sys/fs/cgroup/A
          - cpu_shares // 权重
          - cgroup.procs // 添加进程到 group
            -  sudo sh -c 'echo pid > cgroup.procs'
          - tasks // 添加线程到 group
          - cpu.cfs_preiod_us // 周期
          - cpu_cfs_quota_us // 周期里面最多能跑多少 us，可大于周期值（多核）
        - Android 和 cgroup
          - apps
          - bg_non_interactive
        - Docker 和 cgroup
          - docker run --cpu-quota 25000 --cpu-period 10000 --cpu-shares 30 app_name

17. 硬实时

    - 含义：响应时间（Hard realtime）可预期性

    - cyclicttest 工具评估调度响应时间：最小值、最大值、平均值

    - linux 为什么不是硬实时

      - cpu 时间花在：

        1. 中断 // 不可调度

        2. 软中断 // 软中断可被中断打断，不可调度

        进程：

        3. spin_lock (两核之间抢占..) // 不可调度
        4. 可调度

    - PREEMPT_RT 补丁

      - spin_lock 迁移为 mutex
      - 实现优先级继承协议 // linux 已经自带了
      - 中断线程化
      - 软中断线程化
