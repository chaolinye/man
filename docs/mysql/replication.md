# 主从复制和数据迁移

## 主从复制

### 用途

- 高可用

    > 主-主 + keepalived

- 读写分离

    > 主-从

- 数据备份

    > 主-从(热备)，延时从/双延时从(防止删库)

### 原理

![](../images/mysql-replication.jpeg)

可以分为二个阶段:

1. 主库记录 binlog

    主库开启记录 binlog, 在每次准备提交事务完成数据更新前，主库将数据更新事件记录到到 binlog 中

2. 备份

    1. 备库启动一个 I/O 工作线程，跟主库建立一个客户端连接，主库启动一个 binlog dump 线程，读取 binlog 发送给备库，备库 I/O 线程将接受到的日志写在中继日志(relaylog)
    2. 备库的 SQL 线程从 relaylog 中读取事件并执行。

### binlog 的格式

- Statement: 

    记录更新的 sql 语句

    - 缺点: 备库需要解析 sql 语句，性能差
    - 优点: 方便排查复制中失败的语句

- Row 

    记录更新的数据列

    - 

> 一般情况下 Row 优于 Statement

### 操作流程

1. 创建用于复制的账号

```mysql
GRANT REPLICATION SLAVE, REPLICATION CLIENT ON *.* TO repl@'192.168.0.%' IDENTIFIED BY 'p4ssword';
```

> 在主备库上都创建该账号，`REPLICATION SLAVE` 权限用于主库，`REPLICATION CLIENT` 权限用于备库  
> 主备库同时创建包含两个权限的账号，方便管理和切换主备关系

2. 配置主库

在 `/etc/mysql/my.cnf` 下配置，配置完成后重启 mysql

```ini
[mysqld]
# 在主备库关系中保证唯一
server_id = 1
# 开启 binlog 日志
log_bin = mysql-bin
# 设置 binlog 模式为 row 模式
binlog_format = row
# 设置记录 binlog 的数据库，不设置时就都记录
binlog_do_db = db1, db3
# 设置不记录 binlog 的数据，不设置时就都记录
binlog_ignore_db = db2
```

3. 配置备库

在 `/etc/mysql/my.cnf` 下配置，配置完成后重启 mysql

```ini
[mysqld]
log_bin = mysql-bin
server_id = 2
relay_log = /var/lib/mysql/mysql-relay-bin
# 设置同步的事件也写入自己的 binlog，以实现多级备库
log_slave_updates = 1
# 设置只读，一般可以不设置
read_only = 0
# 设定需要复制的数据库（多数据库使用逗号，隔开）
replicate-do-db = 
# 设定需要忽略的复制数据库 （多数据库使用逗号，隔开）
replicate-ignore-db = 
# 设定需要复制的表
replicate-do-table = 
# 设定需要忽略的复制表
replicate-ignore-table =
# 同 replication-do-table 功能一样，但是可以通配符
replicate-wild-do-table = 
# 同replication-ignore-table功能一样，但是可以加通配符
replicate-wild-ignore-table 
```

4. 启动复制

在备库运行以下命令

```bash
# 设置主库及同步信息
CHANGE MASTER TO MASTER_HOST='source2.example.com', MASTER_USER='replication', MASTER_PASSWORD='password',MASTER_PORT=3306,
MASTER_LOG_FILE='mysql-bin.001',MASTER_LOG_POS=4;
# 开始复制
start slave;
# 检查复制信息
show slave status\G
# 检查 I/O 线程和 SQL 线程
show processlist\G
```

主库

```bash
# 检查备库的连接
show processlist\G
```

### 复制已有数据库

在实际工作中，复制已有数据库是更为常见的场景。

操作流程如下:

1. 创建账号以及主备库的配置和前面一样

2. 使用 mysqldump 将主库已有数据导入备库

```bash
# single-transaction 表示 dump 放在一个事务中，这样新增或者删除的数据就不会被 dump，保证数据的一致性
# master-data 表示生成的 dump 文件中包含了 CHANGE MASTER 语句来设置正确的 logfile 和 position
mysqldump --single-transaction --all-databases --master-data=1 --host=server1 | mysql --host=server2
```

3. 启动复制(和前面一样)

### 相关命令

- 主库

    ```bash
    # 查看当前所写的 binlog 文件及其偏移量
    show master status\G
    # 删除主库的所有 binlog，重置 binlog index 文件
    reset master
    ```

- 备库

    ```bash
    # 查看主从复制信息
    show slave status\G
    # 设置主库及同步信息
    CHANGE MASTER TO MASTER_HOST='source2.example.com', MASTER_USER='replication', MASTER_PASSWORD='password'MASTER_PORT=3306,
    MASTER_LOG_FILE='source2-bin.001',MASTER_LOG_POS=4,MASTER_CONNECT_RETRY=10;
    ```

## 数据迁移

工作中检查会要求数据迁移，比如数据库迁移，数据库上云，单库改分库

### 停机迁移

这种方式最简单

1. 停机或者禁止写入

2. 通过 mysqldump 或者复制数据文件的方式迁移数据即可

### 平滑迁移

### 物理迁移

### 迁移工具

- canal
- XtraBackup

## Reference