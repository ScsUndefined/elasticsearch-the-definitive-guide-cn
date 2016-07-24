# 序

现在已经是信息时代了。长久以来，我们一直淹没在流经或由我们的软件系统产生的海量数据中。而现有的技术则一直专注于如何结构化地存储数据。这种做法本身并没有什么问题，当如果你想要实时地对海量数据进行分析以辅助你决策的话，问题就出现了。

而 Elasticsearch 则是一个分布式的，可拓展的，实时的搜索及分析引擎。它能让你拥有检索，分析，挖掘数据的能力，而这些往往是你在最初设计项目时所没有预想要的功能。之所以会有 Elasticsearch 这个项目，就是为了提高硬盘中的原始数据的可用性。

不管你是想要开发一个全文检索的功能，还是一个实时分析结构化数据的功能，或者两者皆有，这本书都能够传授你一些基本的、不要的概念，以便你能把 Elasticsearch 的基础特性运用起来。当你把这些基础概念掌握透彻并想要进一步丰富你的搜索功能以满足你的客户需求的时候，这本书也将传递给你一些进阶知识。

Elasticsearch 可不仅仅只是一个处理全文检索的工程。我们将要讲述的知识还会涉及到结构化数据的检索，数据分析，如果让搜索引擎理解人类语言，如果对地理位置进行搜索，以及如何处理关系(relationships)等~~还不清楚这边说的关系是指什么~~。我们也会谈及如何对你的数据进行建模，以便能够充分利用起 Elasticsearch 的横向拓展能力，以及当你在生产环境使用 Elasticsearch　时该如何配置及监控你的集群。

***

# Preface

The world is swimming in data. For years we have been simply overwhelmed by the quantity of data flowing through and produced by our systems. Existing technology has focused on how to store and structure warehouses full of data. That’s all well and good—until you actually need to make decisions in real time informed by that data.

Elasticsearch is a distributed, scalable, real-time search and analytics engine. It enables you to search, analyze, and explore your data, often in ways that you did not anticipate at the start of a project. It exists because raw data sitting on a hard drive is just not useful.

Whether you need full-text search, real-time analytics of structured data, or a combination of the two, this book introduces you to the fundamental concepts required to start working with Elasticsearch at a basic level. With these foundations laid, it will move on to more-advanced search techniques, which you will need to shape the search experience to fit your requirements.

Elasticsearch is not just about full-text search. We explain structured search, analytics, the complexities of dealing with human language, geolocation, and relationships. We will also discuss how best to model your data to take advantage of the horizontal scalability of Elasticsearch, and how to configure and monitor your cluster when moving to production.