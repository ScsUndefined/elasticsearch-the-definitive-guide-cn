# 4.3 Multifield Search 跨字段搜索

现实环境中的搜索需求很少是简简单单一个`match`查询就能处理得了的。我们经常需要在一个或多个字段中搜索相同或不同的内容，这也就意味着我们经常需要把多个查询子句联合起来并处理好它们的各自的相关度评分，以使得整个查询语句符合我们的需求。

比如说，有时候我们可能想要搜索一本作者名为“列夫 托尔斯泰”且书名叫“战争与和平”的书；又或者我们可能想要在一本介绍 Elasticsearch 的书籍中查找有关“minimum should match”的内容，而这个查询关键词既可能出现在书的某一章节的标题中，也可能是出现在章节的正文中；再或者，我们可能会查询名字是 John 且姓氏是 Smith 的用户。

在本章节中，我们将为你呈现一些可以将多个查询条件组合起来的工具，并且告诉你怎么在具体的使用场景中正确地使用这些工具。

***

Queries are seldom simple one-clause `match` queries. We frequently need to search for the same or different query strings in one or more fields, which means that we need to be able to combine multiple query clauses and their relevance scores in a way that makes sense.

Perhaps we’re looking for a book called War and Peace by an author called Leo Tolstoy. Perhaps we’re searching the Elasticsearch documentation for “minimum should match,” which might be in the title or the body of a page. Or perhaps we’re searching for users with first name John and last name Smith.

In this chapter, we present the available tools for constructing multiclause searches and how to figure out which solution you should apply to your particular use case.


