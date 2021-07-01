# AQS 原理

AQS 指的是 `AbstractQueuedSynchronizer` 类，用于构建各种同步器、锁的框架。

AQS 提供了原子式管理同步状态、阻塞和唤醒线程功能以及队列模型的能力

核心是

- 通过 CAS 来改变 State

- 通过 LockSupport.park 来阻塞线程，可设置阻塞时间

    > LockSupoort.park 调用 native 方法 UNSAFE.park 来阻塞线程

- 通过队列来管理阻塞的 Thread 对象，加入队列也是通过 CAS

- 通过 LockSupport.unpark(thread) 来唤醒线程

## References

- [从ReentrantLock的实现看AQS的原理及应用](https://tech.meituan.com/2019/12/05/aqs-theory-and-apply.html)

