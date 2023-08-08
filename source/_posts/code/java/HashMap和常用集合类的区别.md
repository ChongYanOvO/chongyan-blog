---
title: HashMap和常用集合类的区别
categories: 数据开发
tags:
  - Java
  - HashMap
  - HashTable
  - HashSet
  - treeMap
  - 集合类
cover: 'https://bu.dusays.com/2023/06/13/64880f6b4a20d.png'
abbrlink: 8f303c09
date: 2023-08-03 17:00:53
ai: true
---
# HashMap和常用集合类的区别

## HashMap 和 HashTable 的区别
- 线程是否安全: HashMap 是非线程安全的,HashTable 是线程安全的,因为 HashTable 内部的方法基本都经过synchronized 修饰。(如果要保证线程安全的话推荐使用 ConcurrentHashMap )
- 效率: 因为线程安全的问题,HashMap 要比 HashTable 效率高一点,HashTable 基本被淘汰,尽量不要在代码中使用它
- 对 Null key 和 Null value 的支持: HashMap 可以存储 null 的 key 和 value,但 null 作为键只能有一个,null 作为值可以有多个,HashTable 不允许有 null 键和 null 值,否则会抛出`NullPointerException`
- 初始容量大小和每次扩充容量大小的不同: 
    1. 创建时如果不指定容量初始值,HashTable 默认的初始大小为 11,之后每次扩充,容量变为原来的 2n + 1,HashMap 默认的初始化大小为 16。之后每次扩充,容量变为原来的 2 倍
    2. 创建时如果给定了容量初始值,那么 HashTable 会直接使用你给定的大小,而 HashMap 会将其扩充为 2 的幂次方大小,也就是说 HashMap 使用 2 的幂作为哈希表的大小
- 底层数据结构: JDK1.8 以后的 HashMap 在解决哈希冲突时有了较大的变化,当链表长度大于阈值(默认为 8)时,将链表转化为红黑树（将链表转换成红黑树前会判断,如果当前数组的长度小于 64,那么会选择先进行数组扩容,而不是转换为红黑树）,以减少搜索时间,HashTable 没有这样的机制

------


## HashMap 和 HashSet 区别
HashSet 底层就是基于 HashMap 实现的

|             **HashMap**              |                         **HashSet**                          |
| :----------------------------------: | :----------------------------------------------------------: |
|          实现了 `Map` 接口           |                       实现 `Set` 接口                        |
|              存储键值对              |                          仅存储对象                          |
|   调用 `put()`向 `Map` 中添加元素    |             调用 `add()`方法向 `Set` 中添加元素              |
| HashMap` 使用键（Key）计算 `hashcode | `HashSet` 使用成员对象来计算 `hashcode` 值，对于两个对象来说 `hashcode` 可能相同，所以`equals()`方法用来判断对象的相等性 |

------

## HashMap 和 TreeMap 区别

`TreeMap` 和`HashMap` 都继承自`AbstractMap` ,但是需要注意的是`TreeMap`还实现了`NavigableMap`接口和`SortedMap` 接口

- 实现 `NavigableMap` 接口让 `TreeMap` 有了对集合内元素的搜索的能力

- 实现`SortedMap`接口让 `TreeMap` 有了对集合中的元素根据键排序的能力

相比于`HashMap`来说 `TreeMap` 主要多了对集合中的元素根据键排序的能力以及对集合内元素的搜索的能力
