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

首先进入 [IAM 控制台](https://console.aws.amazon.com/iam/)，选择左侧 `Access management` 下的 `Roles`，点击 `Create role`。在 `Trusted entity type` 下选择 `AWS service`，然后在 `Use cases for other AWS services` 下选择 `EKS`，接着选择 `EKS - Cluster` 并点击 `Next`。在 `Add permissions` 这步直接点击 `Next`。在最后一步设置所创建的角色的名字，如 `eksClusterRole`，最后点击 `Create role`。

### 创建集群
我们通过 `AWS` 管理后台中的 `Amazon Elastic Kubernetes Service` 界面来创建集群，第一步的 `Configure cluster` 主要设置集群的名称，如 `my-cluster`，以及绑定在之前步骤中所创建的 `Cluster service role`。第二步的 `Specify networking` 这里基本都保持默认，只是将 `Cluster endpoint access` 设置为 `Public and private`。第三步的 `Configure logging` 可以暂时不开启日志监控。最后在第四步的 `Review and create` 点击 `Create` 创建集群。

## 参考
* [Creating the Amazon EKS cluster role](https://docs.aws.amazon.com/eks/latest/userguide/service_IAM_role.html#create-service-role)