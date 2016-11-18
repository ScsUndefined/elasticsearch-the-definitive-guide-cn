# 统计规则

Buckets 允许我们把文档分组成一个个子集，这还不是我们最终想要的统计信息，我们想要的是每组文档对应某个统计规则统计出来的结果。Bucketing 意义也正于此，它提供分组方式来对文档进行归档，然后你就可以对各个分组的文档进行你感兴趣的统计操作。

大多数的 metrics 统计操作都是用文档的值进行简单的数据运算，诸如求最小值，求最大值，求平均值，求和等。换个更贴近实际的说话的话，metrics 统计操作允许你计算诸如，平均薪资，最高售价或者 the 95th percentile for query latency。

---

# Metrics

Buckets allow us to partition documents into useful subsets, but ultimately what we want is some kind of metric calculated on those documents in each bucket. Bucketing is the means to an end: it provides a way to group documents in a way that you can calculate interesting metrics.

Most metrics are simple mathematical operations (for example, min, mean, max, and sum) that are calculated using the document values. In practical terms, metrics allow you to calculate quantities such as the average salary, or the maximum sale price, or the 95th percentile for query latency.