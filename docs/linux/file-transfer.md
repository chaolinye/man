# 文件传输工具

## rsync

rsync 是一个常用的 Linux 应用程序，用于文件同步

它可以在本地计算机与远程计算机之间，或者两个本地目录之间同步文件。它也可以当作文件复制工具，替代 cp 和 mv 命令。

> rsync 其实就是 remote sync （远程同步） 的意思

与其他的文件传输工具（如 FTP 和 scp）不同，rsync 的最大特点是会 **检查发送方和接受方已有的文件，仅传输有变动的部分**（默认规则是文件大小或修改时间有变动）

> 在数据量较大时，可以减少大量传输时间

安装方法

```bash
# CentOS
yum install rsync

# Ubuntu
apt-get install rsync
```

!> 传输的双方都必须安装 rsync

常用命令:

```bash
# 本地同步， -r 指递归
rsync -r source destination

# 同步多个源目录到一个目录
rsync -r source1 source2 destination

# -a, 同步源信息
rsync -a source destination

# 同时删掉不存在于源目录的文件，即目标文件和源文件会完全一致
rsync -av --delete source/ destination

# 排除某些文件
rsync -av --exclude='*.txt' source/ destination

# 远程同步，使用 ssh 协议
rsync -av source/ username@remote_host:destination

# 远程同步使用 rsync 协议，需要远程服务器运行 rsync 守护程序
rsync -av source/ 192.168.2.2::module/destinationn
```

> rsync 协议需要远程服务器运行 rsync daemon，而 ssh 协议更加的方便，建议使用 ssh 协议传输

## scp

Linux scp 命令用于 Linux 之间复制文件和目录

scp 是 secure copy 的缩写，scp 是 linux 系统下基于 ssh 登录进行安全的远程文件拷贝命令

> scp 是加密的

> 大部分 Linux 中，scp 是默认安装的，这也是 scp 优势于 rsync 的地方

常用命令

```bash
# 复制本地文件到远端
scp local_file remote_username@remote_ip:remote_path

# 复制本地目录到远端
scp -r local_folder remote_username@remote_ip:remote_path

# 从远端复制到本地
scp -r remote_username@remote_ip:remote_path local_path
```

## ftp

> TODO

## nfs

> TODO

## References

- [rsync 用法教程](http://www.ruanyifeng.com/blog/2020/08/rsync.html)

- [NFS 服务器](http://cn.linux.vbird.org/linux_server/0330nfs.php)

