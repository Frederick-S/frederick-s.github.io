title: "如何解决 ModuleNotFoundError: No module named 'distutils.util' 错误"
tags:
- Python
---

`Ubuntu` 下安装 `pip` 时遇到 `ModuleNotFoundError: No module named 'distutils.util'` 错误，执行以下命令即可：

```sh
sudo apt-get install python3-distutils
```

参考：

- [Issue with "python3 get-pip.py --user" with python 3.6.7](https://github.com/pypa/get-pip/issues/43)
