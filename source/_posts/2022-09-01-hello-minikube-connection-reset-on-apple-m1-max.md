title: "Hello Minikube - Apple M1 Max connection reset"
tags:
- Kubernetes
- minikube
---

在 `Apple M1 Max` 处理器上按照 [Hello Minikube](https://kubernetes.io/docs/tutorials/hello-minikube/) 进行 `minikube` 的入门教程，不过最后通过本地链接访问的时候出现了 `connection reset`。按照 [这里](https://github.com/kubernetes/minikube/issues/12036) 的描述需要将镜像由 `echoserver:1.4` 换成适用于 `Apple M1 Max` 的 `echoserver-arm:1.8`，即：

```sh
kubectl create deployment hello-arm --image=registry.k8s.io/echoserver-arm:1.8
```

不过帖子中也有人提到换了镜像之后依然无效，所以也不一定对所有人有用。

参考：
* [Cannot connect to service from localhost on M1 Mac](https://github.com/kubernetes/minikube/issues/12036)