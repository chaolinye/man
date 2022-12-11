# Rust 并发编程

安全和高效的处理并发是 Rust 语言的主要目标之一。

可惜的是，在 Rust 中由于语言设计理念、安全、性能的多方面考虑，并没有采用 Go 语言大道至简的方式，而是选择了**多线程与 async/await 相结合**，优点是可控性更强、性能更高，缺点是复杂度并不低，当然这也是系统级语言的应有选择：使用复杂度换取可控性和性能。

协程的实现会显著增大运行时的大小，因此 Rust 只在**标准库中提供了 1:1 的线程模型**，如果你愿意牺牲一些性能来换取更精确的线程控制以及更小的线程上下文切换成本，那么可以选择 Rust 中的 **M:N 模型，这些模型由三方库提供了实现**，例如大名鼎鼎的 tokio。

> 据不精确估算，创建一个线程大概需要 `0.24 毫秒`

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

## aysnc/await

> TODO

## References

- [多线程并发编程](https://course.rs/advance/concurrency-with-threads/intro.html)
- [Rust 异步编程](https://course.rs/async-rust/intro.html)