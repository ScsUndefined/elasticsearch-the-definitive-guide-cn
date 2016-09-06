# 手动排序

传统数据库的查询就不需要考虑相关性，它们只要判断数据是否符合查询条件就可以了。

而全文检索除了要判断文档是否与查询条件相关，还要判断它们到底有多么得相关。所以全文检索引擎不仅需要要判断数据是否与查询条件相关，还要计算它们之间的相关程度，并以此来对查询结果进行排序。

相关程度的计算公式，即术语“相似度算法”，会基于数据的多个维度计算出一个分值，分值高低表示了该数据与查询条件究竟有多相关。在本章，将会介绍多种控制相似度算法的做法。

不仅是全文检索的时候可能会想要控制相似度算法，有时候对结构化的字段进行查询的时候可能也需要进行控制。比如类似这样的业务场景：我们正在找有空调，有wifi的海景度假别墅。是否有空调，是否有wifi，是否海景这三个条件，满足得越多则得分就应该越高。有或者我们除了用相似度算法算出来的相似度评分外，可能还想要综合度假别墅的新旧程序，价格，口碑或者距离等进行最终的权衡。

类似上面举的这种复杂的需求都可以使用 Elasticsearch 来实现，它内置了海量而又强有力的排序功能。

接下来我们先简单介绍下 Lucene 计算相关性的过程，紧接着边演示边说明如何操控这个计算过程。

***

# Controlling Relevance

Databases that deal purely in structured data (such as dates, numbers, and string enums) have it easy: they just have to check whether a document (or a row, in a relational database) matches the query.

While Boolean yes/no matches are an essential part of full-text search, they are not enough by themselves. Instead, we also need to know how relevant each document is to the query. Full-text search engines have to not only find the matching documents, but also sort them by relevance.

Full-text relevance formulae, or similarity algorithms, combine several factors to produce a single relevance _score for each document. In this chapter, we examine the various moving parts and discuss how they can be controlled.

Of course, relevance is not just about full-text queries; it may need to take structured data into account as well. Perhaps we are looking for a vacation home with particular features (air-conditioning, sea view, free WiFi). The more features that a property has, the more relevant it is. Or perhaps we want to factor in sliding scales like recency, price, popularity, or distance, while still taking the relevance of a full-text query into account.

All of this is possible thanks to the powerful scoring infrastructure available in Elasticsearch.

We will start by looking at the theoretical side of how Lucene calculates relevance, and then move on to practical examples of how you can control the process.