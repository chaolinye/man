# 常用命令

[官方命令手册](https://docs.docker.com/engine/reference/commandline/docker/)


## 镜像相关


```bash
# 通过 Dockerfile 构建镜像
docker build -t <image>:<tag> .
# 将容器生成镜像
docker commit <container_id> <image>:<tag>
# 保存镜像到文件中
docker image save <image> > xxxImage.tar
# 从文件中加载镜像
docker image load -i xxxImage.tar
# 删除镜像
docker image rm <image_id>
# 删除没有名称的镜像
docker rmi ${docker images -f "dangling=true" -q}
# 删除没有引用的中间镜像
docker image prune
# 列出已下载镜像列表（默认存储位置：/var/lib/docker/image/）
docker images
# 查询镜像构建历史
docker history <image>:<tag> --no-trunc
# 查询镜像详细信息
docker inspect <image>:<tag>
# 修改镜像名称和标签
docker tag <image_id> <image>:<tag>
# 搜索镜像
docker search <keyword>
# 拉取镜像
docker pull <image>:<tag>
# 推送镜像到 Docker Registry
docker push <image>:<tag>
```

## 容器相关


```bash
# 列出运行中的容器
docker ps
# 列出所有容器
docker ps -a
# 生成并启动容器
docker run -d --name=<container_name> -p 8080:80 <image>:<tag> 
# 以 root 身份进入容器
docker exec -it -u root <container-id> bash

# 获取容器详细信息
docker inspect <container_name>

# 停止容器
docker stop <container_name>
# 启动停掉的容器，常见场景：docker daemon 挂掉后重启之前的容器
docker start <container_id>
# 重启容器，常见场景：修改了主程序 pid=1 的配置，重启让配置生效
docker restart <container_id>

# 删除已停止的容器
docker rm <container_name>
# 强制删除容器
docker rm -f <container_name>

# 将容器生成镜像
docker commit <container_id> <image>:<tag>
```

## volume 相关

```bash
# 列出已创建的 volume（默认存储位置：/var/lib/docker/volumes/）
docker volume ls
# 删除 volume
docker volume rm <volumn_name>
```

## 常见场景

### 基于当前场景下载某些依赖生成新的镜像

使用场景：某些环境机器不能访问外网，如果镜像中需要下载外部依赖，则会失败，这时可以在能访问外网的机制基于当前镜像下载好相应依赖再生成新的镜像。

操作步骤：

- 编写 `Dockerfile`

    ```docker
    FROM <old_image_name>:<old_image_tag>
    # 运行下载依赖的命令
    RUN apt-get install xxx
    ```

- 在可访问外网的机器构建新镜像

    ```bash
    docker build -t <new_image_name>:<new_image_tag>
    ```

- 将镜像保持到文件中

    ```bash
    docker image save <new_image_id> > image.tar
    ```

- 将文件上传到运行机制
- 加载新镜像

    ```bash
    docker image load -i image.tar
    docker image tag <new_image_id> <new_image_name>:<new_image_tag>
    ```