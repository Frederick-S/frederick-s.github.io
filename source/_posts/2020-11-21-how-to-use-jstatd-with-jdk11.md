title: 如何在 JDK 11 中建立 jstatd 连接
tags:
- Java
- VisualVM
---

使用 `VisualVM` 的 `Virsual GC` 插件需要先和服务器建立 `jstatd` 连接，在 `JDK 9` 之前需要首先创建一个 `policy` 文件并声明权限：

```
grant codebase "file:${java.home}/lib/tools.jar" {
   permission java.security.AllPermission;
};
```

然而，从 `JDK 9` 开始，`tools.jar` 已被移除，需要将 `policy` 文件的内容修改为：

```
grant codebase "jrt:/jdk.jstatd" {    
   permission java.security.AllPermission;    
};

grant codebase "jrt:/jdk.internal.jvmstat" {    
   permission java.security.AllPermission;    
};
```

参考：

- [Starting jstatd in Java 9+](https://stackoverflow.com/questions/51032095/starting-jstatd-in-java-9)
