# 全文检索

至此我们已经介绍过了怎么对结构化的数据进行一些简单的搜索操作，现在该接触一点全文检索的知识了，即如何在 full-text 型字段中进行搜索以获得最贴合搜索关键词的文档。

全文检索有两大核心要点：

* 正确地计算相关度

  即要能够正确地甄别与搜索关键词相近的搜索结果的相近程序。无论这个相近程度是通过使用 TF/IDF 算法计算出来的，还是通过地理位置信息求出来的相近的位置，亦或者是模糊相似度算法或者其他算法。

* 解析处理

  所谓解析处理即将一长串的词解析成一个个不同的，格式化过的词元（详细信息请参阅 Analysis and Analyzers）~~对于中文数据的全文搜索的话，可以简单理解成就是将长文本进行拆分，拆成一个个不能再分割的字或者词~~以(a)创建一个倒排索引然后(b)对倒排索引进行查询。
  
  
当我们在探讨相关度算法或者如何对数据进行解析处理的时候，我们其实就已经进入了“查询”领域，我们所探讨的东西已经与如何“过滤”数据无关了。

***

# Full-Text Search

Now that we have covered the simple case of searching for structured data, it is time to explore full-text search: how to search within full-text fields in order to find the most relevant documents.

The two most important aspects of full-text search are as follows:

* Relevance
  
  The ability to rank results by how relevant they are to the given query, whether relevance is calculated using TF/IDF (see What Is Relevance?), proximity to a geolocation, fuzzy similarity, or some other algorithm.

* Analysis
  
  The process of converting a block of text into distinct, normalized tokens (see Analysis and Analyzers) in order to (a) create an inverted index and (b) query the inverted index.

As soon as we talk about either relevance or analysis, we are in the territory of queries, rather than filters.