# RESTful API设计系列七：表单

<!-- TOC -->

- [RESTful API设计系列七：表单](#restful-api设计系列七表单)
    - [说明](#说明)
    - [类型定义](#类型定义)
    - [服务定义](#服务定义)
    - [使用表单引导输入](#使用表单引导输入)
    - [表单描述语言](#表单描述语言)
        - [Form Metadata](#form-metadata)
        - [Fields](#fields)
        - [Constraints](#constraints)
        - [Checking Constraints](#checking-constraints)
        - [Building the Request Entity](#building-the-request-entity)
    - [链接到表单](#链接到表单)
    - [译者说](#译者说)

<!-- /TOC -->

## 说明
- [原文链接](http://restful-api-design.readthedocs.io/en/latest/forms.html)
- [翻译：@adolphlwq](https://github.com/adolphlwq)
- [项目地址](https://github.com/adolphlwq/translate)
- [tt](https://github.com/adolphlwq/tt)：自动生成翻译模板
- 用时: 3h
- <a rel="license" href="http://creativecommons.org/licenses/by-nc/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://i.creativecommons.org/l/by-nc/4.0/80x15.png" /></a>

本节中，我们提出一个解决方案，用于解决现在许多RESTful API中存在的可用性问题。这个问题是，如果没有引用外部文档，API用户不知道要提供什么数据给接受输入的操作。例如，POST调用用来在集合中创建新资源，API用户可以通过OPTIONS调用确定哪些集合支持POST。但是，他却不知道创建新资源需要的数据有哪些。到目前为止，大多数RESTful API都要求用户参考API文档来获取需要的数据信息。这违反了一个重要的RESTful原则：API应该是自我描述的（self-descriptive）。

就我所见，可用性问题影响两个重要的用例：
1. 一位用户正在浏览API，尝试使用和学习它。该用户可以使用Web浏览器，或者像“curl”这样的命令行工具。
2. 开发RESTful API的命令行或图形界面。没有对输入类型的确切描述，这些信息应该在客户端显示编码。（没有确切描述）的缺点是新特征不会被自动暴露，并且存在严格要求：不能开发破坏向后兼容的改变。

当我们通过web访问时，这个问题不复存在。任何时刻都有百万用户通过web和所有类型IT系统交互，他们不需要通过API文档来了解如何交互。和web上的IT系统交互都是自解释的（self-explanatory），web使用超文本，输入通过`表单`引导。

事实上，表单就是我们看到的解决方案。但是在介绍表单之前，让我们看看两类方案，它们都被提出解决这个问题，以及为什么我认为实际上它们无法解决这个问题。一是使用类型定义语言比如XMLSchema来描述输入类型，而是使用服务描述语言比如WADL。

## 类型定义
正如在[Resources](http://restful-api-design.readthedocs.io/en/latest/resources.html)描述的，一些RESTful API使用类型定义语言提供构造请求实体的信息。我认为这是糟糕的方法，并且存在以下问题：
1. 它增加了客户端和服务端间的耦合度。
2. 它依然不允许普通的web浏览器作为HTTP客户端。
3. 它对自动构建CLIs一无是处。

经常，使用XMLSchema作为类型定义语言。它带来了下面一些问题：
1. 一方面XMLSchema是非常复杂可怕的标准，另一方面，它不能很好地表示无序的键/值对这种常见需求（注意<xs：all>不能解决这个问题，它的限制使得它几乎无用）。
2. XMLSchema中的类型定义是上下文自由的（context-free）。这意味着你不能只定义一个类型，你需要为PUT定义一个、POST定义一个等。并且由于一个XML元素只能有一个类型，这意味着你PUT或POST或GET时，使用的是不同的元素。这打破了REST的“homogeneous collection”（均匀收集）原则。

## 服务定义
[WADL](http://www.w3.org/Submission/wadl/)是一种为RESTful API定义服务描述语言的方法。它试图通过精心定义入口点和参数来解决这个问题。我使用WADL的问题是，感觉它非常`非RESTful`。我看不到WADL资源上的方法和RPC入口点之间有很大的区别（WADL风格上偏RPC而非RESTful）。

可以构造更加RESTful的服务描述语言。它将专注于映射资源、集合、关系和链接等RESTful中的概念。它可能需要一个类型定义语言来定义每个方法的类型约束（PUT可能有不同于POST的约束）。这在RHEV-M项目上已经进行了讨论。

最后，这种方法感觉也不对。我认为需要的是一些工作类似于web的方法，其中一切都是自我描述性的，并且仅由实际的URL流引导用户，而不是通过引用一些外部描述。

## 使用表单引导输入
在我看来，指导客户端输入的正确解决方案是使用表单。注意，单词“forms”更一般意义，它作为指导用户输入的实体，而不一定等于HTML中的表单。

在我看来，表单需要指定三个信息：
1. 如何联系目标和格式化输入。
2. 所有可用输入字段的列表。
3. 输入字段必须遵守的约束列表。

HTML表单提供1和2，但不适用于上面的3。3通常使用客户端Javascript实现。因为JavaScript通常不适用于使用API的所有客户端，所以我们需要定义另一种方法来进行验证。我建议的方法是定义一个表单描述语言，捕获1到3的所有信息。通过这种语言描述的表单可以在客户理解表单语言的情况下原样发送到客户端。如果客户端是Web浏览器，表单也可以转换为带有Javascript的HTML表单。


## 表单描述语言
我们的表单被定义为一个特殊的RESTful资源，类型为“form”。这意味着它们使用JSON数据模型，可以根据[resources](http://restful-api-design.readthedocs.io/en/latest/resources.html)中的规则以JSON，YAML和XML表示。

下表列出表单自己的内容类型：

|Type|Content-Type|
|:------:|:------:|
|Form|application/x-form+json
application/x-form+yaml
application/x-form+xml|

我不是要提供一个正式的定义，而是通过使用一个例子来介绍表单描述语言。下面是一个指导虚拟机创建的表单示例。YAML格式：
```
!form
method: POST
action: {url}
type: vm
fields:
- name: name
  type: string
  regex: [a-zA-Z0-9]{5,32}
- name: description
  type: string
  maxlen: 128
- name: memory
  type: number
  min: 512
  max: 8192
- name: restart
  type: boolean
- name: priority
  type: number
  min: 0
  max: 100
constraints:
- sense: mandatory
  field: name
- sense: optional
  field: description
- sense: optional
  field: cpu.cores
- sense: optional
  field: cpu.sockets
- sense: optional
  exclusive: true
  constraints:
  - sense: mandatory
    field: highlyavailable
  - sense: optional
    field: priority
```

可以看到表单由3部分组成：form metadata、field definitions和constraints。

### Form Metadata
Form Metadata非常简单，定义了下面这些属性：
|Attribute|Description|
|:------:|:------:|
|method|The HTTP method to use. Can be GET, POST, PUT or DELETE|
|url|The URL to submit this form to|
|type|The type of the resource to submit|

### Fields
可用字段的列表使用“fields”属性指定。每个字段定义具有以下属性：

|Attribute|Description|
|:------:|:------:|
|name|The field name. Should be in dotted-name notation|
|type|One of “string”, “number” or “boolean”|
|min|Field value must be greater than or equal to this (numbers)|
|max|Field value must be less than or equal to this (numbers)|
|minlen|Minimum field length (strings)|
|maxlen|Maximum field length (strings)|
|regex|Field value needs to match this regular expression (strings)|
|multiple|Boolean that indicates if multiple values are accepted (array)|

### Constraints
首先，我们需要回答在表单描述语言中要表达什么样的约束。我将根据个人观点逐个提到，我认为不可能在客户端表达每一个约束。一些约束例如需要访问其它数据（当创建关系时）是计算密集型的约束，甚至这些约束对API设计者未知，因为它们对于API服务的应用没有文档说明。所以在我看来，我们需要找到一个有用的约束子集，它不太复杂，不用担心一些约束可能表达不到。

于是我定义以下两种约束，这两种约束都是有用的，并且足够我们的使用：
1. 单个数据值的约束。
2. 对字段的存在性约束，即是否允许字段，如果允许，是否是强制允许。

对数据值的约束很有用，因为CLI和GUI可以使用此信息来帮助用户输入。比如根据类型约束，GUI可以给某个字段呈现复选框，文本框或下拉列表。为了简洁起见，对单个数据值的约束被描述为字段定义的一部分，并且在上一节中讨论过。

存在约束也是有用的，它们允许API用户生成一些提示信息。每个存在约束具有以下属性：

|Attribute|Description|
|:------:|:------:|
|sense|One of “mandatory” or “optional”|
|field|This constraint refers to a field|
|constraints|This constraint is a group with nested constraints|
|exclusive|This is an exclusive group (groups only)|

必须指定“字段”或“约束”，但不能同时指定两者。设置了字段属性的约束称为简单约束。具有约束属性集的约束称为组约束。

### Checking Constraints
应当首先检查值约束，并且仅检查非空值。

然后检查存在约束。这有点复杂，因为约束不仅用于确保所有必填字段存在，而且不会出现非可选字段。可以使用以下算法：
1. Start with an empty list called “referenced” that will collect all referenced fields.
2. Walk over all constraints in order. If a constraint is a group, you need to recurse into it, depth first, passing it the “referenced” list.
3. For every simple constraint that validates, add the field name to “referenced”.
4. In exclusive groups, matching stops at the first matching sub-constraint, in which case the group matches. In non-exclusive groups, matching stops at the first non-matching sub-constraint, in which case the group does not match.
5. When matching a group, you need to backtrack to the previous value of “referenced” in case the group does not match.
6. A constraint only fails if it is mandatory and it is a top-level constraint. If a constraint fails, processing may stop.
7. When you’ve walked through all constraints, it is an error if there are fields that have a non-null value but are not in the referenced list.

### Building the Request Entity
满足所有约束后，客户端应该构建一个请求实体，表单数据将在POST，PUT或DELETE方法的主体中传递。如果以JSON，YAML或XML格式请求表单，则假定客户端不是Web浏览器，并且以下内容适用：
首先，表单元数据中某个类型的新资源被创建，`.`表示父子关系。父属性和子属性都是对象。

如果客户端请求表单内容类型为“text/html”，则假定客户端是Web浏览器，并且我们假设表单将作为常规HTML表单处理。在这种情况下：
1. An HTML <form> should be generated, with an appropriate `input` element for each field.
2. The HTML form’s “method” attribute should be set to POST, unconditionally. In case the RESTful form’s method is not POST, the server should include a hidden input element with the name “method” to indicate to the server the original method. (HTML does not support PUT or DELETE in a form).
3. The form’s “enctype” should be set to “multipart/form-data” or “application/x-www-form-urlencoded”, as appropriate for the input elements.
4. A hidden field called `_type` is generated that contains the value of the “type” attribute in the form metadata.
5. The server may generate Javascript and include that in the HTML to check the value and presence constraints.

## 链接到表单
表单通过链接对象与资源和集合关联。链接对象的名称表示表单的含义。以下标准表单名称：

|Name|Scope|Description|
|:------:|:------:|:------:|
|form/search|collection|Form to search for resources|
|form/create|collection|Form to create a new resource|
|form/update|resource|Form to update a resource|
|form/delete|resource|Form to delete a resource|

## 译者说
本节介绍了客户端/GUI输入信息的方案。先介绍了类型定义个服务定义这两种解决方案的缺点。然后详细介绍表单方案。表单方案中涉及表单三大组建：元数据、字段和约束。理解表单三个组建非常重要。
