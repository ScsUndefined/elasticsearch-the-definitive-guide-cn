# 4.3.2 Single Query String 单个查询关键词

`bool`查询是最常用的用来组合多个查询子语句的工具。它适用于很多场景，尤其是当你需要在不同的字段中查询不同的内容的时候。

然而问题是，有时候，用户可能会把所有的查询信息一股脑都提交给系统，然后指望系统能够自动找出他们最想要的结果，此时你的应用会接收到一个单个的查询关键词，然后你可能需要在多个字段中搜索该查询关键词。尽管跨字段的搜索是一种高阶的搜索方式，只有专业的高级用户才能够正确驾驭它，但其实吧，它也没那么复杂，其实也蛮简单的。

在进行多词多字段的查询时，并没有一个万能的查询方法。如果你想要查出更优质的查询结果，那你首先就得深入地了解你的数据，然后才能正确地选择合适的查询方式。

## Know Your Data 了解数据

当你接收到用户输入的一个查询关键词，然后把这个关键词对多个字段同时进行查询的时候，你经常会遇到下面三种情况：

* Best fields 最优字段

  当我们查询表示一个概念的词的时候，比如“brown fox”，这两个单词组合出现的时候要比它们单独出现的时候更符合查询需求。另外当不同的字段，比如标题和正文，里都被查出有相关的信息，这两个字段就会存在竞争。而在相同的字段中，文档应该尽可能地有更多的词，相关度评分来自于最佳匹配的字段。
  
  When searching for words that represent a concept, such as “brown fox,” the words mean more together than they do individually. Fields like the title and body, while related, can be considered to be in competition with each other. Documents should have as many words as possible in the same field, and the score should come from the best-matching field.

* Most fields 越多字段越好

  一个常用的用来多相似度进行微调的技术手段是将相同的信息以不同的解析链进行索引。
  
  主字段可能有各个分词的词根形式，同义词，去音标形式。它
  
  The main field may contain words in their stemmed form, synonyms, and words stripped of their diacritics, or accents. It is used to match as many documents as possible.

  The same text could then be indexed in other fields to provide more-precise matching. One field may contain the unstemmed version, another the original word with accents, and a third might use shingles to provide information about word proximity.

  These other fields act as signals to increase the relevance score of each matching document. The more fields that match, the better.

* Cross fields

  For some entities, the identifying information is spread across multiple fields, each of which contains just a part of the whole:
  
  * Person: first_name and last_name
  * Book: title, author, and description
  * Address: street, city, country, and postcode
  
  In this case, we want to find as many words as possible in any of the listed fields. We need to search across multiple fields as if they were one big field.

All of these are multiword, multifield queries, but each requires a different strategy. We will examine each strategy in turn in the rest of this chapter.

***

The `bool` query is the mainstay of multiclause queries. It works well for many cases, especially when you are able to map different query strings to individual fields.

The problem is that, these days, users expect to be able to type all of their search terms into a single field, and expect that the application will figure out how to give them the right results. It is ironic that the multifield search form is known as Advanced Search—it may appear advanced to the user, but it is much simpler to implement.

There is no simple one-size-fits-all approach to multiword, multifield queries. To get the best results, you have to know your data and know how to use the appropriate tools.

## Know Your Data

When your only user input is a single query string, you will encounter three scenarios frequently:

* Best fields

  When searching for words that represent a concept, such as “brown fox,” the words mean more together than they do individually. Fields like the title and body, while related, can be considered to be in competition with each other. Documents should have as many words as possible in the same field, and the score should come from the best-matching field.

* Most fields
  
  A common technique for fine-tuning relevance is to index the same data into multiple fields, each with its own analysis chain.

  The main field may contain words in their stemmed form, synonyms, and words stripped of their diacritics, or accents. It is used to match as many documents as possible.

  The same text could then be indexed in other fields to provide more-precise matching. One field may contain the unstemmed version, another the original word with accents, and a third might use shingles to provide information about word proximity.

  These other fields act as signals to increase the relevance score of each matching document. The more fields that match, the better.

* Cross fields

  For some entities, the identifying information is spread across multiple fields, each of which contains just a part of the whole:
  
  * Person: first_name and last_name
  * Book: title, author, and description
  * Address: street, city, country, and postcode
  
  In this case, we want to find as many words as possible in any of the listed fields. We need to search across multiple fields as if they were one big field.

All of these are multiword, multifield queries, but each requires a different strategy. We will examine each strategy in turn in the rest of this chapter.

