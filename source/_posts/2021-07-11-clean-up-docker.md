title: Docker 磁盘占用清理
tags:
- Docker
---

日常随着 `Docker` 的使用，`Docker` 会逐渐占用磁盘空间，通过 `docker system df` 可查看 `Docker` 所占用的空间：

```
TYPE            TOTAL     ACTIVE    SIZE      RECLAIMABLE
Images          20        14        22.21GB   17.07GB (76%)
Containers      29        0         6.743GB   6.743GB (100%)
Local Volumes   2         0         417MB     417MB (100%)
Build Cache     0         0         0B        0B
```

其中 `Images` 表示镜像，`Containers` 表示容器，`Local Volumes` 表示本地卷，`Build Cache` 表示构建缓存。

## 整体清理
可以通过 `docker system prune` 进行一次空间清理：

```
WARNING! This will remove:
  - all stopped containers
  - all networks not used by at least one container
  - all dangling images
  - all dangling build cache

Are you sure you want to continue? [y/N]
```

该操作会删除所有停止的容器，所有未被至少一个容器使用的网络，所有的 `dangling` 镜像（在构建镜像时产生的 `tag` 为 `none` 的镜像，没有和任何其他有 `tag` 的镜像有关联），所有的 `dangling` 构建缓存（和 `dangling` 镜像同理）。

更激进一点，还可以执行 `docker system prune -a`，该操作还会删除没有和运行中的容器有关联的镜像。

## 镜像清理
`Docker` 镜像是某个应用（如数据库、某个程序语言的运行时）的磁盘快照，可以通过 `docker image ls -a` 查看所有的镜像（活跃的以及 `dangling` 的镜像）：

```
REPOSITORY    TAG       IMAGE ID       CREATED        SIZE
hello-world   latest    d1165f221234   4 months ago   13.3kB
```

可以通过 `docker image rm <name_or_id>` 来删除某个镜像，支持批量删除多个镜像，多个镜像 `id` 之间使用空格分隔即可。不过，删除镜像要求该镜像没有被某个容器所使用，否则会提示下述类似错误：

```
Error response from daemon: conflict: unable to delete 4cdc5dd7eaad (must be forced) - image is being used by stopped container 3d9f62acc483
Error response from daemon: conflict: unable to delete d1165f221234 (must be forced) - image is being used by stopped container 57027ba35bdd
```

可以通过在执行时增加 `-f` 来强制删除镜像。

## 容器清理
容器是某个镜像的一个运行实例，可以通过 `docker container ls -a` 查看所有的容器：

```
CONTAINER ID   IMAGE          COMMAND                  CREATED          STATUS                      PORTS     NAMES
3d9f62acc483   4cdc5dd7eaad   "/docker-entrypoint.…"   11 minutes ago   Exited (0) 11 minutes ago             sleepy_babbage
```

要删除一个容器必须要先停止该容器（`docker container stop <name_or_id>`），然后通过 `docker container rm <name_or_id>` 删除，同样的，和删除镜像类似，该命令支持批量删除多个容器，多个容器 `id` 之间使用空格分隔。

## 网络清理
`Docker` 网络用于容器间的通信，它们都是一些配置文件，并不会占用多大空间，可以通过 `docker network ls` 查看所有的网络：

```
NETWORK ID     NAME      DRIVER    SCOPE
b96312481a51   bridge    bridge    local
85a64f881d4d   host      host      local
e6808b80f888   none      null      local
```

可以通过 `docker network rm <name_or_id>` 来删除一个网络。

## 数据卷清理
`Docker` 数据卷用于持久化容器运行时保存的数据，例如通过 `Docker` 运行 `MySQL` 时指定数据卷，从而对 `MySQL` 的数据进行备份，可以通过 `docker volume ls` 查看所有的数据卷：

```
DRIVER    VOLUME NAME
local     test-volume
```

同样的，可以通过 `docker volume rm <name>` 来删除指定的数据卷，或者使用 `docker volume prune` 来删除所有未和运行中的容器关联的数据卷，以及通过 `docker volume prune -a` 删除所有的数据卷。

参考：

- [How to clean your Docker data](https://dockerwebdev.com/tutorials/clean-up-docker/)
- [Dangling or Unused Images in Docker](https://jinnabalu.medium.com/docker-frequently-used-commands-on-images-b812d76a4b8e)
