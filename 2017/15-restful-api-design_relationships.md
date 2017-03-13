# RESTful API设计系列六：关系

## 说明
- [原文链接](http://restful-api-design.readthedocs.io/en/latest/relationships.html)
- [翻译：@adolphlwq](https://github.com/adolphlwq)
- [项目地址](https://github.com/adolphlwq/translate)
- [tt](https://github.com/adolphlwq/tt)：自动生成翻译模板
- 用时:
- <a rel="license" href="http://creativecommons.org/licenses/by-nc/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://i.creativecommons.org/l/by-nc/4.0/80x15.png" /></a>

我们已经在[Resources](http://restful-api-design.readthedocs.io/en/latest/resources.html)了解到，资源是RESTful API设计中的基本单元。资源模型对象抽象自应用数据模型。

资源不是孤立存在的，而是与其它资源间存在“关系”。通常，资源间的关系与应用数据模型间关系是对应的，资源间的关系有时在RESTful中指定。

RESTful架构风格的一个原则是，这些资源间关系通过超链接来表示。

资源模型中，我们解释了任何对象的“href”属性的都可以作为超链接。href属性值是一个绝对URL，可以通过GET方法获取。可以确保在这个URL上应用GET请求没有副作用。

通常使用两种超链接：
- 使用“link”对象，它是特殊的link对象，包含了”href”和“rel”属性。”rel“属性表明关系的语义。链接对象借用[HTML](https://www.w3.org/TR/html4/struct/links.html)中的定义。
- 使用任何具有href属性的对象。由对象类型定义关系的语义。它称为“object link”，不要和上文的”link object”混在一起。

下文有一个例子，用YAML格式表示一个虚拟机，它表明了一种关系可以有两种不同的表示：
```
!vm
id: 123
href: /api/vms/123
link:
  rel: collection/nics
  href: /api/vms/123/nics
cluster:
  href: /api/clusters/456
```

link对象和`rel=”collection/nics”`一起使用，表示拥有虚拟网络接口的VM子集。集群链接通过指向包含此VM的集群表示集群对象（The cluster link instead points to the cluster containing this VM by means of a “cluster” object.）。

注意VM本身也有一个href属性。这成为“self”link，它是一种约定：资源提供这样的链接。这样，以后可以容易地重新检索或更新对象而无需保持外部状态。

没有绝对的规则要求何时使用或不使用链接对象。我们在rhevm-api项目中使用了下面的规则，并且很成功。这是一种折衷：简洁（有利于对象链接）和理解链接而不是特定资源（有利于链接对象）。
- 链接对象用来表达API中的结构化关系。例如，顶层集合，单个资源或者子集都可以通过链接对象表示。
- 对象链接用来表达应用数据模型间关系的语义。上面的例子中，vm到集群链接直接来自应用的数据模型，因此被建模成链接。

但是需要注意的是，子集可以表示语义关系或者结构化关系。参考上面示例中VM的“collection/nics”子集合。这种情况下，我们的约定是使用链接对象。

## 标准结构关系（Standard Structural Relationships）
定义一些与不同API相关的标准结构关系名称是有意义的。 这样，客户端作者知道哪些API表示什么，并且可以集成到其它客户端，适配多个API。下表包含一些这样的关系名称。

|Relationship|Semantics|
|:------:|:------:|
|collection/{name}|Link to a related collection {name}.|
|resource/{name}|Link to a related resource {name}.|
|form/{name}|Link to a related form {name}.|

这里的{name}在特定的API中被替换成相应的术语。例如，虚拟机集合可以通过rel=“collection/vms”API来链接。

## 模型语义关系（Modeling Semantic Relationships）
语义关系可以通过对象链接或者子集来表示。我相信选择正确的方式来表示子集有利于客户端获得一致的API体验。我主张使用以下规则：
1. 在1:N（一对多）关系中，目标对象存在依赖（**existentially dependent**）于原对象，我推荐使用子集。存在依赖（**existentially dependent**）的意思是原对象不存在，目标对象也不会存在。在数据库场景中，这类似于外键（FOREIGN KEY）关系。
2. 在1:N（一对多）关系中，数据有相对应的链接，我推荐使用子集。注意这里我谈论的是数据而不是原对象的一部分，也不是目标对象。子集中的资源可以保存额外的数据。这时，子资源中的数据是相关的应用数据模型中的数据和链接数据合并来的。
3. 在任何1:N（一对多）关系中，我推荐使用对象链接。
4. 多对多（N:M）关系中，我推荐使用子集合。

## 译者说
本文介绍链接对象和对象链接表示不同的关系。讲得比较抽象，应该在实践中加强理解，不要过度去理解本文中的相关介绍。
