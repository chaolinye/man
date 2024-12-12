# Python 安装

## 检查安装情况

```bash
# 检查是否安装了 python，可能是 python2
python --version
# 检查是否安装了 python3
python3 --version

# windows 检查是否安装了 python
py --version
```

## Windows

直接[官网](https://www.python.org/downloads/)下载最新 stable 版本

## Linux

```bash
# 查看可安装的 python3 版本
yum search python3
# 安装相应版本的 python3
yum install -y python39
# 关联 python 为 python3
cd /usr/bin && ln -s python3 python
```

## 虚拟环境 Conda

[conda 基本操作及原理](http://www.arclub.cc/?news/346.html)

