title: 使用 GitHub Action 自动构建和推送 Docker 镜像到 Docker Hub
tags:
- GitHub Action
- Docker
---

`Docker Hub` 的免费账户已不再支持关联 `GitHub` 仓库并自动构建镜像的功能，不过可以通过 `GitHub Action` 来自动构建和推送镜像。实现方式非常简单，`Docker` 官方已给出了示例（[Build and push Docker images](https://github.com/marketplace/actions/build-and-push-docker-images)）：

```
name: ci

on:
  push:
    branches:
      - 'master'

jobs:
  docker:
    runs-on: ubuntu-latest
    steps:
      -
        name: Set up QEMU
        uses: docker/setup-qemu-action@v1
      -
        name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v1
      -
        name: Login to DockerHub
        uses: docker/login-action@v1 
        with:
          username: ${{ secrets.DOCKERHUB_USERNAME }}
          password: ${{ secrets.DOCKERHUB_TOKEN }}
      -
        name: Build and push
        id: docker_build
        uses: docker/build-push-action@v2
        with:
          push: true
          tags: user/app:latest
```

一共有三处要注意，第一开头的 `branches` 下对于新建的仓库需要填写 `main` 而不是 `master`。

第二需要为 `Login to DockerHub` 阶段设置 `Docker` 的 `Access Token`，`Access Token` 可以通过 `Docker Hub` 的 `Account Settings -> Security -> New Access Token` 创建，然后通过 `GitHub` 仓库的 `Settings -> Secrets -> New repository secret` 分别创建 `DOCKERHUB_USERNAME` 和 `DOCKERHUB_TOKEN`。

第三最后的 `tags: user/app:latest` 中的 `user` 和 `app` 需要修改为实际的用户名和镜像名。

参考：

- [GitHub Action workflow not running](https://stackoverflow.com/questions/61989951/github-action-workflow-not-running)
