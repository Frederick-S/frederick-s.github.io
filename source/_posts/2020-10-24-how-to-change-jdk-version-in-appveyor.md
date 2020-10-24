title: 如何修改 AppVeyor 的 JDK 版本
tags:
- Java
- AppVeyor
---

使用 `AppVeyor` 的`Visual Studio 2019` 镜像构建 `Java` 项目时默认使用的是 `JDK 1.8`（[这里](https://www.appveyor.com/docs/windows-images-software/#java)说明了 `AppVeyor` 各个镜像下默认使用的 `JDK` 版本，虽然表格里写着 `Visual Studio 2019` 镜像下的默认 `JDK` 是1.7，不过实际是1.8），如果想更换 `JDK` 版本，比如更换为 `JDK 11`，可以重新设置 `JAVA_HOME` 和 `PATH`：

```
before_test:
  - SET JAVA_HOME=C:\Program Files\Java\jdk11
  - SET PATH=%JAVA_HOME%\bin;%PATH%
```

完整的代码可参考 [GitHub](https://github.com/Frederick-S/appveyor-jdk11-demo)。