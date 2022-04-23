# 调度机制

[文档](https://kubernetes.io/zh/docs/concepts/scheduling-eviction/)

可以使用下列方法中的任何一种来选择 Kubernetes 对特定 Pod 的调度：

- 与节点标签匹配的 `nodeSelector`
- 亲和性与反亲和性
- `nodeName` 字段

## nodeSelector


## 节点亲和性

节点亲和性概念上类似于 nodeSelector， 它使你可以根据节点上的标签来约束 Pod 可以调度到哪些节点上。 节点亲和性有两种：

- `requiredDuringSchedulingIgnoredDuringExecution`： 调度器只有在规则被满足的时候才能执行调度。此功能类似于 nodeSelector， 但其语法**表达能力更强**。
- `preferredDuringSchedulingIgnoredDuringExecution`： 调度器会尝试寻找满足对应规则的节点。如果找不到匹配的节点，调度器仍然会调度该 Pod。

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: with-node-affinity
spec:
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
        - matchExpressions:
          - key: kubernetes.io/os
            operator: In
            values:
            - linux
      preferredDuringSchedulingIgnoredDuringExecution:
      - weight: 1
        preference:
          matchExpressions:
          - key: another-node-label-key
            operator: In
            values:
            - another-node-label-value
  containers:
  - name: with-node-affinity
    image: k8s.gcr.io/pause:2.0
```

## pod 间亲和性与反亲和性

基于已经在节点上运行的 Pod 的标签来约束 Pod 可以调度到的节点，而不是基于节点上的标签

> 常见场景是希望某些 Pod 运行在同一节点上，或者不要在同一节点上

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: with-pod-affinity
spec:
  affinity:
    podAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
      - labelSelector:
          matchExpressions:
          - key: security
            operator: In
            values:
            - S1
        topologyKey: topology.kubernetes.io/zone
    podAntiAffinity:
      preferredDuringSchedulingIgnoredDuringExecution:
      - weight: 100
        podAffinityTerm:
          labelSelector:
            matchExpressions:
            - key: security
              operator: In
              values:
              - S2
          topologyKey: topology.kubernetes.io/zone
  containers:
  - name: with-pod-affinity
    image: k8s.gcr.io/pause:2.0
```

### nodeName

直接指定某个 Node

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  containers:
  - name: nginx
    image: nginx
  nodeName: kube-01
```

## 污点和容忍

[文档](https://kubernetes.io/zh/docs/concepts/scheduling-eviction/taint-and-toleration/)

节点被设置了污点（Taint）后，只有设置了相应容忍度（Toleration）的 Pod 才会被调度过来

> nodeSelector 是 Pod 选择 Node，而污点是 Node 选择 Pod

常见使用场景: 设置某些节点只给某些 Pod 使用，其它 Pod 不能用

```bash
# 给节点设置污点
kubectl taint nodes node1 key1=value1:NoSchedule
# 移除污点
kubectl taint nodes node1 key1=value1:NoSchedule-
```

给 Pod 添加容忍度

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
  labels:
    env: test
spec:
  containers:
  - name: nginx
    image: nginx
    imagePullPolicy: IfNotPresent
  tolerations:
  - key: "key1"
    value: "value1"
    effect: "NoSchedule"
```

## References

- [为 Pod 和容器管理资源](https://kubernetes.io/zh/docs/concepts/configuration/manage-resources-containers/)