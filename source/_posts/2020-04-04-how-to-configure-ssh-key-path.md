title: SSH 连接服务器时指定私钥的路径
tags:
- SSH
- Visual Studio Code
---

使用 `Visual Studio Code` 的 `Remote - SSH` 插件连接服务器开发时，有可能会遇到不同的服务器对应不同的私钥的情况，这时就需要单独为各个服务器指定私钥的位置。打开 `SSH` 配置文件（默认路径是 `~/.ssh/config`），在需要指定私钥路径的服务器下添加 `IdentityFile path-to-private-key` 即可，例如：

```
Host your-host
  HostName your-host-name
  User your-host-user
  ForwardAgent yes
  IdentityFile path-to-private-key
```

参考：

- [Configure ssh-key path to use for a specific host](https://superuser.com/questions/1276485/configure-ssh-key-path-to-use-for-a-specific-host)
