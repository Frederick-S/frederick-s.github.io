title: "如何解决 add-apt-repository: not found 错误"
tags:
- Ubuntu
---

`Ubuntu` 下执行 `add-apt-repository` 添加第三方仓库时遇到 `add-apt-repository: not found` 错误，执行以下命令即可：

```sh
sudo apt-get update
sudo apt-get install software-properties-common
```

参考：

- [How To Fix 'Add-Apt-Repository Command Not Found' On Ubuntu & Debian](https://phoenixnap.com/kb/add-apt-repository-command-not-found-ubuntu)
