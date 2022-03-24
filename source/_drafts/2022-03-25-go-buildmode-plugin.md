title: 'MIT 6.824 Lab 1 (3) - Go buildmode'
tags:
- MIT 6.824
- Go
---

`Lab 1` 中遇到的第一个命令是 `go build -race -buildmode=plugin ../mrapps/wc.go`，其中 `-buildmode=plugin` 表示以插件的形式打包源文件，这里的 `wc.go` 是用户实现的 `map` 和 `reduce` 方法，这体现了面向接口编程的思想，只要用户编写的 `map` 和 `reduce` 方法遵循统一的签名，则可以在不重新编译 `MapReduce` 框架代码的情况下，实时替换运行不同的用户应用。

参考：
