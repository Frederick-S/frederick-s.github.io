title: 'MIT 6.824 Lab 1 (2) - Go Race Detector'
tags:
- MIT 6.824
- Go
---

`Lab 1` 中遇到的第一个命令是 `go build -race -buildmode=plugin ../mrapps/wc.go`，其中 `-race` 表示启用 `Go` 的竞争检测，在多线程编程时，数据竞争是必须要考虑的问题，而数据竞争的问题往往难以察觉和排查，所以 `Go` 内置了竞争检测的工具来帮助开发人员提前发现问题。另外，`MIT 6.824` 是一门分布式系统课程，必然会涉及多线程编程，所以竞争检测也是校验 `Lab` 程序正确性的一种方式。

## 介绍
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

竞争检测会提示第13行和第10行存在数据竞争，一共涉及两个 `goroutine`，一个是主 `goroutine`，另一个是手动创建的 `goroutine`。

## 典型数据竞争场景
### 循环计数器竞争
这个例子中每次循环时会创建一个 `goroutine`，每个 `goroutine` 会打印循环计数器 `i` 的值：

```go
package main

import (
	"fmt"
	"sync"
)

func main() {
	var wg sync.WaitGroup
	wg.Add(5)
	for i := 0; i < 5; i++ {
		go func() {
			fmt.Println(i) // Not the 'i' you are looking for.
			wg.Done()
		}()
	}
	wg.Wait()
}
```

我们想要的结果是能输出 `01234` 这5个值（实际顺序并不一定是 `01234`），但由于主 `goroutine` 对 `i` 的更新和被创建的 `goroutine` 对 `i` 的读取之间存在数据竞争，最终的输出结果可能是 `55555` 也可能是其他值。

如果要修复这个问题，每次创建 `goroutine` 时复制一份 `i` 再传给 `goroutine` 即可：

```go
package main

import (
	"fmt"
	"sync"
)

func main() {
	var wg sync.WaitGroup
	wg.Add(5)
	for i := 0; i < 5; i++ {
		go func(j int) {
			fmt.Println(j) // Good. Read local copy of the loop counter.
			wg.Done()
		}(i)
	}
	wg.Wait()
}
```

### 意外地共享变量
在日常开发中，我们可能不经意间在多个 `goroutine` 间共享了某个变量。在下面的例子中，首先 `f1, err := os.Create("file1")` 会创建一个 `err` 变量，接着在第一个 `goroutine` 中对 `file1` 写入时会对 `err` 进行更新（`_, err = f1.Write(data)`），然而在主 `goroutine` 中创建 `file2` 时同样会对 `err` 进行更新（`f2, err := os.Create("file2")`，这里虽然用的是 `:=`，不过 `err` 并不是一个新的变量，在同一个作用域下是不允许重复对某个变量使用 `:=` 创建的，因为 `f2` 是一个新的变量，所以这里可用 `:=`），这就产生了数据竞争：

```go
package main

import "os"

// ParallelWrite writes data to file1 and file2, returns the errors.
func ParallelWrite(data []byte) chan error {
	res := make(chan error, 2)
	f1, err := os.Create("file1")
	if err != nil {
		res <- err
	} else {
		go func() {
			// This err is shared with the main goroutine,
			// so the write races with the write below.
			_, err = f1.Write(data)
			res <- err
			f1.Close()
		}()
	}
	f2, err := os.Create("file2") // The second conflicting write to err.
	if err != nil {
		res <- err
	} else {
		go func() {
			_, err = f2.Write(data)
			res <- err
			f2.Close()
		}()
	}
	return res
}

func main() {
	err := ParallelWrite([]byte("Hello, world!"))
	<-err
}
```

参考：

* [Introducing the Go Race Detector](https://go.dev/blog/race-detector)
* [Data Race Detector](https://go.dev/doc/articles/race_detector)
* [The Go Memory Model](https://go.dev/ref/mem)