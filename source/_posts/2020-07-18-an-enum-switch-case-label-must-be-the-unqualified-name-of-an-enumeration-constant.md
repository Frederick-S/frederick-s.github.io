title: An enum switch case label must be the unqualified name of an enumeration constant
tags:
- Java
- Maven
---

`An enum switch case label must be the unqualified name of an enumeration constant` 是 `Java` 中常见的编译错误，基本上 `Google` 搜索出来的错误场景都是因为在 `switch` 中使用枚举时搭配了类名造成，例如：

```java
Season season = Season.SPRING;

switch (season) {
    // 编译错误，直接使用 SPRING 即可
    case Season.SPRING:
        System.out.println("spring");

        break;
    case Season.SUMMER:
        System.out.println("summer");

        break;
    default:
        break;
}
```

然而，如果某个枚举值不存在，也会提示一样的错误，例如：

```java
Season season = Season.SPRING;

switch (season) {
    case SOME_VALUE_DOES_NOT_EXIST:
        System.out.println("spring");

        break;
    default:
        break;
}
```

这种情况下的错误提示容易让人摸不着头脑，`IntelliJ IDEA` 的错误提示则较为友好：`Cannot resolve symbol 'SOME_VALUE_DOES_NOT_EXIST'`。对于这种错误场景，实际工作中遇到一个例子：

1. 在开发阶段，`A` 拉了个 `some.package` 的分支，更新版本号为 `a.b-SNAPSHOT` 并发布，将其引入 `some.app`，推送代码后触发了 `some.app` 的 `Jenkins` 构建任务
2. `B` 也拉了个 `some.package` 的分支，同样更新版本号为 `a.b-SNAPSHOT` 并发布，并增加了一个新的枚举值到 `SomeEnum`，同样将其引入 `some.app`，推送代码后触发了 `some.app` 的 `Jenkins` 构建任务，此时任务构建失败，提示编译错误：`An enum switch case label must be the unqualified name of an enumeration constant`

出现这样的原因是 `Jenkins` 执行构建任务时执行的编译命令是 `mvn compile`，在 `A` 提交任务时，构建服务器下载了 `some.package` 的 `a.b-SNAPSHOT` 版本，由于是 `SNAPSHOT` 版本，在 `B` 提交任务时，构建服务器没有重新下载 `some.package`，导致服务器中的 `some.package` 没有 `B` 新增的修改，从而出现编译错误，解决方法是在编译时增加 `-U` 参数来强制更新 `SNAPSHOT`。

参考：

- [error: an enum switch case label must be the unqualified name of an enumeration constant](https://stackoverflow.com/questions/54708788/error-an-enum-switch-case-label-must-be-the-unqualified-name-of-an-enumeration/54708876)
