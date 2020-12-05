# Arthas 实践

## 常用命令

```bash
# 监听方法入参、出参、异常
watch *.JedisService isReachLimit '{params, returnObj, throwExp}' -x 3 -n 10

# 调用静态方法
ognl -c 4048ea5f '@org.leaf.StringUtils@doSomething("parameter")'

# 获取 Spring ApplicationContext，并访问 Bean
tt -t *.DispatcherServlet doDispatch
tt -i 1000 -w 'target.getWebApplicationContext().getBean("jedisService")'

# 方法耗时分析
trace *.SomeClass someMethod

# 查看方法调用栈
stack *.SomeClass someMethod
```

## 场景解决方案

### 应用启动耗时分析

Arthas 本身支持 [trace](https://arthas.aliyun.com/doc/trace.html) 和 [profiler](https://arthas.aliyun.com/doc/profiler.html) 功能, 分别用于跟踪每个节点的耗时和生成火焰图

Arthas 常用来分析**运行中**应用的问题，而在 Arthas Attach 上应用的时候，已经错过了跟踪应用启动过程

通用解决方案: 通过 debug 模式暂停应用的启动，等 Arthas attach 好了，再往下运行

如果是在本地 IDE 运行，可以直接在 `main` 入口方法打断点，然后通过 IDE debug 模式启动，进入断点后，使用 Arthas 连接，然后再往下执行

如果是在服务器，可以在引用的启动参数中添加以下的 JVM 参数开启 debug 模式

```bash
# dt_socket：使用 socket 进行传输，address：监听的端口，server: y 代表启动的 JVM 是被调试者，
# suspend：y 代表启动的 JVM 会暂停的等待，直到调试器连接上
-agentlib:jdwp=transport=dt_socket,address=8000,server=y,suspend=y
```

这时启动应用，会 Block 住，等待调试程序的连接

这时启动 Arthas attach 该应用，并进行 profiler 或者 trace

```bash
# 开启 profiler
profiler start -e wall

# trace，这时大部分类都没加载，需要手动加载
# 查看 Classloader
classloader -t
# 是用合适的 Classloader 加载要 trace 的类
classloader -c 18b4aac2 --load org.leaf.SomeClass
# trace
trace org.leaf.SomeClass someMehtod
```

然后取消启动应用的 Block

```bash
jdb -connect com.sun.jdi.SocketAttach:port=8000,hostname=127.0.0.1
```

其它方案: 假如你可以修改代码，在 main 方法入口先 sleep 30 秒，然后打开 Arthas 准备好各种监控的语句，会更简便

## References

- [官方文档](https://arthas.aliyun.com/doc/commands.html)