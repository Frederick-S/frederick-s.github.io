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

可以看到，面向资源的接口设计体现了资源间的层级关系。一般而言，对于 `RESTful` 接口来说，请求 `URL` 中只会包含资源的名称（名词），而不会包含对资源的操作（动词），`HTTP` 的请求方法就对应了资源的标准操作方法。而该指南讨论的是通用的面向资源的设计，其对应的资源标准操作方法为：`List`，`Get`，`Create`，`Update` 和 `Delete`。

## 资源名称
在面向资源的设计下，资源是一个命名实体，每个资源都有一个唯一的名称作为其标识符。一个资源的名称由三部分组成：

1. 资源的 `ID`
2. 所有父资源的 `ID`
3. `API` 服务名，如 `gmail.googleapis.com`

资源集合被视为一种特殊的资源，它包含了一组相同类型的子资源，例如一个目录可以被视为一个资源集合，它包含了一组文件资源。同时，资源集合也有相应的 `ID`。

资源名称由资源 `ID` 和资源集合 `ID` 组成，其定义也体现了资源的层级结构关系，各层级之间使用 `/` 进行分隔。例如，对于某个对象存储服务中的对象来说，其资源名称可能为 `//storage.googleapis.com/buckets/bucket-123/objects/object-123`，其中最顶层为服务名，即 `//storage.googleapis.com`，然后是一个资源集合，即 `buckets`，对象存储服务一般以 `bucket` 为维度来管理对象；接下来为了要定位到某个对象，需要先定位到具体的 `bucket`，`bucket-123` 就是某个 `bucket` 的资源 `ID`，而每个 `bucket` 下包含了多个对象，进而产生了一个资源集合 `objects`，最后的 `object-123` 就是实际对象的资源 `ID`。

一般来说，一个资源在实现上很可能对应一张数据库的表，所以可以用表的主键来作为资源的 `ID`。而由于使用了 `/` 来分隔资源的层级，因此只有最底层的资源才允许资源 `ID` 中包含 `/`，从而避免层级歧义。

> 如果资源 `ID` 中包含了 `/`，则必须在 `API` 文档中明确声明。

资源集合更多的是一种层级上的逻辑概念，所以其 `ID` 命名需要有意义，以及符合以下的规范：

1. 必须是以小驼峰形式命名的英文单词复数，如果对应单词没有复数，则应当使用单词的单数形式
2. 必须使用简洁明了的英文单词
3. 避免使用过于宽泛的英文单词，例如，`rowValues` 优于 `values`。同时应当避免无条件的使用如下的英文单词：
   * elements
   * entries
   * instances
   * items
   * objects
   * resources
   * types
   * values

> 完整的资源名称是协议无关的，

## 标准方法


TODO:
1. 资源更新，/resources/id，实体里就不需要id，见digitalocean api
2. 分页返回结果，github api返回结果没有包含分页信息，以及总数信息

## 参考
* [API design guide](https://cloud.google.com/apis/design)
* [What’s the best RESTful method to return total number of items in an object?](https://stackoverflow.com/questions/3715981/what-s-the-best-restful-method-to-return-total-number-of-items-in-an-object)