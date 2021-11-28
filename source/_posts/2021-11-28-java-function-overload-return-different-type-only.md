title: 为什么 Java 方法重载不允许仅返回值类型不同
tags:
- Java
---

在 `Java` 中，如果两个同名方法仅返回值类型不同，这是不允许的，即编译器不会认为这是方法重载，如下述类中的方法 `f`：

```java
public class Demo {
    public int f() {
        return 0;
    }

    public String f() {
        return "";
    }
}
```

编译器会提示 `'f()' is already defined in 'Demo'`。假设编译器支持这种方式的方法重载，会有什么问题？在某些情况下，编译器无法区分调用的是哪个方法，例如当调用 `f()` 却忽略返回值时：

```java
f();
```

所以仅返回值类型不同不能作为方法重载的形式。

参考：

- Thinking in Java
