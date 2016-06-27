# 5.6.3 Expand or contract 拓展还是收缩

在 [Formatting Synonyms](https://www.elastic.co/guide/en/elasticsearch/guide/current/synonym-formats.html) 一文中，我们已经看到我们可以使用 *简单表达式* *简单缩写式* 或者 *通用扩展形式* 来指明同义词规则。本章节我们将在这三者间做个权衡比较。

> **提示**
> 
> 本章讨论内容仅针对单个词的同义词。多词同义则增加了一层复杂性，并且晚些时候会在 [Multiword Synonyms and Phrase Queries](https://www.elastic.co/guide/en/elasticsearch/guide/current/multi-word-synonyms.html) 一文中另作讨论。

## 简单表达式

使用 *简单表达式* ，我们可以把同义词列表中的任意一个词拓展成该同义词列表中的 *所有* 词：

"jump,hop,leap"

拓展过程既可以发生在入索引的时候，也可以发生在搜索的时候。两种使用时机都有着各自的利(⬆︎)弊（⬇︎）。到底要在哪个时候使用，则取决于性能与灵活性：

||在入索引时使用|在查询时使用|
|--|--|--|
| **索引大小**  |⬇︎ 因为所有的同义词都会被索引，所以索引的大小相对会变大一些|⬆︎ 正常大小
| **相关性**   |⬇︎ 所有的同义词都拥有着相同的 IDF (至于什么是 IDF ，参见 [What Is Relevance?](https://www.elastic.co/guide/en/elasticsearch/guide/current/relevance-intro.html) 一文)，这也即以为着，通用的词和意思相近的生僻词都拥有着相同的权重|⬆︎ 每个同义词的 IDF 都和原来一样
| **性能** |⬆︎ 一个查询动作只需要查找在查询条件中指明的那一个词即可|⬇︎ 针对某个词的查询动作会被重写为针对这个词的本身以及所有的同义词的查询动作，这也导致了性能的降低
| **灵活性** |⬇︎ 已经建好的索引，它的同义词规则就无法改变了，如果需要启用新的同义词规则，那么那些已经存在索引中的文档就都必须要重新索引一次|⬆︎ 同义词规则在不重新索引文档的前提下就可以更新，非常方便

## 简单缩写式

*简单缩写式* 把左边的多个同义词映射到了右边的单个词：

"leap,hop => jump"

这必须在索引时期和查询时期同时被应用，以便能够确保查询的关键词可以被映射成同样的，单个的，已经被索引的词。

这简单表达式相比，这个方法既有自己的优点，也有相比较而言的缺点:

* 索引大小
  
  ⬆︎ 索引大小是正常的，因为只有单个词会被入索引

* 相关性
  
  ⬇︎ 所有词的 IDF 都是相同的，所以如果两个词是意思相近，你就没法分辨它们中哪个是比较常用的词，哪个是不怎么常用的词

* 性能

  ⬆︎ 一个查询动作仅需要查询在索引中被存的那一个词即可

* 灵活性

  ⬆︎ 新的同义词规则可以新增到同义词规则的左边，并在查询的时候使用。举个例子，设想这样一种场景，我们需要加一个新词到先前已经上面那个同义词规则中。那下面这个新的同义词规则在查询的时候或者新的文档要入索引的时候都会起到自己的限制作用：

  "leap,hop,bound => jump"
  
  似乎对旧有的文档不起作用是么？其实我们可以把上面这个同义词规则改写下，以便对旧有文档同样起作用：

  "leap,hop,bound => jump,bound"
  
  等到我们需要重新索引文档的时候，我们可以把这个同义词规则还原成上面那样以改进性能，因为上面那个规则相比这个而言，查询的时候就只要查询一个词了。
  
## 通用拓展形式

*通用拓展形式* 和简单表达式以及缩写式都不尽相同。它并没有把所有的同义词都同等看待，而是拓展了每个词的含义，使得被拓展的词变得更通用。看一下这段示例代：

"cat    => cat,pet",
"kitten => kitten,cat,pet",
"dog    => dog,pet"
"puppy  => puppy,dog,pet"

在索引的时候应用这个同义词规则就可以：

* 一个针对 `kitten` 的查询操作只会查询到有关 `kitten` 的结果
 
* 一个针对 `cat` 的查询操作会查询到有关 `kitten` 和 `cat` 的结果

* 一个这对 `pet` 的查询操作会查询到有关 `kitten`, `cat`, `puppy`, `dog` 或者 `pet`的结果

而如果在查询的时候应用这个同义词规则，一个针对 `kitten` 的查询结果就会被拓展成涉及到 `kitten` , `cat` 或者 `pet` 的结果。

你也可以在索引时应用这个同义词规则而在查询时视具体情况而选择到底要不要使用这个同义词规则，以便获得最佳的效果。具体地说，你需要在索引的时候应用同义词规则，使得通用性能够被存储到索引中。然后在查询的时候，你可以选择不应用同义词规则，这样一个针对 `kitten` 的查询就只会查出有关 `kitten` 的文档，亦或，你可以在查询的时候选择使用同义词规则，这样，针对 `kitten` 的查询操作就会返回包括 `kitten`， `cat` 以及 `pet` （也即包括 `dog` 和 `puppy`）的相关结果。

在上述的例子中，`kitten` 的 IDF 将会是正确的，因为 `cat` 和 `pet` 的 IDF 将会被 Elasticsearch 降权。这对你是有益的—通常而言—当一个针对 `kitten` 的查询被拓展成了针对 `kitten 或者 cat 又或者 pet` 的查询动作，那有关 `kitten` 相关的文档就应该排在最上方，紧跟着的应该是有关 `cat` 的文档，最后是 `pet`. 相关的
***

In [Formatting Synonyms](https://www.elastic.co/guide/en/elasticsearch/guide/current/synonym-formats.html), we have seen that it is possible to replace synonyms by *simple expansion*, *simple contraction*, or *generic expansion*. We will look at the trade-offs of each of these techniques in this section.

> **Tip**
> 
> This section deals with single-word synonyms only. Multiword synonyms add another layer of complexity and are discussed later in [Multiword Synonyms and Phrase Queries](https://www.elastic.co/guide/en/elasticsearch/guide/current/multi-word-synonyms.html).

## Simple Expansion

With *simple expansion*, any of the listed synonyms is expanded into *all* of the listed synonyms:

"jump,hop,leap"
Expansion can be applied either at index time or at query time. Each has advantages (⬆)︎ and disadvantages (⬇)︎. When to use which comes down to performance versus flexibility.

||Index time|	Query time|
|--|--|--|
| **Index size**  |⬇︎ Bigger index because all synonyms must be indexed.|⬆︎ Normal.
| **Relevance**   |⬇︎ All synonyms will have the same IDF (see [What Is Relevance?](https://www.elastic.co/guide/en/elasticsearch/guide/current/relevance-intro.html)), meaning that more commonly used words will have the same weight as less commonly used words.|⬆︎ The IDF for each synonym will be correct.
| **Performance** |⬆︎ A query needs to find only the single term specified in the query string.|⬇︎ A query for a single term is rewritten to look up all synonyms, which decreases performance.
| **Flexibility** |⬇︎ The synonym rules can’t be changed for existing documents. For the new rules to have effect, existing documents have to be reindexed.|⬆︎ Synonym rules can be updated without reindexing documents.

## Simple Contraction

*Simple contraction* maps a group of synonyms on the left side to a single value on the right side:

"leap,hop => jump"

It must be applied both at index time and at query time, to ensure that query terms are mapped to the same single value that exists in the index.

This approach has some advantages and some disadvantages compared to the simple expansion approach:

* Index size
  
  ⬆︎ The index size is normal, as only a single term is indexed.

* Relevance
  
  ⬇︎ The IDF for all terms is the same, so you can’t distinguish between more commonly used words and less commonly used words.

* Performance

  ⬆︎ A query needs to find only the single term that appears in the index.

* Flexibility

  ⬆︎ New synonyms can be added to the left side of the rule and applied at query time. For instance, imagine that we wanted to add the word bound to the rule specified previously. The following rule would work for queries that contain bound or for newly added documents that contain bound:

  "leap,hop,bound => jump"
  
  But we could expand the effect to also take into account existing documents that contain bound by writing the rule as follows:

  "leap,hop,bound => jump,bound"
  
  When you reindex your documents, you could revert to the previous rule to gain the performance benefit of querying only a single term.

## Genre Expansion

Genre expansion is quite different from simple contraction or expansion. Instead of treating all synonyms as equal, genre expansion widens the meaning of a term to be more generic. Take these rules, for example:

"cat    => cat,pet",
"kitten => kitten,cat,pet",
"dog    => dog,pet"
"puppy  => puppy,dog,pet"

By applying genre expansion at index time:

* A query for kitten would find just documents about kittens.
* A query for cat would find documents abouts kittens and cats.
* A query for pet would find documents about kittens, cats, puppies, dogs, or pets.

Alternatively, by applying genre expansion at query time, a query for `kitten` would be expanded to return documents that mention kittens, cats, or pets specifically.

You could also have the best of both worlds by applying expansion at index time to ensure that the genres are present in the index. Then, at query time, you can choose to not apply synonyms (so that a query for `kitten` returns only documents about kittens) or to apply synonyms in order to match kittens, cats and pets (including the canine variety).

With the preceding example rules above, the IDF for `kitten` will be correct, while the IDF for `cat` and `pet` will be artificially deflated. However, this works in your favor—a genre-expanded query for `kitten OR cat OR pet` will rank documents with `kitten` highest, followed by documents with `cat`, and documents with `pet` would be right at the bottom.