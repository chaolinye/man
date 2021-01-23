# Reactor

Reactor 是一个实现了响应式编程模型的 Java8 库

适用场景：

- 构建高可用、实时的数据流应用
- 必须具有低延迟可容错类微服务

> 事件驱动

## Reactive Streams

Reactive Streams 是一个异步处理规范，主要目标是管理跨异步边界的流数据交换-考虑将元素传递到另一个线程或线程池-同时确保不强制接收缓冲任意数量的数据。

换句话说，背压是此模型不可或缺的部分

!> Reactive Streams 的核心: 异步和背压

主要的 API 组件

- Publisher

    ```java
    public interface Publisher<T> {
        public void subscribe(Subscriber<? super T> s);
    }
    ```

- Subscriber

    ```java
    public interface Subscriber<T> {
        public void onSubscribe(Subscription s);
        public void onNext(T t);
        public void onError(Throwable t);
        public void onComplete();
    }
    ```

- Subscription

    ```java
    public interface Subscription {
        public void request(long n);
        public void cancel();
    }
    ```

- Processor

    ```java
    public interface Processor<T, R> extends Subscriber<T>, Publisher<R> {
    }
    ```

    ```mermaid
    sequenceDiagram

    User ->> Publisher: subscribe(Subscriber)
    Publisher ->> Subscription: new(Subscriber, Publisher)
    Publisher ->> Subscriber: onSubscriber(Subscription)
    Subscriber ->> Subscription: onrequest(n)
    Subscription ->> Subscriber: for(i : n) Subscriber.onNext
    Subscriber ->> Subscription: onrequest(n)
    Subscription ->> Subscriber: for(i : n) Subscriber.onNext
    ```

## Backpressure

> 背压是指下游可以告诉上游向其发送较少的数据以防止其不堪重负

每次只连续消费两个：

```java
Flux.just(1, 2, 3, 4)
    .log()
    .subscribe(new Subscriber<Integer>() {
        private Subscription s;
        int onNextAmount;

        @Override
        public void onSubscribe(Subscription s) {
            this.s = s;
            s.request(2);
        }

        @Override
        public void onNext(Integer integer) {
            System.out.println(integer);
            onNextAmount++;
            if (onNextAmount % 2 == 0) {
                s.request(2);
            }
        }

        @Override
        public void onError(Throwable t) {

        }

        @Override
        public void onComplete() {

        }
    });
```

## Reactor vs Java8 Stream

首先是，Reactor 的核心：异步和背压

### 异步

```java
// Java8 Stream 返回同步结果
List<Integer> collected = Stream.of(1, 2, 3, 4).collect(Collectors.toList());

// Reactor 返回流类型
Mono<List<Integer>> mono = Flux.just(1, 2, 3, 4).collectList();

// Reactor 也可以通过 block 方法获取同步结果
List<Integer> blockCollected = Flux.just(1, 2, 3, 4).collectList().block();
```

从代码可以看出，java8 Stream 并没有异步特性

### 背压

由于是同步操作，Java8 Stream 不涉及背压

### 无边界数据流

Reactor 可以处理无边界数据流

```java
ConnectableFlux<Object> publish = Flux.create(fluxSink -> {
    while (true) {
        fluxSink.next(System.currentTimeMillis());
    }
}).publish();

publish.subscribe(System.out::println);
publish.subscribe(System.out::println);
```

Java8 Stream 处理的是有边界数据集

### push 模型

Reactor 使用的是 push 模型，每个事件来了， 就会被推送到 subscribers

> subscriber 调用 subscription.onrequest 只是为了实现背压，并不是 pull 模型

```java
Flux.create(fluxSink -> {
    while(true) {
        fluxSink.next(System.currentTimeMillis());
    }
}).sample(Duration.ofSeconds(2))
        .publish();
```

Java8 Stream 是一个 pull 模型

## Reactor 入门使用

Maven 依赖

```xml
<dependency>
    <groupId>io.projectreactor</groupId>
    <artifactId>reactor-core</artifactId>
    <version>3.3.9.RELEASE</version>
</dependency>
```

### 生产数据

```java
// 生产 0...n 元素
Flux<String> just = Flux.just("1", "2", "3");
Flux<Integer> range = Flux.range(1, 3);

// 生产 0...1 元素
Mono<String> mono = Mono.just("foo");
```

`Flux` 和 `Mono` 都是 `Publisher` 接口的实现

为何要定义两个 `Stream` 数据类型呢？ `Mono` 只生产 0...1 个元素，在某些场景下语义性更好，也可以定义一个特定的场景性操作

```java
// 使用 generate 创建 Flux
public Flux<Integer> generateFibonacciWithTuples() {
    return Flux.generate(() -> Tuples.of(0, 1),
            (state, sink) -> {
                sink.next(state.getT1());
                return Tuples.of(state.getT2(), state.getT1() + state.getT2());
            });
}

// 返回 create 创建异步 Flux
public class SequenceCreator {
    public Consumer<List<Integer>> consumer;

    public Flux<Integer> createNumberSequence() {
        // 返回的只是 Consumer，由后续其他线程调用 consumer.accept() 方法才下发
        return Flux.create(sink -> SequenceCreator.this.consumer = items -> items.forEach(sink::next))
    }
}
```

### 消费数据

```java
List<Integer> elements = new ArrayList<>();

Flux.just(1, 2, 3, 4)
        .log()
        .subscribe(elements::add);

Assertions.assertThat(elements).containsExactly(1, 2, 3, 4);
```

日志

```
00:19:18.843 [main] INFO reactor.Flux.Array.1 - | onSubscribe([Synchronous Fuseable] FluxArray.ArraySubscription)
00:19:18.847 [main] INFO reactor.Flux.Array.1 - | request(unbounded)
00:19:18.848 [main] INFO reactor.Flux.Array.1 - | onNext(1)
00:19:18.848 [main] INFO reactor.Flux.Array.1 - | onNext(2)
00:19:18.848 [main] INFO reactor.Flux.Array.1 - | onNext(3)
00:19:18.848 [main] INFO reactor.Flux.Array.1 - | onNext(4)
00:19:18.848 [main] INFO reactor.Flux.Array.1 - | onComplete()
```

`Subscriber` 完整写法

```java
Flux.just(1, 2, 3, 4)
    .log()
    .subscribe(new Subscriber<Integer>() {
        @Override
        public void onSubscribe(Subscription s) {
            s.request(Long.MAX_VALUE);
        }

        @Override
        public void onNext(Integer integer) {
            elements.add(integer)
        }

        @Override
        public void onError(Throwable t) {

        }

        @Override
        public void onComplete() {

        }
    });
```

### 常用流操作

- Map

    ```java
    Flux.just(1, 2, 3, 4)
        .log()
        .map(i -> i * 2)
        .subscribe(elements::add);
    ```

- handler

    > 比较通用的流操作符

    ```java
    Flux.just(1, 2, 3, 4)
        .handle((number, sink) -> {
            if (number % 2 == 0) {
                sink.next(number / 2);
            }
        })
        .subscribe(elements::add);
    ```

- 合并流

    `zipWith`

    ```java
    Flux.just(1, 2, 3, 4)
        .log()
        .map(i -> i * 2)
        .zipWith(Flux.range(0, Integer.MAX_VALUE),
                (one, two) -> String.format("First Flux: %d, Second Flux: %d", one, two))
        .subscribe(System.out::println);
    ```

    `concatWith`

    ```java
    Flux<Integer> evenNumbers = Flux
            .range(min, max)
            .filter(x -> x % 2 == 0); // i.e. 2, 4

    Flux<Integer> oddNumbers = Flux
            .range(min, max)
            .filter(x -> x % 2 > 0);  // ie. 1, 3, 5

    Flux<Integer> fluxOfIntegers = evenNumbers.concatWith(oddNumbers);

    StepVerifier.create(fluxOfIntegers)
            .expectNext(2)
            .expectNext(4)
            .expectNext(1)
            .expectNext(3)
            .expectNext(5)
            .expectComplete()
            .verify();
    ```

    `combineLatest`

    ```java
    Flux<Integer> fluxOfIntegers = Flux.combineLatest(
        evenNumbers, 
        oddNumbers, 
        (a, b) -> a + b);

    StepVerifier.create(fluxOfIntegers)
        .expectNext(5) // 4 + 1
        .expectNext(7) // 4 + 3
        .expectNext(9) // 4 + 5
        .expectComplete()
        .verify();
    ```

### 实时数据流

```java
ConnectableFlux<Object> publish = Flux.create(fluxSink -> {
    while (true) {
        fluxSink.next(System.currentTimeMillis());
    }
})
        .sample(ofSeconds(2))  // 限流，每2秒下发一个元素
        .publish(); // publish 用于绑定多个 subscriber

// 绑定两个 subscriber ，并不会触发 publisher 的执行
publish.subscribe(System.out::println);
publish.subscribe(System.out::println);

// 开始生产数据
publish.connect();
```

### 异步

```java
Flux.just(1, 2, 3, 4)
        .log()
        .map(i -> i * 2)
        .subscribeOn(Schedulers.parallel())
        .subscribe(System.out::println);
```

```
13:41:37.821 [main] DEBUG reactor.util.Loggers$LoggerFactory - Using Slf4j logging framework
13:41:37.850 [parallel-1] INFO reactor.Flux.Array.1 - | onSubscribe([Synchronous Fuseable] FluxArray.ArraySubscription)
13:41:37.853 [parallel-1] INFO reactor.Flux.Array.1 - | request(unbounded)
13:41:37.854 [parallel-1] INFO reactor.Flux.Array.1 - | onNext(1)
2
13:41:37.854 [parallel-1] INFO reactor.Flux.Array.1 - | onNext(2)
4
13:41:37.854 [parallel-1] INFO reactor.Flux.Array.1 - | onNext(3)
6
13:41:37.854 [parallel-1] INFO reactor.Flux.Array.1 - | onNext(4)
8
13:41:37.855 [parallel-1] INFO reactor.Flux.Array.1 - | onComplete()
```

## 源码导读

### 流操作

以 `map` 操作为例

```java
public abstract class Flux<T> implements CorePublisher<T> {
    public final <V> Flux<V> map(Function<? super T, ? extends V> mapper) {
		if (this instanceof Fuseable) {
			return onAssembly(new FluxMapFuseable<>(this, mapper));
		}
        // 装饰者模式
		return onAssembly(new FluxMap<>(this, mapper));
	}
}
```

```java
final class FluxMap<T, R> extends InternalFluxOperator<T, R> {

	final Function<? super T, ? extends R> mapper;

	FluxMap(Flux<? extends T> source,
			Function<? super T, ? extends R> mapper) {
		super(source);
		this.mapper = Objects.requireNonNull(mapper, "mapper");
	}

	@Override
	@SuppressWarnings("unchecked")
	public CoreSubscriber<? super T> subscribeOrReturn(CoreSubscriber<? super R> actual) {
		if (actual instanceof Fuseable.ConditionalSubscriber) {
			Fuseable.ConditionalSubscriber<? super R> cs =
					(Fuseable.ConditionalSubscriber<? super R>) actual;
			return new MapConditionalSubscriber<>(cs, mapper);
		}
        // 装饰者模式
		return new MapSubscriber<>(actual, mapper);
	}

	static final class MapSubscriber<T, R>
			implements InnerOperator<T, R> {

		final CoreSubscriber<? super R>        actual;
		final Function<? super T, ? extends R> mapper;

		boolean done;

		Subscription s;

		MapSubscriber(CoreSubscriber<? super R> actual,
				Function<? super T, ? extends R> mapper) {
			this.actual = actual;
			this.mapper = mapper;
		}

		@Override
		public void onSubscribe(Subscription s) {
			if (Operators.validate(this.s, s)) {
				this.s = s;

				actual.onSubscribe(this);
			}
		}

		@Override
		public void onNext(T t) {
			if (done) {
				Operators.onNextDropped(t, actual.currentContext());
				return;
			}

			R v;

			try {
				v = Objects.requireNonNull(mapper.apply(t),
						"The mapper returned a null value.");
			}
			catch (Throwable e) {
				Throwable e_ = Operators.onNextError(t, e, actual.currentContext(), s);
				if (e_ != null) {
					onError(e_);
				}
				else {
					s.request(1);
				}
				return;
			}

			actual.onNext(v);
		}

		@Override
		public void onError(Throwable t) {
			if (done) {
				Operators.onErrorDropped(t, actual.currentContext());
				return;
			}

			done = true;

			actual.onError(t);
		}

		@Override
		public void onComplete() {
			if (done) {
				return;
			}
			done = true;

			actual.onComplete();
		}

		@Override
		@Nullable
		public Object scanUnsafe(Attr key) {
			if (key == Attr.PARENT) return s;
			if (key == Attr.TERMINATED) return done;

			return InnerOperator.super.scanUnsafe(key);
		}

		@Override
		public CoreSubscriber<? super R> actual() {
			return actual;
		}

		@Override
		public void request(long n) {
			s.request(n);
		}

		@Override
		public void cancel() {
			s.cancel();
		}
	}
}
```

主要使用了 `装饰者模式`

- 通过一个 `FluxOperation` 实现类包装原 `Flux` 对象

- 调用 `FluexOperation` 的 `subscribe` 方法时，调用的是原 `Flux` 的方法，但是会构造一个 `OperationSubscriber` 对象对真实的 `subscriber` 对象进行包装

- 原 `Flux` 产生的数据传给 `OperationSubscriber`, `OperationSubscriber` 进行转换后再传给真实的 `subscriber`

!> 核心思想是装饰 subscriber

!> 强制增强 subscriber

### FluxSink 实时数据流

```java
final class FluxCreate<T> extends Flux<T> implements SourceProducer<T> {

	enum CreateMode {
		PUSH_ONLY, PUSH_PULL
	}

	final Consumer<? super FluxSink<T>> source;

	final OverflowStrategy backpressure;

	final CreateMode createMode;

	FluxCreate(Consumer<? super FluxSink<T>> source,
			FluxSink.OverflowStrategy backpressure,
			CreateMode createMode) {
		this.source = Objects.requireNonNull(source, "source");
		this.backpressure = Objects.requireNonNull(backpressure, "backpressure");
		this.createMode = createMode;
	}

	static <T> BaseSink<T> createSink(CoreSubscriber<? super T> t,
			OverflowStrategy backpressure) {
		switch (backpressure) {
			case IGNORE: {
				return new IgnoreSink<>(t);
			}
			case ERROR: {
				return new ErrorAsyncSink<>(t);
			}
			case DROP: {
				return new DropAsyncSink<>(t);
			}
			case LATEST: {
				return new LatestAsyncSink<>(t);
			}
			default: {
				return new BufferAsyncSink<>(t, Queues.SMALL_BUFFER_SIZE);
			}
		}
	}
    @Override
	public void subscribe(CoreSubscriber<? super T> actual) {
		BaseSink<T> sink = createSink(actual, backpressure);

		actual.onSubscribe(sink);
		try {
			source.accept(
					createMode == CreateMode.PUSH_PULL ? new SerializedSink<>(sink) :
							sink);
		}
		catch (Throwable ex) {
			Exceptions.throwIfFatal(ex);
			sink.error(Operators.onOperatorError(ex, actual.currentContext()));
		}
	}
    static final class BufferAsyncSink<T> extends BaseSink<T> {

		final Queue<T> queue;

		Throwable error;
		volatile boolean done; //done is still useful to be able to drain before the terminated handler is executed

		volatile int wip;
		@SuppressWarnings("rawtypes")
		static final AtomicIntegerFieldUpdater<BufferAsyncSink> WIP =
				AtomicIntegerFieldUpdater.newUpdater(BufferAsyncSink.class, "wip");

		BufferAsyncSink(CoreSubscriber<? super T> actual, int capacityHint) {
			super(actual);
			this.queue = Queues.<T>unbounded(capacityHint).get();
		}

		@Override
		public FluxSink<T> next(T t) {
			queue.offer(t);
            // 下发，调用 subscriber.onNext()
			drain();
			return this;
		}

        // more
    }
}
```

其中 `BaseSink` 实现了 `FluxSink`, `Subscription` 接口，同时具有 `推送` `背压` 两大功能

### subscribeOn

用新的线程运行 `Publisher.subscribe()`, `Subscriber.onSubscribe()`, `Subscription.request()` 方法

常用于 `slow publisher e.g., blocking IO, fast consumer(s)` 场景

> 即 Publisher 是阻塞性的场景

![](https://raw.githubusercontent.com/reactor/reactor-core/v3.0.5.RELEASE/src/docs/marble/subscribeon.png)

基本用法

```java
flux.subscribeOn(Schedulers.single()).subscribe()
```

源码入口

```java
    public final Flux<T> subscribeOn(Scheduler scheduler, boolean requestOnSeparateThread) {
		// more
		return onAssembly(new FluxSubscribeOn<>(this, scheduler, requestOnSeparateThread));
	}
```

```java
final class FluxSubscribeOn<T> extends InternalFluxOperator<T, T> {

	final Scheduler scheduler;
	final boolean requestOnSeparateThread;

	FluxSubscribeOn(
			Flux<? extends T> source,
			Scheduler scheduler,
			boolean requestOnSeparateThread) {
		super(source);
		this.scheduler = Objects.requireNonNull(scheduler, "scheduler");
		this.requestOnSeparateThread = requestOnSeparateThread;
	}

	@Override
	public CoreSubscriber<? super T> subscribeOrReturn(CoreSubscriber<? super T> actual) {
		Worker worker = Objects.requireNonNull(scheduler.createWorker(),
				"The scheduler returned a null Function");

		SubscribeOnSubscriber<T> parent = new SubscribeOnSubscriber<>(source,
				actual, worker, requestOnSeparateThread);
		actual.onSubscribe(parent);

		try {
			worker.schedule(parent);
		}
		catch (RejectedExecutionException ree) {
			if (parent.s != Operators.cancelledSubscription()) {
				actual.onError(Operators.onRejectedExecution(ree, parent, null, null,
						actual.currentContext()));
			}
		}
		return null;
	}

	static final class SubscribeOnSubscriber<T>
			implements InnerOperator<T, T>, Runnable {

		SubscribeOnSubscriber(CorePublisher<? extends T> source, CoreSubscriber<? super T> actual,
				Worker worker, boolean requestOnSeparateThread) {
			this.actual = actual;
			this.worker = worker;
			this.source = source;
			this.requestOnSeparateThread = requestOnSeparateThread;
		}

		@Override
		public void onSubscribe(Subscription s) {
			if (Operators.setOnce(S, this, s)) {
				long r = REQUESTED.getAndSet(this, 0L);
				if (r != 0L) {
					requestUpstream(r, s);
				}
			}
		}

        void requestUpstream(final long n, final Subscription s) {
			if (!requestOnSeparateThread || Thread.currentThread() == THREAD.get(this)) {
				s.request(n);
			}
			else {
				try {
					worker.schedule(() -> s.request(n));
				}
				catch (RejectedExecutionException ree) {
					if(!worker.isDisposed()) {
						//FIXME should not throw but if we implement strict
						// serialization like in StrictSubscriber, onNext will carry an
						// extra cost
						throw Operators.onRejectedExecution(ree, this, null, null,
								actual.currentContext());
					}
				}
			}
		}

		@Override
		public void onNext(T t) {
			actual.onNext(t);
		}

		@Override
		public void run() {
			THREAD.lazySet(this, Thread.currentThread());
			source.subscribe(this);
		}
	}

}
```

可见，FluxSubscribeOn 的实现方法是 `publisher` 的 `Around` 增强，用一个新的线程运行 `publisher.subscribe`

就是指定了 `Subscription` 和 `Subscriber` 之间交互的线程


### publishOn

用新的线程运行 `Subscriber` 的 `onNext`, `onComplete` and `onError`

常用于 `fast publisher, slow consumer(s)` 场景，即用多线程加快 consume 速度

![](https://raw.githubusercontent.com/reactor/reactor-core/v3.0.5.RELEASE/src/docs/marble/publishon.png)

基本用法

```java
flux.publishOn(Schedulers.single()).subscribe()
```

源码入口

```java
    final Flux<T> publishOn(Scheduler scheduler, boolean delayError, int prefetch, int lowTide) {
		// more
		return onAssembly(new FluxPublishOn<>(this, scheduler, delayError, prefetch, lowTide, Queues.get(prefetch)));
	}
```

```java
final class FluxPublishOn<T> extends InternalFluxOperator<T, T> implements Fuseable {

	FluxPublishOn(Flux<? extends T> source,
			Scheduler scheduler,
			boolean delayError,
			int prefetch,
			int lowTide,
			Supplier<? extends Queue<T>> queueSupplier) {
		super(source);
		if (prefetch <= 0) {
			throw new IllegalArgumentException("prefetch > 0 required but it was " + prefetch);
		}
		this.scheduler = Objects.requireNonNull(scheduler, "scheduler");
		this.delayError = delayError;
		this.prefetch = prefetch;
		this.lowTide = lowTide;
		this.queueSupplier = Objects.requireNonNull(queueSupplier, "queueSupplier");
	}


	@Override
	@SuppressWarnings("unchecked")
	public CoreSubscriber<? super T> subscribeOrReturn(CoreSubscriber<? super T> actual) {
		Worker worker = Objects.requireNonNull(scheduler.createWorker(),
				"The scheduler returned a null worker");

		return new PublishOnSubscriber<>(actual,
				scheduler,
				worker,
				delayError,
				prefetch,
				lowTide,
				queueSupplier);
	}

	static final class PublishOnSubscriber<T>
			implements QueueSubscription<T>, Runnable, InnerOperator<T, T> {

		PublishOnSubscriber(CoreSubscriber<? super T> actual,
				Scheduler scheduler,
				Worker worker,
				boolean delayError,
				int prefetch,
				int lowTide,
				Supplier<? extends Queue<T>> queueSupplier) {
			this.actual = actual;
			this.worker = worker;
			this.scheduler = scheduler;
			this.delayError = delayError;
			this.prefetch = prefetch;
			this.queueSupplier = queueSupplier;
			this.limit = Operators.unboundedOrLimit(prefetch, lowTide);
		}

		@Override
		public void onSubscribe(Subscription s) {
			if (Operators.validate(this.s, s)) {
				this.s = s;

				if (s instanceof QueueSubscription) {
					@SuppressWarnings("unchecked") QueueSubscription<T> f =
							(QueueSubscription<T>) s;

					int m = f.requestFusion(Fuseable.ANY | Fuseable.THREAD_BARRIER);

					if (m == Fuseable.SYNC) {
						sourceMode = Fuseable.SYNC;
						queue = f;
						done = true;

						actual.onSubscribe(this);
						return;
					}
					if (m == Fuseable.ASYNC) {
						sourceMode = Fuseable.ASYNC;
						queue = f;

						actual.onSubscribe(this);

						s.request(Operators.unboundedOrPrefetch(prefetch));

						return;
					}
				}

				queue = queueSupplier.get();

				actual.onSubscribe(this);

				s.request(Operators.unboundedOrPrefetch(prefetch));
			}
		}

		@Override
		public void onNext(T t) {
			if (sourceMode == ASYNC) {
				trySchedule(this, null, null /* t always null */);
				return;
			}

			if (done) {
				Operators.onNextDropped(t, actual.currentContext());
				return;
			}

			if (cancelled) {
				Operators.onDiscard(t, actual.currentContext());
				return;
			}

			if (!queue.offer(t)) {
				Operators.onDiscard(t, actual.currentContext());
				error = Operators.onOperatorError(s,
						Exceptions.failWithOverflow(Exceptions.BACKPRESSURE_ERROR_QUEUE_FULL),
						t, actual.currentContext());
				done = true;
			}
			trySchedule(this, null, t);
		}

		@Override
		public void request(long n) {
			if (Operators.validate(n)) {
				Operators.addCap(REQUESTED, this, n);
				//WIP also guards during request and onError is possible
				trySchedule(this, null, null);
			}
		}

		void trySchedule(
				@Nullable Subscription subscription,
				@Nullable Throwable suppressed,
				@Nullable Object dataSignal) {
			if (WIP.getAndIncrement(this) != 0) {
				if (cancelled) {
					if (sourceMode == ASYNC) {
						// delegates discarding to the queue holder to ensure there is no racing on draining from the SpScQueue
						queue.clear();
					}
					else {
						// discard given dataSignal since no more is enqueued (spec guarantees serialised onXXX calls)
						Operators.onDiscard(dataSignal, actual.currentContext());
					}
				}
				return;
			}

			try {
				worker.schedule(this);
			}
			catch (RejectedExecutionException ree) {
				if (sourceMode == ASYNC) {
					// delegates discarding to the queue holder to ensure there is no racing on draining from the SpScQueue
					queue.clear();
				} else if (outputFused) {
					// We are the holder of the queue, but we still have to perform discarding under the guarded block
					// to prevent any racing done by downstream
					this.clear();
				}
				else {
					// In all other modes we are free to discard queue immediately since there is no racing on pooling
					Operators.onDiscardQueueWithClear(queue, actual.currentContext(), null);
				}
				actual.onError(Operators.onRejectedExecution(ree, subscription, suppressed, dataSignal,
						actual.currentContext()));
			}
		}

		void runAsync() {
			int missed = 1;

			final Subscriber<? super T> a = actual;
			final Queue<T> q = queue;

			long e = produced;

			for (; ; ) {

				long r = requested;

				while (e != r) {
					boolean d = done;
					T v;

					try {
						v = q.poll();
					}
					catch (Throwable ex) {
						Exceptions.throwIfFatal(ex);
						s.cancel();
						if (sourceMode == ASYNC) {
							// delegates discarding to the queue holder to ensure there is no racing on draining from the SpScQueue
							queue.clear();
						} else {
							// discard MUST be happening only and only if there is no racing on elements consumption
							// which is guaranteed by the WIP guard here
							Operators.onDiscardQueueWithClear(queue, actual.currentContext(), null);
						}

						doError(a, Operators.onOperatorError(ex, actual.currentContext()));
						return;
					}

					boolean empty = v == null;

					if (checkTerminated(d, empty, a, v)) {
						return;
					}

					if (empty) {
						break;
					}

					a.onNext(v);

					e++;
					if (e == limit) {
						if (r != Long.MAX_VALUE) {
							r = REQUESTED.addAndGet(this, -e);
						}
						s.request(e);
						e = 0L;
					}
				}

				if (e == r && checkTerminated(done, q.isEmpty(), a, null)) {
					return;
				}

				int w = wip;
				if (missed == w) {
					produced = e;
					missed = WIP.addAndGet(this, -missed);
					if (missed == 0) {
						break;
					}
				}
				else {
					missed = w;
				}
			}
		}

		@Override
		public void run() {
			if (outputFused) {
				runBackfused();
			}
			else if (sourceMode == Fuseable.SYNC) {
				runSync();
			}
			else {
				runAsync();
			}
		}

	}
}
```

本质上，publishOn 实现了 `subscriber` 的 `Around` 增强, 开启新线程运行原 `subscriber` 的 `onNext` 等操作

!>  由于 `publishOn` 增强的是 `subscriber`, 所以只有在 `publishOn` 后面的 `Stream Operator` 才运行在新的线程中

## Reactor 的流操作顺序分析

根据每个流操作的增强对象是 `Publisher` 还是 `Subscriber`, 以及增强的类型(前置、后置、环绕)进行分析

如下代码

```java
Flux.just(1, 2, 3, 4)
    .log()
    .publishOn(Schedulers.parallel())
    .map(x -> x * 2)
    .subscribeOn(Schedulers.parallel())
    .subscribe(System.out::println)
```

可以通过分析画出以下的顺序简图

```
new Tread()
    publisher.subscribe()
        log()
            new Thread()
                map()
                    System.out.println()
```



## References

- [响应式流处理规范首页](http://www.reactive-streams.org/)
- [JVM 响应式流处理规范细则](https://github.com/reactive-streams/reactive-streams-jvm/blob/v1.0.3/README.md)
- [Reactor Core 简介](https://www.baeldung.com/reactor-core)
- [Reactor Core 官方文档](https://projectreactor.io/docs)
- [流的创建](https://www.baeldung.com/flux-sequences-reactor)
- [流的合并](https://www.baeldung.com/reactor-combine-streams)
- [Reactor 指南](https://www.baeldung.com/reactor-combine-streams)