# RESTful API模式系列三：资源

<!-- TOC -->

- [RESTful API模式系列三：资源](#restful-api模式系列三资源)
    - [说明](#说明)
    - [资源数据](#资源数据)
    - [应用数据](#应用数据)
    - [REST元数据](#rest元数据)
    - [其它数据](#其它数据)
    - [表示](#表示)
        - [JSON格式](#json格式)
        - [YAML格式](#yaml格式)
        - [XML格式](#xml格式)
        - [HTML格式](#html格式)
    - [Content-Types](#content-types)
    - [选择表式格式](#选择表式格式)
    - [译者说](#译者说)

<!-- /TOC -->

## 说明
- [原文链接](http://restful-api-design.readthedocs.io/en/latest/resources.html)
- [翻译：@adolphlwq](https://github.com/adolphlwq)
- [项目地址](https://github.com/adolphlwq/translate)
- 翻译/校对：3.5h
- <a rel="license" href="http://creativecommons.org/licenses/by-nc/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://i.creativecommons.org/l/by-nc/4.0/80x15.png" /></a>

资源是任何RESTful API中的基本概念。资源是对象，包括类型、关联的数据、资源间的关系以及资源上的操作集合。它和面向对象编程语言中的对象类似，不同点在于资源
只定义了有限的标准方法（对应HTTP协议中标准的GET，POST，PUT，DELETE方法），而对象实例可以有很多方法。

资源可以被分类到不同的集合中。每个集合都包含一种类型的资源，因此集合都是均匀（`homogeneous`）的。资源也可以不分到集合里，这些资源我们称为`singleton resources`。

集合可以全局存在，即API顶层（译者注：原文at the top level of an API），也可以包含在single resource中。后文中，我们把这种集合称为*sub-collections*，子集通常用来表达“包含于”的关系。我会在[Relationships](http://restful-api-design.readthedocs.io/en/latest/relationships.html)详细介绍。

下图描述了RESTful API的关键概念：

![](http://restful-api-design.readthedocs.io/en/latest/_images/concepts.png)

我们把描述资源的类型、行为和关系的信息称为*API的资源模型*。RESTful中的资源模型可以视为到应用数据模型的映射。

## 资源数据
资源关联数据。API的资源模型还包括关联数据的丰富性。比如，它定义了哪些可用的数据类型和行为。

就我个人经验，我坚信JSON这种数据模型完美满足API的丰富性要求，它是RESTful资源的理想数据模型。我会推荐给每个人。

JSON中已经存在三种类型数据：

- scalar（标量：number, string, boolean, nul）
- array
- object

标量类型只有一个值。数组包含任意类型值的有序列表。对象是无序的`key/value(键/值对)`集合（亦称为属性，但是不要和XML中的属性概念搞混），key是字符串，value可以是任意类型。更多JSON细节请参考[JSON web site](http://www.json.org/)。

为什么如此偏爱JSON？以我个人观点，JSON在表达性和广泛应用上拥有良好的平衡。scalars、arrays和objects这三种类型足够强大，能够以自然的方式描述资源中暴露的数据，同时JSON足够小巧，几乎任何现代语言都可以内置支持JSON。

XML也是以为有力的竞争者（contender）。实际上，RHEV-M（译者注：红帽的一款产品）最终API中就使用XMLSchema来描述资源。事后来看（With hindsight），RESTful API使用XML模型是个糟糕的选择。一方面，它过于丰富；另一方面，它又缺少一些特性。XML作为标准通用标记语言的一个分支（SGML off-shoot），我认为它在表示`结构化文档`是伟大的，但是不适合表示`结构化数据`。

XML一些过于丰富的特性有：

- Attributes vs elements（属性与元素）。XML可以既有属性，也包含子元素。包含数据项的资源可以被编码成任意一种。这导致客户端或者服务端事先不清楚该使用哪一个。
- Relevance of order（顺序相关性）。子元素间的顺序也会关联到XML中，我认为对象间的属性就不是**自然的有序了**。

XML数据模型的缺点有：

- 没有类型。XML文档中的元素没有类型，为了使用类型需要引入XMLSchema，不幸的是XMLSchema规范非常复杂。
- 没有列表。XML不能原生表达列表。这可能导致问题：不清楚某个元素是列表还是对象，或者两者都是。

## 应用数据
我们使用以下规则定义可以与JSON数据模型映射的资源数据：

1. 资源被建模为JSON对象。资源的类型存储在特殊的键值对`_type`中。
2. 资源中的数据表示为JSON对象中的键值对。为了避免和JSON对象内部键值对冲突，**键不能以“_”开头**。
3. 键值对中的值可以是JSON中任意原生类型: string、number、boolean、null或者arrays。值还可以是对象，这种情况下值表示嵌套的资源。
4. 集合表示成对象数组。

我们也会把键值对认为JSON对象中的属性，这里不详细描述区别，都使用统一的术语。这样JSON中的属性就不会和XML中的属性冲突了。

## REST元数据
除了暴露应用数据，资源中还有RESTful API相关特殊的数据。这些信息包括URLs和relationships。

下表列出了所有资源中定义的，通用、有特殊含义的属性：

| Attribute      |    Type | Meaning  |
| :-------- | --------:| :--: |
| id  | String |  Identifies the unique ID of a resource   |
| href     |   String |  	Identifies the URL of the current resource  |
| link      |   object | 	Identifies a relationship for a resource. This attribute is itself an object and has “rel” “href” attributes  |

## 其它数据
通常，除了应用数据、REST元数据外，我们还需要一些数据。这通常是“类RPC”数据，其中需要设置操作，但是设置最终不会作为资源本身一部分。

这里我能列举的例子是，创建新资源过程中需要引用另一个资源，但是被引用的资源最终不会成为创建资源的一部分。

将应用数据、REST元数据和其它数据合并到资源中是API代码的职责，有可能要解决可能出现的名称冲突的问题。

## 表示
我们已经定义了资源，同时也介绍了资源数据和JSON数据模型间的映射关系。但是，这些资源仍然是抽象的实体。在它们通过HTTP链接和客户端通信前，它们需要被**序列化**成文本表现形示。然后这种文本表示就可以作为实体包含在HTTP消息体中。

以下表示类型是资源常用的，该表还可使用的内容类型：

| Type      |    Content-Type|
| :-------- | --------:|
| JSON  | 	application/x-resource+json application/x-collection+json |
| YAML     |  application/x-resource+yaml application/x-collection+yaml |
| XML      |   	application/x-resource+xml application/x-collection+xml |
| HTML      |    text/html  |

注意：所有使用`x-`前缀的内容类型都是实验阶段，[RFC2046](http://www.ietf.org/rfc/rfc2046.txt)也是认可的。

### JSON格式
将资源序列化为JSON格式很简单，因为资源的数据模型是根据JSON模型定义的。下面我们给出一个虚拟机JSON序列化的例子：
```
{
  "_type": "vm",
  "name": "A virtual machine",
  "memory": 1024,
  "cpu": {
    "cores": 4,
    "speed": 3600
  },
  "boot": {
    "devices": ["cdrom", "harddisk"]
  }
}
```

### YAML格式
YAML格式和JSON稍微不同，JSON中键值对里的“_type”在YAML中替换为“!type”注解。上面的虚拟机实例的YAML格式为：
```
!vm
name: A virtual machine
memory: 1024
cpu:
  cores: 4
  speed: 3600
boot:
  devices:
  - cdrom
  - harddisk
```

### XML格式
由于XML的复杂性和限制，XML表示法是最复杂的。我推荐下面的规则：

- 资源映射到XML元素，加上标签名表示资源类型。
- 资源属性映射到XML子元素，标签名表示属性名。
- 标量表示成文本节点。标量元素中关键字“type”表示标量类型，这种映射要遵守[XML Schema Part 2](https://www.w3.org/TR/xmlschema-2/)。
- 列表要存储为单个的容器元素，其中每个列表项都有子元素。容器元素的标签应当是属性名称英文复数，item标签应该是属性名称的英文单数。列表应该具有“xd：list”类型注释。

相同的虚拟机资源的XML表示格式为：
```
<vm xmlns:xs="http://www.w3.org/2001/XMLSchema">
  <name type="xs:string">My VM</name>
  <memory type="xs:int">1024</memory>
  <cpu>
    <cores type="xs:int">4</cores>
    <speed type="xs:int">3600</speed>
  </cpu>
  <boot>
    <devices type="xs:list">
      <device type="xs:string">cdrom</device>
      <device type="xs:string">harddisk</device>
    </devices>
  </boot>
</vm>
```

### HTML格式
HTML响应的精确格式应该是API相关的。HTML是为人类使用设计的，因此唯一要求是易于理解。一个简单实现可以是下面的表示法：

- 对于集合，使用`<table>`标签表示，每一列表示一个属性，每一行表示一个对象。
- 对于资源，使用`<table>`标签和两列表示，一列表示所有的属性名，一列表示属性对应的值。

## Content-Types
根据上文内容，我主张使用的通用内容类型是“application/x-resource+format”和“application/x-collection+format”。在我看来，它们代表了RESTful API中常见的两个极端情形的中间情形：

一类RESTful API只使用“空的”（译者注：bare）XML、JSON或者YAML内容类型。这种情况下，内容类型只表示实体的类型是XML、JSON或者YAML。在我看来，这依然不够。因为资源和集合会有一些特定的语义，例如“href”属性，“link”属性和type。Therefore, these are a specialization of XML, JSON and YAML and should be defined as such（译者注：这里不翻译是因为没看懂）。

另一类RESTful API会为资源模型中的每个资源类型都定义内容类型。一个例子是[vSphere Director API](http://www.vmware.com/pdf/vcd_10_api_guide.pdf)。在我看来这也不妥。指定详细的内容类型会导致API方和客户端方认为这些类型有特定的接口。我认为所有的资源应该共享那些相同的、基本的接口，这些基本接口是符合RESTful设计原则，内容类型表示为“application/x-resource”。

一个原因是，通过 有利于内容类型 细节定义 的方法，内容类型可以和某些`类型定义语言`（如XMLSchema）中的属性相关连。据推测，这有利于客户端自动发现，因为客户端知道某种类型的可用属性。我在[Forms](http://restful-api-design.readthedocs.io/en/latest/forms.html)讨论（go into）了很多这个主题的细节，但总结下来我并不统一这个论点。

## 选择表式格式
客户端可以通过HTTP“Accept”头表示客户端使用哪种合适。HTTP RFC声明了[详尽的规则](http://www.w3.org/Protocols/rfc2616/rfc2616-sec14.html#sec14.1)，规则中可以请求多种格式，没中格式都有自己的优先级。下面的例子中客户端告诉API它只接受YAML格式：
```
GET /api/collection
Accept: application/x-collection+yaml
```

## 译者说
本文在HTTP协议的背景下，介绍了RESTful中的资源包含那些类型的数据；资源与JSON、XML、YAML等格式间的映射规则。作者支持将资源映射称JSON格式。
阅读本文还需要了解HTTP协议，否则很多属于很难理解。