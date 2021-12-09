# Kubernetes

容器编排系统，云原生时代的操作系统

> Docker 封装单体应用，Kubernetes 封装分布式系统

## Pod

Pod 是 k8s 中最小的可部署单元

> 如果说容器封装了进程，那么 Pod 就是封装了进程组

同一个 Pod 中的多个容器会默认共享以下名称空间：

- UTS 名称空间
- 网络名称空间
- IPC 名称空间
- 时间名称空间

### Kubernetes 中 Pod 名称空间共享的实现细节

Pod 内部多个容器共享 UTS、IPC、网络等名称空间是通过一个名为 Infra Container 的容器来实现的，这个容器是整个 Pod 中第一个启动的容器，只有几百 KB 大小（代码只有很短的几十行），Pod 中的其他容器都会以 Infra Container 作为父容器，UTS、IPC、网络等名称空间实质上都是来自 Infra Container 容器。

如果容器设置为共享 PID 名称空间的话，Infra Container 中的进程将作为 PID 1 进程，其他容器的进程将以它的子进程的方式存在，此时将由 Infra Container 来负责进程管理（譬如清理僵尸进程）、感知状态和传递状态。

由于 Infra Container 的代码除了注册 SIGINT、SIGTERM、SIGCHLD 等信号的处理器外，就只是一个以 pause()方法为循环体的无限循环，永远处于 Pause 状态，所以也常被称为“Pause Container”。

## kubectl 常用命令行

语法: `kubectl [command] [TYPE] [NAME] [flags]`

```bash
# 列出 pod 资源
kubectl get pod -n [namespace]

# 获取 pod 描述信息
kubectl describe pod [pod-name] -n [namespace]

# 查看容器日志
kubectl logs [pod-name] -c [container-name] -n [namespace]

# 进入容器
kubectl exec [pod-name] -c [container-name] -it bash -n [namespace]
```

> 涉及指定容器的，如果没有指定，就默认会查询主容器

## References

- [kubectl 官方文档](https://kubernetes.io/docs/reference/kubectl/overview/)