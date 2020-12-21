# 系统服务

Linux 系统服务本质上就是一些后台（`daemon`）进程

> 比如定时任务服务，就是 crond 进程提供的

> 这些后台进程的名称大多以 `d` 结尾

### daemon 管理方式的变化

### 早期的 daemon

Centos 6.x 及之前版本启动系统服务的管理方式被称为 SysV 的 init 脚本程序的处理方式，即系统核心第一支呼叫的程序是 init ，然后 init 去唤起所有的系统所需要的服务，不论是本机服务还是网路服务。

透过 `/etc/init.d/*`, `service`, `chkconfig`, `setup` 等指令来管理服务的启动/关闭/预设启动

具体请 [跳转](./system-service-old)

### systemd 管理的 daemon

从 CentOS 7.x 以后，改用 systemd 这个启动服务管理机制，即系统的第一个进程变成了 `systemd`

使用 systemd 的好处

- 平行处理所有服务，加速开机流程

    > 旧的init启动脚本是『一项一项任务依序启动』的模式，无法利用多核 CPU 的优点，开机速度慢

- 命令简化

    > systemd 全部就是仅有一只 systemd 服务搭配 systemctl 指令来处理，无须其他额外的指令

- 服务相依性的自我检查

    > 如果 B 服务是架构在 A 服务上面启动的，那当你在没有启动 A 服务的情况下仅手动启动 B 服务时， systemd 会自动帮你启动 A 服务

- 依 daemon 功能分类

    > systemd 将服务单位(unit)区分为 service, socket, target, path, snapshot, timer 等多种不同的类型

- 将多个daemons集合成为一个群组

    > target 服务集合了多个 daemon，即是执行某个 target 就可以执行多个daemon

- 向下相容旧有的 init 服务脚本

## systemd 的 unit 分类

systemd 将各服务定义为 unit，而 unit 又分类为service, socket, target, path, timer 等不同的类别，方便管理与维护

| 类型后缀 | 主要服务功能 |
| :--: | :--: | 
|.service|	最常见的类型，主要是系统服务，包括本机服务以及网路服务都是！比较经常被使用到的服务大多是这种类型！|
|.socket|	通过 socket 传递信息的 daemon，使用 socket 类型的服务一般是比较不会被用到的服务，和老的 Super daemon 有点像，一般在有需要的时候才启动 |
|.target|	其实是一群 unit 的集合，执行 target 就是执行一堆其他 .service 或 / 及 .socket 之类的服务 |
|.mount .automount|	系统挂载相关的服务：例如来自网络的自动挂载、NFS 等|
|.path|	某些服务需要侦测某些特定的目录来提供伫列服务，例如最常见的列印服务，就是透过侦测列印伫列目录来启动列印功能 |
|.timer| 循环执行的服务(timer unit)：这个东西有点类似anacrontab 喔！不过是由systemd 主动提供的，比anacrontab 更加有弹性！|

> 常见主要有三个 service, socket, target

## systemctl 管理系统服务

### 启动和关闭服务

```bash
# 启动服务
systemctl start atd.service

# 关闭服务
systemctl stop atd.service

# 重启服务
systemctl restart atd.service

# 查看服务状态
systemctl status atd.service

# 设置服务开机自启动
systemctl enable atd.service

# 取消服务开机自启动
systemctl disable atd.service

# 强制注销服务（无法直接启动，无法依赖启动）
systemctl mask atd.service

# 恢复注销的服务
systemctl unmask atd.service
```

> unit 默认是 service，所以也可以这样子写 `systemctl start atd`

### 观察服务

```bash
# 列出有启动的服务
systemctl list-units

# 列出所有服务
systemctl list-units --all

# 列出某个类型的服务
systemctl list-units --type=service --all

# 查询某个服务是否存在
systemctl list-units --type=service --all | grep xxx

# 查看 socket 服务的 socket 文件位置
systemctl list-sockets
```
### 管理 target unit

systemd 默认是从一个 target 启动，然后启动其 want 的其他服务

这个默认的 target 主要有两个 `multi-user.target`(文本模式)、 `graphical.target`（图形模式）

> 服务器一般都是 `multi-user.target`(文本模式)

> 不同服务下默认启动的服务不一样

```bash
# 查看当前默认启动的target
systemctl get-default 
# 将目前的操作环境改为纯文字模式
systemctl set-default multi-user.target 
# 在不重新开机的情况下，将目前的操作环境改为纯文字模式，关掉图形界面
systemctl isolate multi-user.target
# 重新切回图形
systemctl isolate graphical.target
```

除了上述的 target，还是其它的 target，比如 shutdown.target(执行关机流程) ，systemctl 也对切到常用的 target 提供的简便命令

```bash
systemctl poweroff 系统关机
systemctl reboot    重新开机
systemctl suspend   进入暂停模式
systemctl hibernate 进入休眠模式
systemctl rescue    强制进入救援模式
systemctl emergency 强制进入紧急救援模式
```

### 分析依赖关系

语法: `systemctl list-dependencies [unit] [--reverse]`

```bash
# 查看当前 default target 的依赖
systemctl list-dependencies 

# 查看当前 default target 的 wantBy
systemctl list-dependencies --reverse

# 查看某个服务的 wantBy
systemctl list-dependencies atd.service --reverse
```

## systemd 的配置目录

- `/usr/lib/systemd/system/`：每个服务最主要的启动脚本设定，有点类似以前的/etc/init.d底下的档案；

- `/run/systemd/system/`：系统执行过程中所产生的服务脚本，这些脚本的优先序要比/usr/lib/systemd/system/高

- `/etc/systemd/system/`：管理员依据主机系统的需求所建立的执行脚本，其实这个目录有点像以前 /etc/rc.d/rc5.d/Sxx 之类的功能！执行优先序又比 /run/systemd/system/ 高

> 一般程序安装时的会把服务脚本安装在 `/usr/lib/systemd/system/`

> 修改服务脚本或者自定义服务脚本应该在 `/etc/systemd/system/`

系统服务的常用配置目录

- `/etc/sysconfig/*`: 定义服务启动时需要的环境参数

- `/var/lib/`: 服务运行产出的数据存储的地方

- `/run/`: lock file 以及 PID file 的暂存档

systemd 配置目录文件格式介绍请 [跳转](http://linux.vbird.org/linux_basic/0560daemons.php#systemd_cfg)

> 可以通过 `man systemd.unit`, `man systemd.service`, `man systemd.timer` 查询 `/etc/systemd/system/` 底下配置服务文件的语法

## 创建自己的服务

创建一个 shell script 程序

```bash
vim /backups/backup.sh 
```

```bash
#!/bin/bash
source="/etc /home /root /var/lib /var/spool/{cron,at,mail}"
target="/backups/backup-system-$(date +%Y-%m-%d).tar.gz"
[ ! -d /backups ] && mkdir /backups
tar -zcvf ${target} ${source} &> /backups/backup.log
```

添加可执行权限

```bash
chmod a+x /backups/backup.sh 
```

创建服务启动脚本

```bash
vim /etc/systemd/system/backup.service 
```

内容

```ini
[Unit]
Description=backup my server
#因为ExecStart里面有用到at这个指令，因此， atd.service就是一定要的服务！
Requires=atd.service

[Service]
Type=simple
ExecStart=/bin/bash -c " echo /backups/backup.sh | at now"

[Install]
WantedBy=multi-user.target 
```

```bash
# systemctl 重新导入服务配置
systemctl daemon-reload 

# 启动服务
systemctl start backup.service 

# 查看服务状态
systemctl status backup.service
```

## References

- [鸟叔 Linux 私房菜--认识系统服务](http://linux.vbird.org/linux_basic/0560daemons.php)