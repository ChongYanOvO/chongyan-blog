---
title: 线上调试Arthas的学习使用
categories: 运维
tags:
  - Arthas
  - Java
cover: 'https://bu.dusays.com/2023/07/03/64a2cf9417766.png'
abbrlink: '68784837'
date: 2023-07-06 15:19:51
---
{% image https://bu.dusays.com/2023/07/06/64a677ab88f07.png %}

## 什么是Arthas(阿尔萨斯)
Arthas 是一款线上监控诊断产品,通过全局视角实时查看应用 load、内存、gc、线程的状态信息,并能在不修改应用代码的情况下,对业务问题进行诊断,包括查看方法调用的出入参、异常,监测方法执行耗时,类加载信息等,大大提升线上问题排查效率。

## Arthas(阿尔萨斯)的用途
- 查找类加载jar包,类相关的Exception
- 反编译线上jar包,查找与本地代码不一致的地方
- 线上debug
- 全局视角查看系统的运行状况
- 监控JVM的实时运行状态
- 快速定位应用的热点,生成火焰图
- 直接从JVM内查找某个类的实例

## 安装Arthas

{% btns rounded grid5 %}
{% cell 点击下载Arthas, https://arthas.aliyun.com/download/latest_version?mirror=aliyun, anzhiyufont anzhiyu-icon-bolt %}
{% endbtns %}

解压后，在文件夹里有arthas-boot.jar，直接用java -jar的方式启动:
{% image https://bu.dusays.com/2023/07/06/64a6869dbf56e.png %}
```shell
java -jar arthas-boot.jar
```

打印帮助信息:
{% image https://bu.dusays.com/2023/07/06/64a6868648b4a.png %}
```shell
java -jar arthas-boot.jar -h
```

## Arthas基础入门
### 启动 math-game 进行测试
math-game是一个简单的程序，每隔一秒生成一个随机数，再执行质因数分解，并打印出分解结果。

math-game源代码:[查看](https://github.com/alibaba/arthas/blob/master/math-game/src/main/java/demo/MathGame.java)

```shell
java -jar math-game.jar
```
{% image https://bu.dusays.com/2023/07/06/64a68887425b8.png %}


### 启动 arthas

```shell
java -jar arthas-boot.jar
```
{% image https://bu.dusays.com/2023/07/06/64a68bed77fce.png %}

- 执行该程序的用户需要和目标进程具有相同的权限。比如以admin用户来执行:sudo su admin && java -jar arthas-boot.jar 或 sudo -u admin -EH java -jar arthas-boot.jar。
- 如果 attach 不上目标进程，可以查看~/logs/arthas/ 目录下的日志。
- 如果下载速度比较慢，可以使用 aliyun 的镜像:java -jar arthas-boot.jar --repo-mirror aliyun --use-http
- java -jar arthas-boot.jar -h 打印更多参数信息。

### 查看 dashboard
输入dashboard，按回车/enter，会展示当前进程的信息，按ctrl+c可以中断执行。
{% image https://bu.dusays.com/2023/07/06/64a68c8469bc5.png %}

### 通过 jad 来反编译 Main Class
```shell
jad demo.MathGame
```
{% image https://bu.dusays.com/2023/07/06/64a68d17ebd2c.png %}

### watch
通过`watch`命令来查看demo.MathGame#primeFactors函数的返回值:
{% image https://bu.dusays.com/2023/07/06/64a68d5fcf7ea.png %}

### 退出 arthas
如果只是退出当前的连接，可以用`quit`或者`exit`命令。Attach 到目标进程上的 arthas 还会继续运行，端口会保持开放，下次连接时可以直接连接上。

如果想完全退出 arthas，可以执行`stop`命令。