title: 如何使用 JUnit 测试异步代码
tags:
- Java
- JUnit
---

## 问题
对于以下的异步代码：

```java
public class DemoService {
    public CompletableFuture<String> hello() {
        return CompletableFuture.supplyAsync(() -> "hello");
    }
}
```

我们为其编写一个测试用例，并在 `CompletableFuture#whenComplete` 中判断返回值是否与预期相符，然而即使返回值与预期不符，该测试也不会抛出异常：

```java
@Test
public void exceptionWontBeCaptured() {
    DemoService demoService = new DemoService();

    demoService.hello()
            .whenComplete((result, e) -> {
                Assert.assertEquals("wrongValue", result);
            });
}
```

## 解决
### CompletableFuture#get()
我们可以借助 `CompletableFuture#get()` 阻塞主线程等待结果的特点，将异步代码转成同步：

```java
@Test
public void blockMainThreadByGet() throws ExecutionException, InterruptedException {
    DemoService demoService = new DemoService();

    Assert.assertEquals("hello", demoService.hello().get());
}
```

### CountDownLatch
上述方案依赖了一个具体的异步类方法，如果实际的异步类不提供相应的同步方法，上述方案则不适合。针对这种情况，可以借助 `CountDownLatch`，初始化一个计数为1的 `CountDownLatch` 的实例，在测试方法中调用 `CountDownLatch#await()` 方法进行等待，当异步方法执行成功后在其回调中调用 `CountDownLatch#countDown()` 使计数器减1变为0，从而继续执行后续的测试判断：

```java
@Test
public void waitOnCountDown() throws InterruptedException {
    CountDownLatch countDownLatch = new CountDownLatch(1);
    DemoService demoService = new DemoService();
    AtomicReference<String> actualValue = new AtomicReference<>("");

    demoService.hello()
            .whenComplete((result, e) -> {
                actualValue.set(result);
                countDownLatch.countDown();
            });

    countDownLatch.await();

    Assert.assertEquals("hello", actualValue.get());
}
```

### Awaitility
[Awaitility](https://github.com/awaitility/awaitility) 让测试异步代码变得简单明了：

```java
@Test
public void poweredByAwaitility() {
    DemoService demoService = new DemoService();
    AtomicReference<String> actualValue = new AtomicReference<>("");

    demoService.hello()
            .whenComplete((result, e) -> {
                actualValue.set(result);
            });

    await().atMost(5, SECONDS).untilAsserted(() -> {
        Assert.assertEquals("hello", actualValue.get());
    });
}
```

完整的代码可参考 [GitHub](https://github.com/Frederick-S/test-async-code-with-junit-demo)。

## 参考

- [How to use JUnit to test asynchronous processes](https://stackoverflow.com/questions/631598/how-to-use-junit-to-test-asynchronous-processes)
