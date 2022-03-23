title: 'MIT 6.824 Lab 1 (2) - Go Race Detector'
tags:
- MIT 6.824
- Go
---

`Lab 1` 中遇到的第一个命令是 `go build -race -buildmode=plugin ../mrapps/wc.go`，其中 `-race` 表示启用 `Go` 的竞争检测，在多线程编程时，数据竞争是必须要考虑的问题，而数据竞争的问题往往难以察觉和排查，所以 `Go` 内置了竞争检测的工具来帮助开发人员提前发现问题。另外，`MIT 6.824` 是一门分布式系统课程，必然会涉及多线程编程，所以竞争检测也是校验 `Lab` 程序正确性的一种方式。

`Go` 借助 `goroutine` 来实现并发编程，所以数据竞争发生在多个 `goroutine` 并发读写同一个共享变量时，并且至少有一个 `goroutine` 存在写操作。来看一个简单的例子：

```go
package main

import "fmt"

func main() {
    done := make(chan bool)
    m := make(map[string]string)
    m["name"] = "world"
    go func() {
        m["name"] = "data race"
        done <- true
    }()
    fmt.Println("Hello,", m["name"])
    <-done
}
```

其中 `m` 是一个共享变量，被两个 `goroutine` 并发读写，将上述代码保存为文件 `racy.go`，然后开启竞争检测执行：

```
go run -race racy.go
```

会输出类似如下内容：

```
Hello, world
==================
WARNING: DATA RACE
Write at 0x00c00007e180 by goroutine 7:
  runtime.mapassign_faststr()
      /usr/local/go/src/runtime/map_faststr.go:203 +0x0
  main.main.func1()
      /path/to/racy.go:10 +0x50

Previous read at 0x00c00007e180 by main goroutine:
  runtime.mapaccess1_faststr()
      /usr/local/go/src/runtime/map_faststr.go:13 +0x0
  main.main()
      /path/to/racy.go:13 +0x16b

Goroutine 7 (running) created at:
  main.main()
      /path/to/racy.go:9 +0x14e
==================
==================
WARNING: DATA RACE
Write at 0x00c000114088 by goroutine 7:
  main.main.func1()
      /path/to/racy.go:10 +0x5c

Previous read at 0x00c000114088 by main goroutine:
  main.main()
      /path/to/racy.go:13 +0x175

Goroutine 7 (running) created at:
  main.main()
      /path/to/racy.go:9 +0x14e
==================
Found 2 data race(s)
exit status 66
```

竞争检测会提示第13行和第10行存在数据竞争。

参考：

* [Introducing the Go Race Detector](https://go.dev/blog/race-detector)
* [Data Race Detector](https://go.dev/doc/articles/race_detector)
* [The Go Memory Model](https://go.dev/ref/mem)