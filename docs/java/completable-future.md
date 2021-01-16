# CompletableFuture

为了获取异步计算返回的结果，java5 引入了 `Future` 结果，但是它没有任何方法可以组合这些计算或处理可能的错误。

在 Java8 中，引入了 `CompletableFuture` 类。除 Future 接口外，它还实现了 `CompletionStage` 接口。该接口为异步计算步骤定义了约定，该约定可以与其他步骤结合使用。

同时，`CompletableFuture` 是一个构建块和框架，具有大约 50 种不同的方法，用于组成，组合，执行异步计算步骤和处理错误。

!> `CompletableFuture` 基本等同于 JavaScript 的 `Promise`

下面介绍一下 `CompletableFuture` 的常见用法

## 自定义 CompletableFuture 返回

- 简单的构建方法

    ```java
    Future<String> future = CompletableFuture.completedFuture("Hello");

    String result = future.get();
    Assertions.assertEquals("Hello", result);
    ```

- 完备的自定义方法

    ```java
    public Future<String> calculateAsync() throws InterruptedException {
        CompletableFuture<String> completableFuture = new CompletableFuture<>();

        Executors.newCachedThreadPool().submit(() -> {
            Thread.sleep(500);
            completableFuture.complete("Hello");
            return null;
        });

        return completableFuture;
    }
    ```

- 使用 `cancel`

    ```java
    public Future<String> calculateAsyncWithCancellation() throws InterruptedException {
        CompletableFuture<String> completableFuture = new CompletableFuture<>();

        Executors.newCachedThreadPool().submit(() -> {
            Thread.sleep(500);
            completableFuture.cancel(false);
            return null;
        });

        return completableFuture;
    }

    Future<String> future = calculateAsyncWithCancellation();
    future.get(); // CancellationException
    ```

- 使用 lambda 表达式构建

    ```java
    CompletableFuture<String> future
    = CompletableFuture.supplyAsync(() -> "Hello");

    // ...

    assertEquals("Hello", future.get());
    ```

## 处理异步计算的结果

- `thenApply(Function)`

    ```java
    CompletableFuture<String> completableFuture
    = CompletableFuture.supplyAsync(() -> "Hello");

    CompletableFuture<String> future = completableFuture
    .thenApply(s -> s + " World");

    assertEquals("Hello World", future.get());
    ```

- `thenAccpet(Consumer)`

    ```java
    CompletableFuture<String> completableFuture
    = CompletableFuture.supplyAsync(() -> "Hello");

    CompletableFuture<Void> future = completableFuture
    .thenAccept(s -> System.out.println("Computation returned: " + s));

    future.get();
    ```

- `thenRun(Runable)`

    ```java
    CompletableFuture<String> completableFuture 
    = CompletableFuture.supplyAsync(() -> "Hello");

    CompletableFuture<Void> future = completableFuture
    .thenRun(() -> System.out.println("Computation finished."));

    future.get();
    ```

## 组合 Futures

CompletableFurue API 最好的部分是能够在一系列计算步骤中组合 CompletableFuture 实例的功能。

- `thenCompose`

    ```java
    CompletableFuture<String> completableFuture 
    = CompletableFuture.supplyAsync(() -> "Hello")
        .thenCompose(s -> CompletableFuture.supplyAsync(() -> s + " World"));

    assertEquals("Hello World", completableFuture.get());
    ```

    > `thenApply` 和 `thenCompose` 的关系就像 `map` 和 `flatMap` 的关系

- `thenCombine`

    ```java
    CompletableFuture<String> completableFuture 
    = CompletableFuture.supplyAsync(() -> "Hello")
        .thenCombine(CompletableFuture.supplyAsync(
        () -> " World"), (s1, s2) -> s1 + s2));

    assertEquals("Hello World", completableFuture.get());
    ```

- `thenAcceptBoth`

    ```java
    CompletableFuture future = CompletableFuture.supplyAsync(() -> "Hello")
    .thenAcceptBoth(CompletableFuture.supplyAsync(() -> " World"),
        (s1, s2) -> System.out.println(s1 + s2));
    ```

## 并行多个 Futures

- `allOf`

    ```java
    CompletableFuture<String> future1  
    = CompletableFuture.supplyAsync(() -> "Hello");
    CompletableFuture<String> future2  
    = CompletableFuture.supplyAsync(() -> "Beautiful");
    CompletableFuture<String> future3  
    = CompletableFuture.supplyAsync(() -> "World");

    CompletableFuture<Void> combinedFuture 
    = CompletableFuture.allOf(future1, future2, future3);

    // ...

    combinedFuture.get();

    assertTrue(future1.isDone());
    assertTrue(future2.isDone());
    assertTrue(future3.isDone());
    ```

    `allOf` 返回的是 `CompletableFuture<Void>` 类型，即无法获取每个 Future 的结果

- java8 Stream

    ```java
    String combined = Stream.of(future1, future2, future3)
    .map(CompletableFuture::join)
    .collect(Collectors.joining(" "));

    assertEquals("Hello Beautiful World", combined);
    ```

## 处理错误

```java
String name = null;

// ...

CompletableFuture<String> completableFuture  
  =  CompletableFuture.supplyAsync(() -> {
      if (name == null) {
          throw new RuntimeException("Computation error!");
      }
      return "Hello, " + name;
  })}).handle((s, t) -> s != null ? s : "Hello, Stranger!");

assertEquals("Hello, Stranger!", completableFuture.get());
```

通过 `completeExceptionally` 返回异常

```java
CompletableFuture<String> completableFuture = new CompletableFuture<>();

completableFuture.completeExceptionally(
  new RuntimeException("Calculation failed!"));

try {
    completableFuture.get(); // ExecutionException
} catch (InterruptedException e) {
    e.printStackTrace();
} catch (ExecutionException e) {
    Assertions.assertEquals("Calculation failed!", e.getCause().getMessage())
}
```

## 异步处理

使用新的线程处理 CompletableFuture 返回的结果

```java
CompletableFuture<String> completableFuture  
  = CompletableFuture.supplyAsync(() -> "Hello");

CompletableFuture<String> future = completableFuture
  .thenApplyAsync(s -> s + " World");

assertEquals("Hello World", future.get());
```

## References

- [Guide To CompletableFuture](https://www.baeldung.com/java-completablefuture)