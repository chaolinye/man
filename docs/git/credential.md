# 凭证存储

> 凭证存储指的是如何存储用户凭证

Git 常见有两种连接方式：SSH 和 HTTP

使用 SSH 连接，通过配置 SSH-key，就可以在不输入用户名和密码的情况下安全地传输数据。

然而，这对 HTTP 协议来说，每个连接都是需要输入用户名和密码的。

对此，Git 提供了一个凭证系统来处理这个事情

## 存储模式

| 存储模式 | 含义 |
| :--: | :--: |
| cache | 将凭证存放在内存中一段时间。 密码永远不会被存储在磁盘中，并且在 `15` 分钟后从内存中清除。 |
| store | 将凭证用明文的形式存放在磁盘中，并且永不过期。 这意味着除非你修改了你在 Git 服务器上的密码，否则你永远不需要再次输入你的凭证信息。 这种方式的缺点是你的密码是用明文的方式存放在你的 `home` 目录下的 `git-credentials` 文件中 |
| osxkeychain | Mac 系统特有，会将凭证缓存到你系统用户的钥匙串中。 这种方式将凭证存放在磁盘中，并且永不过期，但是是被加密的，这种加密方式与存放 HTTPS 凭证以及 Safari 的自动填写是相同的。|

!> `store` 模式存在用户账号密码泄露的风险

> 真实开发工作中尽量采用 SSH 连接方式

## 命令行

```bash
# 将凭证存放在内存中 30 分钟
git config --global credential.helper cache --timeout 1800

# 将凭证保存在磁盘，并指定凭证存储文件
git config --global credential.helper stroe --file ~/.my-credentials
```

## Reference

- [Pro Git 2](https://bingohuang.gitbooks.io/progit2/content/07-git-tools/sections/credentials.html)