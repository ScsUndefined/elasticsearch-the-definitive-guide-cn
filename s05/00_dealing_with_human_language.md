# 5 Dealing with Human Language 处理人类语言

“这句子的每个字我都认得，可是整个句子我依然看不懂。。。” -- Matt Groening

全文搜索是一场精确（即尽可能少地搜出不相关的文档）与查全（即尽可能多地搜出相关的文档）之间的斗争。在进行检索处理时，如果只匹配用户输入的文字，那这的确是一种非常严谨的做法，但光这么做还不够。我们还是会漏掉很多用户想搜出的文档。所以我们应该广撒网 ~~然而并没有鱼~~, 把那些并非精确匹配用户输入的信息，但却是相关的内容也给检索出来。

难道你不希望你在搜索“quick brown fox”的时候，搜索结果里也含有“fast brown foxes”？ ~~不希望~~ 或者在搜索“Johnny Walker”的时候，搜出“Johnnie Walker”？ ~~不喝酒~~ 再或者搜“Arnolt Schwarzenneger”的时候搜出“Arnold Schwarzenegger”？ ~~不看终结者~~

如果文档内容的确和用户查询的信息精确匹配，那这些文档就应该现在是搜索结果集的最上面，但是那些非精确匹配但仍有一定相关性的文档也应该紧跟着显示在搜索结果中。这样，即使没有任何精确匹配的结果被检索出来，我们依然可能显示给用户那些有一定相关性的文档，甚至有时候其实这些文档就是用户本意想要检索的东西。

真要这么做的话，那就有许多难点要攻克：
  
  * 移除变音符号，比如`´`， `^`以及`¨`等，这样之后，搜索 `rôle` 的时候也就可以搜出`role`，反之亦然。更多信息在 [Normalizing Tokens](https://www.elastic.co/guide/en/elasticsearch/guide/current/token-normalization.html)
  
  * 把每个单词都*去掉茎*而获取它的词根，比如移除单词中可数名字的单复数格式之间的区别—`fox`,`foxes`—或者移除动词的不同时态之间的区别—`jumping`,`jumped`,`jumps`。更多信息在 [Reducing Words to Their Root Form](https://www.elastic.co/guide/en/elasticsearch/guide/current/stemming.html)

  * 移除常用词或无用词，比如`the`, `and`以及`or`来改进搜索性能。更多信息在  [Stopwords: Performance Versus Precision](https://www.elastic.co/guide/en/elasticsearch/guide/current/stopwords.html)

  * 处理同义词使得再搜索`quick`的时候也能搜出`fast`，或者搜索`UK`的时候同样能搜出`United Kingdom`。更多信息在 [Synonyms](https://www.elastic.co/guide/en/elasticsearch/guide/current/synonyms.html)

  * 检测是否有拼写错误，或者替代写法，或者匹配那些发音相近的 *同音词*,比如`their`与`there`，`meat`，`meet`与`mete`。更多信息在 [Typoes and Mispelings](https://www.elastic.co/guide/en/elasticsearch/guide/current/fuzzy-matching.html)

在我们能都处理单个的字词的时候，我们首先得把一段文字正确地切分出一个个字词，这也就意味着我们需要知道一个 *字词* 如何构成的。我们会在 [Identifying Words](https://www.elastic.co/guide/en/elasticsearch/guide/current/identifying-words.html) 一文中深入介绍。

但现在我们先简单快速地讲点入门知识。

***

“I know all those words, but that sentence makes no sense to me.” -- Matt Groening

Full-text search is a battle between *precision*—returning as few irrelevant documents as possible—and *recall*—returning as many relevant documents as possible. While matching only the exact words that the user has queried would be precise, it is not enough. We would miss out on many documents that the user would consider to be relevant. Instead, we need to spread the net wider, to also search for words that are not exactly the same as the original but are related.

Wouldn’t you expect a search for “quick brown fox” to match a document containing “fast brown foxes,” “Johnny Walker” to match “Johnnie Walker,” or “Arnolt Schwarzenneger” to match “Arnold Schwarzenegger”?

If documents exist that *do* contain exactly what the user has queried, those documents should appear at the top of the result set, but weaker matches can be included further down the list. If no documents match exactly, at least we can show the user potential matches; they may even be what the user originally intended!

There are several lines of attack:

  * Remove diacritics like `´`, `^`, and `¨` so that a search for `rôle` will also match `role`, and vice versa. See [Normalizing Tokens](https://www.elastic.co/guide/en/elasticsearch/guide/current/token-normalization.html).
  * Remove the distinction between singular and plural—`fox` versus `foxes`—or between tenses—`jumping` versus `jumped` versus `jumps`—by *stemming* each word to its root form. See [Reducing Words to Their Root Form](https://www.elastic.co/guide/en/elasticsearch/guide/current/stemming.html).
  * Remove commonly used words or stopwords like `the`, `and`, and `or` to improve search performance. See [Stopwords: Performance Versus Precision](https://www.elastic.co/guide/en/elasticsearch/guide/current/stopwords.html).
  * Including synonyms so that a query for `quick` could also match `fast`, or `UK` could match `United Kingdom`. See [Synonyms](https://www.elastic.co/guide/en/elasticsearch/guide/current/synonyms.html).
  * Check for misspellings or alternate spellings, or match on *homophones—words* that sound the same, like `their` versus `there`, `meat` versus `meet` versus `mete`. See [Typoes and Mispelings](https://www.elastic.co/guide/en/elasticsearch/guide/current/fuzzy-matching.html).

Before we can manipulate individual words, we need to divide text into words, which means that we need to know what constitutes a *word*. We will tackle this in [Identifying Words](https://www.elastic.co/guide/en/elasticsearch/guide/current/identifying-words.html).

But first, let’s take a look at how to get started quickly and easily.