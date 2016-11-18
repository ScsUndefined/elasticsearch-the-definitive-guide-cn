
# High-Level Concepts 进阶概念

和 query DSL 一样，聚合也有一系列的复杂语法：一系列互相独立的功能点可以相互组合使用，来满足复杂的数据分析需求。这也意味着你只需要学习几个基础的概念，然而由此衍生出来的组合却无止无尽。

要掌握聚合功能，你只需要理解这两个基础概念：

* Buckets 统计对象
  
  满足某个条件的所有文档
  
* Metrics 统计规则

  被统计的对象需要怎样进行统计

就这么简单！每个聚合操作都是一个或多个统计对象与零或多个统计点的组合。如果用 SQL 表示的话就类似：

```SQL
SELECT COUNT(color) ①
FROM table
GROUP BY color ②
```

①　COUNT(color) 就相当于一个统计规则

②　GROUP BY color 就相当于一个统计对象

统计对象在概念上类似 sql group 后的每个分组数据集，而统计规则就好比 COUNT(), SUM(), MAX() 等。

恩，下一节我们接着深入探讨这两个概念。

---

# High-Level Concepts

Like the query DSL, aggregations have a composable syntax: independent units of functionality can be mixed and matched to provide the custom behavior that you need. This means that there are only a few basic cons  learn, but nearly limitless combinations of those basic components.

To master aggregations, you need to understand only two main concepts:

* Buckets
  
  Collections of documents that meet a criterion
  
* Metrics

  Statistics calculated on the documents in a bucket
  
That’s it! Every aggregation is simply a combination of one or more buckets and zero or more metrics. To translate into rough SQL terms:


```SQL
SELECT COUNT(color) ①
FROM table
GROUP BY color ②
```

①　COUNT(color) is equivalent to a metric.

②　GROUP BY color is equivalent to a bucket.

Buckets are conceptually similar to grouping in SQL, while metrics are similar to COUNT(), SUM(), MAX(), and so forth.

Let’s dig into both of these concepts and see what they entail.