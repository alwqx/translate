# RESTful API设计系列三：URLs
	
## 说明
- [原文链接](http://restful-api-design.readthedocs.io/en/latest/urls.html)
- [翻译：@adolphlwq](https://github.com/adolphlwq)
- [项目地址](https://github.com/adolphlwq/translate)
- [tt](https://github.com/adolphlwq/tt)：自动生成翻译模板
- 用时: 2h
- <a rel="license" href="http://creativecommons.org/licenses/by-nc/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://i.creativecommons.org/l/by-nc/4.0/80x15.png" /></a>

## Entry Point
RESTful API有且只有一个入口点（entry point）。入口点的URL要告知API客户端，以便它们可以找到。

技术上讲，入口点可以被看作任何集合外的单个资源。通常入口点包含下列部分或全部信息：

- API版本信息，支持的特性等。
- 顶层集合列表。
- 单个资源列表。
- API设计者认为有用的信息，比如：操作状态的简短描述、统计信息等。

## URL结构
API中的每个集合和资源都有自己的URL。URLs不能通过客户端来构造。客户端只能使用API生成的链接。

推荐的URL规范是在API入口点后添加可用的集合或者资源的路径。这最好通过例子来描述。下图表格来自Rails中的“路由”实现，使用“:name”URL变量风格。

| URL | Description |
|:-----:|:-----:|
| /api | The API entry point |
| /api/:coll | A top-level collection named “coll” |
| /api/:coll/:id | 	The resource “id” inside collection “coll” |
| /api/:coll/:id/:subcoll | Sub-collection “subcoll” under resource “id” |
| /api/:coll/:id/:subcoll/:subid | The resource “subid” inside “subcoll” |

尽管子集合可能有任意层嵌套，以我个人经验，如果可以的话最好把嵌套深度限制在2以内。URLs越长，越难使用简单的命令行工具比如[curl](http://curl.haxx.se/)。

## Relative vs Absolute
强烈建议API生成绝对地址的URLs。

理由主要是方便客户端，这样客户端就不要去匹配相对URL对应资源的绝对URL了。毕竟[URL RPF](http://tools.ietf.org/html/rfc3986#section-5.1)中指定的检测基本URL的算法就已经非常复杂了。查找基本URL的方法之一是解析请求资源的URL。由于一个资源可能出现在多个URLs中（比如，资源作为集合的一部分出现在URL，或者单个资源），这样客户端记住每个URL是很大的开销。通过使用绝对URL就避免了这个问题。

## URL模板
已经有关于URL模板的[草案](http://tools.ietf.org/html/draft-gregorio-uritemplate-05)了。当目标URL中存在查询参数时，URL模板会很有帮助。即便如此我还是推荐保守（`conservative `）使用模板。目前为止URL模板唯一的使用案例是在集合中搜索。搜索条件可以作为GET风格的查询参数附加到集合URL后面。

我建议使用URL模板规范的”字面扩展“（literal expansion）部分，因为我认为规范的”表达式扩展“部分非常复杂，几乎没有优势。

## 变量
有时你的工作需要处理资源中的变量部分。以我们的RHEV-M API为例，当虚拟机运行时需要更新虚拟机里面的一些属性。这相当于资源的热插拔（`This amounts to a hot plug/unplug of the resource`），这与改变已经保存的表示是完全不同的操作。好的实现方法是在URL中使用";variant"，比如：

| URL | Description |
|:-----:|:-----:|
| /api/:coll/:id;saved | Identifies the saved variant of a resource. |
| /api/:coll/:id;current | Identifies the current variant of a resource. |

[RFC3986](http://tools.ietf.org/html/rfc3986#section-3.3)允许使用分号来提供特定于路径段的选项。使用”?variant”格式查询参数的优势是，该格式只能用于路径段。

## 译者说
本文作者介绍了API的入口点（entry point），推荐使用RESTful API的绝对URL。同时介绍了URL含有参数时该如何处理。
