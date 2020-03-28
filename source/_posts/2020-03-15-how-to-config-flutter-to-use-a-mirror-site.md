title: 配置 Flutter 国内镜像
tags:
- Flutter
---

由于网络原因，`Flutter` 项目获取包依赖时有可能会失败，可通过设置镜像地址解决，新增如下两个环境变量即可：

```sh
export PUB_HOSTED_URL=https://pub.flutter-io.cn
export FLUTTER_STORAGE_BASE_URL=https://storage.flutter-io.cn
```

参考：

- [Configuring Flutter to use a mirror site](https://flutter.dev/community/china#configuring-flutter-to-use-a-mirror-site)
