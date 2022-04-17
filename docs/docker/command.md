# 常用命令

```bash
# 构建镜像
docker build -t <image>:<tag> .
# 修改镜像名称和标签
docker tag <image_id> <image>:<tag>
# 列出镜像
docker images
# 推送镜像到 Docker Registry
docker push <image>:<tag>
# 拉取镜像
docker pull <image>:<tag>
# 搜索镜像
docker search <keyword>
# 查询镜像构建历史
docker history <image>:<tag> --no-trunc
# 查询镜像详细信息
docker inspect <image>:<tag>
# 删除镜像
docker image rm <image_id>

# 运行容器
docker run -d --name=<container_name> -p 8080:80 <image>:<tag> 
# 列出容器
docker ps -a
# 获取容器详细信息
docker inspect <container_name>
# 以 root 身份进入容器
docker exec -it -u root <container-id> bash
# 停止容器
docker stop <container_name>
# 删除已停止的容器
docker rm <container_name>
# 强制删除容器
docker rm -f <container_name>
```