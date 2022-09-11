title: 创建 EKS 集群
tags:
- Kubernetes
- AWS
- EKS
---

## 介绍
`EKS`（`Amazon Elastic Kubernetes Service`）是 `AWS` 提供的 `Kubernetes` 服务，它能大大减轻创建和维护 `Kubernetes` 集群的负担。

## 创建 EKS 集群
有两种方式来创建 `EKS` 集群，一种是使用本地的 `eksctl` 程序；另一种是通过 `AWS` 的管理后台（`AWS Management Console`），这里选择通过 `AWS` 的管理后台来创建 `EKS` 集群。

### 创建 Cluster service role
创建 `EKS` 集群时需要绑定一个 `IAM` 角色，因为 `Kubernetes` 的 `control plane` 需要管理集群内的资源，所以需要有相应的操作权限。

首先进入 [IAM 控制台](https://console.aws.amazon.com/iam/)，选择左侧 `Access management` 下的 `Roles`，点击 `Create role`。在 `Trusted entity type` 下选择 `AWS service`，然后在 `Use cases for other AWS services` 下选择 `EKS`，接着选择 `EKS - Cluster` 并点击 `Next`。在 `Add permissions` 这步直接点击 `Next`。在最后一步设置所创建的角色的名字，如 `eksClusterRole`，最后点击 `Create role` 创建角色。

### 创建集群
我们通过 `AWS` 管理后台中的 `Amazon Elastic Kubernetes Service` 界面来创建集群，第一步的 `Configure cluster` 主要设置集群的名称，如 `my-cluster`，以及绑定在之前步骤中所创建的 `Cluster service role`。第二步的 `Specify networking` 这里基本都保持默认，只是将 `Cluster endpoint access` 设置为 `Public and private`。第三步的 `Configure logging` 可以暂时不开启日志监控。最后在第四步的 `Review and create` 点击 `Create` 创建集群。

## 创建 Node group
当集群的状态变为 `Active` 后就表示集群创建成功，不过此时集群中还没有任何 `Node`，所以系统级别的 `Pod` 还无法正常工作，比如在集群详情的 `Resources` 下查看某个 `coredns` 的 `Pod` 会显示 `FailedScheduling`，因为 `no nodes available to schedule pods`。

我们需要创建 `Node group` 来为系统添加可用的 `Node`。

### 创建 Node IAM role
在创建 `Node group` 前，需要创建一个 `Node IAM role`。因为集群中的 `Node` 内部会运行着一个叫做 `kubelet` 的程序，它负责和集群的 `control plane` 进行通信，例如将当前 `Node` 注册到集群中，而某些操作需要调用 `AWS` 的接口，所以和 `Cluster service role` 类似，也需要绑定相应的权限。

这里同样也是通过 [IAM 控制台](https://console.aws.amazon.com/iam/) 来创建角色，在 `Trusted entity type` 下选择 `AWS service`，在 `Use case` 下选择 `EC2`，然后点击 `Next`。在第二步的 `Add permissions` 需要添加 `AmazonEKSWorkerNodePolicy`，`AmazonEC2ContainerRegistryReadOnly` 和 `AmazonEKS_CNI_Policy` 三个权限，虽然文档中说不建议将 `AmazonEKS_CNI_Policy` 权限添加到 `Node IAM role` 上，不过这里作为示例教程将三个权限都绑定在了 `Node IAM role` 上。最后也是点击 `Create role` 创建角色。

### 创建 Node group
在集群详情的 `Compute` 下点击 `Add node group` 来创建 `Node group`，在第一步 `Configure node group` 中设置 `node group` 的名称及绑定在之前步骤中所创建的 `Node IAM role`。在第二步 `Set compute and scaling configuration` 里配置节点的类型和数量等信息，作为教程都采用默认配置。第三步 `Specify networking` 同样采用默认配置。最后在第四步的 `Review and create` 点击 `Create` 完成创建。

最后当所创建的 `Node group` 的状态变为 `Active` 以及该 `Node group` 下的 `Node` 的状态变为 `Ready` 时说明节点创建成功。此时再查看集群详情下 `Resources` 的 `coredns` 的 `Pod` 已成功分配了 `Node` 运行。

## 连接 EKS 集群
日常需要通过 `kubectl` 管理集群，所以需要先在本地配置访问 `EKS` 集群的权限。`kubectl` 本质上是和 `Kubernetes API server` 打交道，而创建集群时 `Cluster endpoint access` 部分选择的是 `Public and private`，所以在这个场景下能够从公网管理 `EKS` 集群。

首先需要安装 [AWS CLI](https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html) 和 [kubectl](https://docs.aws.amazon.com/eks/latest/userguide/install-kubectl.html)。然后在本地通过 `aws configure` 来设置 `AWS Access Key ID` 和 `AWS Secret Access Key`。根据 [Enabling IAM user and role access to your cluster](https://docs.aws.amazon.com/eks/latest/userguide/add-user-role.html) 的描述，创建集群的账号会自动授予集群的 `system:masters` 权限，本文是通过 `AWS` 的管理后台创建集群，当前登录的账号为 `root`，所以 `aws configure` 需要设置为 `root` 的 `AWS Access Key ID` 和 `AWS Secret Access Key`：

> When you create an Amazon EKS cluster, the AWS Identity and Access Management (IAM) entity user or role, such as a federated user that creates the cluster, is automatically granted system:masters permissions in the cluster's role-based access control (RBAC) configuration in the Amazon EKS control plane.

一般公司生产环境中的 `AWS` 是不会直接使用 `root` 账户登录的，而是创建 `IAM` 用户，由于这里是个人的 `AWS` 账号所以直接使用了 `root`。设置完成之后可以通过 `aws sts get-caller-identity` 来验证当前用户是否设置正确。然后运行 `aws eks update-kubeconfig --region us-west-2 --name my-cluster` 来更新本地的 `kubeconfig`，其中 `us-west-2` 需要修改为实际的 `AWS Region`，`my-cluster` 需要修改为实际的集群名称。最后就可以通过 `kubectl get all` 来验证能否访问集群，如果没有问题就会输出如下内容：

```
NAME                 TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
service/kubernetes   ClusterIP   10.100.0.1   <none>        443/TCP   175m
```

## 参考
* [Creating the Amazon EKS cluster role](https://docs.aws.amazon.com/eks/latest/userguide/service_IAM_role.html#create-service-role)
* [Creating the Amazon EKS node IAM role](https://docs.aws.amazon.com/eks/latest/userguide/create-node-role.html#create-worker-node-role)
* [Enabling IAM user and role access to your cluster](https://docs.aws.amazon.com/eks/latest/userguide/add-user-role.html)
* [kubectl error You must be logged in to the server (Unauthorized) when accessing EKS cluster](https://stackoverflow.com/questions/50791303/kubectl-error-you-must-be-logged-in-to-the-server-unauthorized-when-accessing)