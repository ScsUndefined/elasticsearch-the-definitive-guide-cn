# 5.1.4 One Language per Document 单个文档对应一门主语言

单个文档对应一门主语言的处理方式相对比较简单。不同语言的文档可以存储在不同的索引中—`blogs-en`（存储英文博客文章），`blogs-fr`（存放法语博客文章）等等—为每个索引设置相同的类型和字段，但使用不同的解析器：

```bash
PUT /blogs-en
{
  "mappings": {
    "post": {
      "properties": {
        "title": {
          "type": "string", ①
          "fields": {
            "stemmed": {
              "type":     "string",
              "analyzer": "english" ②
            }
}}}}}}

PUT /blogs-fr
{
  "mappings": {
    "post": {
      "properties": {
        "title": {
          "type": "string", ③
          "fields": {
            "stemmed": {
              "type":     "string",
              "analyzer": "french" ④
            }
}}}}}}
```

① ③ `blogs-en` 和 `blogs-fr` 索引都有一个叫“post”的类型以及叫做“title”的字段

② ④ 子字段 `title.stemmed` 使用一个针对语言的解析器

这种方式简洁，灵活且易于拓展，当要处理一门新语言的时候—只需要创建一个新的索引—因为每种语言都是完全分开的，我们不会深陷在  [Pitfalls of Mixing Languages](https://www.elastic.co/guide/en/elasticsearch/guide/current/language-pitfalls.html) 一文中讨论到的“如何正确地计算词频”以及“如何正确地提取词根”这两大难题之中。

单语言的文档可以被单独查询，单个查询动作也可以查询多个索引。我们甚至可以通过 `indices_boost` 参数来优先显示某种语言：

```bash
GET /blogs-*/post/_search ①
{
    "query": {
        "multi_match": {
            "query":   "deja vu",
            "fields":  [ "title", "title.stemmed" ] ②
            "type":    "most_fields"
        }
    },
    "indices_boost": { ③
        "blogs-en": 3,
        "blogs-fr": 2
    }
}
```

① 这个查询动作查询了所有以 blogs- 开头的索引

② 每个索引的 title.stemmed 字段都使用了基于语言的特定解析器

③ 或许用户的 HTTP header accept-language 中显示他偏向于英语，其次是法语，这种情况下我们就要相应地对搜索结果进行排序。其他语言的话会有一个中性值为 1

## 外文词汇

当然，这些文档里可能还穿插了一些其他语言书写的字词或者句子，并且这些内容可能没有被正确地提取词根。但对于有个主语言的文档而言，这通常都不是事儿。用户通常都会搜索一个准确的词—比如，一个外文词汇翻译过来后的词—而不是使用一个字词的变体。~~这一句不是太懂，翻译得有点生涩，自己对照下原文吧~~ [Normalizing Tokens](https://www.elastic.co/guide/en/elasticsearch/guide/current/token-normalization.html) 一文中介绍的技术手段可以增加查全率。

另外，类似地名这类的词，应该在主语言和原生语言中都应该可以被检索，比如 Munich 和 München。这两个词其实是同义词，我们将在 [Synonyms](https://www.elastic.co/guide/en/elasticsearch/guide/current/synonyms.html) 一文中进一步讨论。

> **不要用类型来标记语言**
>
> 你或许会试图使用独立的类型来处理多语言而不使用独立的索引。为了能够获得最优质的查询结果，你就不能这么做。在 [Types and Mappings](https://www.elastic.co/guide/en/elasticsearch/guide/current/mapping.html) 一文中有说过，名字相同但类型不同的字段会被索引到同一个倒排索引中。这意味着为每个类型（也即为每种语言和）计算词频的时候，结果会不准确。
>
> 要保证计算语言A中词的出现频率的时候不会影响到语言B，那你就必须为每种语言建立对应的索引或者字段，或者采用我们后续将要介绍的其他方法。

***

A single predominant language per document requires a relatively simple setup. Documents from different languages can be stored in separate indices—`blogs-en`, `blogs-fr`, and so forth—that use the same type and the same fields for each index, just with different analyzers:

```bash
PUT /blogs-en
{
  "mappings": {
    "post": {
      "properties": {
        "title": {
          "type": "string", ①
          "fields": {
            "stemmed": {
              "type":     "string",
              "analyzer": "english" ②
            }
}}}}}}

PUT /blogs-fr
{
  "mappings": {
    "post": {
      "properties": {
        "title": {
          "type": "string", ③
          "fields": {
            "stemmed": {
              "type":     "string",
              "analyzer": "french" ④
            }
}}}}}}
```

① ③ Both `blogs-en` and `blogs-fr` have a type called post that contains the field title.

② ④ The `title.stemmed` subfield uses a language-specific analyzer.

This approach is clean and flexible. New languages are easy to add—just create a new index—and because each language is completely separate, we don’t suffer from the term-frequency and stemming problems described in [Pitfalls of Mixing Languages](https://www.elastic.co/guide/en/elasticsearch/guide/current/language-pitfalls.html).

The documents of a single language can be queried independently, or queries can target multiple languages by querying multiple indices. We can even specify a preference for particular languages with the `indices_boost` parameter:

```bash
GET /blogs-*/post/_search ①
{
    "query": {
        "multi_match": {
            "query":   "deja vu",
            "fields":  [ "title", "title.stemmed" ] ②
            "type":    "most_fields"
        }
    },
    "indices_boost": { ③
        "blogs-en": 3,
        "blogs-fr": 2
    }
}
```

① This search is performed on any index beginning with blogs-.

② The title.stemmed fields are queried using the analyzer specified in each index.

③ Perhaps the user’s accept-language headers showed a preference for English, and then French, so we boost results from each index accordingly. Any other languages will have a neutral boost of 1.

## Foreign Words

Of course, these documents may contain words or sentences in other languages, and these words are unlikely to be stemmed correctly. With predominant-language documents, this is not usually a major problem. The user will often search for the exact words—for instance, of a quotation from another language—rather than for inflections of a word. Recall can be improved by using techniques explained in [Normalizing Tokens](https://www.elastic.co/guide/en/elasticsearch/guide/current/token-normalization.html).

Perhaps some words like place names should be queryable in the predominant language and in the original language, such as Munich and München. These words are effectively synonyms, which we discuss in [Synonyms](https://www.elastic.co/guide/en/elasticsearch/guide/current/synonyms.html).

> **Don’t Use Types for Languages**
>
> You may be tempted to use a separate type for each language, instead of a separate index. For best results, you should avoid using types for this purpose. As explained in [Types and Mappings](https://www.elastic.co/guide/en/elasticsearch/guide/current/mapping.html), fields from different types but with the same field name are indexed into the same inverted index. This means that the term frequencies from each type (and thus each language) are mixed together.
> 
> To ensure that the term frequencies of one language don’t pollute those of another, either use a separate index for each language, or a separate field, as explained in the next section.