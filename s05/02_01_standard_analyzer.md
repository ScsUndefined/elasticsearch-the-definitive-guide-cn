# 5.2.1 standard Analyzer 标注解析器

`标准` 解析器是默认使用于所有的全文`解析`字符串字段。如果我们使用一个 [自定义解析器](https://www.elastic.co/guide/en/elasticsearch/guide/current/custom-analyzers.html) 来重新实现 `标准` 解析器，那写法就大概是这个样子：

```json
{
    "type":      "custom",
    "tokenizer": "standard",
    "filter":  [ "lowercase", "stop" ]
}
```

在  [Normalizing Tokens](https://www.elastic.co/guide/en/elasticsearch/guide/current/token-normalization.html) 和 [Stopwords: Performance Versus Precision](https://www.elastic.co/guide/en/elasticsearch/guide/current/stopwords.html) 这两段中我们就 `小写化` 和 `无用词` *过滤器* 作了相关的讨论，那现在，让我们来对 `标准` *分词器* 来做个深入探讨。


***

The `standard` analyzer is used by default for any full-text `analyzed` string field. If we were to reimplement the `standard` analyzer as a [custom analyzer](https://www.elastic.co/guide/en/elasticsearch/guide/current/custom-analyzers.html), it would be defined as follows:

```json
{
    "type":      "custom",
    "tokenizer": "standard",
    "filter":  [ "lowercase", "stop" ]
}
```

In [Normalizing Tokens](https://www.elastic.co/guide/en/elasticsearch/guide/current/token-normalization.html) and [Stopwords: Performance Versus Precision](https://www.elastic.co/guide/en/elasticsearch/guide/current/stopwords.html), we talk about the `lowercase`, and `stop` *token filters*, but for the moment, let’s focus on the `standard` *tokenizer*.