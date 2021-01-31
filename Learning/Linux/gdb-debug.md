# gdb
1. ulimit  -c  unlimited
2. gdb ./a.out core
3. bt
4. set args 设置参数

# tmux

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

# tcpdump

`tcpdump -i eth0 host 192.254.4.1`