# 命令

|                             cmd                              | 描述                                                         |
| :----------------------------------------------------------: | ------------------------------------------------------------ |
|                            ps aux                            | 查看进程                                                     |
|                            top -H                            | 以线程视角查看 CPU 占用率                                    |
|                         kill 2 12345                         | 杀掉进程                                                     |
|                            kill l                            | 查看所有 signal                                              |
|                            pidof                             | 查看某个进程的 pid                                           |
|                             time                             | 查看进程执行时间                                             |
|                           swapoff                            | 关闭虚拟内存功能                                             |
|                           \time -v                           | 程序运行的具体信息，page fault 次数等                        |
|                           slabtop                            | 查看 slab 内存占用，可观察内存泄漏                           |
|                       apropos funcname                       | 在 man 中搜索带 funcname 函数名的函数                        |
|                           vmstat n                           | n 秒一次查看 vmstat                                          |
|         tar zcvf /test/data.tar.gz -C /some/files .          | 压缩 /some/files/ 目录下的文件到 data.tar.gz，且不包括 /some/files 文件夹 |
|                            xargs                             | shell 命令时，将输出的列转化为行打印                         |
|                           ls -Slh                            | 文件从大到小排序                                             |
|                           ls -Slhr                           | 文件从小到大排序                                             |
|               find -name "*.o" \| xargs ls -lt               | 时间从新到旧排序                                             |
|             find -name "*.o" \| xargs ls -lt -r              | 时间从旧到新排序                                             |
|                             date                             | 设置时间，注意TZ宏                                           |
| sed -i "s/oldstring/newstring/g" `grep oldstring -rl yourdir` | sed 批量替换多个文件中的字符串                               |
|                                                              |                                                              |
|                                                              |                                                              |

## tmux

`session -> windows -> panes`

| 动作 | 命令 |
| --- | --- |
| 创建windows | `tmux new -s name` |
| 切换session | `C-b w` |
| 横分屏 | `C-b %` |
| 竖分屏 | `C-b " |
| 在当前会话中创建串口 | `C-b c` |
| 交换两个pane | `C-b C-o` |
| 调整pane大小 | `C-b C-Left/Right/Up/Down` |

## tcpdump

`tcpdump -i eth0 host 192.254.4.1`

