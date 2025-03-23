# Git Bash

> Git Bash 是 Git 提供的类 linux 控制台，用于运行 Git 命令，也常用于在 windows 上使用 linux 命令

## Git Bash 中文乱码问题解决

运行以下命令，然后关掉 `Git Bash`, 重新打开

```bash
echo "export LC_ALL=zh_CN.UTF-8" >> /etc/bash.bashrc
```

## Git Bash 美化 prompt 

使用 [oh-my-posh](https://ohmyposh.dev/docs/installation/windows) 

步骤：
1. 通过 powershell 运行以下命令安装 oh-my-posh

```
Set-ExecutionPolicy Bypass -Scope Process -Force; Invoke-Expression ((New-Object System.Net.WebClient).DownloadString('https://ohmyposh.dev/install.ps1'))
```

2. 下载[Nerd Font](https://www.nerdfonts.com/)字体，推荐`IntoneMono Nerd Font`。下载解压后，选择所有文件右键点击安装
3. 在 Windows Terminal -> 设置 -> Git Bash -> 外观 -> 字体，下拉搜索选择上面下载安装的字体
4. 在 `~/.bashrc` 文件中增加以下脚本启用 oh-my-posh 和选择某个主题，主题可在[oh-my-posh网站](https://ohmyposh.dev/docs/themes)挑选。

```bash
eval "$(oh-my-posh init bash --config ~/AppData/Local/Programs/oh-my-posh/themes/iterm2.omp.json)"
```
