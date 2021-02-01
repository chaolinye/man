# Docker 网络


## 容器访问宿主机服务

容器中可以通过 `host.docker.internal` 这个 hostname 访问宿主机服务

```bash
curl http://host.docker.internal:8080
```

## 容器内访问 dockerd

- 方法一：挂载 `docker.sock` 到容器内

    ```bash
    # linux
    docker run -itd -v /var/run/docker.sock:/var/run/docker.sock <image:tag>

    # mac or windows wsl
    docker run -itd -v //var/run/docker.sock:/var/run/docker.sock <image:tag>
    ```

    容器内访问 dockerd

    ```bash
    curl --unix-socket /var/run/docker.sock http:/containers/json
    ```

    [dockerd API 文档](https://docs.docker.com/engine/api/latest/)

- 方法二：dockerd 开启 tcp 端口

    ```bash
    dockerd -H tcp://0.0.0.0:2379
    ```

    > 普通的 Windows 宿主机如果不能挂载 docker.sock ， 可以采用这种方式

    容器内访问

    ```bash
    curl http://host.docker.internal:2379/containers/json
    ```