---
title: Hadoop面试知识点
categories: 面试
tags:
  - Hadoop
  - HDFS
  - MapReduce
  - Yarn
cover: "https://bu.dusays.com/2023/06/13/648837ab43865.png"
abbrlink: 19903d30
date: 2023-06-14 13:46:13
---

## 常用端口号

|        常用端口号        |  2.x  |  3.x  |
| :----------------------: | :---: | :---: |
|     访问 HDFS 端口号     | 50070 | 9870  |
|     NN 内部通信端口      | 9000  | 8020  |
| 访问 MR 执行任务情况端口 | 8088  | 8088  |
|    Yarn 内部通信端口     | 8032  | 8032  |
|    访问历史服务器端口    | 19888 | 19888 |
|    历史服务器内部端口    | 10020 | 10020 |

## 常用配置文件

|  常用配置文件   |                                        作用                                        |
| :-------------: | :--------------------------------------------------------------------------------: |
|  core-site.xml  |      配置 Hadoop 的基本属性,例如 HDFS 的默认文件系统、I/O 和记录日志等设置。       |
|  hdfs-site.xml  |   配置 HDFS 的属性,例如数据块大小、副本数量、名字节点和数据节点的地址、缓存等。    |
|  yarn-site.xml  | 配置 YARN 的属性,例如资源管理器和节点管理器的地址、内存和 CPU 的分配、日志聚合等。 |
| mapred-site.xml |  配置 MapReduce 的属性,例如作业跟踪器和任务跟踪器的地址、作业优先级、输出压缩等。  |
|  hadoop-env.sh  |              配置 Hadoop 的环境变量,例如 JAVA_HOME、HADOOP_HOME 等。               |

## HDFS 的构成

> 元数据:目录结构和块的位置信息
> 元数据存放在内存中,默认情况下,每个文件的元数据大概有 150B 字节

- **NameNode**: 负责管理元数据
- **DataNode**: 负责存储实际数据
- **SecondaryNameNode**: 辅助 NameNode 对元数据的管理

## NameNode 概述

- NameNode 是 HDFS 的核心,也被称为 Master
- 仅存储 HDFS 的元数据:目录结构和文件的块列表及其位置信息
- 不存储实际数据或数据集。数据本身实际存储在 DataNode 中
- 知道 HDFS 中任何给定文件的块列表及其位置。使用此信息 NameNode 知道如何从块中构建文件
- 并不持久化存储每个文件中各个块所在的 DataNode 的位置信息,这些信息会在系统启动时从数据节点重建
- 对于 HDFS 至关重要,当 NameNode 关闭时,HDFS/Hadoop 集群无法访问
- 是 Hadoop 集群中的单点故障
- 所在机器通常会配置有大量内存

## DataNode 概述

- 负责将实际数据存储在 HDFS 中,也被称为 Slave
- 启动时,它将自己发布到 NameNode 并汇报自己负责持有的块列表
- 因为实际数据存储在 DataNode 中,所以其机器通常配置有大量的硬盘空间
- 会定期(dfs.heartbeat.interval 配置项配置,默认是 3 秒)向 NameNode 发送心跳,如果 NameNode 长时间没有接受到 DataNode 发送的心跳, NameNode 就会认为该 DataNode 失效（10 分钟 + 30s）
- block(块)汇报时间间隔取参数 dfs.blockreport.intervalMsec,参数未配置的话默认为 6 小时

## NameNode 和 DataNode 对比

{% label NameNode orange %} => HDFS 核心组件 => 负责管理整个 HDFS 集群
不保存具体数据,主要保存元数据 => 放置在内存,所以在配置时需要大量的内存

{% label DataNode blue %} => HDFS 组件 => 负责具体数据/数据集存储,需要占用大量磁盘空间,某个机器故障并不影响整个集群的使用,datanode 需要每个 3s 发送一次心跳信息。
datanode 需要每隔 3s 发送一次心跳信息
datanode 启动时会自动向 namenode 汇报 1 次本节点的块文件信息
datanode 实现数据冗余存储（副本机制）

## HDFS 五大机制

1. 切片机制

   > HDFS 中的文件在物理上是分块（block）存储的,块的大小可以通过配置参数来规定,在 hadoop2.x 版本中默认大小是 128M

2. 汇报机制
   {% note no-icon %}
3. HDFS 集群重新启动的时候,所有的 DataNode 都要向 NameNode 汇报自己的块信息
4. 当集群在正常工作的时,间隔一定时间（6 小时）后 DataNode 也要向 NameNode 汇报一次自己的块信息
   {% endnote %}

5. 心跳检测机制
   {% note no-icon %}
   NameNode 与 DataNode 依靠心跳检测机制进行通信
6. DataNode 每 3 秒给 NameNode 发送自己的心跳信息
7. 如果 NameNode 没有收到心跳信息,则认为 DataNode 进入“假死”状态。DataNode 在此阶段还会再尝试发送 10 次（30s）心跳信息
8. 如果 NameNode 超过最大间隙时间（10 分钟）还未接收到 DataNode 的信息,则认为该 DataNode 进入“宕机”状态
9. 当检测到某个 DataNode 宕机后,NameNode 会将该 DataNode 存储的所有数据重新找台活跃的新机器做备份
   {% endnote %}

10. 负载均衡

    > 让集群中所有的节点（服务器）的利用率和副本数尽量都保持一致或在同一个水平线上

11. 副本机制
    {% note no-icon %}
12. 副本的默认数量为 3
13. 当某个块的副本小于 3 份时,NameNode 会新增副本
14. 当某个块的副本大于 3 份时,NameNode 会删除副本
15. 当某个块的副本数小于 3 份且无法新增的时候,此时集群会强制进入安全模式（只能读,不能写）
    {% endnote %}

## 副本存储策略

<img src="https://bu.dusays.com/2023/06/14/6489785b76db3.png"/>
**通过机架感知原理 + 网络拓扑结构实现副本摆放**
- 第1个副本: 优先本机存放,否则就近随机
- 第2个副本: 放在与第1个副本就近不同机架上的某一个服务器
- 第3个副本: 与第2个副本相同机架的不同服务器。
- 如果还有更多的副本: 随机放在各机架的服务器中。

## HDFS 的读写流程

### 写操作

{% image https://bu.dusays.com/2023/06/14/64899447b1874.png, alt=HDFS 写文件流程 %}

1. 客户端向{% label NameNode orange %}请求上传文件
2. {% label NameNode orange %}检查是否有上传权限以及文件是否存在
3. {% label NameNode orange %}告诉客户端可以上传文件
4. 客户端接收可以上传的信息后,对文件进行切块
5. 客户端重新请求{% label NameNode orange %},询问第一个数据块的上传位置
6. {% label NameNode orange %}接收到客户端的请求后,根据副本机制、负载均衡、机架感知原理和网络拓补图,找到存储第一个数据块的 DataNode 列表
7. {% label NameNode orange %}返回存储数据的{% label DataNode blue %}节点列表
8. 根据{% label NameNode orange %}返回的{% label DataNode blue %}节点列表,各个{% label DataNode blue %}节点之间建立传输管道,通过数据报包的方式建立`ACK确认机制`
9. 节点接收到数据块后,需要告知客户端块信息已上传成功,这里是依次反馈给上一级,也称为`反向应答机制`
10. 第一个数据块上传完成后,客户端继续请求{% label NameNode orange %}询问第二个数据块的上传位置,重复上面的操作,直至所有的数据块上传成功

### 读操作

{% image https://bu.dusays.com/2023/07/05/64a52bb90d5a6.png, alt=HDFS 读文件流程 %}

1. 客户端向{% label NameNode orange %}发起 RPC 请求读取文件
2. {% label NameNode orange %}检查是否有上传权限以及文件是否存在
3. {% label NameNode orange %}根据副本存储机制找到{% label DataNode blue %}节点列表
4. {% label NameNode orange %}向 {% label HDFSClient orange %} 返回
5. 根据{% label NameNode orange %}返回的{% label DataNode blue %}节点列表,各个{% label DataNode blue %}节点之间建立并行传输管道,开始读取数据
6. 传输数据,并行读取 {% label Block green %} 信息,最终读取来所有的 {% label Block green %} 会合并成一个完整的最终文件


## HDFS如何保证数据不丢失
1. 通过数据块的校验和(`checksum`)来检验数据块的完整性
2. 多副本机制
3. {% label DataNode blue %}会周期性的报告{% label Block green %}信息,{% label DataNode blue %}会默认每小时把自己节点上的所有块状态信息报告给{% label NameNode orange %}
4. `safemode模式`, {% label DataNode blue %}上报块信息后,{% label NameNode orange %}会计算{% label Block green %}的损坏率,当阀值 <0.999f 时系统会进入安全模式,HDFS只读不写。

