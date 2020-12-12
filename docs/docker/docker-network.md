# Docker 网络


## 容器访问宿主机

- Windows: 在宿主机运行 `ipconfig`,找到带有 docker 关键词的网卡 IP 地址，容器通过这个 IP 访问宿主机

- MacOS: 容器通过 `docker.for.mac.host.internal` 这个域名访问宿主机