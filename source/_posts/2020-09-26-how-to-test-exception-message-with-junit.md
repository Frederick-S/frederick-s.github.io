title: 如何使用 JUnit 测试异常信息
tags:
- Java
- JUnit
---

假设有如下的 `SumService`：

```java
public class SumService {
    public static int sum(int a, int b) {
        if (a <= 0) {
            throw new IllegalArgumentException("a must be positive");
        }

        if (b <= 0) {
            throw new IllegalArgumentException("b must be positive");
        }
        
        return a + b;
    }
}
```

当 `a` 或者 `b` 非正数时会抛出 `IllegalArgumentException` 异常，由于两者抛出的是同一个异常，所以无法直接使用 `expected = IllegalArgumentException.class` 进行区分测试，故需要测试具体的异常信息。

## 使用 try/catch
用一个 `try/catch` 包裹测试的方法，判断抛出的异常信息：

```java
@Test
public void shouldAssertExceptionMessageByAssertThrows() {
    IllegalArgumentException illegalArgumentException = 
            Assert.assertThrows(IllegalArgumentException.class, () -> SumService.sum(0, 1));

    Assert.assertEquals("a must be positive", illegalArgumentException.getMessage());
}
```

## 使用 assertThrows
借助 `Assert.assertThrows` 执行测试方法返回一个异常，然后判断返回的异常信息：

```java
@Test
public void shouldAssertExceptionMessageByAssertThrows() {
    IllegalArgumentException illegalArgumentException = Assert.assertThrows(IllegalArgumentException.class, () -> SumService.sum(0, 1));

    Assert.assertEquals("a must be positive", illegalArgumentException.getMessage());
}
```

## 使用 ExpectedException
借助 `ExpectedException` 预先设定预期抛出的异常和异常信息，然后执行测试方法：

```java
@Rule
public ExpectedException expectedException = ExpectedException.none();

@Test
public void shouldAssertExceptionMessageByRule() {
    expectedException.expect(IllegalArgumentException.class);
    expectedException.expectMessage("a must be positive");

    SumService.sum(0, 1);
}
```

完整的代码可参考 [GitHub](https://github.com/Frederick-S/test-exception-message-with-junit-demo)。

## 参考

- [How do I assert my exception message with JUnit Test annotation?](https://stackoverflow.com/questions/2469911/how-do-i-assert-my-exception-message-with-junit-test-annotation)
