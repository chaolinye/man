# MySQL 技术内幕--InnoDB 存储引擎

## MySQL 体系结构和存储引擎

### 定义数据库和实例

数据库：物理操作系统文件或其他形式文件类型的集合。在MySQL数据库中，数据库文件可以是 frm、MYD、MYI、ibd结尾的文件。

实例：MySQL 数据库由后台线程以及一个共享内存区组成。

从概念上来说，数据库是文件的集合，是依照某种数据模型组织起来并存放于二级存储器中的数据集合；数据库实例是程序，是位于用户和操作系统之间的一层数据管理软件，用户对数据库数据的任何操作，包括数据库定义、数据查询、数据维护、数据库运行控制等都是在数据库实例下进行的，应用程序只能通过数据库实例才能和数据库打交道。

MySQL 被设计为一个单进程多线程架构的数据库。也就是说 MySQL 数据库实例在系统上的表现就是一个进程。

```bash
# 启动 mysql 进程
./mysqld_safe &
# 查看 mysql 进程
ps -ef | grep mysqld
# 查看当 mysql 数据库实例启动时，会在哪些位置查找配置文件。
# Linux 中默认中 /etc/my.cnf -> /etc/mysql/my.cnf -> /usr/local/mysql/etc/my.cnf -> ~/my.cnf
# 如果多个配置文件存在同一个参数，后者会覆盖前者
mysql --help | grep my.cnf
```

配置文件中有一个参数 `datadir`，该参数指定了数据库所在的路径。Linux 中默认是 `/usr/local/mysql/data/`

```bash
mysql> show variables like 'datadiar' \G;
mysql> system ls-lh /usr/local/mysql/data
```

### MySQL 体系结构

![](../images/mysql-structure.awebp)

### MySQL 存储引擎

MySQL 数据库区别于其它数据库的最重要的一个特点就是其插件式的表存储引擎。

!> 存储引擎是基于表的，而不是数据库。也就是说不同的表可以选择不同的存储引擎。

> 可以通过 `SHOW ENGINES` 语句查看当前使用的 MySQL 数据库所支持的存储引擎，也可以通过查询 `information_schema.engines` 表

InnoDB 存储引擎将数据放在一个逻辑的表空间中，这个表空间就想黑盒一样由 InnoDB 存储引擎自身进行管理。从 MySQL 4.1 版本开始，可以将每个表存放到独立的 `ibd` 文件中。

InnoDB 通过使用多版本并发控制（MVCC）来获得高并发性，并且实现了 SQL 标准的 4 种隔离级别，默认为 REPEATABLE 级别。同时，使用一种被称为 next-key locking 的策略来避免幻读现象的产生。除此之外，InnoDB 存储引擎还提供了插入缓冲（insert buffer），二次写（double write），自适应哈希索引（adaptive hash index），预读（read ahead）等高性能和高可用的功能。

### 连接 MySQL

常用的进程通信方式有管道、命名管道、TCP/IP套接字、UNIX域套接字。

#### TCP/IP

在通过 TCP/IP 连接时，MySQL 数据库会先检查一张权限视图，用来判断发起请求的客户端IP是否允许连接到 MySQL 实例。

`select host,user,passwork from mysql.user;`

#### 命名管道和共享内存

配置文件中启用 `--enable-named-pipe`

配置文件中启用 `--shared-memory`, 同时 MySQL 客户端还必须使用 `protocol=memory`

#### UNIX 域套接字

可以在配置文件中指定套接字文件的路径，如 `--socket=/tmp/mysql.sock`

查看 UNIX 域套接字文件路径：`show variables like 'socket';`

## InnoDB 存储引擎

### 版本

![](../images/innodb-version.png)

```sql
# 查看 mysql 版本
SELECT VERSION();
# 查看 inndob 版本
SHOW VARIABLES LIKE 'innodb_version'\G;
```

### InnoDB 体系结构

![](../images/innodb-arch.png)

#### 后台线程

主要作用是负责刷新内存池中的数据，保证缓冲池中的内存缓存的是最近的数据。此外将已修改的数据文件刷新到磁盘文件，同时保证在数据库发生异常的情况下 InnoDB 能恢复到正常运行状态。

- Master Thread：非常核心的后台线程，主要负责将缓冲池中的数据异步刷新到磁盘中，保证数据的一致性，包括脏页的刷新（1.2版本后由Page Cleaner Thread 处理）、合并插入缓冲（INSERT BUFFER）、UNDO 页的回收（1.1版本后由Purge Thread处理）。
- IO Thread：负责 AIO 请求的回调处理。有4个种类（write、read、insert buffer、log IO thread）

    ```sql
    # 查看 read、write IO Thread 数量
    SHOW VARIABLES LIKE 'innodb_%io_threads'\G
    # 观察 IO Thread 
    SHOW ENGINE INNODB STATUS\G
    ```
- Purge Thread：回收已经使用并分配的 undo 页。

    ```sql
    SHOW VARIABLES LIKE 'innodb_purge_threads'\G
    ```

- Page Cleaner Thread: 刷新脏页

#### 内存

![](../images/innodb-memory.png)

- 缓冲池

    简单来说就是一块内存区域，通过内存的速度来弥补磁盘速度较慢对数据库性能的影响。
    读取页时，先判断缓冲池有没有，没有再读磁盘。
    修改页时，先修改缓冲池，然后再以一定的频率刷新到磁盘上。（Checkpoint 机制）

    > 默认页大小是 16KB

    ```sql
    # 查看缓冲池大小
    SHOW VARIABLES LIKE 'innodb_buffer_pool_size'\G
    # 1.0.x 版本开始，为了减少资源竞争、增加并发处理能力，允许有多个缓冲池实例。每个页根据哈希值平均分配到不同缓冲池实例中。
    # 查看缓冲池实例数量
    SHOW VARIABLES LIKE 'innodb_buffer_pool_instances'\G

    # 查看缓冲池状态，
    SHOW ENGINE INNODB STATUS \G
    # 或者
    SELECT POOL_ID,POOL_SIZE,FREE_BUFFERS,DATABASE_PAGES FROM information_schema.INNODB_BUFFER_POOL_STATS\G
    ```

    缓冲池中的页通过 LRU 算法管理。并对传统的 LRU 算法做了一点优化：新读取的页并不是放到首部，而是放在 midpoint 位置。midpoint 之前的部分称为 new 列表，midpoint 之后的部分称为 old 列表。新读取的页需要在old列表中存活一段时间才能加入new部分（这个动作称为 page made young，如果最后没有加入，则称为 page not made young）。

    ```sql
    # 查看 midpoint 值，单位是 %。（距离尾部的百分比）。如果预估热点数据较多，可以把这个值调小一点。
    SHOW VARIABLES LIKE 'innodb_old_blocks_pct'\G
    # 新读取的页加入 new 列表的等待时间
    SHOW VARIABLES LIKE 'innodb_old_blocks_time'\G

    # 查看缓冲池状态，主要关心缓冲命中率指标（Buffer pool hit rate） ，通常该值不应该小于 95%，否则就要看看是否是由于全表扫描引起的 LRU 列表被污染的问题
    SHOW ENGINE INNODB STATUS \G
    # 或者
    SELECT POOL_ID,HIT_RATE,PAGES_MADE_YOUNG,PAGES_NOT_MADE_YOUNG FROM information_schema.INNODB_BUFFER_POOL_STATS\G

    # 观察每个LRU列表中每个页的具体信息
    SELECT TABLE_NAME,SPACE,PAGE_NUMBER,PAGE_TYPE FROM information_schema.INNODB_BUFFER_PAGE_LRU WHERE SPACE = 1;
    ```

- 重做日志缓冲

    一般不需要设置得很大，因为一般情况下每一秒会将重做日志缓冲刷新到日志文件，只需要保证每秒产生的事务量在这个缓冲大小之内即可。(通常 8MB 就够了)

    重做日志刷到磁盘的三种情况:
    1. Master Thread 每秒刷一次
    2. 每个事务提交时
    3. 重做日志缓冲剩余空间小于 1/2 时

    ```sql
    # 重做日志缓冲大小配置
    SHOW VARIABLES LIKE 'innodb_log_buffer_size'\G
    ```

- 额外的内存池

    在对一些数据结构本身的内存进行分配时，需要从额外的内存池中进行申请。

### Checkpoint 技术

checkpint 所做的事情是将缓冲池中的脏页刷回磁盘

目的是解决以下问题：

- 缩短数据库的恢复时间
    当数据库宕机时，不需要重做所有日志，因为 checkpoint 之前的页都已经刷新会磁盘。只需要恢复checkpoint后的重做日志。
- 缓冲池不够用时，将脏页刷新到磁盘
- 重做日志不可用时，刷新脏页
    重做日志设计上是一个环形，如果不够用，需要checkpoint刷新脏页

对于 InnoDB 存储引擎而言，其是通过 LSN（Log Sequence Number）来标记版本的，而 LSN 是 8 字节的数字，其单位是字节。每个页有 LSN，重做日志中也有 LSN，Checkpoint 也有 LSN。

Checkpoint的关键在于每次刷新多少页到磁盘，每次从哪里取脏页，以及什么时间触发 Checkpoint。

数据库关闭时默认会将所有脏页都刷新会磁盘。

运行时都只刷新一部分脏页，有以下几种 Checkpoint

- Master Thread Checkpoint
    每秒或者每十秒的频率
- FLUSH_LRU_LIST Checkpoint
    需要保证 LRU 列表中有差不多 100 个空闲页可供使用，通过 `innodb_lru_scan_depth` 控制。新版本在 Page Cleaner 线程中进行，淘汰 LRU 尾端的脏页。
- Async/Sync Flush Checkpoint
    重做日志文件不可用时。从脏页列表中选取。新版本在 Page Cleaner 线程中进行
- Dirty Page too much Checkpoint
    由 `innodb_max_dirty_pages_pct` 控制

### Master Thread 工作方式

InnoDB 存储引擎的主要工作都是在 Master Thread 中完成的

#### InnoDB 1.0.x 版本之前

Master Thread 具有最高的线程优先级。其内部由多个循环组成：主循环（loop），后台循环（backgroup loop），刷新循环（flush loop），暂停循环（suspend loop）。Master Thread 会根据数据库运行的状态在这几个循环中进行切换。

大多数操作都在 loop 中，其中有两大部分：每秒的操作和每十秒的操作

每秒的操作：
- 日志缓冲刷新会磁盘，即使这个事务还没有提交（总是）
- 合并插入缓冲（可能）
- 至多刷新100个InnoDB的缓冲池中的脏页到磁盘（可能）
- 如果当前没有用户活动，则切换到 backgroup loop（可能）

每十秒的操作：
- 刷新 100 个脏页到磁盘（可能）
- 合并至多5个插入缓冲（总是）
- 将日志缓冲刷新到磁盘（总是）
- 删除无用的 Undo 页（总是）
- 刷新 100 个或者 10 个脏页到磁盘（总是）

backgroup loop（没有用户活动或者数据关闭会切到该循环）:

- 删除无用的 Undo 页（总是）
- 合并 20 个插入缓冲（总是）
- 跳回到主循环（总是）
- 不断刷新 100 个页知道符合条件（可能，跳转到 flush loop 中完成）

若 flush loop 中也没有什么事情可以做了，InnoDB 存储引擎会切换到 suspend loop，将 Master Thread 挂起。

Master Thread 的伪代码：

```c
void master_thread() {
    goto loop;
loop:
    for (int i = 0; i < 10; i++) {
        thread_sleep(1)
        do log buffer flush to disk
        if (last_one_second_ios < 5) {
            do merge at most 5 insert buffer
        }
        if (buf_get_modified_ratio_pct > innodb_max_dirty_pages_pct) {
            do buffer pool flush 100 dirty page
        }
        if (no user activity) {
            goto backgroup loop
        }
    }
    if (last_ten_second_ios < 200) {
        do buffer pool flush 100 dirty page
    }
    do merge at most 5 insert buffer
    do log buffer flush to disk
    do full purge
    if (buf_get_modified_ratio_pct > 70%) {
        do buffer pool flush 100 dirty page
    } else {
        buffer pool flush 10 dirty page
    }
    goto loop;
backgroup loop:
    do full purge
    do merge 20 insert buffer
    if not idle:
        goto loop;
    else
        goto flush loop;
flush loop:
    do buffer pool flush 100 dirty page
    if (buf_get_modified_ratio_pct > innodb_max_dirty_pages_pct) {
        goto flush loop
    }
    goto suspend loop
suspend loop:
    suspend_thread()
    waiting event
    goto loop;
}
```

#### InnoDB 1.2.x 版本之前

1.0.x 版本之前对于脏页的刷新个数做了很多硬编码，不适合 SSD。

1.0.x 版本提供了参数 innodb_io_capacity，用来表示磁盘 IO 的吞吐量，默认值为 200。对于刷新到磁盘脏页的数量，规则如下：

- 合并插入缓冲数量为 innodb_io_capacity 的 5%
- 刷新脏页的数量为 innodb_io_capacity

1.0.x 版本 innodb_max_dirty_pages_pct 默认值从 90% 改成 75%

添加 innodb_purge_batch_size 控制每次 full purge 回收的 Undo 页数量，默认是 20

#### InnoDB 1.2.x 版本

伪代码：

```c
if InnoDB is idle
    srv_master_do_idle_tasks();
else
    srv_master_do_active_tasks();
```

其中 `srv_master_do_idle_tasks()` 就是之前版本中每10秒的操作，`srv_master_do_active_tasks()` 处理的是之前每秒中的操作。同时对于刷新脏页的操作，从 Master Thread 线程分离到一个单独的 Page Cleaner Thread，从而减轻了 Master Thread 的工作，同时进一步提高了系统的并发性。

### InnoDB 关键特性

