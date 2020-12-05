# Git Bash 中文乱码问题解决

> Git Bash 是 Git 提供的类 linux 控制台，用于运行 Git 命令，也常用于在 windows 上使用 linux 命令

运行以下命令，然后关掉 `Git Bash`, 重新打开

```bash
echo "export LC_ALL=zh_CN.UTF-8" >> /etc/bash.bashrc
```