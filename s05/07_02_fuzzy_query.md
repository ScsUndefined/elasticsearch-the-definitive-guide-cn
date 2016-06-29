# 5.7.2 Fuzzy Query 模糊查询

[`fuzzy` query](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-fuzzy-query.html) 查询有点近似 `term` 查询。虽然你不可能经常直接使用它，但是依然有必要了解下它的工作原理，这有助于你在进行更高级别的 `match` 查询的时候使用“模糊性”这一特性。

在讲解它的工作原理前，我们先把下面这些文档存到缩索引中：

```bash
POST /my_index/my_type/_bulk
{ "index": { "_id": 1 }}
{ "text": "Surprise me!"}
{ "index": { "_id": 2 }}
{ "text": "That was surprising."}
{ "index": { "_id": 3 }}
{ "text": "I wasn't surprised."}
```

现在我们就可以针对 `surprise` 这个词来运行一个 `fuzzy` 请求：

```bash
GET /my_index/my_type/_search
{
  "query": {
    "fuzzy": {
      "text": "surprize"
    }
  }
}
```

`fuzzy` 查询是一个 term 级别的查询动作，所以它不需要做任何解析操作。它使用单个的 term 然后在 term 词典中，根据 `fuzziness` 的值来查找出所有距离相近的 term。默认的 `fuzziness` 的值是 `AUTO`。

`fuzziness` 值为 `AUTO` 就会导致 `surprize` 会被误认为是拼写 `surprise` 或者 `surprised` 时拼错而导致的，所以第一个文档和第三个文档也会被认为是符合搜索条件的文档。可以像下面这样来做，在查询的时候把本次查询的 `fuzziness` 限制为1，以减少不必要的匹配结果：

```bash
GET /my_index/my_type/_search
{
  "query": {
    "fuzzy": {
      "text": {
        "value": "surprize",
        "fuzziness": 1
      }
    }
  }
}
```

## 如何提升性能

根据指定的距离，原始的词可以衍生出一系列词，我们称这些衍生出来的词为 *莱文斯坦自动机*。

elasticsearch 在进行模糊查询的时候，会拿着这个自动机去与 term 字典中的字词一个个地进行快速比对，如果比对过程中发现有匹配的字词则会记录下来，完了之后就可以计算出匹配的文档有哪些了。

当然，一个距离为 2 的模糊查询操作，因为索引中存储的数据类型的不同，相应的匹配的出的字词数量可能也会是一个不小的数目，从而导致整个查询操作的效率巨低。所以为了保证性能，elasticsearch 还提供了下面两个可配置的参数：

* `prefix_length` 举个例子，把这个值设置成 3 的话，elasticsearch 就会假定每个单词的前三个字母是不会被拼错的，这样就可以大幅减少莱文斯坦自动机的大小。提供这个配置项的缘由是，大部分的拼写错误都不会发生在拼写单词的起始部分，而是发生在词尾。

* `max_expansions` 如果从关键词衍生出来的其他词的数量很小的话，那或许这些衍生词还是有意义的。但如果衍生出了成千上万个词，那它们中的绝大多数都是没有意义的。所以我们可以通过 `max_expansions` 来设定一个上限，来限制自动机的大小。

***

The [`fuzzy` query](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-fuzzy-query.html) is the fuzzy equivalent of the `term` query. You will seldom use it directly yourself, but understanding how it works will help you to use fuzziness in the higher-level  `match` query.

To understand how it works, we will first index some documents:

```bash
POST /my_index/my_type/_bulk
{ "index": { "_id": 1 }}
{ "text": "Surprise me!"}
{ "index": { "_id": 2 }}
{ "text": "That was surprising."}
{ "index": { "_id": 3 }}
{ "text": "I wasn't surprised."}
```

Now we can run a `fuzzy` query for the term `surprise`:

```bash
GET /my_index/my_type/_search
{
  "query": {
    "fuzzy": {
      "text": "surprize"
    }
  }
}
```

The `fuzzy` query is a term-level query, so it doesn’t do any analysis. It takes a single term and finds all terms in the term dictionary that are within the specified `fuzziness`. The default `fuzziness` is `AUTO`.

In our example, `surprize` is within an edit distance of 2 from both `surprise and `surprised`, so documents 1 and 3 match. We could reduce the matches to just `surprise` with the following query:

```bash
GET /my_index/my_type/_search
{
  "query": {
    "fuzzy": {
      "text": {
        "value": "surprize",
        "fuzziness": 1
      }
    }
  }
}
```

## Improving Performance

The `fuzzy` query works by taking the original term and building a *Levenshtein automaton* — like a big graph representing all the strings that are within the specified edit distance of the original string.

The fuzzy query then uses the automaton to step efficiently through all of the terms in the term dictionary to see if they match. Once it has collected all of the matching terms that exist in the term dictionary, it can compute the list of matching documents.

Of course, depending on the type of data stored in the index, a fuzzy query with an edit distance of 2 can match a very large number of terms and perform very badly. Two parameters can be used to limit the performance impact:

* `prefix_length` The number of initial characters that will not be “fuzzified.” Most spelling errors occur toward the end of the word, not toward the beginning. By using a `prefix_length` of `3`, for example, you can signficantly reduce the number of matching terms.

* `max_expansions` If a fuzzy query expands to three or four fuzzy options, the new options may be meaningful. If it produces 1,000 options, they are essentially meaningless. Use `max_expansions` to limit the total number of options that will be produced. The fuzzy query will collect matching terms until it runs out of terms or reaches the `max_expansions` limit.

