---
title: ScalaActor并发编程模型
categories: 数据开发
tags:
  - Scala
  - Actor
  - 并发编程
cover: 'https://bu.dusays.com/2023/06/13/64880f6b8fc43.png'
abbrlink: 2502113a
date: 2023-06-13 17:16:13
---

## Scala Actor 并发编程模型

​	`Actor`并发编程模型，是 Scala 提供的一直与 Java 完全不一样的并发编程模型，是一直基于事件模型的并发机制。`Actor`并发编程模型是一种不共享数据，依赖消息传递的并发编程模型，有效避免了资源争夺、死锁等现象。

`Actor`是一种基于事件(消息)的并发编程模型，不共享数据，有效避免了共享数据加锁问题。

### Java并发编程对比 Actor 并发编程模型

| Java并发编程                                            |           `Actor` 并发编程模型           |
| ------------------------------------------------------- | :--------------------------------------: |
| 共享数据锁模型(share data and lock)                     |              share nothing               |
| 每个 object 都有一个`monitor`，用来监视对共享数据的访问 | 不共享数据，`Actor`直接通过`Message`通讯 |
| 加锁代码使用`synchronized`标识                          |                                          |
| 死锁问题                                                |       每个`Actor`内部是顺序执行的        |
| 每个线程内部是顺序执行的                                |       每个`Actor`内部是顺序执行的        |

Scala 在 2.11 及之后的版本中加入了`Akka`并发编程框架，`Actor`并发编程模型已经被废弃了。

### 创建 Actor

可以通过类(Class)或者单例对象(Object)继承`Actor`特质的方式来创建`Actor`对象

#### 通过类实现创建`Actor`对象

````scala
  class myActor1 extends Actor {
    override def act(): Unit = {
      for (i <- 1 to 10) println("myActor1---" + i)
    }
  }

  class myActor2 extends Actor {
    override def act(): Unit = {
      for (i <- 11 to 20) println("myActor2---" + i)
    }
  }

  def main(args: Array[String]): Unit = {
    val myActor1: myActor1 = new myActor1()
    myActor1.start()
    new myActor2().start()
  }
````

通过单例对象实现创建`Actor`对象

```scala
  object myActor1 extends Actor {
    override def act(): Unit = {
      for (i <- 1 to 10) println("myActor1---" + i)
    }
  }

  object myActor2 extends Actor {
    override def act(): Unit = {
      for (i <- 11 to 20) println("myActor2---" + i)
    }
  }

  def main(args: Array[String]): Unit = {
    myActor1.start()
    myActor2.start()
  }
```

### 发送以及接收消息

#### 发送消息

1. `!`：发送异步消息，没有返回值
2. `!?`：发送同步消息，等待返回值
3. `!!`：发送异步消息，返回值是 `Future[Any]`

如下给`myActor1`发送一个异步字符串消息

```scala
myActor1 ! "你好"
```

#### 接收消息

`Actor`中使用`receive` 方法来接收消息，需要传入一个偏函数
`receive` 方法值接收一次消息，接收完成后进行执行`act()`方法

```scala
  object ActorSender extends Actor {
    override def act(): Unit = {
      ActorReceiver ! "你好"
    }
  }

  object ActorReceiver extends Actor {
    override def act(): Unit = {
      receive {
        case msg: String => println(msg)
      }
    }
  }

  def main(args: Array[String]): Unit = {
    ActorSender.start()
    ActorReceiver.start()
  }
```

#### 持续发送和接收消息

用 `while`循环来持续不断的发送和接收消息

```scala
  object ActorSender extends Actor {
    override def act(): Unit = {
      while (true){
        ActorReceiver ! "你好"
        Thread sleep 1000
      }
    }
  }

  object ActorReceiver extends Actor {
    override def act(): Unit = {
      while (true){
        receive {
          case msg: String => println(msg)
        }
      }
    }
  }

  def main(args: Array[String]): Unit = {
    ActorSender.start()
    ActorReceiver.start()
  }
```

**问题**：

1. 如果当前`Actor`没有接收到消息，线程就会处于阻塞状态。如果很多的`Actor`，就会导致很多线程处于阻塞状态
2. 每次有新的消息进来，都会重新创建新的线程来处理。这种频繁的线程创建、销毁和切换会影响影响效率

**解决办法**：通过`loop()`结合`react()`来复用多线程

```scala
  object ActorSender extends Actor {
    override def act(): Unit = {
      loop {
        ActorReceiver ! "你好"
        Thread sleep 1000
      }
    }
  }

  object ActorReceiver extends Actor {
    override def act(): Unit = {
      loop {
        react {
          case msg: String => println(msg)
        }
      }
    }
  }

  def main(args: Array[String]): Unit = {
    ActorSender.start()
    ActorReceiver.start()
  }
```

#### 发送和接收自定义消息

在此之前我们发送的消息都是字符串类型的，显然这样并不常见，因此我们需要能够自定义发送的消息类型。例如可以用`样例类`封装消息，然后进行发送处理

##### 发送接收同步有返回消息

> **使用`!?`**

```scala
  case class Message(id: Int, message: String)

  case class ReplyMessage(message: String, name: String)

  object MessageActor extends Actor {
    override def act(): Unit = {
      loop {
        react {
          case Message(id: Int, message: String) =>
            println(s"id = $id,message = $message ")
            sender ! ReplyMessage("你也好"，"MessageActor")
        }
      }
    }
  }

  def main(args: Array[String]): Unit = {
    MessageActor.start()
    val res: Any = MessageActor !? Message(1, "hello")
    val replyMessage: ReplyMessage = res.asInstanceOf[ReplyMessage]
    println(s"MainActor接收到MessageActor返回的消息是: ${replyMessage.message} and ${replyMessage.name}")
  }
```

##### 发送异步无返回消息

> **使用`!`**

```scala
 object MessageActor extends Actor {
    override def act(): Unit = {
      loop {
        react {
          case Message(id: Int, message: String) =>
            println(s"id = $id,message = $message ")
        }
      }
    }
  }

  def main(args: Array[String]): Unit = {
    MessageActor.start()
    MessageActor ! Message(1, "hello")
  }
```

##### 发送接收异步有返回消息

> **使用`!!`**

```scala
 object MessageActor extends Actor {
    override def act(): Unit = {
      loop {
        react {
          case Message(id: Int, message: String) =>
            println(s"id = $id,message = $message ")
            sender ! ReplyMessage("你也好"， "MessageActor")
        }
        }
      }
    }
  }

  def main(args: Array[String]): Unit = {
    MessageActor.start()
    MessageActor !! Message(1, "hello")
    println("继续执行...")
    val replyMessage: ReplyMessage = res.asInstanceOf[ReplyMessage]
      println(s"MainActor接收到MessageActor返回的消息是: ${replyMessage.message} from ${replyMessage.name}")
    println("MainActor接收成功...")
  }
```

##### 同步消息和异步消息区别

`同步消息`：必须接收到回复信息，程序才会继续执行
`异步消息`：即使没有接收到回复信息，程序也会继续执行

### Actor 实现 WordCount 案例

```scala
package com.chongyan.wordcount

import java.io.File
import scala.actors.{Actor, Future}
import scala.io.Source


object MainActor {
  def main(args: Array[String]): Unit = {
    // 1. 获取所有要统计的文件的路径
    // 1.1 定义变量 dir,记录保存的所有文件的文件夹路径
    var dir = "./data/"
    // 1.2 获取文件夹下所有的文件名
    var fileNameList = new File(dir).list().toList
    //println(fileNameList)
    // 1.3 对获取到的文件名进行拼接
    val fileDirList: List[String] = fileNameList.map(dir + _)
    //println(fileDirList)

    // 2. 根据文件数量创建对应个数的 WordCountActor 对象
    // 2.1 先创建 WordCountActor 类，用来获取 WordCountActor 对象
    case class WordCountTask(fileName: String)
    case class WordCountResult(WordCountList: List[(String, Int)])
    class WordCountActor extends Actor {
      override def act(): Unit = {
        loop {
          // 3.4 接收具体任务
          react {
            case WordCountTask(fileName) =>
            // 3.5 打印具体任务
            println(s"获取到的任务是 $fileName")
            // 4. 统计接收到的文件中的每个单词的数量
            // 4.1 获取指定文件中的所有单词
            val linesList: List[String] = Source.fromFile(fileName).getLines().toList
            // 4.2 将上述获取的数据，转换成一个个的单词
            val wordsList: List[String] = linesList.flatMap(_.split(" "))
            // 4.3 给每个字符串(单词)后面都加上次数，默认为 1
            val wordsTimesList: List[(String, Int)] = wordsList.map((_, 1))
            // 4.4 安按照字符串内容(单词本身的值)进行分组
            val wordsCountList: Map[String, List[(String, Int)]] = wordsTimesList.groupBy(_._1)
            // 4.5 对分组后的内容进行排序，统计每个单词的总数量
            val WordCountList: List[(String, Int)] = wordsCountList.map {
              wordsCountMap =>
              (wordsCountMap._1, wordsCountMap._2.map(_._2).sum)
            }.toList
            // 4.6 打印统计后的结果
            println(WordCountList)

            // 5. 将统计后的结果返回给 MainActor
            // 5.1 返回具体的值
            sender ! WordCountResult(WordCountList)
          }
        }
      }
    }
    // 2.2 根据文件数量创建对应个数的 WordCountActor 对象
    val wordCountActorsList: List[WordCountActor] = fileNameList.map(_ => new WordCountActor)
    // 2.3 将 WordCountActor 对象和文件的全路径关联在一起
    val actorWithFile: List[(WordCountActor, String)] = wordCountActorsList.zip(fileDirList)
    //println(actorWithFile)

    // 3. 启动所有 WordCountActor 对象，并发送单词统计任务消息给每个 WordCountActor 对象
    val fatureList: List[Future[Any]] = actorWithFile.map {
      actorAndFile =>
      // 3.1 获取启动具体 WordCountActor 对象
      val actor: WordCountActor = actorAndFile._1
      val fileName: String = actorAndFile._2
      // 3.2 启动具体 WordCountActor 对象
      actor.start()
      // 3.3 给每个 WordCountActor 发送具体任务(文件路径)，异步有返回
      val fature: Future[Any] = actor !! WordCountTask(fileName)
      fature
    }
    // 5.2 判断所有的 fature 是否都是返回值，如果都是返回值，则继续往下执行
    while (fatureList.exists(!_.isSet)) {}
    // 5.3 从每个 fature 中获取数据
    val WordCountLists: List[List[(String, Int)]] = fatureList
    .map(_.apply()
         .asInstanceOf[WordCountResult]
         .WordCountList)

    val WordCountList: List[(String, Int)] = WordCountLists.flatten.groupBy(_._1).map {
      wordsCountMap =>
      (wordsCountMap._1, wordsCountMap._2.map(_._2).sum)
    }.toList
    // 6. 统计并打印
    println(WordCountList)
  }
}
```

## Scala Akka 并发编程框架

### 什么是 `Akka`?

`Akka`是一个用于构建高并发、分布式和可扩展的基于事件驱动的应用程序工具包。`Akka`是使用 Scala 开发的库，可以支持 Scala 和 Java 语言来开发基于 `Akka` 的应用程序。

### `Akka`的特性

- 通过基于异步非阻塞、高性能的事件驱动编程模型
- 内置容错机制，是循序`Actor`在出错是进行恢复或者重置操作
- 超级轻量级的事件处理(每 GB 对内存可以运行几百万`Actor`)
- 使用`Akka`可以在单机上构建高并发程序，也可以在网络中构建分布式程序

### `Akka`通讯过程

<img src="http://chongyan-blog.test.upcdn.net/md-images/image-20220723201001929.png" alt="image-20220723201001929" style="zoom:50%;" />

1. 学生创建一个`ActorSystem`
2. 通过`ActorSystem`来创建一个`ActorRef`（老师的引用），并将消息发送给`ActorRef`
3. `ActorRef`将消息发送给`Message Dispatcher`（消息分发器）
4. `Message Dispatcher`将消息按照顺序保存到目标Actor的MailBox中
5. `Message Dispatcher`将`MailBox`放到一个线程中
6. `MailBox`按照顺序取出消息，最终将它递给`TeacherActor`接受的方法中

### 创建`Actor`

​	`Akka`中，也是基于`Actor`来进行编程的。类似于`Actor`。但是`Akka`中的`Actor`的编写、创建方法和之前有一些不一样。

#### API介绍

- `ActorSystem`: 它负责创建和监督`Actor`

> 1. 在`Akka`中，`ActorSystem`是一个重量级的结构，它需要分配多个线程
> 1. 在实际应用中， `ActorSystem`通常是一个单例对象， 可以使用它创建很多`Actor`
> 1. 直接使用`context.system`就可以获取到管理该`Actor`的`ActorSystem`的引用

- 实现`Actor`类

> 1. 定义类或者单例对象继承`Actor`**（注意：要导入akka.actor包下的Actor）**
> 2. 实现`receive`方法，`receive`方法中**直接处理消息**即可，不需要添加`loop`和`react`方法调用. Akka会自动调用receive来接收消息
> 3. 还可以实现`preStart()`方法， 该方法在`Actor`对象构建后执行，在`Actor`生命周期中仅执行一次.

- 加载`Actor`

> 1. 要创建Akka的`Actor`，必须要先获取创建一个`ActorSystem`。需要给`ActorSystem`指定一个名称，并可以去加载一些配置项
> 2.  调用`ActorSystem.actorOf(Props(Actor对象)， "Actor名字")`来加载`Actor`

#### Actor Path

每一个`Actor`都有一个`Path`，这个路径可以被外部引用。路径的格式如下：

| **Actor类型** | **路径**                                     | **示例**                                     |
| ------------- | -------------------------------------------- | -------------------------------------------- |
| 本地Actor     | akka://actorSystem名称/user/Actor名称        | akka://SimpleAkkaDemo/user/senderActor       |
| 远程Actor     | akka.tcp://my-sys@ip地址:port/user/Actor名称 | akka.tcp://192.168.10.17:5678/user/service-b |

#### 创建实例

1. 定义`SenderActor`类
2. 定义`ReceiverActor`类
3. 定义`Entrance`主运行类

```scala
import akka.actor.Actor

/**
 * 在 Actor 编程模型中:实现 act() 方法，如果想持续接收消息，需要通过 loop() + react() 组合方式实现
 * 在 Akka 编程模型中:实现 receive() 方法，直接在该方法中提供偏函数来处理数据即可
 */
object SenderActor extends Actor {
  override def receive: Receive = {
    case x => println(x)
  }
}
```

```scala
import akka.actor.Actor

object ReceiverActor extends Actor {
  override def receive: Receive = {
    case x => println(x)
  }
}

```

```scala
object Entrance {
  def main(args: Array[String]): Unit = {
    val actorSystem: ActorSystem = ActorSystem("actorSystem"， ConfigFactory.load())

    val senderActor: ActorRef = actorSystem.actorOf(Props(SenderActor)， "senderActor")
    val receiverActor: ActorRef = actorSystem.actorOf(Props(ReceiverActor)， "receiverActor")
  }
}
```

### 发送和接收消息

1. 使用样例类封装消息

- `SubmitTaskMessage`提交任务消息
- `SuccessSubmitTaskMessage`任务提交成功消息

2. 使用`!`发送消息

`SenderActor.scala`

```scala
object SenderActor extends Actor {
  override def receive: Receive = {
    // 1. 接收 Entrance 发送过来的消息
    case "start" =>
      // 2. 获取 ReceiverActor 的路径
      val receiverActorSelection: ActorSelection = context.actorSelection("akka://actorSystem/user/receiverActor")
      // 3. 给 ReceiverActor 返回消息，用样例类封装
      receiverActorSelection ! SubmitTaskMessage("我是 SenderActor ，我在给你发消息")
    // 4. 接收 ReceiverActor 返回的消息
    case SuccessSubmitTaskMessage(msg) =>
      println(s"SenderActor 接收到的消息是，$msg")
  }
}
```

`ReceiverActor.scala`

```scala
object ReceiverActor extends Actor {
  override def receive: Receive = {
    case SubmitTaskMessage(msg) =>
      println(s"我是 ReceiverActor ，我接收到的消息是:$msg")
      sender ! SuccessSubmitTaskMessage("我是 ReceiverActor ，接收成功")
  }
}
```

`Entrance.scala`

```scala
object Entrance {
  def main(args: Array[String]): Unit = {
    val actorSystem: ActorSystem = ActorSystem("actorSystem"， ConfigFactory.load())
    val senderActor: ActorRef = actorSystem.actorOf(Props(SenderActor)， "senderActor")
    val receiverActor: ActorRef = actorSystem.actorOf(Props(ReceiverActor)， "receiverActor")
    senderActor ! "start"
  }
}
```

### `Akka`定时任务

通过 `ActorSystem.scheduler.schedule()方法`， 启动定时任务

- 方式一：采用`发送消息`方式实现

```scala
final def schedule(
  initialDelay : FiniteDuration, // 首次开始， 按此设定的时间， 延迟后执行
  interval : FiniteDuration, // 每隔多久执行一次(首次开始， 立马执行， 不延时)
  receiver : ActorRef, // 设置目标接收消息的 Actor
  message : Any) // 要发送的消息
(implicit executor : ExecutionContext, sender : ActorRef = {}) // 隐式参数， 需导入
```

- 方式二：采用`自定义消息`方式实现

```scala
final def schedule(
	initialDelay : FiniteDuration, // 首次开始， 按此设定的时间， 延迟后执行
	interval : FiniteDuration // 每隔多久执行一次(首次开始， 立马执行， 不延时
)(f : => Unit) // 定期要执行的函数(消息
(implicit executor : ExecutionContext) // 隐式参数， 需导入
```

**具体实现代码：**

```scala
object MainActor {
  object ReceiverActor extends Actor {
    override def receive: Receive = {
      case msg => println(msg)
    }
  }

  def main(args: Array[String]): Unit = {
    // 创建ActorSystem, 用来负责创建和监督 Actor
    val actorSystem: ActorSystem = ActorSystem("actorSystem"， ConfigFactory.load())
    // 通过 ActorSystem来加载自定义 Actor对象
    val receiverActor: ActorRef = actorSystem.actorOf(Props(ReceiverActor)， "receiverActor")

    // 导入隐式参数和隐式转换
    import actorSystem.dispatcher
    import scala.concurrent.duration._

    // 通过定时器，定时给 ReceiverActor 发送消息
    // 方式 1: 采用提供的 Any 数据类型参数的消息
    actorSystem.scheduler.schedule(0 seconds,
      2 seconds,
      receiverActor,
      "Hello ReceiverActor!， 方式 1...")

    // 方式 2: 采用自定义函数的消息
    actorSystem.scheduler.schedule(0 seconds, 2 seconds) {
      receiverActor ! "Hello ReceiverActor!， 方式 2..."
    }
  }
}
```

### 实现两个进程间的通信

基于`Akka`实现两个**进程**之间发送、接收消息。

1. `WorkerActor`启动后去连接`MasterActor`，并发送消息给`MasterActor`
2. `WorkerActor`在接收到消息后，再回复消息给`MasterActor`

<img src="http://chongyan-blog.test.upcdn.net/md-images/image-20220724053601951.png" alt="image-20220724051541509" style="zoom:50%;" />

##### `MasterActor.scala`

```scala
package com.chongyan.masterAndWorker.master
import akka.actor.Actor
object MasterActor extends Actor {
  override def receive: Receive = {
    case "setup" => println("MasterActor started!")
    // 接收 WorkerActor发的消息
    case "connect" =>
      println("MasterActor, received: connect!")
      // 给发送者(WorkerActor)返回的回执信息
      sender ! "success"
  }
}
```

##### `MasterEntrance.scala`

```scala
package com.chongyan.masterAndWorker.master
import akka.actor.{ActorRef, ActorSystem, Props}
import com.typesafe.config.ConfigFactory
object MasterEntrance {
  def main(args: Array[String]): Unit = {
    val actorSystem: ActorSystem = ActorSystem("actorSystem"， ConfigFactory.load())
    val masterActor: ActorRef = actorSystem.actorOf(Props(MasterActor)， "masterActor")
    // 给 MasterActor发送消息
    masterActor ! "setup"
  }
}
```

##### `WorkerActor.scala`

```scala
package com.chongyan.masterAndWorker.worker
import akka.actor.{Actor, ActorSelection}
object WorkerActor extends Actor {
  override def receive: Receive = {
    case "setup" =>
      println("WorkerActor started!")
      // 远程获取 MasterActor
      val masterActor: ActorSelection = context
        .system
        .actorSelection("akka.tcp://actorSystem@127.0.0.1:8888/user/masterActor")
      // 给 MasterActor发送字符串 connect
      masterActor ! "connect"
    // 接收 MasterActor发的消息
    case "success" => println("MasterActor, received: success!")
  }
}
```

`WorkerEntrance.scala`

```scala
package com.chongyan.masterAndWorker.worker
import akka.actor.{ActorRef, ActorSystem, Props}
import com.typesafe.config.ConfigFactory
object WorkerEntrance {
  def main(args: Array[String]): Unit = {
    val actorSystem: ActorSystem = ActorSystem("actorSystem"， ConfigFactory.load())
    val workerActor: ActorRef = actorSystem.actorOf(Props(WorkerActor)， "workerActor")
    // 给 WorkerActor发送消息
    workerActor ! "setup"
  }
}
```
