# 聚合

前几章节我们聊的一直都是“搜索”。在搜索的时候，我们首先有一个查询条件，然后用这个查询条件从所有的数据中检索出一个符合该条件的某条数据，或某几条数据。听上去有点像谚语大海捞针，对不？

而所谓聚合，则是不再深究某条或某几条符合条件的数据子集，而是通过一个全局的角度去审视我们的整个数据。有时候我们不仅仅需要对数据进行检索，找出我们所想要的某些数据，我们可能还想对我们所拥有的所有数据进行全局的分析或总结操作，比如：

* 我们可能想要知道我们的大海里究竟有几根针？而并不需要真的找出那几根针。
* 这些针的平均长度是多少？
* 这些针长度的中位数是多少？制造商是哪家？
* 每个月又有多少个针被扔进大海？

聚合功能还能处理更精细的问题：

* 最受亲睐的制造商是哪家？
* 有没有什么形状怪异的针？

聚合功能使得我们能够对我们的数据进行更进一步的处理。尽管这些数据分析功能和搜索功能有着本质的区别，但因为使用的是同一套数据结构。所以聚合操作和查询操作一样高效且近乎实时。

这项功能尤其适合于生成报告或制作图表。你不再需要汇总你的数据了，你可以实时地通过可视化的手段直观得看到你的数据，使得你能够实时地做出响应。此时你的数据与你的报表不在是不关联的两个实体而是变得相互关联，一旦数据发生改变，报表也立刻会反映出变化，永远不会失去时效性，也省去了重复地数据计算统计工作。

最后要说明的是，聚合操作总是伴随着请求操作。这也就意味着，你可以在一个请求内就搜索/过滤文档并同时对结果集进行分析。也由于你的聚合操作以及计算都是基于搜索/过滤的结果的，所以，举例，如果你是在显示一系列四星酒店，其实你并没有显示所有的四星酒店，而是显示的符合搜索/过滤条件的四星酒店。

由此可见，聚合功能其实是个非常强大的功能，也因此许多公司仅仅出于数据分析的需要就搭建了大型的　Elasticsearch 集群。

# Aggregations

Until this point, this book has been dedicated to search. With search, we have a query and we want to find a subset of documents that match the query. We are looking for the proverbial needle(s) in the haystack.

With aggregations, we zoom out to get an overview of our data. Instead of looking for individual documents, we want to analyze and summarize our complete set of data:

* How many needles are in the haystack?
* What is the average length of the needles?
* What is the median length of the needles, broken down by manufacturer?
* How many needles were added to the haystack each month?

Aggregations can answer more subtle questions too:

* What are your most popular needle manufacturers?
* Are there any unusual or anomalous clumps of needles?

Aggregations allow us to ask sophisticated questions of our data. And yet, while the functionality is completely different from search, it leverages the same data-structures. This means aggregations execute quickly and are near real-time, just like search.

This is extremely powerful for reporting and dashboards. Instead of performing rollups of your data (that crusty Hadoop job that takes a week to run), you can visualize your data in real time, allowing you to respond immediately. Your report changes as your data changes, rather than being pre-calculated, out of date and irrelevant.

Finally, aggregations operate alongside search requests. This means you can both search/filter documents and perform analytics at the same time, on the same data, in a single request. And because aggregations are calculated in the context of a user’s search, you’re not just displaying a count of four-star hotels—you’re displaying a count of four-star hotels that match their search criteria.

Aggregations are so powerful that many companies have built large Elasticsearch clusters solely for analytics.