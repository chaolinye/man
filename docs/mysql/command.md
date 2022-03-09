# Mysql 常用命令

## 服务器相关

```bash
# 查看表的信息
show table status like 'user' \G;

# 查看线程信息
show processlist

# 查看各种计数器，一般用于性能剖析
show status
show innodb status

# 开启慢查询日志,并设置阈值
set slow_query_log = 1;
set long_query_time = 0;

# 查看配置文件位置
mysqld -v --help | grep -A 1 'Default options'
```

## 事务相关

```sh
# 查看和设置自动提交设置
show variables like 'autocommit';
set AUTOCOMMIT = 1;

# 设置自动提交
set session transaction isolation level read committed;
```

## 用户权限

```sql
-- 查看当前用户权限
show grants;

-- 查看其他用户权限
show grants for 'username'@'10.%';

-- 查看系统有哪些用户
select User,Host from mysql.user;
```