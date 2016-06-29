# 5.7 Typoes and Mispelings

对于像日期啊，价格之类的结构化的数据，我们往往会希望我们查询到的是精确匹配的结果。但是，对全文搜索而言，就不应该有类似这种精确性的限制，而是应该尽可能地搜索出所有相关联的结果，并且最好用相关性排序来把高质量的搜索结果优先显示出来。

而如果让全文搜索尽可能地搜出精确结果的话，只会让用户觉得不够智能。换位思考下，难道你不希望你在搜 “quick brown fox” 的时候查出含有 “fast brown foxes” 的结果？或者在搜索 “Johnny Walker” 的时候，搜到关于 “Johnnie Walker” 的结果？或者搜 “Arnold Shcwarzenneger” 的时候搜出 “Arnold Schwarzenegger”？



If documents exist that *do* contain exactly what the user has queried, they should appear at the top of the result set, but weaker matches can be included further down the list. If no documents match exactly, at least we can show the user potential matches; they may even be what the user originally intended!

We have already looked at diacritic-free matching in *[Normalizing Tokens](https://www.elastic.co/guide/en/elasticsearch/guide/current/token-normalization.html)*, word stemming in *[Reducing Words to Their Root Form](https://www.elastic.co/guide/en/elasticsearch/guide/current/stemming.html)*, and synonyms in *[Synonyms](https://www.elastic.co/guide/en/elasticsearch/guide/current/synonyms.html)*, but all of those approaches presuppose that words are spelled correctly, or that there is only one way to spell each word.

Fuzzy matching allows for query-time matching of misspelled words, while phonetic token filters at index time can be used for *sounds-like* matching.

***

We expect a query on structured data like dates and prices to return only documents that match exactly. However, good full-text search shouldn’t have the same restriction. Instead, we can widen the net to include words that may match, but use the relevance score to push the better matches to the top of the result set.

In fact, full-text search that only matches exactly will probably frustrate your users. Wouldn’t you expect a search for “quick brown fox” to match a document containing “fast brown foxes,” “Johnny Walker” to match “Johnnie Walker,” or “Arnold Shcwarzenneger” to match “Arnold Schwarzenegger”?

If documents exist that *do* contain exactly what the user has queried, they should appear at the top of the result set, but weaker matches can be included further down the list. If no documents match exactly, at least we can show the user potential matches; they may even be what the user originally intended!

We have already looked at diacritic-free matching in *[Normalizing Tokens](https://www.elastic.co/guide/en/elasticsearch/guide/current/token-normalization.html)*, word stemming in *[Reducing Words to Their Root Form](https://www.elastic.co/guide/en/elasticsearch/guide/current/stemming.html)*, and synonyms in *[Synonyms](https://www.elastic.co/guide/en/elasticsearch/guide/current/synonyms.html)*, but all of those approaches presuppose that words are spelled correctly, or that there is only one way to spell each word.

Fuzzy matching allows for query-time matching of misspelled words, while phonetic token filters at index time can be used for *sounds-like* matching.