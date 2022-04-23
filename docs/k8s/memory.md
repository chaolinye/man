# Pod 的内存限制机制

[Pod 内存的定义](https://kubernetes.io/zh/docs/concepts/configuration/manage-resources-containers/#meaning-of-memory)

内存指标

| 指标 | 说明 |
|: -- :|: -- :|
| rss | RSS内存，即常驻内存集（Resident Set Size），是分配给进程使用实际物理内存，而不是磁盘上缓存的虚拟内存。|
| cache | 缓存（`page cache + tmpfs`），为了提高性能，文件系统会把磁盘的内容缓存到主存，读写其实都是在主存中，这就是 page cache， 只在内存中的文件就是 tmpfs |
| swap | 虚拟内存（swap）， 指的是用磁盘来模拟内存使用 |
| usage | 当前使用的内存量，包括所有使用的内存, `usage = rss + cache + swap` |
| inactive_file | 不活跃 LRU 列表中的页内存，可随时回收 | 
| working_set | 当前内存工作集（working set）使用量, `working_set = usage - inactive_file` |

查看以上指标的命令：

```bash
# 查看进程 rss
ps aux

# 查看容器内存状况: rss, cache, swap, total_inactive_file
cat /sys/fs/cgroup/memory/memory.stat

# 查看 usage
cat /sys/fs/cgroup/memory/memory.usage_in_bytes

# 查看 pod working set memory
kubectl top pod -n <namespace>
```

Pod 监控和限制的就是 `working set` 内存

> Java 进程的 rss 占用(G1 收集器) = 堆内存 + 非堆内存（code_cache + metaspace + direct + mapped, stack) + jvm 子系统内存(比如 G1 垃圾收集器用到的 `RememberSet 结构占堆内存的 10%`；另外还有 JIT) + JNI

> 如果 inactive_file 过高，可能是日志打印过多，日志文件过大，而且属于活跃的页缓存，最终导致 inactive_file 过高


## References

- [Cadvisor内存使用率指标](https://www.orchome.com/6745)
- [Java 进程中有哪些组件会占用内存？](https://toutiao.io/posts/slvddp/preview)
- [Kubernetes limit 中的内存指的是什么?](https://www.jianshu.com/p/ab0764c108de)
- [kubernetes上报Pod已用内存不准问题分析](https://cloud.tencent.com/developer/article/1637682)
- [查看linux的page cache文件系统缓存里面是那些文件占用的vmtouch命令](https://gmd20.github.io/blog/%E6%9F%A5%E7%9C%8Blinux%E7%9A%84page-cache%E6%96%87%E4%BB%B6%E7%B3%BB%E7%BB%9F%E7%BC%93%E5%AD%98%E9%87%8C%E9%9D%A2%E6%98%AF%E9%82%A3%E4%BA%9B%E6%96%87%E4%BB%B6%E5%8D%A0%E7%94%A8%E7%9A%84vmtouch%E5%91%BD%E4%BB%A4/)