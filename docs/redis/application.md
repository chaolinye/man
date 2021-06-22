# 应用

## 分布式锁

### 初级版本

- 加锁
   
   通过 `set <lockKey> <requestId> ex <expireTime> nx` 来加锁，并指定过期时间

- 解锁

    ```java
    public boolean unlock(String requestId) {
        if (currentRequestId.equals(redisClient.get(lockKey))) {
            redisClient.del(lockKey);
        }
    }
    ```

### lua 解锁版本

初级版本的解锁并不是原子性操作，可能出现 **删掉其它请求的加锁**

lua 解锁

```java
private static final String UNLOCK_SCRIPT = "if redis.call('get',KEYS[1]) == ARGV[1] then return redis.call('del',KEYS[1]) else return 0 end";

public boolean unlock(String requestId) {
    return redisClient.eval(UNLOCK_SCRIPT) != 0;
}
```

### 看门狗版本

上述版本的加锁会存在一个问题：**过期时间应该设置多长**

如果太长，万一当前实例宕机，而锁的过期时间还很长，影响业务
如果太短，可能出现临界区代码还没执行完，就已经过期自动解锁，这就不能保证正确性

这时，可以通过 `watch dog` 来解决这个问题

1. 加锁，设置过期时间 `30s`,同时提交定时任务，`20s` 执行
2. 定时任务执行时，如果当前还持有锁，则刷新过期时间，并提交定时任务

这样就可以解决 **宕机锁过期时间太长** 以及 **代码没执行完却解锁** 的问题

> Redisson 的分布式锁就是采用看门狗来解决这个问题的

### 可重入版本

加锁和解锁都通过 lua 脚本实现

加锁时在值中添加 count 信息，如果锁的持有者是当前请求则 count + 1

解锁时 count - 1，如果 count == 0 了则删除 key

### RedLock

在 Redis 集群模式下，如果出现锁还没同步到从库，主库就宕机了，就会出现两个请求持有锁。

RedLock 方案：

在多于一个的奇数（3，5）集群加锁，多数集群加锁成功才算加锁成功

