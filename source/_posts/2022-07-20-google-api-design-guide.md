title: '【读】Google API Design Guide'
tags:
- Design
---

## 介绍
[Google API Design Guide](https://cloud.google.com/apis/design) 是 `Google` 设计 [Cloud APIs](https://cloud.google.com/apis/docs/overview) 和其他 [Google APIs](https://github.com/googleapis/googleapis) 的设计指南。

该指南面向的不仅仅是 `REST APIs`，同时也适用于 `RPC APIs`，其中 `RPC APIs` 主要面向的是 `gRPC APIs`。

## 面向资源的设计
传统的 `RPC` 接口设计面向的是操作，各个接口之间是孤立的，没有明确的关联；不同系统的接口也有着不同的设计风格，存在着一定的学习和使用成本。

而面向资源的设计则将系统抽象为一系列资源，开发者则通过有限的几个标准方法来操作资源，从而实现对系统的修改。对于 `RESTful` 接口来说，有限的几个标准方法对应的就是 `HTTP` 请求方法中的 `POST`，`GET`，`PUT`，`PATCH` 和 `DELETE`。另一方面，由于遵循了统一的设计，当开发者调用不同系统的接口时，能够自然的假定各个系统都支持相同的标准方法，从而降低了开发者学习的成本。

> 不管是面向资源的设计还是其他的设计标准，统一的标准胜过百花齐放，开发者应当将更多的精力放在自身系统的业务实现上，而不是耗费时间学习和调试其他系统的接口。

该指南建议按照如下的步骤设计面向资源的接口：

* 确定接口所提供的资源类型
* 确定资源间的关系
* 根据资源类型和资源间的关系确定资源命名模式
* 确定资源体系
* 为每个资源设计最小限度的操作方法

在面向资源的设计体系下，各接口一般会按照资源的层级结构进行组织，层级结构中的每个节点可能是单一的资源，也可能是一个资源集合：

* 一个资源集合包含了一系列相同类型的资源，例如，一个用户拥有一个联系人资源集合
* 一个资源包含了若干的状态，同时也包含了0个或者多个子资源。每个子资源可以是一个单一资源或者是资源集合

以创建邮件接口为例，传统的接口设计可能是如下的方式：

```http
POST /createMail

{
    "userId": 123,
    "title:" "Title",
    "from": "from@example.com"
    "to": "to@example.com",
    "body": "Body"
}
```

而面向资源的接口设计则可能为：

```http
POST /users/{userId}/mails

{
    "title:" "Title",
    "from": "from@example.com"
    "to": "to@example.com",
    "body": "Body"
}
```

可以看到，面向资源的接口设计体现了资源间的层级关系。

## 参考
* [API design guide](https://cloud.google.com/apis/design)
