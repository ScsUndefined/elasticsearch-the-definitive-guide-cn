# 5.1.2 Configuring Language Analyzers 配置语言解析器

尽管语言解析器不需要任何配置，拆箱即可用，但其实大多数的语言解析器都允许你明确地控制它们的行为:

### 词根白名单

试想一下这情况，举个例子，用户在查找“世界卫生组织”（World Health Organization），但是查出来的结果确是和“组织 卫生”(organ health)相关联的一些结果。之所以会造成这样一种乱相是因为动词“组织”（organ）和名字“组织（organization）”都被提取成了词根：“organ”。通常情况下这都不会有什么问题，但在这个特殊的文档中，这会导致搜索结果变得混乱。我们应该防止名词“组织”（organization）以及它的复词形式(organizations)被格式化成词根形式。

### 自定义无用词

英语的无用词列表默认情况下含有这些词：

a, an, and, are, as, at, be, but, by, for, if, in, into, is, it,
no, not, of, on, or, such, that, the, their, then, there, these,
they, this, to, was, will, with

`no`和`not`不同于其他词，它们会否定跟在它们后面的那个词的意思。或许我们应该把它们踢出无用词清单，因为它们的语义含义有时候很重要。

要自定义`英语`解析器的行为，我们可以自定义一个基于`英语`解析器但又有一些新增配置的新解析器：

```bash
PUT /my_index
{
  "settings": {
    "analysis": {
      "analyzer": {
        "my_english": {
          "type": "english",
          "stem_exclusion": [ "organization", "organizations" ], ①
          "stopwords": [ ②
            "a", "an", "and", "are", "as", "at", "be", "but", "by", "for",
            "if", "in", "into", "is", "it", "of", "on", "or", "such", "that",
            "the", "their", "then", "there", "these", "they", "this", "to",
            "was", "will", "with"
          ]
        }
      }
    }
  }
}

GET /my_index/_analyze?analyzer=my_english ③
The World Health Organization does not sell organs.
```

① 防止 `organization` 和 `organizations` 被格式化成词根形式

② 指定一个自定义的无用词列表

③ 被解析出的单词： `world`, `health`, `organization`, `does`, `not`, `sell`, `organ`

我们将在  [Reducing Words to Their Root Form](https://www.elastic.co/guide/en/elasticsearch/guide/current/stemming.html) 和 [Stopwords: Performance Versus Precision](https://www.elastic.co/guide/en/elasticsearch/guide/current/stopwords.html) 中分别对词根和无用词进行更深入的探讨

***

While the language analyzers can be used out of the box without any configuration, most of them do allow you to control aspects of their behavior, specifically:

### Stem-word exclusion

Imagine, for instance, that users searching for the “World Health Organization” are instead getting results for “organ health.” The reason for this confusion is that both “organ” and “organization” are stemmed to the same root word: `organ`. Often this isn’t a problem, but in this particular collection of documents, this leads to confusing results. We would like to prevent the words `organization` and `organizations` from being stemmed.

### Custom stopwords

The default list of stopwords used in English are as follows:

a, an, and, are, as, at, be, but, by, for, if, in, into, is, it,
no, not, of, on, or, such, that, the, their, then, there, these,
they, this, to, was, will, with

The unusual thing about `no` and `not` is that they invert the meaning of the words that follow them. Perhaps we decide that these two words are important and that we shouldn’t treat them as stopwords.

To customize the behavior of the `english` analyzer, we need to create a custom analyzer that uses the `english` analyzer as its base but adds some configuration:

```bash
PUT /my_index
{
  "settings": {
    "analysis": {
      "analyzer": {
        "my_english": {
          "type": "english",
          "stem_exclusion": [ "organization", "organizations" ], ①
          "stopwords": [ ②
            "a", "an", "and", "are", "as", "at", "be", "but", "by", "for",
            "if", "in", "into", "is", "it", "of", "on", "or", "such", "that",
            "the", "their", "then", "there", "these", "they", "this", "to",
            "was", "will", "with"
          ]
        }
      }
    }
  }
}

GET /my_index/_analyze?analyzer=my_english ③
The World Health Organization does not sell organs.
```

① Prevents `organization` and `organizations` from being stemmed

② Specifies a custom list of stopwords

③ Emits tokens `world`, `health`, `organization`, `does`, `not`, `sell`, `organ`

We discuss stemming and stopwords in much more detail in [Reducing Words to Their Root Form](https://www.elastic.co/guide/en/elasticsearch/guide/current/stemming.html) and [Stopwords: Performance Versus Precision](https://www.elastic.co/guide/en/elasticsearch/guide/current/stopwords.html), respectively.