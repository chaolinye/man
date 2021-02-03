## 镜像 tag 说明

Docker 镜像的 tag 可能包含某些关键词，这些约定俗成的关键词可以辅助我们选择合适的镜像

| 关键词 | 含义 | 例子 |
| :--: | :--: | :--: |
| stretch | 表明这个镜像的操作系统是debian9 | 8-jre-stretch |
| alpine | 表明镜像的操作系统是alpine linux，alpine linux 镜像很小，只有 5M 左右 | 13-ea-19-jdk-alpine3.9 |
| slim | 表明镜像为了减少体积对功能做了缩减，并不包含应用的全功能 | 8-jre-slim |