# Linux

- 查看文件和进程的关联

    ```bash
    # lsof(list open files)
    # 查看某个文件被哪些进程在读写
    lsof filename

    # 查看某个进程打开了哪些文件
    lsof -p pid
    ```

- 查看 Linux 版本

    ```bash
    # 查看内核版本
    cat /proc/version
    uname -a

    # 查看发行版本
    cat /etc/os-release
    ```

- 查看程序执行时间

    ```bash
    time <program>
    ```

- 查看本机公网IP

    ```bash
    curl cip.cc
    ```

- 查看系统信息

    ```bash
    # 查看系统服务
    systemctl

    # 查看当前登录的用户
    loginctl

    # 查看本地化设置
    localectl

    # 查看当前时区设置
    timedatectl

    # 查看当前主机的信息
    hostnamectl

    # 查看启动耗时
    systemd-analyze
    ```

- 创建目录并进入目录

    ```bash
    # $_ 获取上一个命令的最后一个参数
    mkdir -p foo/bar && cd $_
    ```

- 脚本排错、快速失败

    ```bash
    #!/usr/bin/env bash

    # 遇到不存在变量报错
    set -u

    # 打印出每一行执行的命令
    set -x

    # 脚本只要发生错误，就终止执行
    set -e

    # 只要一个子命令失败，整个管道命令就失败，脚本就会终止执行
    set -eo pipefail

    echo $a
    echo bar

    foo | echo a
    ```

    合并设置

    ```bash
    # 写法一
    set -euxo pipefail

    # 写法二
    set -eux
    set o pipefail
    ```

    > 所有的 Bash 脚本的头部建议都加上这些设置

- 分割大文件

    > 场景：文件传输有单文件大小限制，比如 github 单文件不能超过 100M

    ```bash
    # 分割大文件成 bigfile-aa, bigfile-bb 等小文件
    split -b 50m bigfile bigfile-

    # 合并小文件
    cat bigfile-* > bigfile
    ```

## References

- [Bash Reference Manual](http://www.gnu.org/software/bash/manual/html_node/)

- [Bash 脚本教程](https://wangdoc.com/bash/)