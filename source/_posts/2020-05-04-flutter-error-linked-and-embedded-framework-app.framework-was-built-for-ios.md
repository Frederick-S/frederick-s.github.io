title: Flutter error - Building for iOS Simulator, but the linked and embedded framework 'App.framework' was built for iOS
tags:
- Flutter
---

`Flutter` 项目打包 `iOS` 应用的时候遇到个错误：

```
Building for iOS Simulator, but the linked and embedded framework 'App.framework' was built for iOS. (in target 'Runner' from project 'Runner')
```

这个问题在 `Flutter` 的 `GitHub` 仓库中也有人提到，解决方法也比较简单，删除 `App.framework` 文件夹即可，即 `rm -rf ios/Flutter/App.framework`。

参考：

- [[App.framework] Linked and embedded framework 'App.framework' was built for iOS/iOS Simulator](https://github.com/flutter/flutter/issues/50568)