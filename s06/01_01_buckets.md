# 统计对象

一个统计对象即满足相同条件的同一组数据：

* 一个员工可以被分为男性或女性。
* Albany 城可以被分到 New York 州下。
* 2014年10月28日可以被分入十月份下。

在聚合操作在执行的时候，每个文档的值是否被统计，取决与该文档是否满足分组的条件。如果满足，那么该文档就会被归档入该分组中，然后聚合操作继续执行后续的操作。

分组之间甚至还可以存在嵌套关系，使得你能够构建一个层级或带条件的分组方案。例如，Cincinnati 可以被归入 Ohio 州下，而 Ohio 由可以归入整个美联邦下。

Elasticsearch 提供多种多样的分组，这使得你可以丰富文档的划分规则，比如按照最热的词汇，按照年龄区间，按照地理位置，等等。但其实根本上来讲，这些种类繁多的分组条件的原则都是一致的，即你必须按照某个条件来对文档进行分组归档。

# Buckets

A bucket is simply a collection of documents that meet certain criteria:

* An employee would land in either the male or female bucket.
* The city of Albany would land in the New York state bucket.
* The date 2014-10-28 would land within the October bucket.

As aggregations are executed, the values inside each document are evaluated to determine whether they match a bucket’s criteria. If they match, the document is placed inside the bucket and the aggregation continues.

Buckets can also be nested inside other buckets, giving you a hierarchy or conditional partitioning scheme. For example, Cincinnati would be placed inside the Ohio state bucket, and the entire Ohio bucket would be placed inside the USA country bucket.

Elasticsearch has a variety of buckets, which allow you to partition documents in many ways (by hour, by most-popular terms, by age ranges, by geographical location, and more). But fundamentally they all operate on the same principle: partitioning documents based on criteria.