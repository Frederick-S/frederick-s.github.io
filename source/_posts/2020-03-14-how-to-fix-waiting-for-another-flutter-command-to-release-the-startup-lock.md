title: 如何解决“Waiting for another flutter command to release the startup lock...”
categories:
- Flutter
---

执行 `Flutter` 包管理相关命令时有可能遇到 `Waiting for another flutter command to release the startup lock...` 这样的错误，可尝试杀死所有的 `dart` 进程解决：

```sh
// Linux
killall -9 dart

// Windows
taskkill /F /IM dart.exe
```

参考：

- [Waiting for another flutter command to release the startup lock](https://stackoverflow.com/questions/51679269/waiting-for-another-flutter-command-to-release-the-startup-lock)
