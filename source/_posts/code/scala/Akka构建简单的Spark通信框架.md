---
title: Akka构建简单的Spark通信框架
categories: 数据开发
tags:
  - Scala
  - Akka
  - Spark
cover: 'https://bu.dusays.com/2023/06/13/64880f6b4a20d.png'
ai: true
abbrlink: a2e68d58
date: 2023-06-13 17:14:52
---

## 通过 Akka 构建简单的 Spark 通信框架

**实现思路：**

1. 构建`Master`、`Worker`阶段

- 构建 `Master ActorSystem、 Actor`
- 构建 `Worker ActorSystem、 Actor`

2. `Worker`注册阶段

- `Worker`进行向`Master`注册(将自己的`ID`、`CPU 核数`、`内存大小(MB)`发送给 `Master`)

3. `Worke`定时发送心跳阶段

- `Worke`定期向`Master`发送心跳消息

4. `Master`定时心跳检测阶段

- `Master`定期检测`Worker`心跳,将一些超时的`Worker`移除,并对`Worker`按照内存进行倒序排列

5. 多个`Worker`测试阶段

- 启动多个`Worker`,查看是否能注册成功,并停止某个`Worker`查看是否能正常移除

<img src="https://chongyandocs-1304373775.cos.ap-nanjing.myqcloud.com/chongyandocs/akka-spark.png" alt="akka-spark" style="zoom:50%;" />

