# AQS 原理

AQS 指的是 `AbstractQueuedSynchronizer` 类，用于构建各种同步器、锁的框架。

AQS 提供了原子式管理同步状态、阻塞和唤醒线程功能以及队列模型的能力

核心是

- 通过 CAS 来改变 State

- 通过 LockSupport.park 来阻塞线程，可设置阻塞时间

    > LockSupoort.park 调用 native 方法 UNSAFE.park 来阻塞线程

- 通过队列来管理阻塞的 Thread 对象，加入队列也是通过 CAS

- 通过 LockSupport.unpark(thread) 来唤醒线程

## 公平锁和非公平锁

- 公平锁：多个线程按照申请锁的顺序去获得锁，线程会直接进入队列去排队，永远都是队列的第一位才能得到锁。

    优点：所有的线程都能得到资源，不会饿死在队列中。
    缺点：吞吐量会下降很多，队列里面除了第一个线程，其他的线程都会阻塞，cpu唤醒阻塞线程的开销会很大。

    ```java
    static final class FairSync extends Sync {
            /**
            * Fair version of tryAcquire.  Don't grant access unless
            * recursive call or no waiters or is first.
            */
            @ReservedStackAccess
            protected final boolean tryAcquire(int acquires) {
                final Thread current = Thread.currentThread();
                int c = getState();
                if (c == 0) {
                    if (!hasQueuedPredecessors() &&
                        compareAndSetState(0, acquires)) {
                        setExclusiveOwnerThread(current);
                        return true;
                    }
                }
                else if (current == getExclusiveOwnerThread()) {
                    int nextc = c + acquires;
                    if (nextc < 0)
                        throw new Error("Maximum lock count exceeded");
                    setState(nextc);
                    return true;
                }
                return false;
            }
        }
    ```

- 非公平锁：多个线程去获取锁的时候，会直接去尝试获取，获取不到，再去进入等待队列，如果能获取到，就直接获取到锁。

    优点：可以减少 CPU 唤醒线程的开销，整体的吞吐效率会高点，CPU 也不必取唤醒所有线程，会减少唤起线程的数量。
    缺点：你们可能也发现了，这样可能导致队列中间的线程一直获取不到锁或者长时间获取不到锁，导致饿死。

    ```java
        @ReservedStackAccess
        final boolean nonfairTryAcquire(int acquires) {
            final Thread current = Thread.currentThread();
            int c = getState();
            if (c == 0) {
                if (compareAndSetState(0, acquires)) {
                    setExclusiveOwnerThread(current);
                    return true;
                }
            }
            else if (current == getExclusiveOwnerThread()) {
                int nextc = c + acquires;
                if (nextc < 0) // overflow
                    throw new Error("Maximum lock count exceeded");
                setState(nextc);
                return true;
            }
            return false;
        }
    ```

!> 源码上差别在于公平锁获取锁的时候需要判断 `hasQueuedPredecessors`，即是否有排在当前线程前面的 waiter。


## References

- [从ReentrantLock的实现看AQS的原理及应用](https://tech.meituan.com/2019/12/05/aqs-theory-and-apply.html)

