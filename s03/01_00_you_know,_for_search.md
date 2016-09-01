# 3.1 You Know, for Search…
Elasticsearch 是一个开源的基于 Apache Lucene 的开源搜索引擎，一个全文搜索引擎库。而不管是在开源界还是非开源界， Apache Lucene 都可以称得上是最先进、性能最好并且功能最全的搜索引擎库。

但是 Lucene 仅仅是个库。如果你要使用它的话，你就得把它整个的整合到你的应用中来。而且你可能还需要一点信息检索相关的专业知识，以便能够理解 Lucene 是怎么工作的。Lucene 相对比较复杂，要用好它的话，学习成本比较大。

Elasticsearch 也是使用 Java 语言编写的（最初是由 java 编写的，现在已经有许多其他语言改写的版本了）。在内部通过 Lucence 去索引数据或者检索数据。而它的目标则是让全文检索变得简单，并且通过一个简洁的 RESTful 的 API 来屏蔽掉 Lucene 的复杂性。

正所谓青出于蓝而胜于蓝，Elasticsearch 并不像 Lucene 那样仅仅是个全文检索引擎。它其实还可以被这么来称呼：

* 一个分布式的实时的数据库
* 可以进行实时分析的分布式的搜索引擎
* 可以横向扩展成上百台服务器集群，并处理PB级的结构化或非结构化的数据。并可以把所有功能通过一个简单的 RESTful API 来暴露出去，使得你可以使用任意一种你喜欢的编程语言来构建一个 web 客户端去调用这个 API，甚至是直接通过命令行。

Elasticsearh 相对来说非常适合新手。它预置了各种默认设置并且屏蔽了复杂的搜索理论，让新手可以非常傻瓜式地进行使用。拆箱即用，你甚至不需要了解地多深入就可以在生产环境使用了。（然后用着用着就发现怎么这么多坑？！！！）

等你知识渐长，羽翼渐丰的时候，你就可以使用更多的 Elasticsearch 的高阶特性了。Elasticsearch 本身就是可配置的，非常灵活。你可以根据你的业务场景来配置 Elasticsearch，进行个性化的定制。

下载，使用或者修改源码都是免费的。Elasticsearch 使用 Apache 2 license来发布。这个 license 可以说是限制最小的开源许可了。源代码被挂到 github 上了。如果你有心想要加入狂拽炫酷吊炸天的 Elasticsearch 贡献者社区而又不清楚门路在哪的话，看下 《Contributing to Elasticsearch》 ！！！

如果你有任何的疑问，不管是关于 Elasticsearch 项目本身还是具体的某个特性或者各种语言平台的客户端，或者插件什么的，欢迎访问 discuss.elastic.co。

大雾

故事发生在好几年前，有这个一个会写点代码的小屌丝，叫 Shay Banon，新婚没多久（卧槽，你让那么多年薪百万还追不到妹子的码农情何以堪），没有工作，跟着他老婆来到了大雾都伦敦。他老婆要在这学习怎么当一个大厨（卧槽，还是个有一手好厨艺的妹子！！！我特么要给你寄刀片喏！！！）。在求职的同时，他也开始玩 Lucene 了，当时 Lucene 的版本还很新，想给他最爱的夫人做一个菜单的搜索引擎。

直接使用 Lucene 非常的恶心（当年 Solr 肯定还没横空出世），上文提到过了，直接使用 Lucene 的话需要你有一定的信息检索相关的专业知识。所以 Shay 开始尝试着在 Lucene 之上再做一层抽象，使得 Java 开发人员能够更轻松地使用 Lucene 来给自己的应用添加搜索功能。他把这块代码开源了出去并且起名叫“Compass”。（这名字比 Elasticsearch 逼格高得不知道要到那里去）

后来这小屌丝终于找到了一份工作，天天跟一个高性能的分布式的环境打交道。这导致他迫切地需要一个高性能的，实时的，分布式的搜索引擎，于是乎他决定重写 Compass 类库，把它改写成一个独立的服务，并改名叫 Elasticsearch。（刚真，不改名逼格会高很多。。。）

第一版发布于 2010年2越。自此 Elasticsearch 就成了 Github 上最火热的项目之一，有超过 300 名贡献者为它贡献过代码（如果 Elasticsearch 的文档也算其一部分的话，我也是贡献者的一员啊～～不知廉耻地捂脸～～～～），然后这个小屌丝还自己组了一个公司出任CEO，迎娶～～哦已婚了，经营范围为提供商业的 Elasticsearch 支持并开发新特性，但要强调的是，Elasticsearch 现在是并且将永远是一个开源项目。（开源万岁～～～～）

哦，对了，他老婆还在等她的菜谱搜索应用。。。

***

# 3.1 You Know, for Search…
Elasticsearch is an open-source search engine built on top of Apache Lucene™, a full-text search-engine library. Lucene is arguably the most advanced, high-performance, and fully featured search engine library in existence today—both open source and proprietary.

But Lucene is just a library. To leverage its power, you need to work in Java and to integrate Lucene directly with your application. Worse, you will likely require a degree in information retrieval to understand how it works. Lucene is very complex.

Elasticsearch is also written in Java and uses Lucene internally for all of its indexing and searching, but it aims to make full-text search easy by hiding the complexities of Lucene behind a simple, coherent, RESTful API.

However, Elasticsearch is much more than just Lucene and much more than “just” full-text search. It can also be described as follows:

* A distributed real-time document store where every field is indexed and searchable
* A distributed search engine with real-time analytics
* Capable of scaling to hundreds of servers and petabytes of structured and unstructured data And it packages up all this functionality into a standalone server that your application can talk to via a simple RESTful API, using a web client from your favorite programming language, or even from the command line.

It is easy to get started with Elasticsearch. It ships with sensible defaults and hides complicated search theory away from beginners. It just works, right out of the box. With minimal understanding, you can soon become productive.

As your knowledge grows, you can leverage more of Elasticsearch’s advanced features. The entire engine is configurable and flexible. Pick and choose from the advanced features to tailor Elasticsearch to your problem domain.

You can download, use, and modify Elasticsearch free of charge. It is available under the Apache 2 license, one of the most flexible open source licenses available. The source is hosted on GitHub at github.com/elastic/elasticsearch. See Contributing to Elasticsearch if you would like to join our amazing community of contributors!

If you have any questions related to Elasticsearch, including specific features, language clients and plugins, join the conversation at discuss.elastic.co.

The Mists of Time

Many years ago, a newly married unemployed developer called Shay Banon followed his wife to London, where she was studying to be a chef. While looking for gainful employment, he started playing with an early version of Lucene, with the intent of building his wife a recipe search engine.

Working directly with Lucene can be tricky, so Shay started work on an abstraction layer to make it easier for Java programmers to add search to their applications. He released this as his first open source project, called Compass.

Later Shay took a job working in a high-performance, distributed environment with in-memory data grids. The need for a high-performance, real-time, distributed search engine was obvious, and he decided to rewrite the Compass libraries as a standalone server called Elasticsearch.

The first public release came out in February 2010. Since then, Elasticsearch has become one of the most popular projects on GitHub with commits from over 300 contributors. A company has formed around Elasticsearch to provide commercial support and to develop new features, but Elasticsearch is, and forever will be, open source and available to all.

Shay’s wife is still waiting for the recipe search…