# MySQL 技术内幕--InnoDB 存储引擎

## MySQL 体系结构和存储引擎

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


