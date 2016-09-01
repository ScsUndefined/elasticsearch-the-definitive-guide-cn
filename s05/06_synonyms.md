# 5.6 Synonyms 同义词
提取词根(termming)会导致搜索结果集变大，同义词也一样。或许现在并没有一个文档匹配“English queen”，但包含“British monarch”的内容或许也可以认为是一个不错的搜索结果。

扩大搜索结果的同时还得保证准确性。一个搜索“the US”的用户会希望搜索出含有 *United States*, *USA*, *U.S.A.*, *America*, 或者 *the States* 的文档，但不会希望搜索出关于`the states of matter` 或者 `state machines`的结果。

从上面这个例子可以看出要一个人去识别同义词有多么轻松而让机器去识别就变得非常困难（主要是机器很难去根据语境来判断同义词）当你要处理同义词的时候最直接的想法或许是试图为每种语言中的每个词汇提供它的同义词列表，以保证不管文档之间有多么微小的关联性都能够被检索出来。

但其实这是一个错误的想法。就如同我们应该倾向于尽可能少地进行词根提取操作，因为词根和它的变体可能意思相去甚远，同义词也只应在必要的时候才使用。用户明白为什么他们的查询结果会受限于他们的查询操作。但他们可不会理解他们的查询结果为什么看起来像是随机的（言下之意是大规模使用同义词会导致查询结果趋向于让人觉得是随机的）。

同义词可以用来合并拥有着相近释义的字词，比如`jump`, `leap`, 和 `hop`（三个单词都有跳跃的意思）, 或者 `pamphlet`, `leaflet`, 和 `brochure`（三者都有小册子的意思）。但从另一个角度来说，同义词也会使得词失去独有的含义而变得普通。举个例子，`bird`（鸟）可以认为是`owl`或者`pigeon`的更普通的同义词，同样的还有`adult`(成人)，可以被认为是`man`（男人）或`woman`（女人）的更普通的同义词。

同义词看上去是一个简单的概念但其实很难被用好。在本章内容里，我们会阐述使用同义词的机制以及探讨它的局限性以及易造成的陷阱。

> **提示**
> 
> 同义词会被用来扩大匹配范围。就像 [stemming](https://www.elastic.co/guide/en/elasticsearch/guide/current/stemming.html) ~~提取词根~~或者 [partial matching](https://www.elastic.co/guide/en/elasticsearch/guide/current/partial-matching.html) ~~部分匹配~~ 那样，同义词字段不应该被单独使用，而应该与一个针对主字段的查询操作一起使用，这个主字段应该包含纯净格式的原始文本。你可以查阅 [Most Fields](https://www.elastic.co/guide/en/elasticsearch/guide/current/most-fields.html) 一文来了解在使用同义词同时怎么来维护相关性。

***

While stemming helps to broaden the scope of search by simplifying inflected words to their root form, synonyms broaden the scope by relating concepts and ideas. Perhaps no documents match a query for “English queen,” but documents that contain “British monarch” would probably be considered a good match.

A user might search for “the US” and expect to find documents that contain *United States*, *USA*, *U.S.A.*, *America*, or *the States*. However, they wouldn’t expect to see results about `the states of matter` or `state machines`.

This example provides a valuable lesson. It demonstrates how simple it is for a human to distinguish between separate concepts, and how tricky it can be for mere machines. The natural tendency is to try to provide synonyms for every word in the language, to ensure that any document is findable with even the most remotely related terms.

This is a mistake. In the same way that we prefer light or minimal stemming to aggressive stemming, synonyms should be used only where necessary. Users understand why their results are limited to the words in their search query. They are less understanding when their results seems almost random.

Synonyms can be used to conflate words that have pretty much the same meaning, such as `jump`, `leap`, and `hop`, or `pamphlet`, `leaflet`, and `brochure`. Alternatively, they can be used to make a word more generic. For instance, `bird` could be used as a more general synonym for `owl` or `pigeon`, and `adult` could be used for `man` or `woman`.

Synonyms appear to be a simple concept but they are quite tricky to get right. In this chapter, we explain the mechanics of using synonyms and discuss the limitations and gotchas.

> **Tip**
> 
> Synonyms are used to broaden the scope of what is considered a matching document. Just as with [stemming](https://www.elastic.co/guide/en/elasticsearch/guide/current/stemming.html) or [partial matching](https://www.elastic.co/guide/en/elasticsearch/guide/current/partial-matching.html), synonym fields should not be used alone but should be combined with a query on a main field that contains the original text in unadulterated form. See [Most Fields](https://www.elastic.co/guide/en/elasticsearch/guide/current/most-fields.html) for an explanation of how to maintain relevance when using synonyms.

