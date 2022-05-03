title: 如何减小 Docker 镜像的大小
tags:
- Docker
---

## 问题
首先来看一个例子，构建一个 `C` 语言版的 `hello world` 镜像：

```c
/* hello.c */
int main () {
  puts("Hello, world!");
  return 0;
}
```

对应的 `Dockerfile` 为：

```
FROM gcc
COPY hello.c .
RUN gcc -o hello hello.c
CMD ["./hello"]
```

然后执行 `docker build -t hello-world .` 构建一个名为 `hello-world` 的镜像，然而以这种方式构建的镜像的大小竟然有1.19 GB：

```
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
hello-world         latest              b11e170bd1d2        6 minutes ago       1.19GB
```

因为这种构建方式生成的镜像会同时包含 `gcc` 镜像的内容，查看 `gcc` 镜像大小发现达到了1.19 GB：

```
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
gcc                 latest              21f378ba43ec        11 days ago         1.19GB
```

如果我们把基础镜像换成 `Ubuntu` 并安装 `gcc` 编译 `hello.c` 重新构建镜像，最后的镜像大小为213 MB：

```
FROM ubuntu
COPY hello.c .
RUN apt-get update && apt-get install gcc -y
RUN gcc -o hello hello.c
CMD ["./hello"]
```

```
REPOSITORY          TAG                 IMAGE ID            CREATED              SIZE
hello-world         latest              42f17d1d12a5        About a minute ago   213MB
```

虽然新镜像相比1.19 GB有大幅减少，但相比于 `hello-world` 程序本身的大小（17k）来说，213 MB依然是个庞大的数字：

```
$ ls hello -hl
-rwxr-xr-x 1 root root 17K Aug  4 13:54 hello
```

## 解决
### Multi-stage
对于 `hello-world` 这个镜像来说，我们真正需要的只是最终的可执行程序，而并不关心中间的编译过程，如果能将编译阶段作为一个临时阶段而并不包含在最终的镜像中，则可有效减少最终的镜像大小。针对此，`Docker` 在 17.05 版本开始提供了名为 `multi-stage` 构建的功能。我们将原来的 `Dockerfile` 稍作修改，将原来的编译阶段抽取为一个 `stage`，然后将编译好的可执行文件复制到最终的 `stage` 中：

```
FROM gcc AS mybuildstage
COPY hello.c .
RUN gcc -o hello hello.c
FROM ubuntu
COPY --from=mybuildstage hello .
CMD ["./hello"]
```

最终的镜像大小只有73.9 MB：

```
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
hello-world         latest              7dd2b51c53b2        7 minutes ago       73.9MB
```

### `FROM scratch`
在上一步中，我们使用 `Ubuntu` 作为基础镜像来运行 `hello-world`，相比于一个可执行程序，`Ubuntu` 依然过于庞大，有没有比 `Ubuntu` 更轻量的镜像呢？有，那就是 `scratch`，这表示一个空的镜像，继续将 `Dockerfile` 稍作修改：

```
FROM gcc AS mybuildstage
COPY hello.c .
RUN gcc -o hello hello.c
FROM scratch
COPY --from=mybuildstage hello .
CMD ["./hello"]
```

最终的镜像大小只有16.4 KB：

```
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
hello-world         latest              676253b0e9c4        31 minutes ago      16.4kB
```

不过在运行该镜像时却提示错误：

```
standard_init_linux.go:211: exec user process caused "no such file or directory"
```

这是因为这种方式构建出的镜像缺少 `hello-world` 运行时依赖的库。我们可以在编译 `hello-world` 时通过指定 `-static` 参数将依赖的库包含到最后的可执行文件中来解决这个问题：

```
FROM gcc AS mybuildstage
COPY hello.c .
RUN gcc -o hello hello.c -static
FROM scratch
COPY --from=mybuildstage hello .
CMD ["./hello"]
```

不过包含了依赖的库后最终镜像的大小也上涨为945 KB：

```
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
hello-world         latest              e6a1fccc2de7        9 seconds ago       945kB
```

另外，如果不想将依赖的库包含到最终的镜像中，可以使用 `busybox:glibc` 这个基础镜像，该镜像包含了 `C` 语言的标准库，有了这个镜像在编译 `hello-world` 时则无需指定 `-static` 参数：

```
FROM gcc AS mybuildstage
COPY hello.c .
RUN gcc -o hello hello.c
FROM busybox:glibc
COPY --from=mybuildstage hello .
CMD ["./hello"]
```

不过由于该镜像本身有一定大小，最终镜像的大小达到了5.22 MB：

```
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
hello-world         latest              e2f2c0544800        7 seconds ago       5.22MB
```

## 总结
通过 `multi-stage` 构建可以有效的减少 `Docker` 镜像的大小，而基础镜像的选择则要具体情况分析，在满足需求的情况下选择合理的基础镜像。

## 参考

- [Docker Images : Part I - Reducing Image Size](https://www.ardanlabs.com/blog/2020/02/docker-images-part1-reducing-image-size.html)
