---
title: Scala高阶函数
categories: 数据开发
tags:
  - Scala
  - 高阶函数
cover: 'https://bu.dusays.com/2023/06/13/64880f6b92bc1.png'
abbrlink: 7245d322
date: 2023-06-13 17:18:31
---

## 高阶函数(high-order function) 及 函数柯里化

能够接收函数作为参数的函数，叫做高阶函数`(high-order function)`

可以使应用程序更加健壮

```scala
def highFunc(f: Double => Double, num: Double): Unit = {
  f(num)
}

def sum(num: Double): Double = {
  num + num
}

highFunc(sum,6.0)
```

高阶函数可以接收多个函数作为参数

**高阶函数可以返回一个函数**

```scala
def minusxy(x: Int) = {
  // 匿名函数
  (y: Int) => x - y
}

// 输出结果是 8 - 5 = 3
println(minusxy(8)(5))
```

**说明:**

1. `minusxy` 是高阶函数，返回了一个匿名函数
2. 返回的匿名函数 `(y: Int) => x - y`
3. 返回的匿名函数可以使用变量接收

### 闭包(closure)

**闭包就是`一个函数`和与其`相关的引用环境`组合的一个`整体`**

```scala
def minusxy(x: Int) = (y: Int) => x - y

val closureFunc: Int => Int = minusxy(20)
println(closureFunc(5)) // 15
println(closureFunc(10)) // 20
```

1. `(y: Int) => x - y`返回的是一个匿名函数，因为该函数引用到了函数外的 x ，那么该函数和 x 整体形成了一个闭包
2. 用对象的方式理解的话，返回的函数是一个对象，而 x 是该对象的一个字段，它们共同形成了一个闭包
3. 当多次调用闭包时，使用的 x 其实是同一个，所以 x 不变
4. 在使用闭包时，主要看返回函数引用了哪些函数外的变量，因为它们会组合成一个整体，从而形成一个闭包

### 函数柯里化(curry)

1. 在函数编程中，接收`多个参数的函数`都可以转化为接收`单个参数的函数`，这个转化过程就叫做柯里化
2. 柯里化就是证明了函数只需要一个参数而已
3. 柯里化就是以`函数为主体这种思想`发展的必然结果

```scala
def minusxy(x: Int) = (y: Int) => x - y

println(minusxy(8)(5)) // 3
```

### 抽象控制

1. 参数是函数
2. 函数参数没有输入也没有返回值

 ```scala
 def runInThread(func: => Unit): Thread = {
   new Thread {
     override def run(): Unit = {
       func
     }
   }
 }
 
 runInThread {
   println("线程启动")
 }
 ```
