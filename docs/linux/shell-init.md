# Shell 的初始化执行脚本

> 这里使用的 Shell 发行版是 Bash

## 初始化脚本执行次序

- `/etc/profile`: 所有用户的全局配置脚本

- `/etc/priofile.d/*.sh`

- `~/.bash_profile`: 用户的个人配置脚本。**如果该脚本存在，则执行完就不再往下执行**

- `~/.bash_login`: 如果 `~/.bash_profile` 没找到，则尝试执行这个脚本（C shell 的初始化脚本）。**如果该脚本存在，则执行完就不再往下执行**

- `~/.profile`: 如果 `~/.bash_profile` 和 `~/.bash_login` 都没找到，则尝试读取这个脚本（Bourne shell 和 Korn shell 的初始化脚本）

另外 `~/.bash_profile` 里面还会执行 `~/.bashrc`

```bash
# .bash_profile

# Get the aliases and functions
if [ -f ~/.bashrc ]; then
	. ~/.bashrc
fi

# User specific environment and startup programs

PATH=$PATH:$HOME/bin

export PATH
```

## 修改初始化脚本

### 作用于所有用户

Linux 发行版更新的时候，会更新 `/etc` 里面的文件，包括 `/etc/profile`, 因此不建议修改这个文件

因此，如果想修改所有用户的 Shell 环境，就在 `/etc/profile.d` 目录里面新建 `.sh` 脚本

### 作用于当前用户

建议修改 `~/.bash_profile` 文件

> `~/.bash_login` 和 `~/.profile` 属于 Bash 兼容其他 Shell 发行版本的文件

> `~/.bashrc` 在打开子 bash 事会被再次执行

## 退出脚本

`~/.bash_logout` 脚本在每次退出 Shell 时执行，通常用来做一些清理工作和记录工作，比如删除临时文件，记录用户在本次 Shell 花费的时间

## 学习资料

- [Bash 启动环境](https://wangdoc.com/bash/startup.html)