# 同步、异步、阻塞、非阻塞和 IO 模型

- Bootchart：开机优化工具

## I/O 模型

I/O 模型经常被搞混淆，阻塞、非阻塞、同步、异步傻傻分不清楚。首先要搞清楚概念：

### 什么是阻塞、非阻塞？

阻塞、非阻塞是针对**调用者**（线程）来说的。在阻塞模式下，读取或写入时将一直等待（线程挂起），直到操作完成，而非阻塞模式下，读取或写入立即返回一个状态值。

### 什么是同步、异步？

\* A synchronous I/O operation causes the requesting process to be blocked until that I/O operation completes;

\* An asynchronous I/O operation does not cause the requesting process to be blocked;

摘自 Richard Stevens 的 “UNIX® Network Programming Volume 1, Third Edition: The Sockets Networking ” 6.2 节“I/O Models ”

### 五种 IO 模型

1. 阻塞（blocking IO）

    进程阻塞等 I/O ready，Linux 默认 socket 都是阻塞的

    有两种返回情况：

    1. 等到资源
    2. 在设置 xx 的情况下，收到 EINTR 信号

2. 非阻塞（nonblocking IO）

    没有资源返回 EAGAIN

3. 多路复用（IO multiplexing）

    - selset
    - epoll

4. SIGNO（signal driven IO）

5. 异步 I/O（asynchronous IO）

    - glibc：
        - aio_read()
        - aio_suspend()
    - kernel-aio：它自己本身就是系统调用
        - io_setip()
        - io_submit()
        - io_getevents()
        - io_destory()

event 事件驱动：libevent

c10k 问题：http://www.kegel.com/c10k.html

### 同步异步 IO

以  read 为例，操作发生时，它会经历两个阶段：
1）等待数据准备（Waiting for the data to be ready）
2）将数据从内核拷贝到进程中（Copying the data from the kernel to the process）

因此前四种 IO 模型称为同步 IO，后一种称为异步 IO。

- 有人可能会说，non-blocking IO 并没有被 block 啊。这里有个非常“狡猾”的地方，定义中所指的“IO operation”是指真实的 IO 操作，例如 recvfrom 这个系统调用。non-blocking IO在执行recvfrom 这个系统调用的时候，如果 kernel 的数据没有准备好，这时候不会 block 进程。但是当 kernel 中数据准备好的时候，recvfrom 会将数据从 kernel 拷贝到用户内存中，这个时候进程是被 block 了，在这段时间内进程是被 block 的。

- asynchronous IO 则不一样，当进程发起 IO 操作之后，就直接返回再也不理睬了，直到 kernel 发送一个信号，告诉进程说 IO 完成。在这整个过程中，进程完全没有被 block。

- IO 复用同非阻塞 IO 本质一样，不过利用了新的 select/epoll 系统调用，由内核来负责本来是请求进程该做的轮询操作。看似比非阻塞IO还多了一个系统调用开销，不过因为可以支持多路IO，才算提高了效率。

- 信号驱动 IO 调用 sigaction 系统调用，当内核中 IO 数据就绪时以 SIGIO 信号通知请求进程，请求进程再把数据从内核读入到用户空间，这一步是阻塞的。

### 各种情况的例子

以小明下载文件为例，对上述概念做一梳理：

- **同步阻塞**：小明一直盯着下载进度条，到 100% 的时候就完成。
    - 同步：等待下载完成通知；
    - 阻塞：等待下载完成通知过程中，不能做其他任务处理；
- **同步非阻塞**：小明提交下载任务后就去干别的，每过一段时间就去瞄一眼进度条，看到 100% 就完成。
    - 同步：等待下载完成通知；
    - 非阻塞：等待下载完成通知过程中，去干别的任务了，只是时不时会瞄一眼进度条；【小明必须要在两个任务间切换，关注下载进度】
- **异步阻塞**：小明换了个有下载完成通知功能的软件，下载完成就“叮”一声。不过小明仍然一直等待“叮”的声音（看起来很傻，不是吗）。
    　　- 异步：下载完成“叮”一声通知；
          　　- 阻塞：等待下载完成“叮”一声通知过程中，不能做其他任务处理；
- **异步非阻塞**：仍然是那个会“叮”一声的下载软件，小明提交下载任务后就去干别的，听到“叮”的一声就知道完成了。
    　　- 异步：下载完成“叮”一声通知；
          　　- 非阻塞：等待下载完成“叮”一声通知过程中，去干别的任务了，只需要接收“叮”声通知。

### 参考资料：

https://blog.csdn.net/z_ryan/article/details/80873449?spm=1001.2014.3001.5502

https://www.jianshu.com/p/aed6067eeac9

https://blog.csdn.net/xiexievv/article/details/44976215

