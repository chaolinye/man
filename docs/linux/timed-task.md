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

# 删除全部任务，删除单个任务用 crontab -e
crontab -r

# 创建任务, 会打开一个 vim 编辑窗口
crontab -e
```

定时任务内容：每天凌晨 03:00 清理 /home/user/temp/ 文件夹下 **上次访问时间在 24 小时前** 的文件

```
00 03 * * * /usr/sbin/tmpwatch 24 /home/user/temp/
```

### 任务配置

crond 一般从以下三个地方读取配置

> crond 每分钟会重新读取一次配置

- `/etc/crontab`

- `/etc/cron.d/*`

- `/var/spool/cron/*`

> `crontab -e` 创建的任务就是存在 `/var/spool/cron/<usernname>` 文件中

!> `crontab -e` （即 `/var/spool/cron/<username>` 文件）定义任务的格式是 `分 时 日 月 周 指令`  
而 `/etc/crontab` 文件和 `/etc/cron.d/*` 目录下的文件定义任务的格式还需要指定用户，即 `分 时 日 月 周 用户名 指令`

### 权限控制

通过 `/etc/cron.allow` 和 `/etc/cron.deny` 文件进行 cron 的权限控制

> 两个文件的权限控制和 at 的逻辑一致

### 任务执行记录查看

```bash
cat /var/log/cron | grep -i xxx
```

## 重新执行停机期间错过的定时任务

需要用到工具 `anacron`，目前已集成到 `cron` 中

anacron 每个小时被 crond 执行一次，然后 anacron 再去检测相关的排程任务有没有被执行，如果有超过期限的工作在， 就执行该排程任务，执行完毕或无须执行任何排程时，anacron 就停止了。

> `cat /ect/cron.hourly/0anacron` 可查看定时任务触发 anacron

anacron 的配置文件是 `/etc/anacrontab`，默认内容如下

```
# /etc/anacrontab: configuration file for anacron

# See anacron(8) and anacrontab(5) for details.

SHELL=/bin/sh
PATH=/sbin:/bin:/usr/sbin:/usr/bin
MAILTO=root
# the maximal random delay added to the base delay of the jobs
RANDOM_DELAY=45
# the jobs will be started during the following hours only
START_HOURS_RANGE=3-22

#period in days   delay in minutes   job-identifier   command
1	5	cron.daily		nice run-parts /etc/cron.daily
7	25	cron.weekly		nice run-parts /etc/cron.weekly
@monthly 45	cron.monthly		nice run-parts /etc/cron.monthly
```

以 `cron.daily` 这行分析 anacron 的工作流程

- 由 `/etc/anacrontab` 分析到 `cron.daily `这项工作名称的天数为 `1` 天；由 `/var/spool/anacron/cron.daily` 取出最近一次执行 anacron 的时间戳, 和目前的时间比较，若差异天数为 1 天以上(含 1 天)，就准备进行指令；

- 若准备进行指令，根据 `/etc/anacrontab` 的设定，将延迟 `5` 分钟 + `3` 小时(看 `START_HOURS_RANGE` 的设定)；

- 延迟时间过后，开始执行后续指令，亦即 `run-parts /etc/cron.daily` 这串指令；

!> 可见默认只有 `/etc/cron.{daily, weekly, monthly}` 三个目录下的任务才会在停机错过后被 anacron 重新执行

## References

- [鸟叔 Linux 私房菜--例行性工作排程](http://linux.vbird.org/linux_basic/0430cron.php)