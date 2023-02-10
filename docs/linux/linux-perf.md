# Linux 性能分析

## 安装常用观测工具

```bash
yum install -y sysstat
yum install -y strace
yum install -y procps
yum install -y coreutils
```

## 了解系统性能概况（**性能分析第一分钟要执行的 10 个命令**）

```bash
uptime
dmesg | tail
vmstat 1
mpstat -P ALL 1
pidstat 1
iostat -xz 1
free -m
sar -n DEV 1
sar -n TCP,ETCP 1
top
```

## CPU Profile

### 追踪

```bash
# 实时追踪进程的系统调用，ttt：time(us) since epoch, -T: syscall tims(s)
strace -tttT -p `pgrep lab002`
```

### 动态追踪

```bash
# objdump -tT /bin/bash | grep readline
00000000007003f8 g    DO .bss	0000000000000004  Base        rl_readline_state
0000000000499e00 g    DF .text	00000000000001c5  Base        readline_internal_char
00000000004993d0 g    DF .text	0000000000000126  Base        readline_internal_setup
000000000046d400 g    DF .text	000000000000004b  Base        posix_readline_initialize
000000000049a520 g    DF .text	0000000000000081  Base        readline
[...]
```

```bash
# bpftrace -e 'uretprobe:/bin/bash:readline { printf("read a line\n"); }'
Attaching 1 probe...
read a line
read a line
read a line
^C
```

## 工具

![](../images/linux_observability_tools.png)

bpf

![](../images/bpf_book_tools.png)

## References

- [动态追踪技术漫谈](https://blog.openresty.com.cn/cn/dynamic-tracing/)
- [工欲性能调优，必先利其器（1）](https://cn.pingcap.com/blog/iostat-perf-strace)
- [工欲性能调优，必先利其器（2）- 火焰图](https://cn.pingcap.com/blog/flame-graph)
- [性能分析大佬 Brendan Gregg](https://brendangregg.com/)
- [Linux 系统动态追踪技术介绍](https://blog.arstercz.com/introduction_to_linux_dynamic_tracing/)
- [Linux 性能分析前60s](https://netflixtechblog.com/linux-performance-analysis-in-60-000-milliseconds-accc10403c55)
- [USE Method: Linux Performance Checklist](https://brendangregg.com/USEmethod/use-linux.html)
- [Netflix at Velocity 2015: Linux Performance Tools](https://netflixtechblog.com/netflix-at-velocity-2015-linux-performance-tools-51964ddb81cf)
- [bpftrace uprobe](https://github.com/iovisor/bpftrace/blob/master/docs/reference_guide.md#3-uprobeuretprobe-dynamic-tracing-user-level)