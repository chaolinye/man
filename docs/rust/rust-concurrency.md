# Rust 并发编程

安全和高效的处理并发是 Rust 语言的主要目标之一。

可惜的是，在 Rust 中由于语言设计理念、安全、性能的多方面考虑，并没有采用 Go 语言大道至简的方式，而是选择了**多线程与 async/await 相结合**，优点是可控性更强、性能更高，缺点是复杂度并不低，当然这也是系统级语言的应有选择：使用复杂度换取可控性和性能。

协程的实现会显著增大运行时的大小，因此 Rust 只在**标准库中提供了 1:1 的线程模型**，如果你愿意牺牲一些性能来换取更精确的线程控制以及更小的线程上下文切换成本，那么可以选择 Rust 中的 **M:N 模型，这些模型由三方库提供了实现**，例如大名鼎鼎的 tokio。

async 和多线程的性能对比

| 操作 | async | 线程 |
| :--: | :--: | :--: |
| 创建 | 0.3us | 17us |
| 线程切换 | 0.2us | 1.7us |

async 和多线程的选择：

- 有大量 IO 任务需要并发运行时，选 async 模型
- 有部分 IO 任务需要并发运行时，选多线程，如果想要降低线程创建和销毁的开销，可以使用线程池
- 有大量 CPU 密集任务需要并行运行时，例如并行计算，选多线程模型，且让线程数等于或者稍大于 CPU 核心数
- 无所谓时，统一选多线程

## 多线程

使用 `thread::spawn` 创建线程, 它接收一个 FnOnce 闭包或函数.

```rust
use std::thread;
use std::time::Duration;

fn main() {
    // 创建线程，返回 JoinHandler 对象
    let handle = thread::spawn(|| {
        for i in 1..5 {
            println!("hi number {} from the spawned thread!", i);
            thread::sleep(Duration::from_millis(1));
        }
    });

    // 等待子线程结束，返回 Result，可用于处理子线程的 panic
    handle.join().unwrap();

    for i in 1..5 {
        println!("hi number {} from the main thread!", i);
        // 线程休眠
        thread::sleep(Duration::from_millis(1));
    }
}
```

> main 线程结束，程序就立即结束了

Rust 中可以捕获子线程的错误：一个线程的 panic 在其他线程中会体现为包含错误的 Result。

> Java 和 C# 的默认行为是将子线程的异常抛到终端，然后就不管了。在 C++ 中，默认行为是中断进程

由于无法确定子线程什么时候结束所以传给线程闭包不能捕获引用，因为无法保证生命周期。如果不能 move 的话，可以使用 `Arc` 原子引用计数

```rust
fn process_files_in_parallel(filenames: Vec<String>, glossary: Arc<GigabyteMap>) -> io::Result<()>
{
    ...
    for worklist in worklists {
        // 这里调用.clone()只是克隆Arc并触发引用计数。并不会克隆 GigabyteMap
        let glossary_for_child = glossary.clone();
        thread_handles.push(
            spawn(move || process_files(worklist, &glossary_for_child))
        );
    }
    ...
}
```

在多线程间有多种方式可以共享、传递数据，最常用的方式就是通过消息传递或者将锁和 Arc 联合使用，

### 消息传递

与 Go 语言内置的chan不同，Rust 是在标准库里提供了消息通道(channel)。

标准库提供了通道 `std::sync::mpsc`。该通道支持多个发送者，但是只支持唯一的接收者

```rust
use std::sync::mpsc;
use std::thread;

fn main() {
    let (tx, rx) = mpsc::channel();
    let tx1 = tx.clone();
    thread::spawn(move || {
        tx.send(String::from("hi from raw tx")).unwrap();
    });

    thread::spawn(move || {
        tx1.send(String::from("hi from cloned tx")).unwrap();
    });

    // 接送者是 Iterable
    for received in rx {
        println!("Got: {}", received);
    }
}
```

> 对于通道而言，消息的发送顺序和接收顺序是一致的，满足FIFO原则(先进先出)。

Rust 标准库的 `mpsc` 通道其实分为两种类型：**同步和异步**。上面的 `channel()` 返回的就是异步通道，通道无穷大。`sync_channel()` 返回的同步通道需要指定大小，如果通道满了，发送消息是阻塞的。

```rust
use std::sync::mpsc;
use std::thread;
use std::time::Duration;
fn main() {
    // 通道大小为 0，每次发送消息都是阻塞的，只有在消息被接收后才解除阻塞
    let (tx, rx)= mpsc::sync_channel(0);

    let handle = thread::spawn(move || {
        println!("发送之前");
        tx.send(1).unwrap();
        println!("发送之后");
    });

    println!("睡眠之前");
    thread::sleep(Duration::from_secs(3));
    println!("睡眠之后");

    println!("receive {}", rx.recv().unwrap());
    handle.join().unwrap();
}
```

> 所有发送者被drop或者所有接收者被drop后，通道会自动关闭。

一个消息通道只能传输一种类型的数据，如果你想要传输多种类型的数据，可以为每个类型创建一个通道，你也可以使用枚举类型来实现

```rust
use std::sync::mpsc::{self, Receiver, Sender};

enum Fruit {
    Apple(u8),
    Orange(String)
}

fn main() {
    let (tx, rx): (Sender<Fruit>, Receiver<Fruit>) = mpsc::channel();

    tx.send(Fruit::Orange("sweet".to_string())).unwrap();
    tx.send(Fruit::Apple(2)).unwrap();

    for _ in 0..2 {
        match rx.recv().unwrap() {
            Fruit::Apple(count) => println!("received {} apples", count),
            Fruit::Orange(flavor) => println!("received {} oranges", flavor),
        }
    }
}
```

### 锁、条件变量、信号量

共享内存可以说是同步的灵魂，因为消息传递的底层实际上也是通过共享内存来实现。

共享内存的优点是简洁、性能高，缺点是容易出错

Rust 的互斥锁是 `std::sync::Mutex`，和其它语言不一样，Rust 把 Mutex 和要保护的数据绑定在一起，这样出错的概率更小，也更清晰地知道这个数据是要保护的。

`mutex.lock()` 方法触发加锁并返回 `LockResult<MutexGuard<'_, T>>`，其中 `MutexGuard` 是数据的智能指针

```rust
use std::rc::Rc;
use std::sync::Mutex;
use std::thread;

fn main() {
    // 通过`Rc`实现`Mutex`的多所有权
    let counter = Arc::new(Mutex::new(0));
    let mut handles = vec![];

    for _ in 0..10 {
        let counter = Arc::clone(&counter);
        // 创建子线程，并将`Mutex`的所有权拷贝传入到子线程中
        let handle = thread::spawn(move || {
            let mut num = counter.lock().unwrap();

            *num += 1;
        });
        handles.push(handle);
    }

    // 等待所有子线程完成
    for handle in handles {
        handle.join().unwrap();
    }

    // 输出最终的计数结果
    println!("Result: {}", *counter.lock().unwrap());
}
```

读写锁： `std::sync::RwLock`

```rust
use std::sync::RwLock;

fn main() {
    let lock = RwLock::new(5);

    // 同一时间允许多个读
    {
        let r1 = lock.read().unwrap();
        let r2 = lock.read().unwrap();
        assert_eq!(*r1, 5);
        assert_eq!(*r2, 5);
    } // 读锁在此处被drop

    // 同一时间只允许一个写
    {
        let mut w = lock.write().unwrap();
        *w += 1;
        assert_eq!(*w, 6);

        // 以下代码会panic，因为读和写不允许同时存在
        // 写锁w直到该语句块结束才被释放，因此下面的读锁依然处于`w`的作用域中
        // let r1 = lock.read();
        // println!("{:?}",r1);
    }// 写锁在此处被drop
}
```

条件变量: `std::sync::Condvar`

> 条件变量始终代表由某个 Mutex 保护的数据或真或假的条件。

> 不如 C++ 的强大，其可以传递闭包来判断，也能解决假唤醒的问题（Rust 还需要自行用循环来解决）

```rust
use std::sync::{Arc,Mutex,Condvar};
use std::thread::{spawn,sleep};
use std::time::Duration;

fn main() {
    let flag = Arc::new(Mutex::new(false));
    let cond = Arc::new(Condvar::new());
    let cflag = flag.clone();
    let ccond = cond.clone();

    let hdl = spawn(move || {
        let mut m = { *cflag.lock().unwrap() };
        let mut counter = 0;

        while counter < 3 {
            while !m {
                // wait 传入 MutexGuard，也把释放互斥量的特权授予你
                m = *ccond.wait(cflag.lock().unwrap()).unwrap();
            }

            {
                m = false;
                *cflag.lock().unwrap() = false;
            }

            counter += 1;
            println!("inner counter: {}", counter);
        }
    });

    let mut counter = 0;
    loop {
        sleep(Duration::from_millis(1000));
        *flag.lock().unwrap() = true;
        counter += 1;
        if counter > 3 {
            break;
        }
        println!("outside counter: {}", counter);
        cond.notify_one();
    }
    hdl.join().unwrap();
    println!("{:?}", flag);
}
```

信号量：

> 本来 Rust 在标准库中有提供一个信号量实现, 但是由于各种原因这个库现在已经不再推荐使用了，因此我们推荐使用tokio中提供的Semaphore实现: `tokio::sync::Semaphore`。

```rust
use std::sync::Arc;
use tokio::sync::Semaphore;

#[tokio::main]
async fn main() {
    let semaphore = Arc::new(Semaphore::new(3));
    let mut join_handles = Vec::new();

    for _ in 0..5 {
        let permit = semaphore.clone().acquire_owned().await.unwrap();
        join_handles.push(tokio::spawn(async move {
            //
            // 在这里执行任务...
            //
            drop(permit);
        }));
    }

    for handle in join_handles {
        handle.await.unwrap();
    }
}
```

Barrier:

```rust
use std::sync::{Arc, Barrier};
use std::thread;

fn main() {
    let mut handles = Vec::with_capacity(6);
    let barrier = Arc::new(Barrier::new(6));

    for _ in 0..6 {
        let b = barrier.clone();
        handles.push(thread::spawn(move|| {
            println!("before wait");
            b.wait();
            println!("after wait");
        }));
    }

    for handle in handles {
        handle.join().unwrap();
    }
}
```

### 原子操作

> 原子操作是实现无锁的关键

> 和 Mutex 一样，Atomic的值具有内部可变性

`std::sync::atomic` 模块包含无锁并发编程要使用的原子类型

标准库中的原子类型：

- `AtomicIsize` and `AtomicUsize`
- `AtomicI8`, `AtomicI16`, `AtomicI32`, `AtomicI64`
- `AtomicU8`, `AtomicU16`, `AtomicU32`, `AtomicU64`
- `AtomicBool`
- `AtomicPtr<T>`

Rust 的原子类型支持指定内存操作，这和 C++ 基本一致，具体请参考 [C++ 章节](/docs/cpp/cpp-concurrent.md?id=同步操作和强制顺序)。

```rust
#[non_exhaustive]
pub enum Ordering {
    Relaxed,
    Release,
    Acquire,
    AcqRel,
    SeqCst,
}
```

```rust
use std::sync::Arc;
use std::sync::atomic::{AtomicUsize, Ordering};
use std::{hint, thread};

fn main() {
    let spinlock = Arc::new(AtomicUsize::new(1));

    let spinlock_clone = Arc::clone(&spinlock);
    let thread = thread::spawn(move|| {
        spinlock_clone.store(0, Ordering::SeqCst);
    });

    // 等待其它线程释放锁
    while spinlock.load(Ordering::SeqCst) != 0 {
        hint::spin_loop();
    }

    if let Err(panic) = thread.join() {
        println!("Thread had an error: {:?}", panic);
    }
}
```

### 基于 Send 和 Sync 的线程安全

`Send` 和 `Sync` 是 Rust 安全并发的重中之重，但是实际上它们只是标记特征。

- 实现 `Send` 的类型可以在线程间安全的传递其所有权
- 实现 `Sync` 的类型可以在线程间安全的共享(通过引用)

在 Rust 中，几乎所有类型都默认实现了 Send 和 Sync。除了以下几个

- 裸指针两者都没实现，因为它本身就没有任何安全保证
- `UnsafeCell` 不是 `Sync`，因此 `Cell` 和 `RefCell` 也不是
- `Rc` 两者都没实现(因为内部的引用计数器不是线程安全的)

由于这两个特征都是可自动派生的特征(通过derive派生)，意味着一个复合类型(例如结构体), 只要它内部的所有成员都实现了Send或者Sync，那么它就自动实现了Send或Sync。

> 手动实现 Send 和 Sync 是不安全的，通常并不需要手动实现 Send 和 Sync trait

### 多线程的延迟初始化

在 Rust 标准库中提供 `lazy::OnceCell` 和 `lazy::SyncOnceCell` 两种 Cell，前者用于单线程，后者用于多线程，它们用来存储堆上的信息，并且具有最多只能赋值一次的特性。

> C++11 更简单，使用 static 局部变量就可以实现

```rust
#![feature(once_cell)]

use std::{lazy::SyncOnceCell, thread};

fn main() {
    // 子线程中调用
    let handle = thread::spawn(|| {
        let logger = Logger::global();
        logger.log("thread message".to_string());
    });

    // 主线程调用
    let logger = Logger::global();
    logger.log("some message".to_string());

    let logger2 = Logger::global();
    logger2.log("other message".to_string());

    handle.join().unwrap();
}

#[derive(Debug)]
struct Logger;

static LOGGER: SyncOnceCell<Logger> = SyncOnceCell::new();

impl Logger {
    fn global() -> &'static Logger {
        // 获取或初始化 Logger
        LOGGER.get_or_init(|| {
            println!("Logger is being created..."); // 初始化打印
            Logger
        })
    }

    fn log(&self, message: String) {
        println!("{}", message)
    }
}
```


### 线程局部变量

> 功能类似于 C++ 的 `thread_local` 关键字和 Java 的 `ThreadLocal<T>` 类型

> Golang 就是因为没有线程局部变量，只能通过传递 `Context` 的方式替代

Rust 标准库提供的 `thread_local 宏`可以初始化线程局部变量，然后在线程内部使用该变量的 with 方法获取变量值：

```rust
thread_local!(static FOO: RefCell<u32> = RefCell::new(1));

// 每个线程开始时都会拿到线程局部变量的FOO的初始值
let t = thread::spawn(move|| {
    FOO.with(|f| {
        assert_eq!(*f.borrow(), 1);
        *f.borrow_mut() = 3;
    });
});
```

## aysnc/await 异步编程

上面的多线程编程本质上是同步编程，IO 会阻塞线程，而每个线程会占用一定的系统资源，系统的线程数量是有限的，IO 阻塞线程也就导致并发度受限。

Unix 系统支持 [5 种 IO 模型](/docs/netty/?id=_5-种-io-模型)，由于当前异步 IO 还不完善，大多数异步编程都是通过 IO 多路复用实现（在 Linux 下就是 epoll 了）。

Nodejs 默认就是异步编程，其实现异步编程是通过事件循环来（使用线程池通过IO多路复用监听事件，有事件完成时再把事件和回调放入主循环中）。

Rust 的异步编程也是差不多的原理。关键字也是采用了 `async/await` (await 的语法有点差别)。

> Go 的协程本质上也是基于异步 IO 实现的。不过其是有栈协程。`async/await` 是无栈携程。

async 的底层实现非常复杂，且会导致编译后文件体积显著增加，因此 Rust 没有选择像 Go 语言那样内置了完整的特性和运行时，而是选择了通过 Rust 语言提供了必要的特性支持，再通过社区来提供 async 运行时的支持。 因此要完整的使用 async 异步编程，你需要依赖以下特性和外部库:

- Rust 语言提供 `async/await` 关键字, 并进行了编译器层面的支持
- 标准库提供所必须的特征(例如 `Future` )、类型和函数
- 官方开发的 `futures` 包提供众多实用的类型、宏和函数
- async 代码的执行、IO 操作、任务创建和调度等等复杂功能由社区的 async 运行时提供，例如 `tokio` 和 `async-std`

### Futures

> 由标准库提供

Rust 为支持异步编程提供了 `std::future::Future`。其提供一个 `poll` 拉取异步任务结果，会立即返回 `Ready(output)` 或者 `Pending`。另外 `poll` 可以传入一个 `wake` 回调函数，等异步任务完成了，会调用 `wake` 函数。

> Rust 的运行时就是通过 `wake` 函数来唤醒阻塞的任务（在 wake 函数中把任务重新加到任务列表）

> 和其它语言的 Future 只是一个句柄不一样，Rust 的 Future 是惰性的，只有在被 poll 时才会运行。里面是个状态机，每次 poll 才会执行，一般会执行到下一个状态。也就是说任务的内容就在 Future 中。

一个简化版的特征：

```rust
trait SimpleFuture {
    type Output;
    fn poll(&mut self, wake: fn()) -> Poll<Self::Output>;
}

enum Poll<T> {
    Ready(T),
    Pending,
}
```

### async/await 表达式

> 由 Rust 语言提供

async 可以用于函数即 `async fn`，也可以用于 block 即 `async { ... }`

`aysnc` 返回实现 `Future` 特征的类型(类似于 Javascript 的 async 返回 promise)

和 Javascirpt 类似，只有在 `async` 中调用 `async` 才可以使用 `.wait`，同步代码中调用 `block_on(future)`。

>  `block_on` 会阻塞线程，`.wait` 并不会阻塞当前的线程。

!> 
同步代码中需要把最外层的 Future 放到三方库提供的**运行时**的任务列表中。

```toml
[dependencies]
futures = "0.3"
```

```rust
use futures::executor::block_on;

async fn hello_world() {
    hello_cat().await;
    println!("hello, world!");
}

async fn hello_cat() {
    println!("hello, kitty!");
}
fn main() {
    let future = hello_world();
    block_on(future);
}
```

### Pin 和 Unpin

在 Rust 中，所有的类型可以分为两类:

- 类型的值可以在内存中安全地被移动，例如数值、字符串、布尔值、结构体、枚举，总之你能想到的几乎所有类型都可以落入到此范畴内
- 自引用类型

    ```rust
    // pointer_to_value 指向 value，如果 SelfRef 移动后，pointer_to_value 还是指向原来的变量字段，而这个字段已经是为初始化
    struct SelfRef {
        value: String,
        pointer_to_value: *mut String,
    }
    ```

绝大多数类型都不在意是否被移动(开篇提到的第一种类型)，因此它们都自动实现了 `Unpin` 特征。对于不能安全移动的类型需要实现 `!Unpin` 特征。

由于自引用类型，才需要实现 `!Unpin` 特征

如果 async 语句块中使用了引用类型

```rust
async {
    let mut x = [0; 128];
    let read_into_buf_fut = read_into_buf(&mut x);
    read_into_buf_fut.await;
    println!("{:?}", x);
}
```

被编译成两个 Future，

```rust
struct ReadIntoBuf<'a> {
    buf: &'a mut [u8], // 指向下面的`x`字段
}

struct AsyncFuture {
    x: [u8; 128],
    read_into_buf_fut: ReadIntoBuf<'what_lifetime?>,
}
```

可以看出编译得到的 Future 很容易就存在自引用结构，所以需要用 `!Unpin` 保护。 

而 Pin 只是一个结构体，用于

> 

```rust
pub struct Pin<P> {
    pointer: P,
}
```

### Rust 异步编程原理

1. 编译器把 `async fn` 和 `async` block 编译成一个实现 `Future` 的类型，这个类型是个状态机，根据 `.await` 切分状态。

    > 每个状态就是个恢复点，下次唤醒时执行对应状态的代码

    ```rust
    #[tokio::main]
    async fn main() {
        let when = Instant::now() + Duration::from_millis(10);
        let future = Delay { when };

        // 运行并等待 Future 的完成
        let out = future.await;

        // 判断 Future 返回的字符串是否是 "done"
        assert_eq!(out, "done");
    }
    ```

    会被编译成

    ```rust
    use std::future::Future;
    use std::pin::Pin;
    use std::task::{Context, Poll};
    use std::time::{Duration, Instant};

    enum MainFuture {
        // 初始化，但永远不会被 poll
        State0,
        // 等待 `Delay` 运行，例如 `future.await` 代码行
        State1(Delay),
        // Future 执行完成
        Terminated,
    }

    impl Future for MainFuture {
        type Output = ();

        fn poll(mut self: Pin<&mut Self>, cx: &mut Context<'_>)
            -> Poll<()>
        {
            use MainFuture::*;

            loop {
                match *self {
                    State0 => {
                        let when = Instant::now() +
                            Duration::from_millis(10);
                        let future = Delay { when };
                        *self = State1(future);
                    }
                    State1(ref mut my_future) => {
                        match Pin::new(my_future).poll(cx) {
                            Poll::Ready(out) => {
                                assert_eq!(out, "done");
                                *self = Terminated;
                                return Poll::Ready(());
                            }
                            Poll::Pending => {
                                return Poll::Pending;
                            }
                        }
                    }
                    Terminated => {
                        panic!("future polled after completion")
                    }
                }
            }
        }
    }
    ```

2. 同步代码把 `Future` 封装成 `Task`（Task 实现了 `Wake` 特征） 加入**运行时**的任务列表中

    ```rust
    #[tokio::main]
    async fn main() {
        let when = Instant::now() + Duration::from_millis(10);
        let future = Delay { when };

        // 运行并等待 Future 的完成
        let out = future.await;

        // 判断 Future 返回的字符串是否是 "done"
        assert_eq!(out, "done");
    }
    ```

    会编译成

    > 暂时忽略 Future 的编译

    ```rust
    fn main() {
        let mut mini_tokio = MiniTokio::new();

        mini_tokio.spawn(async {
            let when = Instant::now() + Duration::from_millis(10);
            let future = Delay { when };

            let out = future.await;
            assert_eq!(out, "done");
        });

        mini_tokio.run();
    }
    ```

    一个简化的 Tokio 实现如下：

    ```rust
    struct MiniTokio {
        scheduled: channel::Receiver<Arc<Task>>,
        sender: channel::Sender<Arc<Task>>,
    }

    impl MiniTokio {
        fn new() -> MiniTokio {
            let (sender, scheduled) = channel::unbounded();

            MiniTokio { scheduled, sender }
        }

        /// 生成一个 Future并放入 mini-tokio 实例的任务队列中
        fn spawn<F>(&self, future: F)
        where
            F: Future<Output = ()> + Send + 'static,
        {
            Task::spawn(future, &self.sender);
        }

        fn run(&self) {
            while let Ok(task) = self.scheduled.recv() {
                task.poll();
            }
        }
    }
    ```

    任务的封装：

    ```rust
    struct Task {
        // `Mutex` 是为了让 `Task` 实现 `Sync` 特征，它能保证同一时间只有一个线程可以访问 `Future`。
        // 事实上 `Mutex` 并没有在 Tokio 中被使用，这里我们只是为了简化： Tokio 的真实代码实在太长了 :D
        future: Mutex<Pin<Box<dyn Future<Output = ()> + Send>>>,
        executor: channel::Sender<Arc<Task>>,
    }

    impl Task {
        fn schedule(self: &Arc<Self>) {
            self.executor.send(self.clone());
        }

        fn poll(self: Arc<Self>) {
            // 基于 Task 实例创建一个 waker, 它使用了之前的 `ArcWake`
            let waker = task::waker(self.clone());
            let mut cx = Context::from_waker(&waker);

            // 没有其他线程在竞争锁时，我们将获取到目标 future
            let mut future = self.future.try_lock().unwrap();

            // 对 future 进行 poll
            let _ = future.as_mut().poll(&mut cx);
        }

        fn spawn<F>(future: F, sender: &channel::Sender<Arc<Task>>)
        where
            F: Future<Output = ()> + Send + 'static,
        {
            let task = Arc::new(Task {
                future: Mutex::new(Box::pin(future)),
                executor: sender.clone(),
            });

            let _ = sender.send(task);
        }

    }

    impl ArcWake for Task {
        fn wake_by_ref(arc_self: &Arc<Self>) {
            arc_self.schedule();
        }
    }
    ```

3. 运行时的 `Executor` 从任务列表获取任务，调用其 `poll`，传入 `context`(包含 `waker` 对象，也就是 Task 对象)

4. poll 时，Future 执行其内部状态机的第一个状态方法，执行完第一个状态后，执行第二个状态内的方法，poll 其它 future，并传入 `waker`。

5. 一直调用到最里面的 future，这时一般是需要执行异步IO（IO 多路复用），或者定时器，把 `waker` 传给对应的线程或者线程池。

    一个定时器 Future

    ```rust
    struct Delay {
        when: Instant,
    }

    impl Future for Delay {
        type Output = &'static str;

        fn poll(self: Pin<&mut Self>, cx: &mut Context<'_>)
            -> Poll<&'static str>
        {
            if Instant::now() >= self.when {
                println!("Hello world");
                Poll::Ready("done")
            } else {
                // 为当前任务克隆一个 waker 的句柄
                let waker = cx.waker().clone();
                let when = self.when;

                // 生成一个计时器线程
                thread::spawn(move || {
                    let now = Instant::now();

                    if now < when {
                        thread::sleep(when - now);
                    }

                    waker.wake();
                });

                Poll::Pending
            }
        }
    }
    ```

6. 异步 IO 或者定时器完成时，调用 `waker.wake()` ，这个会把 Task 对象再次放入任务列表，让 Executor 继续调用其 poll。



## References

- [多线程并发编程](https://course.rs/advance/concurrency-with-threads/intro.html)
- [深入 Tokio 背后的异步原理](https://course.rs/async-rust/tokio/async.html)
- [Rust 异步编程](https://course.rs/async-rust/intro.html)
- [浅谈有栈协程与无栈协程](https://zhuanlan.zhihu.com/p/347445164)
- [深入了解 Rust 异步开发模式](https://cloud.tencent.com/developer/news/686021)
