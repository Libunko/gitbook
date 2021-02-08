# Linux铁三角之 - 内存管理

1. 硬件原理和分页管理

   内存空间：分为内存和寄存器

   1. 进程访问虚拟地址 v = (p, d)
   2. MMU 以 p 作为索引页表
   3. page frame (f)，加上偏移 d，得到物理地址

2. kill -l 查看所有信号

   signal 11: segameation fault

3. 物理地址不是指针，而是 uint32(32 bit) uint64(64 bit)

4. meltdown 漏洞 （用户空间访问内核空间数据）

   基于时间的旁路攻击

5. 内存分 ZONE 的一般概念（理解）

   - ZONE_HIGH
   - ZONE_NORMAL
   - ZONE_DMA
   - 低端内存开机就线性映射到低端内存映射区，且偏移可配置

6. Buddy 算法（基于一个内存页管理，避免外部碎片）

   - 最小单位是一个物理页
   - cat /proc/buddyinfo
7. 常见申请内存 api

   kmalloc: 申请的内存来自低端内存区域，不需要做内存映射，一开机就映射好的，只需要线性偏移

   vmalloc: 找 vmalloc 映射区虚拟地址和高端内存或者低端内存并建立映射

   ioremap: 也是类似 vmalloc，只不过 ioremap 针对寄存器，底层不是 buddy 算法

   kmap: 内核空间访问高端内存

8. CMA 算法

   - movable
   - 平时 DMA 区域给 app 用，当 DMA 要用了把已用的内存移到其他地方去

   宋宝华文章：dma 若干误解的彻底...

9. Slab 算法

   - 解决 Buddy 粒度太大问题

   - cat /proc/slabinfo

   - kmalloc 申请用

10. malloc

    - 分配的连续内存在物理内存上不连续
    - 和 c 库是二级分配关系
    - malloc 和 free 可能并不是直接从 Buddy 上拿内存，而是通过 c 库申请释放，mallopt 可设置 c 库申请释放时内存的门限大小，c 库通过 brk 或者 mmap 从内存拿内存

11. malloc: VSS vs RSS

    malloc: 申请内存过程（lazy allocation）：
    
    1. 在进程空间找一段与申请大小相同的内存段
    2. 将该内存段的页表都指向 zero page，权限为 RD-ONLY
    3. 创建 vma（virtual memory area）权限为 R + W
    4. 当应用程序写该段内存时，由于页表没有写权限，cpu 会发出 page fault
    5. cpu 查找 page fautl 原因，发现是写 vma 内的内存，且有写权限
    6. 这时系统申请 1 页内存，并将页表指向新分配的内存，权限为 R + W
    7. 这被称为 demanding page，按需分配页
    8. 这时进程空间的中内存成为 VSS，真实拿到的内存称为 RSS
    9. 代码段、栈页都是 lazy allocation

12. OOM

    malloc 是 lazy allocation，当申请的内存兑现不了时，内核会走到 OOM (Out of Memory)，
    
    这时内核根据程序打分高低，杀掉分数最高的程序，释放内存。
    
    oom score 取决于：
    
    - 驻留内存，pagetable 和 swap
    - oom_scor_adj
    - oom_adj: -15~15 的系数调整

13. 安卓生命周期靠 OOM 驱动

14. 用户空间的进程究竟耗了多少内存？

    不指内核 3~4G 内核空间的内存，指 0~3G 用户空间的耗费的内存。
    
    进程中所有的段都用 vma 来描述。
    
    task_struct -> mm -> vma
    
    通过 pmap、/proc/pid/maps、/proc/pid/smaps 查看 vma
    
    所有 vma 加起来是程序消耗的内存吗？
    
    答案不是，多个相同程序会分享代码段，看宋宝华内存管理视频3，28:12
    
    衍生出了：vss、rss、pss、uss（独占且驻留）

15. VSS、RSS、PSS、USS

    参考：https://www.jianshu.com/p/3bab26d25d2e
    
    - VSS - Virtual Set Size （用处不大）
    
       虚拟耗用内存（包含共享库占用的全部内存，以及分配但未使用内存）。其大小还包括了可能不在RAM中的内存（比如虽然 malloc 分配了空间，但尚未写入）。VSS 很少被用于判断一个进程的真实内存使用量。
    
    - RSS - Resident Set Size （用处不大） 
    
      实际使用物理内存（包含共享库占用的全部内存）。但是 RSS 还是可能会造成误导，因为它仅仅表示该进程所使用的所有共享库的大小，它不管有多少个进程使用该共享库，该共享库仅被加载到内存一次。所以 RSS 并不能准确反映单进程的内存占用情况
    
    - PSS - Proportional Set Size （仅供参考） 
    
      实际使用的物理内存（比例分配共享库占用的内存，按照进程数等比例划分）。例如：如果有三个进程都使用了一个共享库，共占用了 30 页内存。那么 PSS 将认为每个进程分别占用该共享库 10 页的大小。 PSS 是非常有用的数据，因为系统中所有进程的 PSS 都相加的话，就刚好反映了系统中的 总共占用的内存。 而当一个进程被销毁之后， 其占用的共享库那部分比例的 PSS，将会再次按比例分配给余下使用该库的进程。这样 PSS 可能会造成一点的误导，因为当一个进程被销毁后， PSS 不能准确地表示返回给全局系统的内存。
    
    - USS - Unique Set Size （非常有用）
    
      进程独自占用的物理内存（不包含共享库占用的内存）。USS 是非常非常有用的数据，因为它反映了运行一个特定进程真实的边际成本（增量成本）。当一个进程被销毁后，USS 是真实返回给系统的内存。当进程中存在一个可疑的内存泄露时，USS 是最佳观察数据。

16. Page fault 的几种可能性

    1. 可读可写 VMA（比如堆）：页表只有 R 权限，第一次写发生 page fault，并申请内存，页表改为 R+W（minor）
    2. 非法地址，不是任何一个 VMA 地址中：segv，地址非法
    3. 可读可执行 VMA（比如代码段）：页表只有 R+X，当写时 segv，地址合法权限非法
    4. 可读可执行 VMA（比如代码段）：页表只有 R+X，当执行时，发生page fault，申请一页内存，在把代码从硬盘读出来，填到该也（major），有 IO 操作，会影响程序性能和延迟。

17. 内存泄漏（堆内存）到底是什么

    不是：进程死了，内存没释放（进程死了除了 task_struct 没释放，其他资源都灰飞烟灭了）

    而是：进程活了，运行越久，耗费内存越多。

    观察方法：连续多点采样法

18. 观察内存泄漏方法

    1. 用 meminfo\free 看系统空闲内存是不是随时间推移而降低
    2. 用 smem 看是不是 application 有泄漏，USS 增大
    3. 再看 slab，vmalloc 区是不是有泄漏

19. 在确认内存泄漏的情况下，可以用如下方法：

    1. 内存泄漏的检查 - valgrind（对性能有影响）

       - valgrind:
         gcc -g leak-example.c
         valgrind --tool=memcheck --leak-check=yes ./a.out

       ```
       ➜  day3 git:(master) ✗ valgrind --tool=memcheck --leak-check=yes ./a.out
       ==3704== Memcheck, a memory error detector
       ==3704== Copyright (C) 2002-2017, and GNU GPL'd, by Julian Seward et al.
       ==3704== Using Valgrind-3.15.0 and LibVEX; rerun with -h for copyright info
       ==3704== Command: ./a.out
       ==3704==
        ^C==3704==
       ==3704== Process terminating with default action of signal 2 (SIGINT)
       ==3704==    at 0x4938334: clock_nanosleep@@GLIBC_2.17 (clock_nanosleep.c:78)
       ==3704==    by 0x493E046: nanosleep (nanosleep.c:27)
       ==3704==    by 0x493DF7D: sleep (sleep.c:55)
       ==3704==    by 0x1091FF: main (leak-example.c:14)
       ==3704==
       ==3704== HEAP SUMMARY:
       ==3704==     in use at exit: 1,400,832 bytes in 114 blocks
       ==3704==   total heap usage: 228 allocs, 114 frees, 1,517,568 bytes allocated
       ==3704==
       ==3704== 1,388,544 bytes in 113 blocks are definitely lost in loss record 2 of 2
       ==3704==    at 0x483B7F3: malloc (in /usr/lib/x86_64-linux-gnu/valgrind/vgpreload_memcheck-amd64-linux.so)
       ==3704==    by 0x10919E: main (leak-example.c:6)
       ==3704==
       ==3704== LEAK SUMMARY:
       ==3704==    definitely lost: 1,388,544 bytes in 113 blocks
       ==3704==    indirectly lost: 0 bytes in 0 blocks
       ==3704==      possibly lost: 0 bytes in 0 blocks
       ==3704==    still reachable: 12,288 bytes in 1 blocks
       ==3704==         suppressed: 0 bytes in 0 blocks
       ==3704== Reachable blocks (those to which a pointer was found) are not shown.
       ==3704== To see them, rerun with: --leak-check=full --show-leak-kinds=all
       ==3704==
       ==3704== For lists of detected and suppressed errors, rerun with: -s
       ==3704== ERROR SUMMARY: 1 errors from 1 contexts (suppressed: 0 from 0)
       ```

    2. 内存泄漏的检查 - addresssanitizer（gcc 新版支持）

       - asan:
             gcc -g  -fsanitize=address  ./lsan.c
             gcc-8 -fuse-ld=gold -fsanitize=address -g ./lsan.c

       ```
       ➜  day3 git:(master) ✗ ./a.out
       
       =================================================================
       ==3760==ERROR: LeakSanitizer: detected memory leaks
       
       Direct leak of 122880 byte(s) in 10 object(s) allocated from:
           #0 0x7fbca40b4bc8 in malloc (/lib/x86_64-linux-gnu/libasan.so.5+0x10dbc8)
           #1 0x5555bbafb225 in main lsan.c:9
           #2 0x7fbca3ddc0b2 in __libc_start_main (/lib/x86_64-linux-gnu/libc.so.6+0x270b2)
       
       SUMMARY: AddressSanitizer: 122880 byte(s) leaked in 10 allocation(s).
       ```

20. 内核内存泄漏

    slabtop 观察哪个 slab obj 随着时间推移占用越来越大。

    menuconfig KMEMLEAK 选项可调试内核内存泄漏

21. 观察内存几个利器

    用户空间：

    ​	cat /proc/meminfo

    ​	cat /proc/buddyinfo

    ​	cat /proc/zoneinfo

    ​	cat /proc/slabinfo

    内核空间：kmemleak

    安卓：procmem

22. 内存与 IO 之间的交换

    1. page cache

       在 linux 读写文件时，它用于缓存文件的逻辑内容，从而加快对磁盘映像和数据的访问。

       用户空间有两种方式读写文件：

       1. read/write 经过 page cache 读写文件（与 mmap 相比多了 user copy）
       2. mmap 经过 page cache 读写文件

    2. mmap 的内存视图

       看起进程中 mmap 指针指向的是一个文件，其实本质上指向的是文件在内存中的副本。

    3. 代码段的本质就是 page cache

    4. page cache 的两种形式

       - 以文件系统中的文件为背景：cached
       - 以裸分区 /dev/sdax 等作为背景：buffers

    5. file-backed 和 anonymous page

       - file-backed （文件背景）映射把进程的虚拟空间映射都 files，比如代码段、mmap 的一个文件

       - anonymous 没有映射到任何 files，比如 stack、heap、cow pages，这种被称为匿名页

    6. swap

       - 作名称时，linux 有一种机制：伪造个文件来存放 anon pages 匿名页，达到释放内存的目的，存放伪造的文件的分区被称为 swap
       - 作动词时，是指 file-backed 和 anonymous page 与硬盘交换的这个动作被称为 swap
       - 在 windows 中被称为虚拟内存

    7. LRU 算法（Linux Page Replacement 还是 Least Recently Used？最近最少使用，常用于页面置换算法)

       用 LRU 算法来进行 swap 和 page cache 的页面交换

    8. 老版 free 命令详细解释

       ```
                       total          used          free     shared       buffers        cached
       Mem:      24677460[0]   23276064[1]    1401396[2]          0     870540[3]   12084008[4]
       -/+ buffers/cache:      10321516[5]   14355944[6]
       Swap:        25151484        224188      24927296
       ```

       说明：

       [5] = [1] - [3] - [4]

       [6] = [2] + [3] + [4]

       [0] = [1] + [2] = [5] + [6]

    9. 新版 free 命令详细解释

       新版 free 相较于老版，第一行加了 available ，删除了第二行。

       ```
                     total        used        free      shared  buff/cache   available
       Mem:        3891360      405136     3126028       44488      360196     3283508
       Swap:       1945676           0     1945676
       ```

       available 是预估的可用内存。

    10. 内存回收（reclaim）

    11. zRAM

        用 cpu 算力换内存

        用法：

        ```
        echo size > /sys/block/zram0/disksize
        swapon -p 10 /dev/zram0
        
        然后就能看到：
        ➜  ~ cat /proc/swaps 
        Filename								Type		Size		Used		Priority
        /dev/zram0                              partition	1945676		   0		      10
        ```

23. DMA cache 一致性

24. cgroup 

    sudo cgexec -g memory:A ./a.out	意思是把 a.out 放到 cgroup 中的 memory 中的 A group中去

    - memory.limit_in_bytes // 限制最大内存

25. 文件 Dirty Page 的写回

    文件路径：/proc/sys/vm/

    - 时间维度

      - dirty_expire_centisecs

        指定脏数据能存活的时间。在这里它的值是 30 秒。当 `pdflush/flush/kdmflush` 在运行的时候，他们会检查是否有数据超过这个时限，如果有则会把它异步地写到磁盘中。

      - dirty_writeback_centisecs

        指定多长时间 `pdflush/flush/kdmflush` 这些进程会唤醒一次，然后检查是否有缓存需要清理。

    - 空间维度

      - dirty_ratio

        是可以用脏数据填充的绝对最大系统内存量，当系统到达此点时，必须将所有脏数据提交到磁盘，同时所有新的`I/O`块都会被阻塞，直到脏数据被写入磁盘。这通常是长`I/O`卡顿的原因，但这也是保证内存中不会存在过量脏数据的保护机制。

      - dirty_background_ratio

        是内存可以填充脏数据的百分比。这些脏数据稍后会写入磁盘，`pdflush/flush/kdmflush`这些后台进程会稍后清理脏数据。比如，我有 32G 内存，那么有 3.2G 的脏数据可以待着内存里，超过 3.2G 的话就会有后台进程来清理。

26. 内存收回

    - high_free_byte: 内存到此点，停止回收
    - low_free_byte: 内存到此点，kswapd 启动 reclaim（后台内存收回）
    - min_free_byte: 内存到此点，做 direct reclaim（直接内存回收，堵住前台进程）
    - /proc/sys/vm/ 下的 min_free_kbytes 和 lowmem_reserve_ratio 可设置min 水位和低水位

    swappiness: 反映是否积极回收 swap（0~100）

    - 默认 60

    - 越大越倾向回收匿名页
    - 越小越倾向回收 file-backed，即使设为 0 ，当 free 内存 + file-backed < high water mark 时，还是会交换匿名页
    - 对 cgroup 里的 memory.swappiness 为 0 时，匿名页交换被关闭

    vfs_cache_pressure: 表示内核回收用于 dentries cache 和 inode cache 内存倾向

    ```
    To free pagecache::
    	echo 1 > /proc/sys/vm/drop_caches
    To free reclaimable slab objects (includes dentries and inodes)::
    	echo 2 > /proc/sys/vm/drop_caches
    To free slab objects and pagecache::
    	echo 3 > /proc/sys/vm/drop_caches
    ```
    
27. 内存延迟观察工具

    - getdelays  docs: Documentation/accounting/delay-accounting.rst
    - vmstat