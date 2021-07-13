# Innodb 锁

## 共享锁和排他锁

共享(S)锁(shared lock): 用于 select 操作，多个事务可同时持有一个共享锁

排它(X)锁(exclusive lock): 用于 insert、update 操作，一个排它锁同时只能被一个事务持有，其它事务阻塞等待

## 意向锁

Innodb 支持多粒度锁，即行锁和表锁。

为了同时兼容支持行锁和表锁，Innodb 引入了意向锁(Intention Lock)

意向锁作用于表级别，即当加行锁时，需先加意向锁到表级别

比如 `SELECT ... FOR SHARE` 会在表上加意向共享(IS)锁, `SELECT ... FOR UPDATE` 会在表上加意向排它(IX)锁.

表锁和意向锁的兼容性如下：

|  | X | IX | S | IS |
| :--: | :--: | :--: | :--: | :--: |
|X	| Conflict	| Conflict	| Conflict	| Conflict |
| IX	| Conflict	| Compatible	| Conflict	| Compatible |
| S	| Conflict	| Conflict	| Compatible	| Compatible |
| IS	| Conflict	| Compatible	| Compatible	| Compatible |

## 记录锁

记录锁（Record Lock）是索引记录上的锁，即聚簇索引叶子节点上锁

## 间隙锁

间隙锁 (Gap Lock) 是在记录之间或者第一条记录之前或者最后一条记录之后的锁。

## Next-key Lock

Next-Key Lock 等于 Record Lock + Gap Lock

## 当前读和快照读

## References

- [MySQL InnoDB锁机制全面解析分享](https://segmentfault.com/a/1190000014133576)

