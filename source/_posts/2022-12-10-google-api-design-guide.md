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

> 对于 `Google` 的服务来说，资源集合 `ID` 还会经常出现在自动生成的客户端类库代码中，所以它们的命名也必须是合法的 `C/C++` 标识符。

完整的资源名称是协议无关的，虽然它看起来像 `RESTful` 服务的 `HTTP` 接口请求路径，但本质上这是两个东西。实际的资源请求还需要附带版本号，协议等信息，例如对于资源名称 `//calendar.googleapis.com/users/john smith/events/123` 来说，实际的 `RESTful` 请求路径可能是 `https://calendar.googleapis.com/v3/users/john%20smith/events/123`，和原本的资源名称相比有三点不同：
1. 指明了具体的协议，`HTTPS`
2. 指明了版本号，`v3`
3. 对资源名称进行了 `URL` 转义

`Google` 的 `API` 服务要求资源名称必须是字符串，除非有后向兼容的问题，资源名称在跨模块间传递时必须确保没有任何数据丢失。对于资源定义来说，第一个字段应该命名为 `name`，类型为字符串，用于表示资源的名称，以下是 `gRPC` 服务的定义示例：

```
// 图书馆服务
service LibraryService {
  // 获取一本书
  rpc GetBook(GetBookRequest) returns (Book) {
    option (google.api.http) = {
      get: "/v1/{name=shelves/*/books/*}"
    };
  };

  // 创建一本书
  rpc CreateBook(CreateBookRequest) returns (Book) {
    option (google.api.http) = {
      post: "/v1/{parent=shelves/*}/books"
      body: "book"
    };
  };
}

// 资源定义
message Book {
  // 资源名称，必须以 "shelves/*/books/*" 的形式。
  // 例如，"shelves/shelf1/books/book2"。
  string name = 1;

  // ... 其他属性
}

// 获取书籍请求
message GetBookRequest {
  // 资源名称，例如 "shelves/shelf1/books/book2"。
  string name = 1;
}

// 创建书籍请求
message CreateBookRequest {
  // 父资源的名称，例如 "shelves/shelf1"。
  string parent = 1;
  // 需要创建的资源实体，客户端不允许设置 `Book.name` 属性。
  Book book = 2;
}
```

> 为什么这里资源的名称字段要定义为 `name` 而不是 `id`？首先从命名上来说 `name` 本身要比 `id` 更适合作为 `名称` 一词的命名。其次，`name` 也是一个较为宽泛的词语，例如文件资源的 `name` 代表的是文件的名称还是完整的路径？通过将 `name` 作为标准字段，使得开发人员必须要选择更适合的命名，例如 `display_name`，`title` 或者 `full_name`。

为什么不直接使用资源 `ID` 来定位资源？一个系统中往往有多个资源，单纯的资源 `ID` 不具有辨识度以及缺少上下文信息。例如，如果使用数据库表的自增主键作为资源 `ID`，则无法简单的通过数字来定位资源。如果想要通过资源 `ID` 来定位资源，则势必要扩展资源 `ID` 的定义，例如使用元组来表示资源 `ID`，如 `(bucket, object)` 用于定位某个对象存储服务的对象。不过，这也带来了几个问题：
1. 对开发人员不友好，需要额外理解和记忆（例如不同资源 `ID` 的元组元素个数不同，每个元组元素代表的含义是什么）
2. 解析元组比解析字符串更为困难
3. 对基础设施组件不友好，例如日志和访问控制系统无法直接理解元组
4. 限制了 `API` 设计的灵活性，如提供可复用的 `API` 接口

## 标准方法
标准方法的作用在于为大多数的服务场景提供统一、易用的接口，超过 70% 的 `Google APIs` 都是标准方法。`Google APIs` 设计了5种标准方法：

1. `List`
2. `Get`
3. `Create`
4. `Update`
5. `Delete`

下表是标准方法和 `HTTP` 请求方法的映射：

|标准方法   |`HTTP` 请求方法映射   |`HTTP` 请求体   |`HTTP` 响应体   |
|---|---|---|---|
|List   |Get <资源集合 `URL`>   |无   |资源集合   |
|Get   |GET <资源 `URL`>   |无   |资源   |
|Create   |POST <资源集合 `URL`>   |资源   |资源   |
|Update   |PUT 或者 PATCH <资源 `URL`>   |资源   |资源   |
|Delete   |DELETE <资源 `URL`>   |无   |空   |

> `HTTP` 响应体返回的资源可能不会包含资源的全部字段，例如客户端请求时可以指定只返回需要的字段。
> 如果 `Delete` 操作不是立即删除资源，例如只是更新资源的某个字段标记或者是创建一个 [长时间运行任务](https://github.com/googleapis/googleapis/blob/master/google/longrunning/operations.proto) 来删除资源，则 `HTTP` 响应体应该包含修改后的资源或者任务信息。

### List
`List` 方法用于返回一系列同类的资源，同时该接口支持额外的参数从而允许只返回匹配的资源。它适合用于获取有限大小、未缓存的单一资源集合，对于更复杂的场景则可以考虑使用自定义方法中的 `Search` 接口。

如果想要批量获取资源，例如入参一组资源 `ID` 来返回每个资源 `ID` 所对应的资源，则应该考虑实现 `BatchGet` 的自定义方法，而不是在 `List` 方法上扩展。

`List` 方法和 `HTTP` 请求的映射关系如下：

* `List` 方法必须对应 `HTTP` 的 `GET` 请求
* `List` 方法的 `RPC` 请求参数的 `name` 字段（也就是资源集合名称）应该和 `HTTP` 的请求路径匹配，如果相匹配，则 `HTTP` 请求路径的最后一个段必须是字面量（即资源集合的 `ID`）
* `List` 方法的 `RPC` 请求参数的其他字段应该和 `HTTP` 请求路径的查询参数相匹配
* 对应的 `HTTP` 请求无请求体；`List` 的 `API` 定义中不允许声明 `body` 语句
* `HTTP` 响应体应该包含一组资源及可选的元数据信息

```
// 获取书架上的书
rpc ListBooks(ListBooksRequest) returns (ListBooksResponse) {
  // List method maps to HTTP GET.
  option (google.api.http) = {
    // The `parent` captures the parent resource name, such as "shelves/shelf1".
    get: "/v1/{parent=shelves/*}/books"
  };
}

message ListBooksRequest {
  // The parent resource name, for example, "shelves/shelf1".
  string parent = 1;

  // The maximum number of items to return.
  int32 page_size = 2;

  // The next_page_token value returned from a previous List request, if any.
  string page_token = 3;
}

message ListBooksResponse {
  // The field name should match the noun "books" in the method name.  There
  // will be a maximum number of items returned based on the page_size field
  // in the request.
  repeated Book books = 1;

  // Token to retrieve the next page of results, or empty if there are no
  // more results in the list.
  string next_page_token = 2;
}
```

TODO:
1. 资源更新，/resources/id，实体里就不需要id，见digitalocean api
2. 分页返回结果，github api返回结果没有包含分页信息，以及总数信息

## 参考
* [API design guide](https://cloud.google.com/apis/design)
* [What’s the best RESTful method to return total number of items in an object?](https://stackoverflow.com/questions/3715981/what-s-the-best-restful-method-to-return-total-number-of-items-in-an-object)