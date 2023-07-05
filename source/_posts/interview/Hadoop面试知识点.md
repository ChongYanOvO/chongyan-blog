---
title: Hadoop面试知识点
categories: 面试
tags:
  - Hadoop
  - HDFS
  - MapReduce
  - Yarn
cover: 'https://bu.dusays.com/2023/06/13/648837ab43865.png'
abbrlink: 19903d30
date: 2023-06-14 13:46:13
---
## 常用端口号

|       常用端口号       |  2.x  |  3.x  |
| :--------------------: | :---: | :---: |
|     访问HDFS端口号     | 50070 | 9870 |
|     NN内部通信端口     | 9000 | 8020 |
| 访问MR执行任务情况端口 | 8088 | 8088 |
|    Yarn内部通信端口    | 8032 | 8032 |
|   访问历史服务器端口   | 19888 | 19888 |
|   历史服务器内部端口   | 10020 | 10020 |

## 常用配置文件

|  常用配置文件  |                                        作用                                        |
| :-------------: | :---------------------------------------------------------------------------------: |
|  core-site.xml  |       配置 Hadoop 的基本属性,例如HDFS的默认文件系统、I/O 和记录日志等设置。       |
|  hdfs-site.xml  |   配置 HDFS 的属性,例如数据块大小、副本数量、名字节点和数据节点的地址、缓存等。   |
|  yarn-site.xml  | 配置 YARN 的属性,例如资源管理器和节点管理器的地址、内存和 CPU 的分配、日志聚合等。 |
| mapred-site.xml |   配置MapReduce的属性,例如作业跟踪器和任务跟踪器的地址、作业优先级、输出压缩等。   |
|  hadoop-env.sh  |              配置 Hadoop 的环境变量,例如 JAVA_HOME、HADOOP_HOME 等。              |

## HDFS的构成
> 元数据:目录结构和块的位置信息
> 元数据存放在内存中,默认情况下,每个文件的元数据大概有150B字节

- **NameNode**: 负责管理元数据
- **DataNode**: 负责存储实际数据
- **SecondaryNameNode**: 辅助NameNode对元数据的管理

## NameNode概述
- NameNode是HDFS的核心,也被称为Master
- 仅存储HDFS的元数据:目录结构和文件的块列表及其位置信息
- 不存储实际数据或数据集。数据本身实际存储在DataNode中
- 知道HDFS中任何给定文件的块列表及其位置。使用此信息NameNode知道如何从块中构建文件
- 并不持久化存储每个文件中各个块所在的DataNode的位置信息,这些信息会在系统启动时从数据节点重建
- 对于HDFS至关重要,当NameNode关闭时,HDFS/Hadoop集群无法访问
- 是Hadoop集群中的单点故障
- 所在机器通常会配置有大量内存

## DataNode概述
- 负责将实际数据存储在HDFS中,也被称为Slave
- 启动时,它将自己发布到NameNode并汇报自己负责持有的块列表
- 因为实际数据存储在DataNode中,所以其机器通常配置有大量的硬盘空间
- 会定期(dfs.heartbeat.interval配置项配置,默认是3秒)向NameNode发送心跳,如果NameNode长时间没有接受到DataNode发送的心跳, NameNode就会认为该DataNode失效（10分钟 + 30s）
- block(块)汇报时间间隔取参数dfs.blockreport.intervalMsec,参数未配置的话默认为6小时


## NameNode和DataNode对比
`NameNode` => HDFS核心组件 => 负责管理整个HDFS集群
    不保存具体数据,主要保存元数据 => 放置在内存,所以在配置时需要大量的内存

`DataNode` => HDFS组件 => 负责具体数据/数据集存储,需要占用大量磁盘空间,某个机器故障并不影响整个集群的使用,datanode需要每个3s发送一次心跳信息。
    datanode需要每隔3s发送一次心跳信息
    datanode启动时会自动向namenode汇报1次本节点的块文件信息
    datanode实现数据冗余存储（副本机制）


## HDFS五大机制
1. 切片机制
> HDFS中的文件在物理上是分块（block）存储的,块的大小可以通过配置参数来规定,在hadoop2.x版本中默认大小是128M

2. 汇报机制
{% note no-icon %}
1. HDFS集群重新启动的时候,所有的DataNode都要向NameNode汇报自己的块信息
2. 当集群在正常工作的时,间隔一定时间（6小时）后DataNode也要向NameNode汇报一次自己的块信息
{% endnote %}

3. 心跳检测机制
{% note no-icon %}
    NameNode与DataNode依靠心跳检测机制进行通信
1. DataNode每3秒给NameNode发送自己的心跳信息
2. 如果NameNode没有收到心跳信息,则认为DataNode进入“假死”状态。DataNode在此阶段还会再尝试发送10次（30s）心跳信息
3. 如果NameNode超过最大间隙时间（10分钟）还未接收到DataNode的信息,则认为该DataNode进入“宕机”状态
4. 当检测到某个DataNode宕机后,NameNode会将该DataNode存储的所有数据重新找台活跃的新机器做备份
{% endnote %}

4. 负载均衡
> 让集群中所有的节点（服务器）的利用率和副本数尽量都保持一致或在同一个水平线上

5. 副本机制 
{% note no-icon %}
1. 副本的默认数量为3
2. 当某个块的副本小于3份时,NameNode会新增副本
3. 当某个块的副本大于3份时,NameNode会删除副本
4. 当某个块的副本数小于3份且无法新增的时候,此时集群会强制进入安全模式（只能读,不能写）
{% endnote %}


## 副本存储策略
<img src="https://bu.dusays.com/2023/06/14/6489785b76db3.png"/>
**通过机架感知原理 + 网络拓扑结构实现副本摆放**
- 第1个副本: 优先本机存放,否则就近随机
- 第2个副本: 放在与第1个副本就近不同机架上的某一个服务器
- 第3个副本: 与第2个副本相同机架的不同服务器。
- 如果还有更多的副本: 随机放在各机架的服务器中。


## HDFS的读写流程
### 写操作
<img src="https://bu.dusays.com/2023/06/14/64899447b1874.png"/>
1. 客户端向NameNode请求上传文件
2. NameNode检查是否有上传权限以及文件是否存在
3. NameNode告诉客户端可以上传文件
4. 客户端接收可以上传的信息后,对文件进行切块
5. 客户端重新请求NameNode,询问第一个数据块的上传位置
6. NameNode接收到客户端的请求后,根据副本机制、负载均衡、机架感知原理和网络拓补图,找到存储第一个数据块的DataNode列表
7. NameNode返回存储数据的DataNode节点列表
8. 根据NameNode返回的DataNode节点列表,各个DataNode节点之间建立传输管道,通过数据报包的方式建立`ACK确认机制`
9. 节点接收到数据块后,需要告知客户端块信息已上传成功,这里是依次反馈给上一级,也称为`反向应答机制`
10. 第一个数据块上传完成后,客户端继续请求NameNode询问第二个数据块的上传位置,重复上面的操作,直至所有的数据块上传成功