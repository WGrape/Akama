---
layout:     post
title:      【译】解释 UNION 和 OR
subtitle:   UNION & OR
date:       2018-12-03
author:     Lvsi
header-img: 
catalog: true
tags:
    - SQL
---

## 【译】解释 UNION 和 OR

> 原文 [《Union and OR and the Explanation》](https://leetcode.com/problems/big-countries/discuss/103561/Union-and-OR-and-the-Explanation)<br/>
> 译者：[Lvsi](https://github.com/Lvsi-China)

> LeetCode : 595. 大的国家<br>
> 如果一个国家的面积超过300万平方公里，或者人口超过2500万，那么这个国家就是大国家。<br>
> 编写一个SQL查询，输出表中所有大国家的名称、人口和面积。

两个明显的解题方法:

```sql
#OR
SELECT name, population, area
FROM World
WHERE area > 3000000 OR population > 25000000
```

使用更快的 UNION

```
#Union
SELECT name, population, area
FROM World
WHERE area > 3000000 

UNION

SELECT name, population, area
FROM World
WHERE population > 25000000
```

所以为什么 ```UNION``` 比 ```OR``` 更快呢？

严格的说，对于像扫描两个不同列的这种情况，使用 ```UNION``` 会更快。
(当然使用 ```UNION ALL``` 比 ```UNION``` 要快得多，因为我们不需要对结果排序，但是它违反了很多要求。 )

假设我们正在搜索```population``` 和 ```area```。鉴于MySQL通常在给定查询的中每个表使用一个索引，所以当它使用第一个索引而不是第二个索引时，它仍然需要进行全表扫描才能找到适合第二个索引的行。

使用```UNION```时，每个子查询都可以使用其查询的索引，然后使用```UNION```把子查询组合起来。

下面我引用一个关于```UNION```和```OR```的测试标准，以便随时查看：

```
Scenario 3: Selecting all columns for different fields
            CPU      Reads        Duration       Row Counts
OR           47       1278           443           1228
UNION        31       1334           400           1228

Scenario 4: Selecting Clustered index columns for different fields
            CPU      Reads        Duration       Row Counts
OR           0         319           366           1228
UNION        0          50           193           1228
```

## Comments

```UNION```并不总是比```OR```的速度快!

大多数优秀的关系型数据库系统会在SELECT语句执行之前，使用内置的查询优化器来组合SELECT语句。 从理论上讲，这意味着从性能角度分析，使用多个```WHERE```子句条件或```UNION```之间应该没有真正的区别。 我只是从理论上讲，因为在实践中，大多数查询优化器的实际表现并没有理论上的优异。所以最好的方法是测试两种方法，看看哪种方法最适合你。



