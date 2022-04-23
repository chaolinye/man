# 访问 Kubernetes 的元数据

## Downward API

[通过环境变量将 Pod 信息呈现给容器](https://kubernetes.io/zh/docs/tasks/inject-data-application/environment-variable-expose-pod-information/)
[通过文件将 Pod 信息呈现给容器](https://kubernetes.io/zh/docs/tasks/inject-data-application/downward-api-volume-expose-pod-information/)

Downward API 允许我们通过环境变量或者卷传递 pod 的元数据

## 与 Kubernetes API 服务器交互

[Kubernetes API 文档](https://kubernetes.io/zh/docs/reference/using-api/api-concepts/#retrieving-large-results-sets-in-chunks)

Downward API 仅仅可以暴露 Pod 自身的元数据，如果要获取集群中其它资源的信息，就得通过访问 API 服务器了

获取 API 服务器地址 `kubectl cluster-info`

由于访问 API 服务器需要认证，直接通过 curl 访问会认证不通过

调试时，可以通过 `kubectl proxy` 开启代理，然后用 curl 访问代理来和 API 服务器交互

在 Pod 中访问可以通过以下命令来访问

```bash
# 信任 API server 的根证书
export CURL_CA_BUNDLE=/var/run/secrets/kubernetes.io/serviceaccount/ca.crt
# 获取鉴权 token
TOKEN=$(cat /var/run/secrets/kubernetes.io/serviceaccount/token)
# API server 会通过 Service 对象暴露，Service 的名称就是 kubernetes
curl -H "Authorization: Bearer $TOKEN" https://kubernetes
```

如果每个 Pod 都需要做这些鉴权步骤，那就会比较繁琐

参考 `kubectl proxy` 的思路，可以用一个 sidecar 容器作为代理处理这些步骤，主容器只需要访问 sidecar 容器的代理接口即可

```yaml
apiVersion: v1
kind: Pod
metadata:
    name: curl-with-sidecar
spec:
    containers:
    - name: main
        image: tutum/curl
        commmand: ["sleep", "999999"]
    - name: proxy
        image: luksa/kubectl-proxy:1.6.2
```

> 在实际操作中，可以使用[客户端库](https://kubernetes.io/zh/docs/reference/using-api/client-libraries/)访问 API Server，可以大大简化编码


