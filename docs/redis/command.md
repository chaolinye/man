# 常用命令

## 管理命令

### 服务器状态和配置查询

```bash
# 查询全部
info
# 查询服务器
info server
# 查询复制
info replication
# 查询集群
info cluster
```

### 客户端相关

```bash
# 查询客户端状态和配置
info clients
# 查看当前连接的所有客户端
client list
# kill 某个客户端
client kill <client-id>
```

### 集群相关

```bash
# 集群信息
cluster info
# 集群节点信息
cluster nodes
# key 对应的 slot
cluster keyslot <key>
```

## 数据操作命令