---
title: HashMap的底层实现
categories: 数据开发
tags:
  - Java
  - HashMap
cover: 'https://bu.dusays.com/2023/06/13/648837ab43865.png'
ai: true
abbrlink: ae26a7fb
date: 2023-08-05 17:30:56
---
# HashMap的底层实现
## JDK1.8 之前

JDK1.8 之前 `HashMap` 底层是 **数组和链表** 结合在一起使用也就是 **链表散列**。HashMap 通过 key 的 `hashcode` 经过扰动函数处理过后得到 hash 值,然后通过 `(n - 1) & hash` 判断当前元素存放的位置（这里的 n 指的是数组的长度）,如果当前位置存在元素的话,就判断该元素与要存入的元素的 hash 值以及 key 是否相同,如果相同的话,直接覆盖,不相同就通过拉链法解决冲突

所谓扰动函数指的就是 HashMap 的 `hash` 方法

使用 `hash` 方法也就是扰动函数是为了防止一些实现比较差的 `hashCode()` 方法 换句话说使用扰动函数之后可以减少碰撞

**JDK 1.7 HashMap 的 hash 方法源码:**

```java
static int hash(int h) {
    // This function ensures that hashCodes that differ only by
    // constant multiples at each bit position have a bounded
    // number of collisions (approximately 8 at default load factor).

    h ^= (h >>> 20) ^ (h >>> 12);
    return h ^ (h >>> 7) ^ (h >>> 4);
}
```

所谓 **“拉链法”** 就是: 将链表和数组相结合,创建一个链表数组,数组中每一格就是一个链表,如果遇到哈希冲突,则将冲突的值加到链表中即可

<img src="https://bu.dusays.com/2023/08/08/64d20ccf6c5ae.png" alt="Snipaste_2023-08-08_17-36-47.png" style="zoom:50%;" />

## JDK1.8 之后

JDK1.8 之后在解决哈希冲突时有了较大的变化,当链表长度大于阈值(默认为 8)时,将链表转化为红黑树

将链表转换成红黑树前会判断,如果当前数组的长度小于 64,那么会选择先进行数组扩容,而不是转换为红黑树以减少搜索时间

<img src="https://bu.dusays.com/2023/08/08/64d20e07e1d0d.png" alt="Snipaste_2023-08-08_17-42-19.png" style="zoom:50%;" />

> TreeMap、TreeSet 以及 JDK1.8 之后的 HashMap 底层都用到了红黑树
>
> 红黑树就是为了解决二叉查找树的缺陷,因为二叉查找树在某些情况下会退化成一个线性结构



## HashMap 的长度为什么是 2 的幂次方

为了能让 HashMap 存取高效,尽量较少碰撞,也就是要尽量把数据分配均匀

Hash 值的范围值-2147483648 到 2147483647,前后加起来大概 40 亿的映射空间,只要哈希函数映射得比较均匀松散,一般应用是很难出现碰撞的

但问题是一个 40 亿长度的数组,内存是放不下的

所以这个散列值在用之前还要先做对数组的长度取模运算,得到的余数才能用来要存放的位置也就是对应的数组下标

这个数组下标的计算方法是“ `(n - 1) & hash`”(n 代表数组长度)

这也就解释了 HashMap 的长度为什么是 2 的幂次方