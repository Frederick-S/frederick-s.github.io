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
* `List` 方法的 `RPC` 请求消息体的 `name` 字段（也就是资源集合名称）应该和 `HTTP` 的请求路径匹配，如果相匹配，则 `HTTP` 请求路径的最后一个段必须是字面量（即资源集合 `ID`）
* `List` 方法的 `RPC` 请求消息体的其他字段应该和 `HTTP` 请求路径的查询参数相匹配
* 对应的 `HTTP` 请求无请求体；`List` 方法的 `API` 定义中不允许声明 `body` 语句
* `HTTP` 响应体应该包含一组资源及可选的元数据信息

接口定义示例：

```
// 获取书架上的书
rpc ListBooks(ListBooksRequest) returns (ListBooksResponse) {
  // List 方法映射为 HTTP GET 请求
  option (google.api.http) = {
    // parent 对应父资源的名称，如 shelves/shelf1
    get: "/v1/{parent=shelves/*}/books"
  };
}

// 获取书籍集合请求
message ListBooksRequest {
  // 父资源的名称，如shelves/shelf1
  string parent = 1;

  // 返回资源的最大个数
  int32 page_size = 2;

  // 返回第几页的资源集合，其值为前一个 List 请求返回的 next_page_token 字段
  string page_token = 3;
}

// 获取书籍集合响应
message ListBooksResponse {
  // 返回的书籍资源集合，该字段名应该和方法名中的 Books 相匹配。其数量上限由 ListBooksRequest 中的 page_size 决定
  repeated Book books = 1;

  // 下一页资源集合的页码信息，用于获取下一页的资源集合；没有下一页时为空
  string next_page_token = 2;
}
```

### Get
`Get` 方法接收一个资源名称及其他参数来返回某个指定的资源。

`Get` 方法和 `HTTP` 请求的映射关系如下：

* `Get` 方法必须对应 `HTTP` 的 `GET` 请求
* `Get` 方法的 `RPC` 请求消息体的 `name` 字段（也就是资源名称）应该和 `HTTP` 的请求路径匹配
* `Get` 方法的 `RPC` 请求消息体的其他字段应该和 `HTTP` 请求路径的查询参数相匹配
* 对应的 `HTTP` 请求无请求体；`Get` 方法的 `API` 定义中不允许声明 `body` 语句
* `Get` 方法返回的资源实体应该和 `HTTP` 的整个响应体相匹配

接口定义示例：

```
// 获取一本书
rpc GetBook(GetBookRequest) returns (Book) {
  // Get 方法映射为 HTTP GET 请求，资源名称映射到请求路径，无请求体
  option (google.api.http) = {
    // 所请求的资源名称，如 shelves/shelf1/books/book2
    get: "/v1/{name=shelves/*/books/*}"
  };
}

// 获取单个书籍请求
message GetBookRequest {
  // 所请求的资源名称，如 shelves/shelf1/books/book2
  string name = 1;
}
```

### Create
`Create` 方法接收一个父资源名称，一个资源实体，以及其他的参数来在指定的父资源下创建一个新的资源，并返回创建的资源。

如果某个 `API` 服务支持创建资源，则应当为每一类的资源创建对应的 `Create` 方法。

`Create` 方法和 `HTTP` 请求的映射关系如下：

* `Create` 方法必须对应 `HTTP` 的 `POST` 请求
* `Create` 方法的 `RPC` 请求消息体应当包含一个 `parent` 字段用于表示所创建的资源的父资源的名称
* `Create` 方法的 `RPC` 请求消息体中表示资源实体的各字段应当和 `HTTP` 请求体中的字段相对应。如果 `Create` 方法定义中标注了 `google.api.http`，则必须声明 `body: "<resource_field>"` 语句
* `Create` 方法的 `RPC` 请求消息体可能包含一个 `<resource>_id` 字段来允许调用方指定所创建的资源的 `id`。这个字段可能会包含在资源字段实体内
* `Create` 方法的其余参数应当和 `URL` 的查询参数相匹配
* `Create` 方法返回的资源实体应该和 `HTTP` 的整个响应体相匹配

如果 `Create` 方法允许由调用方指定所创建的资源的名称，并且该资源已经存在，则该请求应当作失败处理并返回错误码 `ALREADY_EXISTS`；或者由服务端重新生成一个资源名称，并返回创建的资源，同时接口文档应当清晰的标注最终创建的资源的名称有可能和调用方传入的资源名称不同。

`Create` 方法的 `RPC` 请求消息体中必须包含资源实体，这样当资源实体的定义发生变更时，就无需同时变更请求消息体的定义。如果资源实体中的某些字段无法由客户端设置，则必须将其标注为 `Output only` 字段。

接口定义示例：

```
// 在书架上创建一本书
rpc CreateBook(CreateBookRequest) returns (Book) {
  // Create 方法对应 HTTP 的 POST 请求，URL 请求路径为资源集合名称
  // HTTP 请求体中包含需要创建的资源
  option (google.api.http) = {
    // parent 对应父资源的名称，如 shelves/1
    post: "/v1/{parent=shelves/*}/books"
    body: "book"
  };
}

// 创建书籍请求
message CreateBookRequest {
  // 父资源名称
  string parent = 1;

  // 资源 id
  string book_id = 3;

  // 需要创建的资源
  // 字段名称需要和 RPC 方法中的名词对应，即 book -> Book
  Book book = 2;
}

// 创建一个书架
rpc CreateShelf(CreateShelfRequest) returns (Shelf) {
  option (google.api.http) = {
    post: "/v1/shelves"
    body: "shelf"
  };
}

// 创建书架请求
message CreateShelfRequest {
  Shelf shelf = 1;
}
```

### Update
`Update` 方法接收一个资源实体，以及其他的参数来更新指定的资源及其属性，并返回更新后的资源。

资源的可变属性应当能够通过 `Update` 方法修改，除非该属性包含资源的名称或父资源的名称。资源重命名或者将资源移动到另一个父资源下都不允许在 `Update` 方法中实现，而应当由自定义方法来实现。

`Update` 方法和 `HTTP` 请求的映射关系如下：

* 标准的 `Update` 方法应该能够支持更新资源的部分属性，并通过 `update_mask` 字段来指明需要更新的属性，对应的 `HTTP` 请求方法为 `PATCH`。资源实体中标注为 `Output only` 的属性应该在资源更新时忽略
* 如果要求 `Update` 方法实现更为高级的局部更新语义则应当将其作为自定义方法来实现，例如追加新值到资源的某个列表类型的属性上
* 如果 `Update` 方法不支持局部属性更新，则对应的 `HTTP` 请求方法必须是 `PUT`。不过不建议 `Update` 方法仅支持全局更新，因为后续如果为资源添加新的属性则可能会有后向兼容问题
* `Update` 方法的 `RPC` 请求消息体中表示资源名称的字段值必须和 `URL` 中的请求路径相匹配。这个资源名称字段可能包含在资源实体内
* `Update` 方法的 `RPC` 请求消息体中表示资源实体的各字段必须和 `HTTP` 请求体中的字段相对应
* `Update` 方法的其余参数必须和 `URL` 的查询参数相匹配
* `Update` 方法的返回结果必须是更新后的资源实体

> 既然 URL 中已经有了资源名称，为什么请求体里面还要再传一遍资源名称？关于这一点不同的服务有不同的实现，例如 `DigitalOcean` 的更新接口就不要求请求体中再传一遍 `id`：[Update an App](https://docs.digitalocean.com/reference/api/api-reference/#operation/apps_update)。

> `Update` 方法的返回结果必须是更新后的资源实体看起来是多此一举，但是某些资源的属性必须由服务端来更新，例如资源的更新时间，或者对于 `Git` 服务来说文件更新后的版本号等等，这些属性更新后也需要返回给客户端。

如果后端服务允许客户端指定资源名称则 `Update` 方法允许客户端调用时发送一个不存在的资源名称，然后服务端会自动创建一个新的资源。否则，`Update` 方法应当作失败处理并返回 `NOT_FOUND` 的错误码（如果这是唯一的错误的话）。

即使 `Update` 方法本身支持新建资源也应当提供额外的 `Create` 方法，否则服务的使用者可能会感到迷惑。

接口定义示例：

```
// 更新一本书
rpc UpdateBook(UpdateBookRequest) returns (Book) {
  // Update 方法对应 HTTP 的 PATCH 请求，资源名称映射到请求路径
  // 资源实体包含在 HTTP 请求体中
  option (google.api.http) = {
    // 请求路径包含了需要更新的资源名称
    patch: "/v1/{book.name=shelves/*/books/*}"
    body: "book"
  };
}

// 更新书籍请求
message UpdateBookRequest {
  // 需要更新的书籍
  Book book = 1;

  // 指定需要更新的书籍属性，具体定义见 https://developers.google.com/protocol-buffers/docs/reference/google.protobuf#fieldmask
  FieldMask update_mask = 2;
}
```

### Delete
`Delete` 方法接受一个资源名称和其他参数来删除或者计划删除某个指定的资源。`Delete` 方法返回的消息体类型应当为 `google.protobuf.Empty`。

服务调用方不应该依赖 `Delete` 方法返回的任何信息，因为 `Delete` 方法被重复调用时不一定每次都返回相同的信息。

`Delete` 方法和 `HTTP` 请求的映射关系如下：

* `Delete` 方法必须对应 `HTTP` 的 `DELETE` 请求
* `Delete` 方法的 `RPC` 请求消息体中表示资源名称的字段值应当和 `URL` 中的请求路径相匹配
* `Delete` 方法的其余参数应当和 `URL` 的查询参数相匹配
* 对应的 `HTTP` 请求无请求体；`Delete` 方法的 `API` 定义中不允许声明 `body` 语句
* 如果 `Delete` 方法在实现时是立即删除资源则该方法返回的消息体为空
* 如果 `Delete` 方法在实现时是创建一个 [长时间运行任务](https://github.com/googleapis/googleapis/blob/master/google/longrunning/operations.proto) 来删除资源，则该方法返回的消息体应当为对应的任务信息
* 如果 `Delete` 方法在实现时仅将资源标记为删除而不是物理删除，则该方法应当返回更新后的资源

`Delete` 方法应当是幂等的，但并不要求每次都返回相同的信息。对同一个资源的多个 `Delete` 请求应当使得该资源（最终）被删除，不过只有第一个成功删除资源的请求应当返回相应的成功信息，其余的请求应当返回 `google.rpc.Code.NOT_FOUND` 错误码。

接口定义示例：

```
// 删除一本书
rpc DeleteBook(DeleteBookRequest) returns (google.protobuf.Empty) {
  // Delete 方法对应 HTTP 的 DELETE 请求，资源名称映射到请求路径，无 HTTP 请求体
  option (google.api.http) = {
    // 请求路径包含了需要删除的资源名称，例如 shelves/shelf1/books/book2
    delete: "/v1/{name=shelves/*/books/*}"
  };
}

// 删除书籍请求
message DeleteBookRequest {
  // 需要被删除的资源名称，如 shelves/shelf1/books/book2
  string name = 1;
}
```

## 自定义方法
标准方法提供了对资源的基础操作功能，它们的职责都较为单一，基本上对应了基础的 `CRUD` 操作。不过，并不是所有对资源的操作都能或者适合抽象为 `CRUD` 操作，这也是对于 `RESTful` 风格的服务经常争论的地方。因此，自定义方法就应运而生。

不过，对于 `API` 的设计者而言应当尽可能的首选使用标准方法，因为标准方法有着更为统一的语义，对开发者而言更为简单易懂。

自定义方法可以应用于资源，资源集合或者服务。它可能会接收任意类型的输入和返回任意类型的输出，并且支持流式的请求和响应。

### HTTP 请求映射
对于自定义方法来说，应当使用如下的 `HTTP` 请求映射：

```
https://service.name/v1/some/resource/name:customVerb
```

这里之所以选择 `:` 而不是 `/` 将 `name` 和 `customVerb` 分开是为了支持 `name` 中包含 `/` 的情况，例如将文件路径作为资源名称时则获取资源的请求可能为 `GET /files/a/long/file/name`，则撤销对该文件的删除操作所对应的自定义方法可能为 `POST /files/a/long/file/name:undelete`，如果 `undelete` 前用 `/` 分割则无法识别完整的资源名称。

自定义方法和 `HTTP` 请求的映射关系应当遵循如下规则：

* 自定义方法应当使用 `HTTP` 请求的 `POST` 方法，除非该自定义方法是作为标准方法中的 `List` 或者 `Get` 方法的扩展，此时可以使用 `HTTP` 请求的 `GET` 方法
* 自定义方法不应该使用 `HTTP` 请求的 `PATCH` 方法，但是可以使用其他 `HTTP` 请求方法
* 对于使用 `HTTP` 请求的 `GET` 方法的自定义方法来说必须保证接口的幂等性
* 自定义方法的 `RPC` 请求消息体中表示资源或者资源集合名称的字段值应当和 `URL` 中的请求路径相匹配
* `HTTP` 请求的路径必须以 `:customVerb` 的形式结尾
* 如果自定义方法对应的 `HTTP` 请求方法允许 `HTTP` 请求体（如 `POST`，`PUT`，`PATCH` 或者自定义的 `HTTP` 方法），则该自定义方法的 `HTTP` 配置中必须声明 `body: "*"` 语句，并且 `RPC` 消息体中的剩余字段应当和 `HTTP` 请求体中的字段相匹配
* 如果自定义方法对应的 `HTTP` 请求方法不接受 `HTTP` 请求体（如 `GET`，`DELETE`），则该自定义方法的 `HTTP` 配置中不允许声明 `body` 语句，并且 `RPC` 消息体中的剩余字段应当和 `URL` 的查询参数相匹配

接口定义示例：

```
// 服务级别的自定义方法
rpc Watch(WatchRequest) returns (WatchResponse) {
  // 对应 HTTP 的 POST 请求，所有请求参数都来自于 HTTP 的请求体
  option (google.api.http) = {
    post: "/v1:watch"
    body: "*"
  };
}

// 资源集合级别的自定义方法
rpc ClearEvents(ClearEventsRequest) returns (ClearEventsResponse) {
  option (google.api.http) = {
    post: "/v3/events:clear"
    body: "*"
  };
}

// 资源级别的自定义方法
rpc CancelEvent(CancelEventRequest) returns (CancelEventResponse) {
  option (google.api.http) = {
    post: "/v3/{name=events/*}:cancel"
    body: "*"
  };
}

// 一个批量获取资源的自定义方法
rpc BatchGetEvents(BatchGetEventsRequest) returns (BatchGetEventsResponse) {
  // 对应 HTTP 的 GET 请求
  option (google.api.http) = {
    get: "/v3/events:batchGet"
  };
}
```

### 使用场景
以下是一些自定义方法比标准方法更适合的场景：

* 重启一台虚拟机：其中一种反直觉的设计是在重启资源集合下创建一个重启资源，这属于生搬硬套；或者为虚拟机增加一个状态字段，重启操作就等价于资源的局部更新操作，即将虚拟机的状态由 `RUNNING` 改为 `RESTARTING`，虽然看似合理但是增加了使用人员的心智负担，例如除了这两种状态之外还有其他什么状态？另一方面也增加了接口实现的复杂度，标准方法的 `Update` 接口需要额外针对状态字段进行逻辑处理，违背了单一职责原则
* 批处理：对于性能敏感的场景而言，提供批处理的自定义方法比一系列独立的标准方法可能有着更好的性能

> 对于 `RESTful` 服务的争论中最常提到的例子就是如何使用 `RESTful` 接口表示注册/登陆？注册/登陆并不适合作为标准方法来实现，使用自定义方法会更好。一般而言，标准方法的应当实现尽量简单直白，一旦需要对标准方法扩展处理额外的逻辑，就需要考虑是否使用自定义方法更合适。

## 异常处理
异常处理是 `RESTful` 又一个争论的点，即业务异常可能有很多，`HTTP` 的状态码根本不够，以及业务的状态码不应该和协议层的状态码相混淆。

### 异常模型
`Google API` 将异常统一封装为 [google.rpc.Status](https://github.com/googleapis/googleapis/blob/master/google/rpc/status.proto)：

```
package google.rpc;

// 适用于不同编程环境的统一异常模型，包括 REST 接口和 RPC 接口
message Status {
  // 错误码，具体错误码的定义见 google.rpc.Code
  int32 code = 1;

  // 错误信息，对错误原因的描述以及可能的修复方式
  string message = 2;

  // 错误的详细信息，开发人员可以通过详细信息找到一些有用的信息
  repeated google.protobuf.Any details = 3;
}
```

由于大部分的 `Google APIs` 都是面向资源的设计，异常处理同样遵循了这样的设计，即使用一系列的标准异常来应对大多数的资源异常场景。例如使用标准的 `google.rpc.Code.NOT_FOUND` 异常来统一表示某个资源不存在，而不是为每一个资源定义一个 `SOME_RESOURCE_NOT_FOUND` 异常。

#### 错误码
`Google APIs` 必须使用 [google.rpc.Code](https://github.com/googleapis/googleapis/blob/master/google/rpc/code.proto) 中定义的错误码，不允许独自额外定义错误码。

#### 错误信息
错误信息应当能够帮助用户简单快速的理解和解决 `API` 错误。一般而言，描述错误信息可以遵循如下的规则：

* 不要假设用户是你所开发的服务的专家，他们可能是客户端开发者，运维人员，`IT` 人员以及应用的终端用户
* 不要假设用户知晓你所开发的服务任何的实现细节，或者熟悉错误的上下文
* 尽可能的使得错误信息有助于技术用户（不一定是服务的开发人员）响应错误并修正
* 保持错误信息简洁。如果可能的话在错误信息中提供一个帮助链接，以便于用户能够提问，反馈或者查找一些有用的信息

> 错误信息可能会随时变动，应用开发人员不应该强依赖错误信息。

#### 错误详情

TODO:
1. 分页返回结果，github api返回结果没有包含分页信息，以及总数信息

## 参考
* [API design guide](https://cloud.google.com/apis/design)
* [What’s the best RESTful method to return total number of items in an object?](https://stackoverflow.com/questions/3715981/what-s-the-best-restful-method-to-return-total-number-of-items-in-an-object)
* [DigitalOcean API (2.0)](https://docs.digitalocean.com/reference/api/api-reference/)