title: Permissions 0644 for 'xxx.pem' are too open
tags:
- SSH
---

使用 `SSH` 连接到 `Azure` 的虚拟机时遇到错误：

```
➜  ~ ssh -i /path/to/some.pem xxx@x.x.x.x
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
@         WARNING: UNPROTECTED PRIVATE KEY FILE!          @
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
Permissions 0644 for '/path/to/some.pem' are too open.
It is required that your private key files are NOT accessible by others.
This private key will be ignored.
Load key "/path/to/some.pem": bad permissions
xxx@x.x.x.x: Permission denied (publickey).
```

这是因为创建虚拟机时从 `Azure` 下载的私钥默认权限太大，需要将其权限改为只读且仅当前用户可见：

```
chmod 400 some.pem
```

参考：

- [Trying to SSH into an Amazon Ec2 instance - permission error](https://stackoverflow.com/questions/8193768/trying-to-ssh-into-an-amazon-ec2-instance-permission-error)
