---
title: JUC面试题
categories: 面试
tags:
  - Java
  - JUC
  - 多线程
cover: 'https://bu.dusays.com/2023/07/24/64be41720d297.png'
abbrlink: 69e9aee0
date: 2023-07-24 17:14:07
---
# JUC面试题

## volatile关键字的作用?
一个共享变量(类的成员变量、类的静态成员变量)在被 `volatile`关键字 修饰之后,那么就具备了两层语
义:
- 保证了不同线程对这个变量进行操作时的可见性,即一个线程修改了某个变量的值,这新值对其他线程来说是立即可见的
- 禁止进行指令重排序

## volatile和synchronized的区别
1.  > volatile 本质是在告诉 jvm 当前变量在寄存器(工作内存)中的值是不确定的,需要从主存中读取
    > synchronized 则是锁定当前变量,只有当前线程可以访问该变量,其他线程被阻塞住
2.  > volatile 仅能使用在变量级别
    > synchronized则可以使用在变量、方法、和类级别的
3.  > volatile仅能实现变量的修改可见性,并不能保证原子性
    > synchronized则可以保证变量的修改可见性和原子性
4.  > volatile不会造成线程的阻塞
    > synchronized可能会造成线程的阻塞
5.  > volatile标记的变量不会被编译器优化
    > synchronized标记的变量可以被编译器优化