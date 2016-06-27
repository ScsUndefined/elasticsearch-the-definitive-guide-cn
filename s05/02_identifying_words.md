# 5.2 Identifying Words 识别字词

英语中的单词还是比较好定位出来的：单词通常是由空格符或者（多个）标点符号分隔开的。但即便是在类似英语这类分词清晰的语言中，也有一些难以处理的地方：*you're*究竟算一个单词还是两个？那*o'clock*呢？*cooperate*, *half-baked*, 或者 *eyewitness*呢？

类似德语或者荷兰语之类的语言会把多个独立的单词组合在一起，创造出一个新的长而复杂的词，比如 *Weißkopfseeadler*(白头的海鹰)，但为了用户在查询`Adler`(鹰)的时候能返回`Weißkopfseeadler`，我们需要知道该如何把组合词正确地拆开。

亚洲的语言就更复杂了：一些单词，句子甚至段落之间有时候压根就没有空格符来分割。~~请你讲中文！yoyoyo~~一些词可以由单个字符构成，但同样的字符，当它被紧跟在其他字符之后的时候，又可以与它前面的字符组合成一个更长的词，并且意思还相去甚远。~~生是中国人，死亦中国魂，要我过四级？绝逼不可能！~~

要清楚，还没有一个万能解析器能惊为天人地处理所有的人类语言。Elasticsearch 搭载有针对多种语言的专用解析器，除了这些之外还有许许多多的针对语言的解析器可以以插件的形式被你找到。

但是，并非所有的语言都有它专属的解析器，甚至有时候你都不知道你在处理的东西是啥语言写的。为了应对这种蛋疼的场景，我们需要一个优秀的标准的工具集来合理地处理一些事务，而不用在意它是用什么语言写的。

***

A word in English is relatively simple to spot: words are separated by whitespace or (some) punctuation. Even in English, though, there can be controversy: is *you’re* one word or two? What about *o’clock*, *cooperate*, *half-baked*, or *eyewitness*?

Languages like German or Dutch combine individual words to create longer compound words like Weißkopfseeadler (white-headed sea eagle), but in order to be able to return *Weißkopfseeadler* as a result for the query `Adler` (eagle), we need to understand how to break up compound words into their constituent parts.

Asian languages are even more complex: some have no whitespace between words, sentences, or even paragraphs. Some words can be represented by a single character, but the same single character, when placed next to other characters, can form just one part of a longer word with a quite different meaning.

It should be obvious that there is no silver-bullet analyzer that will miraculously deal with all human languages. Elasticsearch ships with dedicated analyzers for many languages, and more language-specific analyzers are available as plug-ins.

However, not all languages have dedicated analyzers, and sometimes you won’t even be sure which language(s) you are dealing with. For these situations, we need good standard tools that do a reasonable job regardless of language.
