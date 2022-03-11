# 构建文档平台

## 个人文档

### docsify

[官方文档](https://docsify.js.org/#/)

优点:

- 好看

- 运行时 js 编译 Markdown，无需额外的构建工具，只需要一个静态服务器即可

### Gitbook

[入门教程](https://chaolinye.github.io/2018/11/23/gitbook/)

优点：

- 插件众多

## 个人站点

### wordpress

[Docker 构建 wordpress](https://gitee.com/yechaolin/hello-docker/tree/master/wordpress)

## 工程文档

### 工程 README

[README 文件写法建议](../home/hello-readme)

优点:

- 寻找方便

- 无需额外平台

### 团队 wiki

优点:

- 方便统一查看

## 团队 wiki

团队想要长足的发展，知识的沉淀是必不可少的，而知识的沉淀，最有效的方式就是编写文档，这时用 wiki 平台来承载这些文档是很有效的一种方式

### DokuWiki

DokuWiki 是目前最好的开源 wiki 软件，功能简洁强大，插件众多，是快速构建团队 wiki 的不二之选

> 优点：功能完善；
> 缺点：界面样式比较老式，原生不支持 Markdown，需要通过插件和特殊的标签来实现.

[Docker 快速构建 DokuWiki](https://gitee.com/yechaolin/hello-docker/tree/master/dokuwiki)

[官方文档](https://www.dokuwiki.org/page#create_a_page)

支持 Markdown [插件](https://www.dokuwiki.org/plugin:mdpage?s[]=markdown)

### MinDoc

MinDoc 是一款针对 IT 团队开发的简单好用的文档管理系统。

> 特点是简单，适合中小开发团队

[官方文档](https://www.iminho.me/)

Docker 构建

```bash
docker run --name=mindoc --restart=always -e DB_ADAPTER=sqlite3 -e MYSQL_INSTANCE_NAME=./database/mindoc.db -e CACHE=true -e CACHE_PROVIDER=file -e ENABLE_EXPORT=true -p 8181:8181 -d registry.cn-hangzhou.aliyuncs.com/mindoc/mindoc:v0.12
```

### MM-Wiki

> MM-Wiki 一个轻量级的企业知识分享与团队协同软件，可用于快速构建企业 Wiki 和团队知识分享平台。部署方便，使用简单，帮助团队构建一个信息共享、文档管理的协作环境。

> 相对于 DokuWiki 更为现代化，原生支持 Markdown，但是更像是个文档管理平台，不算是完整的 wiki 系统
> 相对于 MinDoc 功能更为强大，更适合多团队的文档管理

[官网地址](https://github.com/phachon/mm-wiki)
[Docker 快速构建 mm-wiki](https://gitee.com/yechaolin/hello-docker/tree/master/mm-wiki)




