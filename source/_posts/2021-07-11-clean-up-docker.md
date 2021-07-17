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

参考：

- [How to clean your Docker data](https://dockerwebdev.com/tutorials/clean-up-docker/)
- [Dangling or Unused Images in Docker](https://jinnabalu.medium.com/docker-frequently-used-commands-on-images-b812d76a4b8e)
