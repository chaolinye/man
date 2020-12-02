# 网络代理

在一些安全级别比较高的公司，不允许直接访问 GitHub，需要经过代理来访问

> 这类安全代理会监控和限制 GitHub 的访问

Git 代理设置命令:

```bash
git config --global http.proxy http://${user}:${passwod}@proxy.xxx.com
git config --global https.proxy http://${user}:${passwod}@proxy.xxx.com
git config --global http.sslVerify false
```