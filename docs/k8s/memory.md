# Pod 的内存限制机制

[Pod 内存的定义](https://kubernetes.io/zh/docs/concepts/configuration/manage-resources-containers/#meaning-of-memory)

内存指标

| 指标 | 说明 |
|: -- :|: -- :|
| rss | RSS内存，即常驻内存集（Resident Set Size），是分配给进程使用实际物理内存，而不是磁盘上缓存的虚拟内存。|
| cache | 缓存（`page cache + tmpfs`），为了提高性能，文件系统会把磁盘的内容缓存到主存，读写其实都是在主存中，这就是 page cache， 只在内存中的文件就是 tmpfs |
| swap | 虚拟内存（swap）， 指的是用磁盘来模拟内存使用， `K8S 默认关闭 swap` |
| usage | 当前使用的内存量，包括所有使用的内存, `usage = rss + cache + swap` |
| active_file | 活跃 LRU 列表中 page cache，内存压力下可回收 | 
| inactive_file | 不活跃 LRU 列表中的 page cache，可随时回收 | 
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

> 

> Java 进程的 rss 占用(G1 收集器) = 堆内存 + 非堆内存（code_cache + metaspace + direct + mapped, stack) + jvm 子系统内存(比如 G1 垃圾收集器用到的 `RememberSet 结构占堆内存的 10%`；另外还有 JIT) + JNI

由于 page cache 会在内存限制使用足够多的内存，因此需要频繁读写文件的 Pod 的 usage 内存很容易就接近 limit 上限

如果短时间内读写大量不同文件，会导致 active_file 过高，进而导致 working_set 过高。
如果对于 working_set 有监控就很容易触发告警；这种情况下可以先尽量减少读写文件的大小来缓解，比如日志文件是否打印过多等

> 监控 working_set 其实并不是十分合理，因为 active_file 在内存压力下也是可以释放的，并不会导致 OOM。监控 RSS 会是个更好的选择
> Github 上也有相关的 [issue](https://github.com/kubernetes/kubernetes/issues/43916)

## References

- [Linux 内存指标](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/6/html/deployment_guide/s2-proc-meminfo)
- [/PROC/MEMINFO之谜](http://linuxperf.com/?p=142)
- [解读VMSTAT中的ACTIVE/INACTIVE MEMORY](http://linuxperf.com/?p=97)
- [Cadvisor内存使用率指标](https://www.orchome.com/6745)
- [Java 进程中有哪些组件会占用内存？](https://toutiao.io/posts/slvddp/preview)
- [Kubernetes limit 中的内存指的是什么?](https://www.jianshu.com/p/ab0764c108de)
- [kubernetes上报Pod已用内存不准问题分析](https://cloud.tencent.com/developer/article/1637682)
- [查看linux的page cache文件系统缓存里面是那些文件占用的vmtouch命令](https://gmd20.github.io/blog/%E6%9F%A5%E7%9C%8Blinux%E7%9A%84page-cache%E6%96%87%E4%BB%B6%E7%B3%BB%E7%BB%9F%E7%BC%93%E5%AD%98%E9%87%8C%E9%9D%A2%E6%98%AF%E9%82%A3%E4%BA%9B%E6%96%87%E4%BB%B6%E5%8D%A0%E7%94%A8%E7%9A%84vmtouch%E5%91%BD%E4%BB%A4/)
- [Grafana上监控kubernetes中Pod已用内存不准问题分析](https://ormissia.github.io/posts/problems/5002-k8s-memory/)