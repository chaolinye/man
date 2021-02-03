
# Docker 容器服务发现

无论是在本地，还是服务器上的 Docker，往往都会运行着众多的容器

如果想要从外部访问这个容器中的服务，最简单的做法是将容器中的服务端口绑定在宿主机的某个端口上

但是当容器较多时，宿主机上被绑定的端口过多导致记忆困难

这时可以构建一个 `反向代理`，来统一处理请求，然后转发到相应的容器服务

`Traefik` 就是一个云原生的反向代理软件，支持 Docker 服务发现、k8s 服务发现

> 即使用 Traefik 时，新增 docker 容器，会被 Traefik 自动发现，无需重启，就可以通过 Traefik 代理新容器的访问请求

## 实践

1. 首先构建 `Tracefik` 服务

    ```yaml
    version: '3.7'

    services:

    traefik:
        container_name: traefik
        image: traefik:v2.1.3
        restart: always
        ports:
        - 80:80
        - 443:443
        networks:
        - traefik
        command: traefik --configFile /etc/traefik.toml
        volumes:
        - /var/run/docker.sock:/var/run/docker.sock:ro
        - ./traefik.toml:/etc/traefik.toml:ro
        healthcheck:
        test: ["CMD-SHELL", "wget -q --spider --proxy off localhost:8080/ping || exit 1"]

    # 先创建外部网卡
    # docker network create traefik
    networks:
    traefik:
        external: true
    ```

    `traefik.toml`

    ```toml
    [global]
    checkNewVersion = false
    sendAnonymousUsage = false

    [log]
    level = "WARN"
    format = "common"

    [api]
    dashboard = true
    insecure = true

    [ping]

    [accessLog]

    [providers]
    [providers.docker]
        watch = true
        exposedByDefault = false
        endpoint = "unix:///var/run/docker.sock"
        swarmMode = false
        useBindPortIP = false
        network = "traefik"

    [entryPoints]
    [entryPoints.http]
        address = ":80"
    ```

2. 构建应用

    > 应用要加上 traefik 要求的 labels，以提供给 tracefix 动态配置

    ```yaml
    version: '3'
    services:
    dozzle:
        container_name: 'dozzle'
        image: 'amir20/dozzle:latest'
        ports:
        - '18888:8080'
        volumes:
        - '/var/run/docker.sock:/var/run/docker.sock:ro'
        networks:
        - traefik
        labels:
        - "traefik.enable=true"
        - "traefik.docker.network=traefik"
        - "traefik.http.routers.dozzle-web.entrypoints=http"
        - "traefik.http.routers.dozzle-web.rule=Host(`dozzle.ye.com`)"
        - "traefik.http.services.dozzle-web-backend.loadbalancer.server.scheme=http"
        - "traefik.http.services.dozzle-web-backend.loadbalancer.server.port=8080"

    networks:
    traefik:
        external: true

    ```

    配置的含义就是要求 traefik 将 Host 为 `dozzle.ye.com` 的请求转发到 dozzle 容器的 8080 端口

3. 配置 `/etc/hosts` 或者 DNS

    配置 hosts

    ```bash
    sudo echo '127.0.0.1 dozzle.ye.com' >> /etc/hosts
    ```

    如果容器很多，需要经常改动 hosts ，也是件烦人的事情。由于 `/etc/hosts` 不支持通配符，只能通过 DNS 来统一配置

    Mac OS

    ```bash
    # 安装 dns 软件
    brew install dnsmasq
    # 在 dns 配置文件配置所需的规则
    echo 'address=/.ye.com/127.0.0.1' >> $(brew --prefix)/etc/dnsmasq.conf
    # 启动 dns 软件
    sudo brew services start dnsmasq
    # 配置 `ye.com` 域名解析使用本机的 dns 软体
    sudo mkdir -v /etc/resolver && sudo bash -c 'echo "nameserver 127.0.0.1" > /etc/resolver/ye.com'
    ```

## References

- [使用服务发现改善开发体验](https://soulteary.com/2018/06/11/use-server-side-discovery-improve-development.html)
- [Traefik 2 使用指南，愉悦的开发体验](https://juejin.cn/post/6844904053722316813)
- [Traefik 官方文档](https://doc.traefik.io/traefik/)
- [一文搞懂 Traefik2.1 的使用](https://www.qikqiak.com/post/traefik-2.1-101/)
- [Mac OS 安装 DNS](https://gist.github.com/ogrrd/5831371)