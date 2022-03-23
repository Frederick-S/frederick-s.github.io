title: 'MIT 6.824 Lab 1 (2) - Go Race Detector'
tags:
- MIT 6.824
- Go
---

`Lab 1` 中遇到的第一个命令是 `go build -race -buildmode=plugin ../mrapps/wc.go`，其中 `-race` 表示启用 `Go` 的竞争检测，在多线程编程时，数据竞争是必须要考虑的问题，而数据竞争的问题往往难以察觉和不易排查，所以 `Go` 内置了竞争检测的工具来帮助开发人员提前发现问题。另外，`MIT 6.824` 是一门分布式系统课程，比如会涉及多线程编程，所以竞争检测也是校验 `Lab` 程序正确性的一种方式。

参考：

* [Introducing the Go Race Detector](https://go.dev/blog/race-detector)
* [Data Race Detector](https://go.dev/doc/articles/race_detector)
* [The Go Memory Model](https://go.dev/ref/mem)