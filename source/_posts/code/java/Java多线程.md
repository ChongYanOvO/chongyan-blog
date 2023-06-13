---
title: Java多线程
categories: 数据开发
tags:
  - Java
  - 多线程
cover: https://bu.dusays.com/2023/06/13/64880f6b37b95.png
abbrlink: a2c3ad19
date: 2023-06-13 17:09:21
---

## Java 多线程-认真学习版

### 线程相关概念

#### 进程

1. 进程指定的运行中的程序，操作系统会为进程分配内存空间
2. 进程是程序的一次执行过程，或者说是一个正在运行的程序。这个进程有它自己的生命周期，产生、存在、销毁的过程

<img src="https://chongyandocs-1304373775.cos.ap-nanjing.myqcloud.com/chongyandocs/java-thread-1.png" alt="java-thread-1" style="zoom:30%;" />

在这里`IDEA`程序就是一个进程

#### 线程

1. 线程是由进程创建的，是进程的一个实体
2. 一个进程可以拥有多个线程

<img src="https://chongyandocs-1304373775.cos.ap-nanjing.myqcloud.com/chongyandocs/java-thread-2.png" alt="java-thread-2" style="zoom:50%;" />

我们可以看到在`IDEA`整个进程中有 142 个线程

#### 线程的一些补充概念

1. 单线程：在一个时刻，只允许执行一个线程

2. 多线程：同一时刻可以执行多个线程，这里如上图所示，`IDEA`就是一个多线程应用
3. 并发：同一时刻，多个任务交替执行，单核 CPU 实现的多任务处理就是并发
4. 并行：同一时刻，多个任务同时执行。多核 CPU 才能实现并行。

```java
public class CpuNum {
    public static void main(String[] args) {
        // 获取挡圈端脑 CPU 数量
        Runtime runtime = Runtime.getRuntime();
        int cpuNums = runtime.availableProcessors();
        System.out.println("CPU 数量："+cpuNums);
    }
}
```

### 线程的基本使用

Thread类图

<img src="https://chongyandocs-1304373775.cos.ap-nanjing.myqcloud.com/chongyandocs/java-thread-3.png" alt="java-thread-3" style="zoom:50%;" />

创建线程的两种方式

1. 继承 Thread 类，重写 run 方法
2. 实现 Runnable 接口，重写 run 方法

#### **方式一、继承 Thread 类**

```java
    public static void main(String[] args) {
        myThread myThread = new myThread();
        myThread.start();
    }

    static class myThread extends Thread{

        @Override
        public void run() {
            System.out.println("我重写了 Thread 的 run 方法!!!");
        }
    }
```

#### **方式二、实现 Runnable 接口**

```java
   public static void main(String[] args) {
        myThread myThread = new myThread();
        Thread thread = new Thread(myThread);
        thread.start();
    }

    static class myThread implements Runnable{

        @Override
        public void run() {
            System.out.println("我重写了 Thread 的 run 方法!!!");
        }
    }
```

#### 线程执行示意图

当 `main 进程`启动一个 `Thread-0 线程`后，主线程不会阻塞，会继续执行

这里 `main 线程`和`Thread-0 线程`交替执行

<img src="https://chongyandocs-1304373775.cos.ap-nanjing.myqcloud.com/chongyandocs/java-thread-4.png" alt="java-thread-4" style="zoom:50%;" />

这里注意一点：在`继承 Thread 类`执行创建线程的代码中，调用的是`start()`方法，而在不是`run()`方法，那为什么调用`run()`方法呢?
因为`继承 Thread 类`之后执行的`run()`方法，实际上`run()`方法只是一个普通的方法，并没有真正的启动线程，相当于串行化的执行，会导致 main 线程阻塞，意思就是说当 main 线程执行到`run()`方法时，必须要等到`run()`方法执行结束后才能继续执行
换句话说:
调用`start()`方法会在 `main 栈`外单独开一块栈空间，调用`run()`方法则还是在` main 栈`内执行

在源码中是这样执行的：

Thread.start() 调用 start0() 方法

```java
public synchronized void start() {
    ...
      start0();  
    ...
}
```

这里 start0() 方法是本地方法，由 JVM调用，底层是 C/C++执行
真正实现多线程的效果，是 start0() ，而不是run()

```java
private native void start0();
```

 start0() 调用后，该线程并不一定会立马执行，只是将线程变为可执行状态，然后等待 CPU 的统一调度执行

#### 啥时候使用`实现 Runnable 接口`来创建线程?

在 Java 中是单继承的，在某些情况下一个类继承了某个父类，此时就没办法`继承 Thread 类`来创建线程了，就需要通过`实现 Runnable 接口`来创建线程

这里也要注意一点：`Thread thread = new Thread(myThread)`在创建线程时直接将`myThread`作为参数来创建线程，这里底层调用的其实是代理模式

这里模拟一下代理模式

```java
class ProxyThread implements Runnable {

    private Runnable target = null;

    @Override
    public void run() {
        if (target != null) {
            target.run();
        }
    }

    public ProxyThread(Runnable target) {
        this.target = target;
    }
    
    public void start(){
        start0();
    }
    
    public void start0(){
        run();
    }
    
}
```

将一个实现了`Runnable接口`的类作为参数传入，然后调用其`run()`方法，在通过`start()`方法创建线程时，还是会调用`start0()`方法，由此可以看出，在 Java 中真正实现多线程的其实就是`start0()`方法

<img src="https://chongyandocs-1304373775.cos.ap-nanjing.myqcloud.com/chongyandocs/java-thread-5.png" alt="java-thread-5" style="zoom:50%;" />

Java 的`Thread`类中也确实是这样使用的
`target.run()`是动态绑定传进来的`myThread`类中的`run()`方法

#### 多个线程执行示意图

![java-thread-6](https://chongyandocs-1304373775.cos.ap-nanjing.myqcloud.com/chongyandocs/java-thread-6.png)

#### 练习：使用多线程模拟三个窗口同时售票 100 张

```java
public class SellTicket {
    public static void main(String[] args) {
        mySellTicket mySellTicket = new mySellTicket();
        Thread sellTicket01 = new Thread(mySellTicket);
        Thread sellTicket02 = new Thread(mySellTicket);
        Thread sellTicket03 = new Thread(mySellTicket);
        sellTicket01.start();
        sellTicket02.start();
        sellTicket03.start();
    }
}

class mySellTicket implements Runnable {
    private static int ticketNum = 100;

    @Override
    public void run() {
        while (true) {
            if (ticketNum <= 0) {
                System.out.println("售票结束...");
                break;
            }
            try {
                Thread.sleep(50);
            } catch (InterruptedException e) {
                throw new RuntimeException(e);
            }
            System.out.println("窗口 " + Thread.currentThread().getName()
                    + " 售出一张票" + "剩余票数= " + (--ticketNum));
        }
    }
}
```

**出现问题了，这里车票超售了**

<img src="https://chongyandocs-1304373775.cos.ap-nanjing.myqcloud.com/chongyandocs/java-thread-7.png" alt="java-thread-7" style="zoom:50%;" />

在同一时刻，三个线程都执行了售票操作，`ticketNum`被三个线程同时修改后，还没有进入到下一次判断，那么票数就会变成 0 和 -1

#### 线程终止

1. 当线程完成任务时，会自动退出
2. 还可以通过使用变量来控制 run 方法退出从而实现停止线程，即通知方式

```java
public class ThreadExit {
    public static void main(String[] args) throws InterruptedException {
        myThread myThread = new myThread();
        Thread thread = new Thread(myThread);
        thread.start();
        // 如果希望 main 线程去控制 myThread 线程的终止，修改 loop 变量即可
        // 修改 loop 让 myThread 退出 run 方法，从而退出进程，这种方式是通知方式
        System.out.println("main 线程休眠 10s");
        Thread.sleep(10 * 1000);
        myThread.setLoop(false);
    }
}

class myThread implements Runnable {
    private int count = 0;
    private boolean loop = true;

    @Override
    public void run() {
        while (loop) {
            try {
                Thread.sleep(50);
            } catch (InterruptedException e) {
                throw new RuntimeException(e);
            }
            System.out.println("运行中..." + (++count));
        }
    }
    public void setLoop(boolean loop) {
        this.loop = loop;
    }
}
```

#### 第一组线程常用方法

1. `setName`：设置线程名称，使之与参数 name 相同
2. `getName`：返回该线程的名称
3. `start`：是该线程开始执行,JVM 虚拟机底层调用 start0 方法
4. `run`： 调用线程对象 run 方法
5. `setPriority`： 更改线程优先级
6. `getPriority`：获取线程优先级
7. `sleep`：在指定的毫秒数内让当前正在执行的线程休眠(暂停执行)
8. `interrupt`： 中断线程

**注意：**

1. start 底层会创建新的线程，调用 run,run 就是一个简单的方法调用，不会启动新的线程
2. 线程优先级的的范围
3. `interrupt`，中断线程，但没有真正接收线程。所以一般用于中断正在休眠的线程
4. sleep 是线程的静态方法，使线程休眠

```java
public class ThreadMethod {
    public static void main(String[] args) throws InterruptedException {
        myThread myThread = new myThread();
        Thread thread = new Thread(myThread);
        thread.setName("重言");
        thread.setPriority(Thread.MAX_PRIORITY);
        thread.start();
        System.out.println("当前线程名称:" + thread.getName());
        System.out.println(thread.getName() + " 的优先级: " + thread.getPriority());
        for (int i = 1; i <= 5; i++) {
            Thread.sleep(1000);
            System.out.println("hi " + i);
        }
        thread.interrupt();
    }
}

class myThread implements Runnable {
    @Override
    public void run() {
        while (true) {
            for (int i = 0; i < 20; i++) {
                System.out.println("当前线程名称:" + Thread.currentThread().getName() + i);
            }
            try {
                System.out.println("线程:" + Thread.currentThread().getName() + " sleep 5s...");
                Thread.sleep(5 * 1000);
            } catch (InterruptedException e) {
                System.out.println("线程:" + Thread.currentThread().getName() + " 被中断了");
            }
        }
    }
}
```

#### 第二组线程常用方法

1. `yied`：线程的礼让。让出 CPU ，让其他线程先执行，但礼让的时间不确定，所以说不一定礼让成功。在 CPU 资源紧张的情况下，更容易礼让成功
2. `join`：线程的插队。插队的线程一旦插队成功，则肯定先执行完插入的线程的所有任务

`yied`执行示意图
`yied`在执行后线程会进入就绪状态，只是将资源优先给` Thread-2 `去使用，如果 CPU 资源充足时，` Thread-1 `也会继续执行

<img src="https://chongyandocs-1304373775.cos.ap-nanjing.myqcloud.com/chongyandocs/java-thread-8.png" alt="java-thread-8" style="zoom:50%;" />

`join`执行示意图
`join`在执行后线程会进入阻塞状态，只有将` Thread-2 `线程完全执行结束后，才会执行` Thread-1 `线程，这是不管 CPU 资源是否还有空余，都需要等待` Thread-2 `线程执行结束

<img src="https://chongyandocs-1304373775.cos.ap-nanjing.myqcloud.com/chongyandocs/java-thread-9.png" alt="java-thread-9" style="zoom:50%;" />

这里说一点：`sleep`在执行后线程会进入阻塞状态

```java
public class ThreadMethod {
    public static void main(String[] args) throws InterruptedException {
        myThread myThread = new myThread();
        Thread thread = new Thread(myThread);
        thread.start();

        for (int i = 1; i <= 20; i++) {
            Thread.sleep(1000);
            System.out.println("MainThread---" + i);
            // 当 i== 5 时，先执行 myThread 线程，当 myThread 线程执行结束后再执行 mainThread
            if (i == 5) {
                thread.join();
            }
        }
    }
}

class myThread implements Runnable {
    @Override
    public void run() {
        for (int i = 1; i <= 20; i++) {
            try {
                Thread.sleep(1000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println("myThread---" + i);
        }
    }
}
```

### 用户线程和守护线程

1. `用户线程`：也叫作工作线程，当线程的任务执行完或以通知方式来结束线程
2. `守护线程`：一般是为工作线程服务的，当使用用户线程结束，守护线程自动结束
3. 常见的守护线程： JVM 垃圾回收机制

```java
public class ThreadMethod {
    public static void main(String[] args) throws InterruptedException {
        myThread myThread = new myThread();
        Thread thread = new Thread(myThread);
        // 将 thread 设为守护线程，注意要在 start 之前设置
        thread.setDaemon(true);
        thread.start();
        for (int i = 1; i <= 10; i++) {
            System.out.println("Main 线程在工作..." + i);
            Thread.sleep(1000);
        }
    }
}

class myThread implements Runnable {
    private int count = 0;
    @Override
    public void run() {
        for (; ; ) {
            try {
                Thread.sleep(1000);
            } catch (InterruptedException e) {
                throw new RuntimeException(e);
            }
            System.out.println("守护进程 daemon 在运行..." + (++count));
        }
    }
}
```

没将`thread线程`设为`守护线程`前，当 `Main 线程`结束后，`thread线程`依然还在工作

<img src="https://chongyandocs-1304373775.cos.ap-nanjing.myqcloud.com/chongyandocs/java-thread-10.png" alt="java-thread-10" style="zoom:50%;" />

当将`thread线程`设为`守护线程`后，`Main 进程`接收后，作为`守护线程`的`thread线程`也会被关闭

<img src="https://chongyandocs-1304373775.cos.ap-nanjing.myqcloud.com/chongyandocs/java-thread-11.png" alt="java-thread-11" style="zoom:50%;" />

#### 线程的生命周期

线程的 7 大状态

1. `NEW`：尚未启动的线程处于此状态。

2. `Runnable` ：在Java 虚拟机中执行的线程处于此状态。
- `Ready`：线程在可运行`线程池`中，但未获得CPU执行权，和`Running`并称运行。
- `Running`：线程执行并获得 CPU 执行权，和`Ready`并称运行。

3. `Blocked`：被阻塞等待监视器锁定的线程处于此状态。

4. `Wating`：正在等待可另一个执行特定动作的线程处于此状态。

5. `TimeWaiting`：正在等待可另一个执行特定动作达到指定等待时间的线程处于此状态。

6. `Terminated`：已退出的线程处于此状态。

<img src="https://chongyandocs-1304373775.cos.ap-nanjing.myqcloud.com/chongyandocs/java-thread-12.png" alt="java-thread-12" style="zoom:50%;" />

```java
public class ThreadState_ {
    public static void main(String[] args) throws InterruptedException {
        myThread myThread = new myThread();
        Thread thread = new Thread(myThread);
        System.out.println(thread.getName() + " 状态: " + thread.getState());
        thread.start();
        while (Thread.State.TERMINATED != thread.getState()) {
            System.out.println(thread.getName() + " 状态: " + thread.getState());
            Thread.sleep(1000);
        }
        System.out.println(thread.getName() + " 状态: " + thread.getState());
    }
}

class myThread implements Runnable {
    @Override
    public void run() {
        while (true) {
            for (int i = 0; i < 3; i++) {
                System.out.println("hi" + i);
                try {
                    Thread.sleep(1000);
                } catch (InterruptedException e) {
                    throw new RuntimeException(e);
                }
            }
            break;
        }
    }
}
```

打印结果

```java
Thread-0 状态: NEW
Thread-0 状态: RUNNABLE
hi0
Thread-0 状态: RUNNABLE
hi1
Thread-0 状态: RUNNABLE
hi2
Thread-0 状态: TIMED_WAITING
Thread-0 状态: TERMINATED
```

### 线程同步机制

1. 在多线程编程中，一些敏感时间不允许被多个线程同时访问，此时就使用同步访问技术，保证数据在任何同一时刻，最多有一个线程访问，以保证数据的完整性。
2. 或者说：线程同步，即是当有一个线程在对内存进行操作时，其他线程都不可以对于这个内存地址进行操作，直到该线程完成操作，洽谈线程才能对这个内存地址进行操作。

#### 同步的实现方式 Synchronized

1. 同步代码块`synchronized(对象)`得到对象的锁，才能同步代码
2. synchronized还可以放在方法声明中，表示让整个方法为同步方法
3. 使用`synchronized关键字`来完成线程同步

#### 使用 Synchronized 解决车票超售问题

```java
public class SellTicket {
    public static void main(String[] args) {
        mySellTicket mySellTicket = new mySellTicket();
        Thread sellTicket01 = new Thread(mySellTicket);
        Thread sellTicket02 = new Thread(mySellTicket);
        Thread sellTicket03 = new Thread(mySellTicket);
        sellTicket01.start();
        sellTicket02.start();
        sellTicket03.start();
    }
}
class mySellTicket implements Runnable {
    private static int ticketNum = 100;
    private boolean loop = true;
    // 同步方法，在同一时刻只能有一个线程执行 sell 方法
    public synchronized void sell() {
        if (ticketNum <= 0) {
            System.out.println("售票结束...");
            loop = false;
            return;
        }
        try {
            Thread.sleep(100);
        } catch (InterruptedException e) {
            throw new RuntimeException(e);
        }
        System.out.println("窗口 " + Thread.currentThread().getName()
                + " 售出一张票" + "剩余票数= " + (--ticketNum));
    }
    @Override
    public void run() {
        while (loop) {
            sell();
        }
    }
}
```

在售票方法`sell()`上加上`synchronized线程同步关键字`使在同一时刻只能有一个线程执行 sell 方法

```java
public  void sell() {
        synchronized (this){
            if (ticketNum <= 0) {
                System.out.println("售票结束...");
                loop = false;
                return;
            }
            try {
                Thread.sleep(50);
            } catch (InterruptedException e) {
                throw new RuntimeException(e);
            }
            System.out.println("窗口 " + Thread.currentThread().getName()
                    + " 售出一张票" + "剩余票数= " + (--ticketNum));
        }
    }
```

这种方法就是在代码块上加上锁

### 互斥锁

1. 互斥锁用来保证数据操作的完整性。
2. 每个对象对应于一个可称为`互斥锁`的标记，这个标记用来同一时刻只能由一个线程访问。
3. `关键字synchronized`来与对象的互斥锁联系。当某个对象用`synchronized`修饰时，表明该对象在同一时刻只能由一个线程访问。
4. 同步的局限性：会导致查询的执行效率降低。
5. 同步方法(非静态)的锁可以是 this,也可以是其他对象(要求是同一个对象)。
6. 同步方法(静态)的锁为当前类本身。即为`当前类.class`

### 死锁

什么是死锁?
多个线程都占用了对方的锁资源，但不肯让，导致了死锁。

#### 模拟线程死锁

```java
public class DeadLock_ {
    public static void main(String[] args) {
        // 模拟死锁现象
        myThread myThread1 = new myThread(true);
        myThread myThread2 = new myThread(false);
        Thread thread1 = new Thread(myThread1);
        Thread thread2 = new Thread(myThread2);

        thread1.start();
        thread2.start();
    }
}

class myThread implements Runnable {
    static Object object1 = new Object();
    static Object object2 = new Object();
    boolean flag;
    public myThread(boolean flag) {
        this.flag = flag;
    }
    @Override
    public void run() {
        if (flag) {
            synchronized (object1) {
                System.out.println(Thread.currentThread().getName() + "进入 1");
                synchronized (object2) {
                    System.out.println(Thread.currentThread().getName() + "进入 2");
                }
            }
        } else {
            synchronized (object2) {
                System.out.println(Thread.currentThread().getName() + "进入 2");
                synchronized (object1) {
                    System.out.println(Thread.currentThread().getName() + "进入 1");
                }
            }
        }
    }
}
```

1. 如果 flag 为 true,`线程 Thread-0 `就会先持有 object1 对象锁，然后尝试获取 object2 对象锁。
2. 如果`线程 Thread-0 `获取不到 object2 对象锁，就会`Blocked`。
3. 如果 flag 为 flase,`线程 Thread-1 `就会先持有 object2 对象锁，然后尝试获取 object1 对象锁。
4. 如果`线程 Thread-1 `获取不到 object1 对象锁，就会`Blocked`。

### 释放锁

1. 当前线程的同步方法、同步代码块执行结束。
2. 当前线程在同步方法、同步代码块中遇到 break、return。
3. 当前线程在同步方法、同步代码块中出现未处理的`Error`或`Exception`，导致异常结束。
4. 当前线程的同步方法、同步代码块执行了线程对象的 `wait()`方法，当前线程暂停，并释放锁。

#### 下面这些操作不会释放锁

1. 线程执行同步方法、同步代码块时，程序调用`Thread.sleep()`、`Thread.yeild()`方法暂停当前线程执行，不会释放锁。
2. 线程执行同步代码块时时，其他线程调用了该线程的`suspend()`方法将该线程挂起，该线程不会释放锁。
