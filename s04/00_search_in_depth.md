# 搜索进阶

在 Getting Started 中我们简单介绍了一些基础的工具，以便你先能够简单运用起 Elasticsearch 来。等你开始用起 Elasticsearch 之后用不了多久，你就会发现实际生产环境中的用户需求往往复杂得多，光运用这些基础的工具集远远满足不了用户的实际需求。比如你可能想要更灵活的搜索方式，或者更精确地对搜索结果进行排序亦或者对不同的场景进行更定制化的查询等等。

如果你想更深入一步，那么仅靠 match 查询明显不能够满足需求的。你需要更清楚地了解你的数据，然后才能够开发出更精密的搜索功能。本章节将会进一步指导你如果索引并检索你的数据，以便你能够运用搜索的一些高阶特性，比如识别词在文档中出现的前后次序，部分匹配，模糊查询以及理解人类语言等等。

清楚地理解每种搜索方式是怎么计算相关度评分的，将会有助于你微调你的搜索请求：以保证你认为最相符的结果的确能够被搜索出来并置顶显示在搜索结果集中，并且能够摈弃掉那些几乎不相关的“长尾”结果。

所谓搜索也不仅仅就是全文检索：你的数据中肯定也有很多结构化的数据，比如时间、数字等等。我们也将会开始介绍如何将全文检索和结构化检索有效得结合起来使用。

***

# Search in Depth

In Getting Started we covered the basic tools in just enough detail to allow you to start searching your data with Elasticsearch. It won’t take long, though, before you find that you want more: more flexibility when matching user queries, more-accurate ranking of results, more-specific searches to cover different problem domains.

To move to the next level, it is not enough to just use the match query. You need to understand your data and how you want to be able to search it. The chapters in this part explain how to index and query your data to allow you to take advantage of word proximity, partial matching, fuzzy matching, and language awareness.

Understanding how each query contributes to the relevance _score will help you to tune your queries: to ensure that the documents you consider to be the best results appear on the first page, and to trim the “long tail” of barely relevant results.

Search is not just about full-text search: a large portion of your data will be structured values like dates and numbers. We will start by explaining how to combine structured search with full-text search in the most efficient way.