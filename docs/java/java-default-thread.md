# Java 内置线程

在使用 `jstack <pid>` 查看 JVM 线程，会发现除了 `main` 等用户线程外，还存在着其它内置线程

下面是 JDK8 程序的线程栈

```bash
Full thread dump Java HotSpot(TM) 64-Bit Server VM (11.0.3+12-LTS mixed mode):

# 主要用于处理引用对象本身（软引用、弱引用、虚引用）的垃圾回收问题
"Reference Handler" #2 daemon prio=10 os_prio=31 cpu=0.98ms elapsed=66.02s tid=0x00007fbd64057000 nid=0x3003 waiting on condition  [0x0000700009ae6000]
   java.lang.Thread.State: RUNNABLE
	at java.lang.ref.Reference.waitForReferencePendingList(java.base@11.0.3/Native Method)
	at java.lang.ref.Reference.processPendingReferences(java.base@11.0.3/Reference.java:241)
	at java.lang.ref.Reference$ReferenceHandler.run(java.base@11.0.3/Reference.java:213)

# 用于在垃圾收集前，调用对象的 finalize() 方法
"Finalizer" #3 daemon prio=8 os_prio=31 cpu=0.63ms elapsed=66.02s tid=0x00007fbd65013000 nid=0x4603 in Object.wait()  [0x0000700009be9000]
   java.lang.Thread.State: WAITING (on object monitor)
	at java.lang.Object.wait(java.base@11.0.3/Native Method)
	- waiting on <0x00000007803003e0> (a java.lang.ref.ReferenceQueue$Lock)
	at java.lang.ref.ReferenceQueue.remove(java.base@11.0.3/ReferenceQueue.java:155)
	- waiting to re-lock in wait() <0x00000007803003e0> (a java.lang.ref.ReferenceQueue$Lock)
	at java.lang.ref.ReferenceQueue.remove(java.base@11.0.3/ReferenceQueue.java:176)
	at java.lang.ref.Finalizer$FinalizerThread.run(java.base@11.0.3/Finalizer.java:170)

# Attach Listener 线程的职责是接收外部 jvm 命令，当命令接收成功后，会交给 signal dispather 线程去进行
# signal dispather 线程也是在第一次接收外部 jvm 命令时，进行初始化工作
"Signal Dispatcher" #4 daemon prio=9 os_prio=31 cpu=0.26ms elapsed=66.00s tid=0x00007fbd64058800 nid=0x4003 runnable  [0x0000000000000000]
   java.lang.Thread.State: RUNNABLE

# 即时编译（JIT）线程
"C1 CompilerThread0" #5 daemon prio=9 os_prio=31 cpu=407.92ms elapsed=66.00s tid=0x00007fbd6500f000 nid=0x3a03 waiting on condition  [0x0000000000000000]
   java.lang.Thread.State: RUNNABLE
   No compile task


"Sweeper thread" #8 daemon prio=9 os_prio=31 cpu=6.37ms elapsed=66.00s tid=0x00007fbd65838800 nid=0x3d03 runnable  [0x0000000000000000]
   java.lang.Thread.State: RUNNABLE

"Common-Cleaner" #9 daemon prio=8 os_prio=31 cpu=3.21ms elapsed=65.94s tid=0x00007fbd64889000 nid=0x5603 in Object.wait()  [0x000070000a17e000]
   java.lang.Thread.State: TIMED_WAITING (on object monitor)
	at java.lang.Object.wait(java.base@11.0.3/Native Method)
	- waiting on <0x0000000780300bb8> (a java.lang.ref.ReferenceQueue$Lock)
	at java.lang.ref.ReferenceQueue.remove(java.base@11.0.3/ReferenceQueue.java:155)
	- waiting to re-lock in wait() <0x0000000780300bb8> (a java.lang.ref.ReferenceQueue$Lock)
	at jdk.internal.ref.CleanerImpl.run(java.base@11.0.3/CleanerImpl.java:148)
	at java.lang.Thread.run(java.base@11.0.3/Thread.java:834)
	at jdk.internal.misc.InnocuousThread.run(java.base@11.0.3/InnocuousThread.java:134)

"JDWP Transport Listener: dt_socket" #10 daemon prio=10 os_prio=31 cpu=4.17ms elapsed=65.89s tid=0x00007fbd648a3000 nid=0xa503 runnable  [0x0000000000000000]
   java.lang.Thread.State: RUNNABLE

"JDWP Event Helper Thread" #11 daemon prio=10 os_prio=31 cpu=155.71ms elapsed=65.89s tid=0x00007fbd658a9800 nid=0x5d03 runnable  [0x0000000000000000]
   java.lang.Thread.State: RUNNABLE

"JDWP Command Reader" #12 daemon prio=10 os_prio=31 cpu=0.78ms elapsed=65.89s tid=0x00007fbd6485e800 nid=0xa103 runnable  [0x0000000000000000]
   java.lang.Thread.State: RUNNABLE

# 用于启动服务的线程
"Service Thread" #13 daemon prio=9 os_prio=31 cpu=0.05ms elapsed=65.28s tid=0x00007fbd65840000 nid=0xa003 runnable  [0x0000000000000000]
   java.lang.Thread.State: RUNNABLE

# 该线程负责接收外部命令，执行该命令并把结果返回给调用者，此种类型的线程通常在桌面程序中出现
"Attach Listener" #16 daemon prio=9 os_prio=31 cpu=10.25ms elapsed=64.63s tid=0x00007fbd65a2e000 nid=0x6803 waiting on condition  [0x0000000000000000]
   java.lang.Thread.State: RUNNABLE

"DestroyJavaVM" #42 prio=5 os_prio=31 cpu=2311.26ms elapsed=62.33s tid=0x00007fbd6480f000 nid=0x1603 waiting on condition  [0x0000000000000000]
   java.lang.Thread.State: RUNNABLE

# JVM 中线程的母体，是一个单例的对象（最原始的线程）会产生或触发所有其他的线程，这个单例的 VM 线程是会被其他线程所使用来做一些 VM 操作（如清扫垃圾等）
"VM Thread" os_prio=31 cpu=19.81ms elapsed=66.03s tid=0x00007fbd6487e000 nid=0x2f03 runnable

# 垃圾回收线程
"GC Thread#0" os_prio=31 cpu=17.41ms elapsed=66.05s tid=0x00007fbd6401d000 nid=0x2c03 runnable

# 垃圾回收线程
"GC Thread#1" os_prio=31 cpu=27.30ms elapsed=64.87s tid=0x00007fbd6490c000 nid=0x6303 runnable

# 垃圾回收线程
"GC Thread#2" os_prio=31 cpu=28.40ms elapsed=64.87s tid=0x00007fbd648f4800 nid=0x6503 runnable

# 垃圾回收线程
"GC Thread#3" os_prio=31 cpu=13.71ms elapsed=64.03s tid=0x00007fbd65a58000 nid=0x6e03 runnable

"G1 Main Marker" os_prio=31 cpu=0.50ms elapsed=66.05s tid=0x00007fbd6403e000 nid=0x5103 runnable

"G1 Conc#0" os_prio=31 cpu=13.09ms elapsed=66.05s tid=0x00007fbd6403f000 nid=0x2d03 runnable

"G1 Refine#0" os_prio=31 cpu=1.06ms elapsed=66.05s tid=0x00007fbd64857800 nid=0x4c03 runnable

"G1 Young RemSet Sampling" os_prio=31 cpu=17.40ms elapsed=66.05s tid=0x00007fbd64858000 nid=0x4a03 runnable
"VM Periodic Task Thread" os_prio=31 cpu=48.51ms elapsed=64.86s tid=0x00007fbd65a28000 nid=0x9d03 waiting on condition

JNI global refs: 66, weak refs: 11071
```

## References

- [JVM故障分析及性能优化系列之二：jstack生成的Thread Dump日志结构解析](https://www.javatang.com/archives/2017/10/19/51301886.html)