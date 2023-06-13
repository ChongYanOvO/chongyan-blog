---
title: Scala泛型
categories: 数据开发
tags:
  - Scala
  - 泛型
cover: 'https://bu.dusays.com/2023/06/13/64880f6b82d0c.png'
abbrlink: 962cd171
date: 2023-06-13 17:17:17
---

## Scala泛型

泛型的意思是 泛指某种具体的数据类型，在 Scala 中泛型用 `[数据类型]` 表示。

### 泛型方法

通过`getMiddleElement`方法获取任意数据类型的数组中的中间元素

```scala
def getMiddleElement[T](array: Array[T]): T = {
  array(array.length / 2)
}
```

### 泛型类

定义一个` Pair `泛型类，这个类中包含两个字段且字段类型不固定

```scala
class Pair[T](var x: T, var y: T)
```

### 泛型特质

`Loggers`是一个泛型特质，`ConsoleLogger`继承了`Loggers`，并重写了其 x 字段和 show 方法

```scala
trait Logger[T] {
  val x: T

  def show(y: T)
}

object ConsoleLogger extends Logger[String] {
  override val x: String = "log"

  override def show(y: String): Unit = println(y)
}
```

### 泛型上下界

#### 泛型上界

使用 `T <: 类型名` 表示给类型添加一个上界，表示该类型必须是 T 或 T 的子类。

#### 泛型下界

使用 `T >: 类型名` 表示给类型添加一个下界，表示该类型必须是 T 或 T 的父类。

#### 泛型上下界

如果泛型既有上界又有下界，下界写在前面，`[T >: A <: B]`

### 协变、逆变、非变

- 协变：类 A 和 类 B 之间是父子类关系，Pair[A] 和 Pari[B] 之间也有父子关系。

```scala
class Pair[+T]{}
```

- 逆变：类 A 和 类 B 之间是父子类关系，但 Pair[A] 和 Pari[B] 之间是子父关系。

```scala
class Pair[-T]{}
```

- 非变：类 A 和 类 B 之间是父子类关系，Pair[A] 和 Pari[B] 之间没有任何关系。

```scala
class Pair[T]{} //默认类型是非变的
```

