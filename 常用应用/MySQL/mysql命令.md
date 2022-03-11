### group by
假设我有下表T：
|a|b|
|-|-|
|1|abc|
|2|def|
|3|ghi|
|4|jkl|
|5|mno|
|6|pqr|

做以下查询
```
select a, b from T group by a
```
输出应该有两行，一行a=1，一行a=2<br>
但是，b的值应该显示在这两行中的每一行上？每种情况都有三种可能性，查询中没有任何内容可以清楚地说明每个组中为b选择哪个值。这是模棱两可的。

这演示了单值规则，它禁止运行GROUP BY查询时得到的未定义结果，并且您在select-list中包含既不属于分组条件的任何列，也不会出现在聚合函数中(SUM， MIN，MAX等)。

修复它可能如下所示：
```
SELECT a, MAX(b) AS x
FROM T
GROUP BY a
```