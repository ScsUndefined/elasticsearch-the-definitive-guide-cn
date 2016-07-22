# 以字段为中心的查询

上述三种问题的根源是 most_fields 是一种以字段为中心的查询方式而非以词元为中心：它去寻找所有含有搜索关键词中某个词元的字段，然后根据匹配的字段数来判断查询结果的相关度，但其实我们想要的是那些与整个搜索关键词吻合程度高的数据。

> **注意**
> 
> best_fields 同样是一种以字段为中心的查询方式，所以同样有这些问题

首先我们来分析一下为什么会出现这些问题，然后我们再来解释如何解决这些问题

# 问题 1: 在多个字段中匹配到同一个词

回忆一下 most_fields 查询是怎么被执行的： Elasticsearch 会为每个字段创建一个match子查询，然后把这些子查询都包装成一个 bool 查询。

我们可以用 validate-query API 来解析一下我们的查询逻辑：

```bash
GET /_validate/query?explain
{
  "query": {
    "multi_match": {
      "query":   "Poland Street W1V",
      "type":    "most_fields",
      "fields":  [ "street", "city", "country", "postcode" ]
    }
  }
}
```

[VIEW IN SENSE](http://localhost:5601/app/sense/?load_from=https://www.elastic.co/guide/en/elasticsearch/guide/current/snippets/110_Multi_Field_Search/40_Entity_search_problems.json) 

最终结果会含有这些输出信息：

```
(street:poland   street:street   street:w1v)
(city:poland     city:street     city:w1v)
(country:poland  country:street  country:w1v)
(postcode:poland postcode:street postcode:w1v)
```

这你就会发现一个文档如果在两个字段里匹配到同一个词会高于另一个，在一个字段里匹配到两个词的文档。

# 问题 2: 摈弃长尾匹配结果

在 Controlling Precision 一节，我们谈论到了使用 operator 或者 minimum_should_match 参数来摈弃掉那些相关度程度极低的长尾匹配结果。所以或许我们这么来做：

```bash
{
    "query": {
        "multi_match": {
            "query":       "Poland Street W1V",
            "type":        "most_fields",
            "operator":    "and", ①
            "fields":      [ "street", "city", "country", "postcode" ]
        }
    }
}
```

[VIEW IN SENSE](http://localhost:5601/app/sense/?load_from=https://www.elastic.co/guide/en/elasticsearch/guide/current/snippets/110_Multi_Field_Search/40_Entity_search_problems.json) 

① 所有的词元都必须出现

但是，best_fields 或者 most_fields 会把这些参数都传递给生成的子 match 查询里，最终导致查询语句的内部逻辑变成这样：

```
(+street:poland   +street:street   +street:w1v)
(+city:poland     +city:street     +city:w1v)
(+country:poland  +country:street  +country:w1v)
(+postcode:poland +postcode:street +postcode:w1v)
```

~~看不懂？看不懂就对了！~~

换句话说，使用 operator 意味着所有这些词都必须存在在同一个字段中，这明显就不对嘛！结果什么都搜不出来！！

# 问题 3: 词频

在 What Is Relevance? 一章中我们解释了，计算每个词元的相关度评分的默认算法，即 TF/IDF:

* TF(Term frequency) 
  
  一个词元在某个文档中出现得越频繁则该文档的相关性越强。
  
* IDF(Inverse document frequency)

  在一个索引中的所有文档的某个字段中，一个词元出现的频数越高，则它的权重越低
  
但当我们跨字段进行搜索的时候，这个默认算法就会产生一些不符合搜索意图的结果了。

比如当我们在 first_name 和 last_name 字段中搜索“Peter Smith”的时候。Perter 是一个常见的名字，然后 Smith 也是一个常见的姓氏，这也就导致两个词的 IDF 都非常低。但是如果我们的索引里还有一个人叫“Smith Williams”的人那结果会怎么样呢？“Smith”是一个罕见的名字，所以它的 IDF 就会非常高！

所以像下面这样的查询就会导致 Smith Williams 会排在 Peter Smith 之前，尽管事实上我们更想要有关 Peter Smith 的结果：

```bash
{
    "query": {
        "multi_match": {
            "query":       "Peter Smith",
            "type":        "most_fields",
            "fields":      [ "*_name" ]
        }
    }
}
```

[VIEW IN SENSE](http://localhost:5601/app/sense/?load_from=https://www.elastic.co/guide/en/elasticsearch/guide/current/snippets/110_Multi_Field_Search/40_Bad_frequencies.json)

名字字段中的 Smith 的 IDF 太高了，导致它压制了有着较低 IDF 的名字字段中的 Peter 以及姓氏字段中的 Smith。

# 解决方案

这种问题只有当我们进行跨字段查询的时候才会遇到，所以假如我们把这些字段中的数据整合到一起，当作一个单个的字段进行查询，那么问题不就迎仍而解了嘛！~~so easy mama wouldn't worry about my study any more.~~。

```bash
{
    "first_name":  "Peter",
    "last_name":   "Smith",
    "full_name":   "Peter Smith"
}
```

如果我们除了 last_name 姓氏和 first_name 名字这两个字段之外还有一个叫做 full_name 全名的字段，那么当我们当我们在搜索 full_name 全名的时候：

  * 匹配了更多词元的文档的优先级就会比重复匹配了单个词元的文档高
  * minimum_should_match 和 operator parameters 参数就可以达到预期的作用
  * first_name 和 last_name 的 IDF 会被联合起来计算，这就不会导致 Smith 这个词在姓氏和名字中的 IDF 不同而导致最终结果的相关度评分不准的问题了
  
尽管这个方法的确有效，但其实你也没必要把 first_name 和 last_name 的数据重复入一遍索引入成 full_name 字段。因为 Elasticsearch 已经提供了两种可行的方案让你达到相同的目的，一种方案需要在入索引的时候执行，而另一种则是在搜索的时候执行，我们稍后会详细介绍。

***

# Field-Centric Queries

All three of the preceding problems stem from most_fields being field-centric rather than term-centric: it looks for the most matching fields, when really what we’re interested in is the most matching terms.

> **Note**
> 
> The best_fields type is also field-centric and suffers from similar problems.
!!
First we’ll look at why these problems exist, and then how we can combat them.

# Problem 1: Matching the Same Word in Multiple Fields

Think about how the most_fields query is executed: Elasticsearch generates a separate match query for each field and then wraps these match queries in an outer bool query.

We can see this by passing our query through the validate-query API:

```bash
GET /_validate/query?explain
{
  "query": {
    "multi_match": {
      "query":   "Poland Street W1V",
      "type":    "most_fields",
      "fields":  [ "street", "city", "country", "postcode" ]
    }
  }
}
```

[VIEW IN SENSE](http://localhost:5601/app/sense/?load_from=https://www.elastic.co/guide/en/elasticsearch/guide/current/snippets/110_Multi_Field_Search/40_Entity_search_problems.json) 

which yields this explanation:

```
(street:poland   street:street   street:w1v)
(city:poland     city:street     city:w1v)
(country:poland  country:street  country:w1v)
(postcode:poland postcode:street postcode:w1v)
```

You can see that a document matching just the word poland in two fields could score higher than a document matching poland and street in one field.

# Problem 2: Trimming the Long Tail

In Controlling Precision, we talked about using the and operator or the minimum_should_match parameter to trim the long tail of almost irrelevant results. Perhaps we could try this:

```bash
{
    "query": {
        "multi_match": {
            "query":       "Poland Street W1V",
            "type":        "most_fields",
            "operator":    "and", ①
            "fields":      [ "street", "city", "country", "postcode" ]
        }
    }
}
```

[VIEW IN SENSE](http://localhost:5601/app/sense/?load_from=https://www.elastic.co/guide/en/elasticsearch/guide/current/snippets/110_Multi_Field_Search/40_Entity_search_problems.json) 

① All terms must be present.

However, with best_fields or most_fields, these parameters are passed down to the generated match queries. The explanation for this query shows the following:

```
(+street:poland   +street:street   +street:w1v)
(+city:poland     +city:street     +city:w1v)
(+country:poland  +country:street  +country:w1v)
(+postcode:poland +postcode:street +postcode:w1v)
```

In other words, using the and operator means that all words must exist in the same field, which is clearly wrong! It is unlikely that any documents would match this query.

# Problem 3: Term Frequencies

In What Is Relevance?, we explained that the default similarity algorithm used to calculate the relevance score for each term is TF/IDF:

* Term frequency
  
  The more often a term appears in a field in a single document, the more relevant the document.
  
* Inverse document frequency

  The more often a term appears in a field in all documents in the index, the less relevant is that term.
  
When searching against multiple fields, TF/IDF can introduce some surprising results.

Consider our example of searching for “Peter Smith” using the first_name and last_name fields. Peter is a common first name and Smith is a common last name—both will have low IDFs. But what if we have another person in the index whose name is Smith Williams? Smith as a first name is very uncommon and so will have a high IDF!

A simple query like the following may well return Smith Williams above Peter Smith in spite of the fact that the second person is a better match than the first.

```bash
{
    "query": {
        "multi_match": {
            "query":       "Peter Smith",
            "type":        "most_fields",
            "fields":      [ "*_name" ]
        }
    }
}
```

[VIEW IN SENSE](http://localhost:5601/app/sense/?load_from=https://www.elastic.co/guide/en/elasticsearch/guide/current/snippets/110_Multi_Field_Search/40_Bad_frequencies.json)

The high IDF of smith in the first name field can overwhelm the two low IDFs of peter as a first name and smith as a last name.

# Solution

These problems only exist because we are dealing with multiple fields. If we were to combine all of these fields into a single field, the problems would vanish. We could achieve this by adding a full_name field to our person document:

```bash
{
    "first_name":  "Peter",
    "last_name":   "Smith",
    "full_name":   "Peter Smith"
}
```

When querying just the full_name field:

  * Documents with more matching words would trump documents with the same word repeated.
  * The minimum_should_match and operator parameters would function as expected.
  * The inverse document frequencies for first and last names would be combined so it wouldn’t matter whether Smith were a first or last name anymore.

While this would work, we don’t like having to store redundant data. Instead, Elasticsearch offers us two solutions—one at index time and one at search time—which we discuss next.