---
title: Ubuntu搭建Hadoop伪分布式
categories: 数据开发
tags:
  - Ubuntu
  - Hadoop
cover: 'https://bu.dusays.com/2023/06/13/648837aac4b4d.png'
abbrlink: efa88e6a
date: 2023-06-13 17:35:07
---

> 基于 Ububtu Server 20.04.1 LTS 版本
> Hadoop 3.1.3 版本

## 一、重新设置主机名

```Shell
hostnamectl set-hostname localhost
bash
```

## 二、关闭防火墙

### 防火墙常用命令

> Ubuntu安装防火墙 `sudo apt-get install ufw -y`

  1. 查看防火墙开启状态 `sudo ufw status`

  2. 开启某个端口(以8866为例) `sudo ufw allow 8866`

  3. 开启防火墙 `sudo ufw enable`

  4. 关闭防火墙 `sudo ufw disable`

  5. 重启防火墙 `sudo ufw reload`

  6. 禁止某个端口(以8866为例) `sudo ufw delete allow 8866`

  7. 查看端口IP `netstat -ltn`

### 2.1 关闭防火墙

```Shell
ufw disable
```

### 2.2 检查防火墙是否关闭

```Shell
ufw status
```

## 三、设置IP映射

### 2.1  主节点配置 hosts 文件

```Shell
vim /etc/hosts
```

### 3.2 添加ip地址和主机名

```vim
10.211.55.60 localhost
```

### 3.3 测试IP映射配置

```Shell
ping localhost
```

## 四、配置免密登录

### 4.1 生成两个文件，一个公钥（id_rsa.pub），一个私钥（id_rsa）

```Shell
ssh-keygen -t rsa
```

### 4.2 将公匙上传到主节点

**注意:在每台机器上都要输入**

```Shell
ssh-copy-id localhost
```

### 4.3 分发 authorized_keys 文件

```Shell
scp ~/.ssh/authorized_keys root@localhost:~/.ssh/
```

### 4.4 测试免密登录

```Shell
ssh localhost
```

## 五、安装 JDK

### 5.1 解压JDK安装包

```Shell
tar -zxvf jdk-8u212-linux-x64.tar.gz -C /usr/local/src/
```

### 5.2 移动并重命名JDK包

```Shell
mv /usr/local/src/jdk1.8.0_212 /usr/local/src/java
```

### 5.3 配置Java环境变量

```Shell
vim /etc/profile
```

```vim
# JAVA_HOME
export JAVA_HOME=/usr/local/src/java
export PATH=$PATH:$JAVA_HOME/bin
export JRE_HOME=/usr/local/src/java/jre
export CLASSPATH=.:$CLASSPATH:$JAVA_HOME/lib:$JRE_HOME/lib
```

```Shell
source /etc/profile
```

### 5.4 查看Java是否成功安装

```Shell
java -version
```

![image.png](https://chongyandocs-1304373775.cos.ap-nanjing.myqcloud.com/chongyandocs/java-success.png)

## 六、安装Hadoop

### 6.1 解压Hadoop安装包

```Shell
tar -zxvf hadoop-3.1.3.tar.gz -C /usr/local/src/
```

### 6.2 移动并重命名Hadoop包

```Shell
mv /usr/local/src/hadoop-3.1.3 /usr/local/src/hadoop
```

### 6.3 配置`Hadoop`环境变量

```Shell
vim /etc/profile
```

```vim
# HADOOP_HOME
export HADOOP_HOME=/usr/local/src/hadoop/
export PATH=$PATH:$HADOOP_HOME/bin
export PATH=$PATH:$HADOOP_HOME/sbin
```

```Shell
source /etc/profile
```

### 6.4 修改配置文件`yarn-env.sh`和`hadoop-env.sh`

#### 6.4.1 修改`yarn-env.sh`

```Shell
vim /usr/local/src/hadoop/etc/hadoop/yarn-env.sh
```

```Shell
export JAVA_HOME=/usr/local/src/java
```

#### 6.4.2 修改hadoop-env.sh

```Shell
vim /usr/local/src/hadoop/etc/hadoop/hadoop-env.sh
```

```Shell
export JAVA_HOME=/usr/local/src/java
export HDFS_NAMENODE_USER=root
export HDFS_DATANODE_USER=root
export HDFS_SECONDARYNAMENODE_USER=root
export YARN_RESOURCEMANAGER_USER=root
export YARN_NODEMANAGER_USER=root
export HDFS_JOURNALNODE_USER=root
export HDFS_ZKFC_USER=root
export HADOOP_SHELL_EXECNAME=root
```

### 6.5 测试Hadoop是否安装成功

```Shell
hadoop version
```

![image.png](https://chongyandocs-1304373775.cos.ap-nanjing.myqcloud.com/chongyandocs/hadoop-success.png)

## 七、配置Hadoop

> Hadoop集群节点规划

`常用端口号`

HDFS NameNode 网页查看端口: [http://master:9870](http://master:9870)

Yarn网页查看端口: [http://master:8088/cluster](http://master:8088/cluster)

历史服务器网页查看端口:[http://slave2:19888/jobhistory](http://slave2:19888/jobhistory)

### 7.1 修改core-site.xml

```Shell
vim /usr/local/src/hadoop/etc/hadoop/core-site.xml
```

```XML
<configuration>
        <property>
               <name>fs.defaultFS</name>
               <value>hdfs://localhost:9000</value>
        </property>
        <property>
               <name>hadoop.tmp.dir</name>
               <value>/usr/local/src/hadoop/tmp</value>
        </property>
</configuration>
```

### 7.2 修改 hdfs-site.xml

```Shell
vim /usr/local/src/hadoop/etc/hadoop/hdfs-site.xml
```

```XML
<configuration>
        <property>
                <name>dfs.namenode.name.dir</name>
                <value>/usr/local/src/hadoop/tmp/hdfs/name</value>
        </property>
        <property>
                <name>dfs.datanode.data.dir</name>
                <value>/usr/local/src/hadoop/tmp/hdfs/data</value>
        </property>
        <property>
                <name>dfs.replication</name>
                <value>1</value>
        </property>
</configuration>
```

### 7.3 修改 yarn-site.xml

```Shell
vim /usr/local/src/hadoop/etc/hadoop/yarn-site.xml
```

```XML
<configuration>
      <property>         
                <name>yarn.resourcemanager.hostname</name>
                <value>localhost</value>
      </property>

      <property>
                <name>yarn.nodemanager.aux-services</name>
                <value>mapreduce_shuffle</value>
      </property>
</configuration>
```

### 7.4 修改 mapred-site.xml

```Shell
vim /usr/local/src/hadoop/etc/hadoop/mapred-site.xml
```

```XML
<configuration>
     <property>
         <name>mapreduce.framework.name</name>
         <value>yarn</value>
     </property>
</configuration>
```

### 7.5 修改 workers

> `注意:` 在 Hadoop3.0 以上的版本,使用的是 workers 配置文件,而在 Hadoop3.0 以下,使用的是 slaves 配置文件

```Shell
vim /usr/local/src/hadoop/etc/hadoop/workers
```

```vim
localhost
```

## 八、格式化及启动 Hadoop

### 8.1 格式化namenode

```Shell
hdfs namenode -format
```

### 8.2 启动并查看jps

```Shell
start-all.sh && jps
```
