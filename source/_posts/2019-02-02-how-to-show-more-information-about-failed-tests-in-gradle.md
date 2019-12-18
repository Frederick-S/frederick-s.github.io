title: 如何在 Gradle 的测试中显示详尽的错误信息
categories:
- Gradle
---

在`build.gradle`中添加如下设置：

```bash
test {
    testLogging {
        exceptionFormat = 'full'
    }
}
```

参考：

- [Gradle Goodness: Show More Information About Failed Tests](http://mrhaki.blogspot.com/2013/05/gradle-goodness-show-more-information.html)
