# Select VS Epoll

## Select

1. 使用 `copy_from_user` 从用户空间拷贝 `fd_set` 到内核空间

2. 把当前线程挂到各 fd 对应设备的等待队列中，不同的设备有不同的等待队列。在设备收到一条消息（网络设备）或填写完文件数据（磁盘设备）后，会唤醒设备等待队列上睡眠的进程，这时current便被唤醒了，并给 fd_set 赋值。

3. 如果遍历完所有的fd，还没有返回一个可读写的mask掩码，则会调用schedule_timeout是调用select的进程（也就是current）进入睡眠。当设备驱动发生自身资源可读写后，会唤醒其等待队列上睡眠的进程。如果超过一定的超时时间（schedule_timeout指定），还是没人唤醒，则调用select的进程会重新被唤醒获得CPU，进而重新遍历fd，判断有没有就绪的fd。

4. 把fd_set从内核空间拷贝到用户空间。

缺点:

- 每次调用select，都需要把fd集合从用户态拷贝到内核态，这个开销在fd很多时会很大
- 每次调用select都需要在内核遍历传递进来的所有fd，这个开销在fd很多时也很大
- select支持的文件描述符数量太小了，默认是1024

## Poll

Poll 和 Select 的差别主要是 Poll 使用链表描述 fd_set, 所以 fd_set 没有数量限制

## Epoll

> epoll 可以理解为 event poll

epoll 采用内核态的红黑树来存储 fd，没有数量限制

epoll 每次添加 fd 的时候，就会把 fd 放在内核态的红黑树中，这样就不需要在每次 select 的时候传递全部的 fd，减少了开销

epoll 给每个 fd 注册了回调函数，当 fd 可读写时，回调函数会把 fd 放置在内核态的链表上，这样 epoll 只需要遍历链表即可，也只需要把链表复制到用户态

epoll 分为两种模式:

- LT(Level-triggered)，默认的模式（水平触发） 只要该fd还有数据可读，每次 epoll_wait 都会返回它的事件，提醒用户程序去操作，
- ET(edge-triggered) 是“高速”模式（边缘触发), 可读可写状态变化时触发。

## References

- [一文搞懂select、poll和epoll区别](https://zhuanlan.zhihu.com/p/272891398)