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

## 科学计算 python 运行环境 -- Jupyter Notebook

[Jupyter安装](https://jupyter.org/install)

```bash
# 安装 Jupyter Notebook
pip install notebook
# 运行 Jupyter Notebook
jupyter notebook
```

常用快捷键：

- `J/K` : 在 Cell 之间上下移动
- `Y/M` : 切换 Cell 为 Code/Markdown 模式
- `Enter` : 进入编辑 Cell
- `ESC` : 退出编辑 Cell
- `Ctrl_Enter` : 运行当前 Cell，Markdown Cell 也需要运行
- `Shift_Enter` : 运行当前Cell并移动到下一个Cell

可以安装 [vim 插件](https://github.com/jupyterlab-contrib/jupyterlab-vim)

```bash
# 安装 vim 插件
pip install jupyterlab-vim
# 重启 jupyter notebook
```
