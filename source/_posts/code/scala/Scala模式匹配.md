---
title: Scala模式匹配
categories: 数据开发
tags:
  - Scala
  - 模式匹配
cover: 'https://bu.dusays.com/2023/06/13/64880f6b9632e.png'
abbrlink: 4a6243a2
date: 2023-06-13 17:20:29
---

## Scala 模式匹配

### 模式匹配简单示例

```scala
val oper: Char = '+'
val n1: Int = 20
val n2: Int = 10
var res: Int = 0

oper match {
  case '+' => res = n1 + n2
  case '-' => res = n1 - n2
  case '*' => res = n1 * n2
  case '/' => res = n1 / n2
  case _ => println("oper error")
}

println("res:\t" + res)
```

**说明:**

	1. match 和 case 是 scala 中模式匹配的关键字
	1. 如果匹配成功，执行 => 后的代码块
	1. 匹配顺序是从上到下的，匹配到就执行对应的代码
	1. => 还没的代码块不需要写 break ，会自动退出 match
	1. 如果都没有匹配到，则会执行 case _ 后的代码

### 条件守卫

**如果想要表达`匹配某个范围的数据`，就需要在模式匹配中增加条件守卫**

```scala
    val flag: String = "mysql"

    flag match {
      case _ if flag.equals("mysql") => println("mysql")
      case _ if flag.equals("kafka") => println("kafka")
      case _ if flag.equals("redis") => println("redis")
    }
```

### 模式变量

如果在 case 关键字后跟变量，那么 match 表达式的值会赋给那个变量

```scala
val flag: String = "kafka"

flag match {
  case value => println("value:" + value)
  case _ if flag.equals("mysql") => println("mysql")
  case _ if flag.equals("kafka") => println("kafka")
  case _ if flag.equals("redis") => println("redis")
}
```

### 模式匹配返回值

在模式匹配中,match 是一个表达式，因此可以拥有返回值

```scala
val flag: String = "kafka"

val res: String = flag match {
  case _ if flag.equals("mysql") => "mysql"
  case _ if flag.equals("kafka") => "kafka"
  case _ if flag.equals("redis") => "redis"
}
```

### 类型匹配

可以匹配对象任意类型，避免使用 `isInstanceOf` 和 `asInstanceOf` 方法

```scala
val obj: Any = ""

obj match {
  case int: Int => println("Int")
  case string: String => println("String")
  case map: Map[String,String] => println("Map[String,String]")
  ...
}
```

**注意:**

	1. `Map[String,String]` 和 `Map[String,Int]`是两种不同类型
	2. 在进行类型匹配时，编译器会`预先检测是否有可能的匹配`，如果没有则会报错
	3. `case int: Int => println("Int")`表示将 `int=obj`，然后再判断类型
	4. 如果`case _`出现在 match 中间，则表示隐藏变量名，而`不是表示默认匹配`

```scala
val obj: Any = ""

obj match {
  case _: Int => println("Int")
  case _: String => println("String")
  case _: Map[String,String] => println("Map[String,String]")
}
```

### 匹配数组

1. `Array(0)`只匹配一个元素且为 0 的数组
2. `Array(x,y)`匹配数组有两个元素，并`将两个元素赋值为 x 和 y`。可以依次类推`Array(x,y,z)...`
3. `Array(0,_*)`匹配数组以 0 开始

```scala
val array: Array[Array[Int]] = Array(Array(0)， Array(1, 0)， Array(0, 1, 0)， Array(1, 1, 0)， Array(1, 1, 0, 1))

for (arr <- array) {
  val res: String = arr match {
    case Array(0) => "0"
    case Array(x, y) => x + "-" + y
    case Array(0, _*) => "以 0 开头的数组"
    case _ => "什么数组都不是"
  }
  println("res = " + res)
}
```

### 匹配列表

```scala
val array: Array[List[Int]] = Array(List(0)， List(1, 0)， List(0, 0, 0)， List(1, 0, 0))
for (list <- array) {
  val res: String = list match {
    case 0 :: Nil => "0"
    case x :: y :: Nil => x + "-" + y
    case 0 :: tail => "0..."
    case _ => "SomeThings else"
  }
  println("res = " + res)
}
```

### 元组匹配

```scala
val array: Array[Product] = Array((0, 1)， (1, 0)， (1, 1)， (1, 0, 2))
for (pair <- array) {
  val res: Any = pair match {
    case (0, _) => "0..."
    case (x, _) => x
    case _ => "others"
  }
  println(res)
}
```

### 对象匹配

1. case 中对象的`unapply`方法(提取器)返回 `some` 集合则为匹配成功
2. 返回 `none` 集合则为匹配失败

```scala
val number: Double = 36.0
number match {
  case Square(n) => println(n)
  case _ => println("nothing matched")
}

object Square {
  def unapply(z: Double): Option[Double] = Some(math.sqrt(z))

  def apply(z: Double): Double = z * z
}
```

**执行顺序:**

1. 在构建对象时`apply`会被调用
2. `case Square(n)` 语句会默认调用`unapply`方法`(对象提取器)`
3. `number`会被作为参数传递给`def unapply(z: Double)`中的 z 形参
4. 如果返回的是`Some`集合，则`unapply提取器`返回的结果会返回个`n`这个形参

### 变量声明中的模式

match 中的每一个 case 都可以单独提取出来

1. x = 1，y = 2
2. q = BigInt(10) / 3，r = BigInt(10) % 3
3.  first = 1，second = 2

```scala
val (x, y): (Int, Int) = (1, 2)
val (q, r): (BigInt, BigInt) = BigInt(10) /% 3
val array: Array[Int] = Array(1, 2, 3, 4, 5)
val Array(first, second, _*) = array
```

### for表达式中的模式

for 循环中也能进行模式匹配

```scala
val map: Map[String, Int] = Map("A" -> 1, "B" -> 2, "C" -> 3)
for ((k, v) <- map) {
  println(k + "->" + v)
}

for ((k, 0) <- map) {
  println(k)
}

for ((k, v) <- map if k.equals("A")) {
  println(k + "->" + v)
}
```

### 样例类

```scala
abstract class Amount
case class Dollar(value:Double) extends Amount
case class Currency(value:Double,unit:String) extends Amount
case object NoAmount extends Amount
```

**说明:**

1. 样例类仍然是类
2. 样例类用`case 关键字进行声明`  
3. 样例类是为`模式匹配而优化`的类
4. 构造器中的每一个参数都成为 val (`除非被显式的声明为 var`)
5. 在样例类对应的伴生对象中`提供 apply 方法`，可以不使用 new关键字就能构造出相应的对象
6. `提供 unapply 方法`让模式匹配可以工作
7. 将`自动生成 toString、equals、hashcode 和 copy方法`
8. 除此之外，样例类和其他类型完全一致，仍可以添加字段和方法扩展他们

### 匹配嵌套结果

**类似于正则表达式**

 ```scala
 val sale: Bundle = Bundle("书籍"， 10, Book("漫画"， 40)， Bundle("文学作品"， 20, Book("阳关"， 80)， Book("围城"， 30)))
 
 val res1: String = sale match {
   case Bundle(_, _, Book(desc, _)， _*) => desc
 }
 
 val res2: Any = sale match {
   case Bundle(_, _, book@Book(_, _)， rest@_*) => (book, rest)
 }
 
 val res3: Any = sale match {
   case Bundle(_, _, book@Book(_, _)， rest) => (book, rest)
 }
 
 abstract class Item
 case class Book(desc: String, price: Double) extends Item
 case class Bundle(desc: String, discount: Double, item: Item*) extends Item
 
 ```

**说明:**

1. 通过签到匹配获取到书籍的 desc
2. 通过@表示法将嵌套的值绑定到变量， _* 绑定剩余 Iterm 到 rest
3. 如果剩余内容只有一个时，可以不用写，直接通过 rest 变量获取剩余内容

### 密封类

1. 如果想让 case 类的所有子类都必须在声明该类的`相同源文件中定义`,可以将样例类通过超类声明为`sealed`,这个超类称之为密封类
2. 密封就是不能在其他文件中定义子类