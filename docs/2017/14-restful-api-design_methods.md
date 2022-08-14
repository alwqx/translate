# RESTful API设计系列五：方法

## 说明
- [原文链接](http://restful-api-design.readthedocs.io/en/latest/methods.html)
- [翻译：@alwqx](https://github.com/alwqx)
- [项目地址](https://github.com/alwqx/translate)
- [tt](https://github.com/alwqx/tt)：自动生成翻译模板
- 用时:
- <a rel="license" href="http://creativecommons.org/licenses/by-nc/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://i.creativecommons.org/l/by-nc/4.0/80x15.png" /></a>

## 标准方法
我已经阐明资源是RESTful API中的基本概念。每个资源都有单独对应的URL。方法可以通过API作用到资源上。

下表列出了标准方法，以及对资源或集合操作的明确解释：

| Method | Scope | Semantics |
|:------:|:------:|:------:|
|GET|collection| Retrieve all resources in a collection |
|GET|resource|Retrieve a single resource|
|HEAD|collection|Retrieve all resources in a collection (header only)|
|HEAD|resource|	Retrieve a single resource (header only)|
|POST|collection|Create a new resource in a collection|
|PUT|resource|Update a resource|
|PATCH|resource|Update a resource|
|DELETE|resource|Delete a resource|
|OPTIONS|any|Return available HTTP methods and other options|

通常，并非所有资源和集合都实现所有方法。有两种方法可以判断资源和集合能接受的方法：
- URL中使用`OPTIONS`方法，查看返回结果中"Allow"头部信息。Allow头部包含一个逗号分隔的方法列表，表明资源或者集合支持的方法。
- 直接使用你想用的方法，可能返回“405 Method Not Allowed”响应，表明不支持这个方法。

## 动作
Sometimes, it is required to expose an operation in the API that inherently is non RESTful。有时需要在API上暴露本质上不是RESTful的操作。一个例子是资源状态变化的操作，多个不同的方法可以达到相同的最终状态，并且这些方法看起来一样但内部原理是不同的。有人认为这种设计是糟糕的，（我认为）不模拟所有的状态变化可以大大简化API。很贴切的例子是虚拟机的“shutdown”和“power off”操作。两者都会导致虚拟机处于“DOWN”状态。但是它们是不同的。

作为这些非RESTful操作的解决方案，可以在资源子集合上使用“actions”。动作是类RPC的消息，告诉资源执行某个操作。动作子集合可以视为命令队列，可以添加新的动作，稍后会被API执行。每个资源接受的动作都是“POST”，动作附带“type”，表明是哪种类型，也可以添加其它类型给操作添加参数。

应当注意，当动作不能映射到一个标准的RESTful时，动作要抛出异常。如果一个API有太多的动作，这表明要么它设计有一个RPC viewpoint，而不是使用RESTful设计原则，要么这个API自然更适合RPC类型模型。

## PATCH vs PUT
HTTP RFC指定PUT方法必须采用一个完整的新资源作为请求实体。This means that if for example only certain attributes are provided, those should be removed (i.e. set to null)。

另一个方法PATCH草案在[这里](http://tools.ietf.org/html/rfc5789)。PATCH在语义上类似PUT，它更新资源，但是又有不同，PATCH应用增量而不是更新整个资源。PATCH还在投票阶段。

对于简单的资源表示，它们间的差异通常不重要，许多API将PUT和PATCH视为同义词。这通常不会出现问题，因为设置属性为null不是很常见。如果你需要，你可以总是显式声明出来。

但是，对于更复杂的表示，尤其是包含列表的资源，准确表达要改变什么非常重要。因此，现在我的建议是既提供PATCH和PUT，并用PATCH做相对更新，用PUT替换整个资源。

意识到PATCH方法请求实体与被修改的实体内容类型不同非常重要。PATCH描述的是对资源进行的修改。对JSON格式，我认为有两种明智的方式来定义PATCH的格式：
1. 非正式，你接受一个dict，仅仅表示对象的一部分，只有dict中出现的属性才会更新。没出现的不变。这种方法简单但是有一个缺点：资源内部结构可能非常复杂。比如PATCH实体包含一个很大的列表，列表存储dict。这是PARCH类似PUT。
2. 更正式。将改变放到PATCH实体中，每个修改可以是dict，指定要修改（add， remove，change）节点的JSON路径和新值。

## Link Headers
[这份互联网草案](https://tools.ietf.org/html/draft-nottingham-http-link-header-10)描述了“Link：”头部类型。头部把link属性当作响应实体，并把它格式化为HTTP头部。

有人认为Link头很有用，因为它允许客户端快速从响应中获取链接，而不必解析响应实体，甚至直接使用HEAD方法直接不需要响应试题。

在我看来，它的可用性是可疑的。首先，它显著增加响应的大小。第二，它只能在资源被返回时使用，它与集合一起使用没有意义。因为我还没有看到任何在RESTful API使用这个头的好的案例，所以我建议不要使用链接头。

## Asynchronous Requests(异步请求)
有时动作在单个HTTP请求上下文中耗时很长，客户端可能收到“202 Accepted”状态码。这样的响应只能返回给POST, PUT, PATCH或者DELETE的方法。

202状态码对应的实体是常规资源，只有请求的信息是可获取的时才会接受请求。资源应包含“链接”属性，指向状态监视器，可以轮询该状态监视器获取更新的状态信息。

当轮询状态监视器时，它应当返回一个“响应”对象，其中包含有关异步请求的当前状态的信息（When polling the status monitor, it should return a “response” object with information on the current status of the asynchronous request）。 如果请求仍在进行中，则此类响应可能如下所示（YAML格式）：
```
!response
status: 202 Accepted
progress: 50%
```

如果调用完成，响应应当包含相同的头部信息，相应主体也同步完成满足请求：
```
!response
status: 201 Created
headers:
 - name: content-type
   value: applicaton/x-resource+yaml
response: !!str
  Response goes here
```

响应被获取一次，并且状态码不等于“202 Accepted”，API代码会对它进行垃圾回收，因此客户端不应该假设它继续可用。

客户端可能会通过下面的“Except”头部请求告诉服务端修改异步请求：
- “Expect: 200-ok/201-created/204-no-content” ，关闭所有异步功能。如果服务器不愿意等待操作完成，可能返回“417 Expectation Failed”。
- “Expect: 202-accepted”，请求明确表明需要异步响应。如果服务器不愿意响应异步请求，可能返回“417 Expectation Failed”。

如果没有异常，客户端必须准备接受除GET方法请求外的202 Accepted status。

## 范围/页码
当集合包含很多资源时，客户端只获取部分资源是很常见的需求。可以通过在请求头中用“Range”指定资源范围实现：
```
GET /api/collection
Range: resources=100-199
```

上面的请求会返回资源中的100-199（包括）项。注意，实现API的开发人员要确保请求合适的资源且保证资源有意义。

服务端支持范围查询时，要在返回的信息中提供“Accept-Ranges: resource”头部信息。这个头部信息在OPTIONS响应中提供：
```
OPTIONS /api/collection HTTP/1.1

HTTP/1.1 200 OK
Accept-Ranges: resources
```

## 通知
另一个常用的需求是当某种类型的事件发生时立刻通知客户端。

理想情况下，通过服务端想客户断call-out实现通知。但是，通过HTTP这样做还没有很好的、可移植的标准。（HTTP通知客户端）也打破了网络地址转换和HTTP代理的现有标准。第二种方法是循环轮询，它很低效。

我认为最佳途径是“long polling”。长轮询中，客户端会获取一个URL但是服务端不会生成响应。客户端会等待一会（时间长短需要配置），直到时间到期关闭链接重新建立链接。如果服务端意识到有个事件需要通知客户端，它可以立即向当前等待的客户端通知该事件。

长轮询默认情况下是被禁止的，客户端可以通过头部信息指定。例如，客户端可以组合长轮询和基于资源的范围查询请求集合中的新资源：
```
GET /api/collection
Range: 100-
Expect: nonempty-response
```
这个例子中，集合中的资源“100”是上次请求中的最后一个资源，这次请求调用API返回的资源ID至少大于100。

**服务端的编程人员要决定对每个长轮询的等待客户端都开启一个线程，或者是一个线程IO多路复用等待所有的客户端。**这个决定是易于实现和可扩展性间的权衡（现代操作系统中线程相对廉价）。

## 译者说
通知和异步请求个人还要多学习。文中的实现方案值得借鉴。
