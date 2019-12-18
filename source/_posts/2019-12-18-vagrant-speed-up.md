title: Vagrant 镜像下载加速
categories:
- Vagrant
---

可以使用[清华大学开源软件镜像站](https://mirrors.tuna.tsinghua.edu.cn/)上的镜像。例如需要下载 `Ubuntu 18` 的镜像，先在站内找到对应镜像的地址，然后在终端执行 `vagrant box add https://mirrors.tuna.tsinghua.edu.cn/ubuntu-cloud-images/bionic/20191211/bionic-server-cloudimg-amd64-vagrant.box --name ubuntu/bionic`，下载完成后即可执行 `vagrant init ubuntu/bionic` 等操作。

参考：

- [vagrant box国内有镜像地址可以下载吗？](https://segmentfault.com/q/1010000011063709/a-1020000011064302)
