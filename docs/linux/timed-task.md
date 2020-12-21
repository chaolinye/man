# 定时任务

## 分类

- 周期性任务：每隔一定的周期都要执行的任务

- 一次性任务：执行一次的任务

## 一次性任务

一次任务常用 `at` 工具

### 安装和启动 at

> linux 一般预设了 `at`

```bash
# 查看是否存在 atd 服务
systemctl status atd

# 安装 at
yum install -y at

# 启动 atd 服务
systemctl start atd

# 设置 atd 服务开机自启动(可选)
systemctl enable atd

# 查看 atd 服务状态
systemctl status atd
```

### 常用命令

```bash
# 查看 at 任务列表
at -l

# 创建个 5 分钟后执行的任务
at now + 5 minutes
at > echo '2' > /root/b.txt # 编写任务内容，可以多行
at > <EOT> # Ctrl + d 结束任务编写

# 查看某个任务内容
at -c <jobnumber>
```

创建任务的语法

```bash
at <TIME>
```

```
TIME：時間格式，這裡可以定義出『什麼時候要進行 at 這項工作』的時間，格式有：
  HH:MM				ex> 04:00
	在今日的 HH:MM 時刻進行，若該時刻已超過，則明天的 HH:MM 進行此工作。
  HH:MM YYYY-MM-DD		ex> 04:00 2015-07-30
	強制規定在某年某月的某一天的特殊時刻進行該工作！
  HH:MM[am|pm] [Month] [Date]	ex> 04pm July 30
	也是一樣，強制在某年某月某日的某時刻進行！
  HH:MM[am|pm] + number [minutes|hours|days|weeks]
	ex> now + 5 minutes	ex> 04pm + 3 days
	就是說，在某個時間點『再加幾個時間後』才進行。
```

### at 的权限控制

通过 `/etc/at.allow` 和 `/etc/at.deny` 文件进行 at 的权限控制

- 如果 `at.allow` 文件有值，则只有该文件中的用户才能使用 at

- 如果 `at.allow` 不存在或者没值, 而 `at.deny` 文件中有值，则除了 `at.deny` 文件中的用户，其它用户都能使用 at

- 如果两个文件都不存在，则只有 root 用户可以使用 at

### 系统空闲时执行任务

batch 基于 at， 不过 batch 可在 CPU 工作负载小于 0.8 时才执行任务

> 安装 at 的时候自动安装了 batch

batch 创建任务

```bash
[root@study ~]# batch 
at> /usr/bin/updatedb
at> <EOT>
job 4 at Thu Jul 30 19:57:00 2015
```

> batch 不支持设置执行时间，而是在 CPU loaded 小于 0.8 时就会执行

## 周期性任务

Linux 实现周期性任务的工具是 `cron`

### 安装和启动 cron

> Linux 一般都预设了 cron

```bash
# 查看是否存在 crond 服务
systemctl status crond

# 安装 cron
yum install -y crontabs

# 启动 crond 服务
systemctl start crond

# 设置开机自启动
systemctl enable crond

# 查看 crond 服务状态
systemctl status crond
```

### 常用命令

```bash
# 查看任务列表
crontab -l

# 创建任务
crontab -e
```

### 权限控制

通过 `/etc/cron.allow` 和 `/etc/cron.deny` 文件进行 cron 的权限控制

> 两个文件的权限控制和 at 的逻辑一致

## References

- [鸟叔 Linux 私房菜--例行性工作排程](http://linux.vbird.org/linux_basic/0430cron.php)