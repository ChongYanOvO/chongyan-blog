---
title: Scala偏函数
categories: 数据开发
tags:
  - Scala
  - 偏函数
cover: 'https://bu.dusays.com/2023/06/13/64880f6b7c235.png'
abbrlink: bff5a009
date: 2023-06-13 17:19:36
---

## 偏函数(partial function)

**这里通过一个例子来了解一下什么是偏函数**

**需求:将一个 Any 类型的列表中的数字+1，非数字忽略**

```scala
val list: List[Any] = List(1, 2, 3, 4, "hello")

list
// 过滤 Int 类型之外的类型
.filter(_.isInstanceOf[Int])
// 将 Any 类型转为 Int 类型
.map(_.asInstanceOf[Int])
// 数字+1
.map(_ + 1)
```

**这里虽然解决了问题，但是太过于麻烦了**

**这里还可以通过模式匹配的方法去解决**

```scala
list.map {
  case x: Int => x + 1
  case _ =>
}
```

**虽然使用模式匹配比较简洁，但还是不够完美，这里我们就要通过偏函数去达到更加简洁方便的去解决这个问题**

### **那么什么是偏函数呢?**

1. 在对`符合某个条件`时，而不是所有情况进行逻辑操作时，使用偏函数是个不错的选择
2. 将包括在大括号内的一组 case 语句封装为函数，称之为`偏函数`，它只会对作用于指定类型的参数或者指定范围值的参数实施计算，超出范围的值会忽略(或者指定处理方式)
3. 偏函数在 Scala 中是一个特质`PartialFunction`

```scala
// 需求:将一个 Any 类型的列表中的数字+1,非数字忽略
val list: List[Any] = List(1, 2, 3, 4, "hello")

// 定义一个偏函数
val partialFunction: PartialFunction[Any, Int] = new PartialFunction[Any, Int] {
  override def isDefinedAt(x: Any): Boolean = x.isInstanceOf[Int]

  override def apply(v1: Any): Int = {
    v1.asInstanceOf[Int] + 1
  }
}

// 使用偏函数，使用偏函数不能使用 map,得使用 collect
val res: List[Int] = list.collect(partialFunction)
```

**说明:**

1. `PartialFunction[Any,Int]`中`Any`表示偏函数接收的是 Any 类型的参数，`Int`表示偏函数返回类型是 Int 
2. `isDefinedAt`表示如果返回 true,就会去调用 `apply` 方法去构建对象实例，如果是 false 则过滤
3. `apply` 方法相当于构造器，在这里对传入的值+1,并返回新的集合

**偏函数执行流程:**

1. 遍历 list 所有元素
2. 然后调用偏函数中的`isDefinedAt`方法，如果为 true 则调用`apply`方法，得到list 中每一个被偏函数处理的值
3. 每得到一个值，就放入到新的集合中，最后返回一个集合

### 偏函数的简化方法

在声明偏函数的时候，需要重写`trait`中方法，有时候会很麻烦，其实 Scala 提供了简单的方法

#### 简化方法 1:

case 语句可以通过隐式转换转为偏函数

```scala
def partialFunction(): PartialFunction[Any, Int] = {
  case x: Int => x + 1
}
```

#### 简化方法 2:

```scala
list.collect{case x: Int => x + 1}
```