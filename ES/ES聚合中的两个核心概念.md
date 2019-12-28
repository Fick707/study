# ES聚合中的两个核心概念

桶和指标是ES聚合中的两个核心概念。
桶：满足某个条件的文档集合。
指标：为某个桶中的文档计算得到的统计信息。
就是这样！每个聚合只是简单地由一个或者多个桶，零个或者多个指标组合而成。
可以将它粗略地转换为SQL：

```
SELECT COUNT(color) 
FROM table
GROUP BY color
```

以上的COUNT(color)就相当于一个指标。GROUP BY color则相当于一个桶。桶和SQL中的组(Grouping)拥有相似的概念，而指标则与COUNT()，SUM()，MAX()等相似。

让我们仔细看看这些概念。

## 桶（buckets）
```满足某个条件的文档集合。```



## 指标（Metrics）
```为某个桶中的文档计算得到的统计信息。```