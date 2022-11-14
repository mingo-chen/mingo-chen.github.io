---
title       : "clickhourse调研"
author      : "mingo"
date        : "2021-03-27 14:26:20"

layout      : post
category    : bigdata
tag         : ["OLAP","click_hourse","survey","笔记"]
blog        : true
star        : false
---

## 概述

目前对于数据仓库的要求越来越高, 从之前的T+1逐步变为实时级别; 基于传统的Hadoop+Hive等方式, 数据的处理过程比较慢, 查询也是几十分钟级别, 使用体验很差

基于此, 使用MPP思想的数据库产品逐渐成为实时数仓的新思路, 目前最热门MPP数据库的当属毛子的ClickHouse及百度开源的Doris, 都可以对于10亿级的数据或100GB的数据可以秒级查询, 并且都是列式数据库

本文主要是对ClickHouse做一个简单的入门介绍, 数仓选型的一些理解

## What 

ClickHouse是一个用于联机分析(OLAP)的列式数据库管理系统(DBMS)。

### 一. 列式存储
### 二. [性能数据](https://clickhouse.tech/docs/zh/introduction/performance/)

```
1. 单个大查询的吞吐量

对于简单的查询，速度可以达到30GB／s

2. 处理短查询的延迟时间

查找时间（10 ms） * 查询的列的数量 * 查询的数据块的数量

3. 处理大量短查询的吞吐量

建议每秒最多查询100次

4. 数据的写入性能

建议每次写入不少于1000行的批量写入，或每秒不超过一个写入请求
```

## Why

为什么这么快

（回答来自：https://blog.csdn.net/wenyusuran/article/details/107832079）

1. 着眼硬件，先想后做

基于将硬件功效最大化的目的，ClickHouse会在内存中进行GROUP BY，并且使用HashTable装载数据。与此同时，他们非常在意CPU L3级别的缓存，因为一次L3的缓存失效会带来70～100ns的延迟。这意味着在单核CPU上，它会浪费4000万次/秒的运算；而在一个32线程的CPU上，则可能会浪费5亿次/秒的运算。所以别小看这些细节，一点一滴地将它们累加起来，数据是非常可观的。正因为注意了这些细节，所以ClickHouse在基准查询中能做到1.75亿次/秒的数据扫描性能。

2. 算法在前，抽象在后

性能是ClickHouse算法选择的首要考量指标。

3. 勇于尝鲜，不行就换

如果世面上出现了号称性能强大的新算法，ClickHouse团队会立即将其纳入并进行验证。行就用，不行就不用。

4. 特定场景，特殊优化

ClickHouse会针对不同场景使用最合适、最快的算法。针对同一个场景的不同状况，选择使用不同的实现方式，尽可能将性能最大化。

5. 持续测试，持续改进

由于Yandex的天然优势，ClickHouse经常会使用真实的数据进行测试，这一点很好地保证了测试场景的真实性。ClickHouse差不多每个月都能发布一个版本。

## How 

ClickHouse支持一种基于SQL的声明式查询语言，它在许多情况下与ANSI SQL标准相同。

问题:
1. 如何更新数据

    - 通过ALTER UPDATE/DELETE语义来更新数据
        + 重量级操作([参见ClickHourse Mutations操作原理说明](https://www.jianshu.com/p/521f2d1611f8))
        + 按提交顺序执行, 即使重启ClickHourseServer也会继续执行, 可通过`KILL MUTATION`指令取消
        + 非原子操作
        + 异步执行
        + 不可回退
        + 可以在 `system.mutations` 表查看所有已经提交的mutations操作

    - ReplacingMergeTree表引擎来更新
        + 指定ver列, 只保留最大的

    - 通过MegreTree引擎, Sign+version方式来更新数据

2. 腾讯云上CH的价格, 配置, 时间长度

3. 有无golang API

    目前TRPC上已经封装好API

4. ClickHourse数据类型
5. ClickHourse存储引擎
    
    db引擎: Lazy, Atomic, MySQL
    
    表引擎: 

### 限制 
1. 没有完整的事务支持。
2. 缺少高频率，低延迟的修改或删除已存在数据的能力。仅能用于批量删除或修改数据，但这符合 GDPR。
3. 稀疏索引使得ClickHouse不适合通过其键检索单行的点查询。

### ClickHouse大数据处理架构优点

![ch_bigdata_advantage](/assets/images/clickhouse/ch_bigdata_advantage.jpg)

### ClickHouse与ES,HBase的对比

![ch_vs_es_vs_hbase](/assets/images/clickhouse/ch_es_hbase.jpg)

## 进阶

XX实现原理, 架构设计, 指导理论, 精妙算法

## 其它

- 使用过程中有没有遇到坑
- 使用技巧, 骚操作
- 八卦,趣闻

## 参考
- [clickhourse官方文档](https://clickhouse.tech/docs/zh/getting-started/playground/)
- [通过alter来更新数据](https://www.jianshu.com/p/521f2d1611f8)
