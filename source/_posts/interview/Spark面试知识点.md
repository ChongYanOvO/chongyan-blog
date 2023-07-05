---
title: Spark面试知识点
categories: 面试
tags:
  - Spark
  - Spark-Core
  - Spark-Sql
cover: 'https://bu.dusays.com/2023/07/03/64a2cf90d4401.png'
abbrlink: ecb2a13e
date: 2023-07-03 21:35:57
---

## Spark的宽窄依赖
### 如何划分宽窄依赖
如果子RDD的一个分区完全依赖父RDD的一个或多个分区,则是窄依赖,否则就是宽依赖。这个完全依赖怎么理解呢？其实就是父RDD一个分区的数据是否需要切分,或者说子RDD分区要依赖父RDD分区的全部而不仅仅是一部分。上面这样说相对比较严谨,但也会有特殊情况,比如在只有一个分区的情况下,强行使用repartiton操作,即使父子RDD各自只有一个分区,也是宽依赖。这种情况生产中不会遇到,但要知晓。
Narrow Dependies
{% p center logo large, Volantis %}