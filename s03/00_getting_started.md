# 基础入门

Elasticsearch 是一个实时的，分布式的搜索及分析引擎。它能让你能够以前所未有的速度来检索你的数据，并且数量级也将超过你原本无法处理的级别。它可以用来实现全文检索，结构化搜索亦可以用来进行数据分析：

  * 维基百科网站使用 Elasticseach 来提供全文检索功能并对搜索结果进行高亮处理，并通过 Elasticsearch 来提供“搜我所输”及“搜我所想”的建议型搜索功能。
  
  * 《卫报》使用 Elasticsearch 来对访客日志和社交网络数据进行数据挖掘并实时地给新闻编辑人员以反馈，反馈公众对新文章的反应。
  
  * Stack Overflow 使用 Elasticsearch 来提供全文检索和基于地理位置的检索功能，并使用相似检索来找出用户想要的问题及回答。
  
  * GitHub 则用 Elasticsearch 来处理 1300 亿行数据。

但 Elasticsearch 并非专为特大型企业级开发而生的一个项目。它也帮助了很多诸如 Datadog 和 Klout 等初创型公司将它们的创业理念成功地变为了现实，并让它们提供的服务拥有了可伸缩性。Elasticsearch 即可运行在你的笔记本上，也可以部署在大规模的服务器集群上并处理PB级的数据量。

尽管 Elasticsearch 版本一直在更迭，但它并没有在某次版本更迭中加入了新的核心功能或者某项革命性的重大特性。自 Elasticsearch 推出之始，它就已经具备了全文检索功能，数据分析功能以及分布式的数据存储特性。Elasticsearch 这个革命性的项目将这些有用的特性组合成了单个的，连贯的，实时地应用。它的门槛极低，但当你技能有所提升的时候或者需求变得复杂得时候，它依然能够满足你的要求。

既然你在翻阅这本书，那你手头肯定有一些数据，而光有数据是没有意义的，除非你能用它来做些什么。

而不幸的是，大多数数据库引擎都没有分析数据的功能。诚然你可以通过时间戳或者类似的精确的值来进行数据筛查，但你没办法进行全文检索，更处理不了数据中的同义字词，更别提根据数据的相关性来进行数据归档了。它们也没法生成数据的分析或者统计信息。更重要的是，它们没办法在实时地处理大批量的批处理任务。

~~关系数据库的软肋促进了 NoSQL 技术的发展~~

这正是 Elasticsearch 与众不同的地方:Elasticsearch鼓励你去探索和利用数据,而不是让它因为太难被查询而烂在数据库里。

你的好友 Elasticsearch 已上线。

***

# Getting Started

Elasticsearch is a real-time distributed search and analytics engine. It allows you to explore your data at a speed and at a scale never before possible. It is used for full-text search, structured search, analytics, and all three in combination:

  * Wikipedia uses Elasticsearch to provide full-text search with highlighted search snippets, and search-as-you-type and did-you-mean suggestions.
  * The Guardian uses Elasticsearch to combine visitor logs with social -network data to provide real-time feedback to its editors about the public’s response to new articles.
  * Stack Overflow combines full-text search with geolocation queries and uses more-like-this to find related questions and answers.
  * GitHub uses Elasticsearch to query 130 billion lines of code.

But Elasticsearch is not just for mega-corporations. It has enabled many startups like Datadog and Klout to prototype ideas and to turn them into scalable solutions. Elasticsearch can run on your laptop, or scale out to hundreds of servers and petabytes of data.

No individual part of Elasticsearch is new or revolutionary. Full-text search has been done before, as have analytics systems and distributed databases. The revolution is the combination of these individually useful parts into a single, coherent, real-time application. It has a low barrier to entry for the new user, but can keep pace with you as your skills and needs grow.

If you are picking up this book, it is because you have data, and there is no point in having data unless you plan to do something with it.

Unfortunately, most databases are astonishingly inept at extracting actionable knowledge from your data. Sure, they can filter by timestamp or exact values, but can they perform full-text search, handle synonyms, and score documents by relevance? Can they generate analytics and aggregations from the same data? Most important, can they do this in real time without big batch-processing jobs?

This is what sets Elasticsearch apart: Elasticsearch encourages you to explore and utilize your data, rather than letting it rot in a warehouse because it is too difficult to query.

Elasticsearch is your new best friend.