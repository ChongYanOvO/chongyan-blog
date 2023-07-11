---
title: Java多线程学习2.0
categories: 数据开发
tags:
  - Java
  - 多线程
cover: 'https://bu.dusays.com/2023/07/11/64ad210403f5f.png'
abbrlink: ccc9d50d
date: 2023-07-11 17:26:35
---

# Java 多线程

## 什么是并发和并行

- 并发:同一时刻多个指令在单个 CPU 上交替执行

- 并行:同一时刻多个指令在多个 CPU 上同时执行

## 线程实现方式

### Thread 实现

通过将类继承 `Thread` 类,并重写 `Run` 方法

- 优点:实现简单,可以直接使用`Thread` 类中的方法
- 缺点:可扩展性差,不能继承其他的类

```java
class ExtendThread extends Thread {
    @Override
    public void run() {
        for (int i = 0; i < 20; i++) {
            System.out.println(getName() + " : " + "ExtendThread-Thread");
        }
        super.run();
    }
}

private static void runExtendThread() {
    ExtendThread thread1 = new ExtendThread();
    ExtendThread thread2 = new ExtendThread();
    thread1.setName("Thread 继承实现线程 - 1");
    thread2.setName("Thread 继承实现线程 - 2");
    thread1.start();
    thread2.start();
}
```

### Runable 接口实现

通过将类实现 `Runable` 接口,并实现 `Run` 方法

- 优点:可扩展性强,实现接口之后还可以继承其他类
- 缺点:实现复杂,不能直接使用`Thread` 类中的方法

```java
class ImplementRunnable implements Runnable {

    @Override
    public void run() {
        for (int i = 0; i < 20; i++) {
            System.out.println(Thread.currentThread().getName() + " : " + "ImplementRunnable-Runnable");
        }
    }
}

private static void runImplementRunnable() {
    ImplementRunnable runnable = new ImplementRunnable();
    Thread thread1 = new Thread(runnable);
    Thread thread2 = new Thread(runnable);
    thread1.setName("Runnable 实现线程 - 1");
    thread2.setName("Runnable 实现线程 - 2");
    thread1.start();
    thread2.start();
}
```



### Callable<T> 接口实现

通过将类实现 `Callable` 接口,并实现 `call` 方法,通过 `FutureTask` 用来管理多线程运行的结果

- 优点:可扩展性强,实现接口之后还可以继承其他类,还可以获取到多线程运行的结果
- 缺点:实现复杂,不能直接使用`Thread` 类中的方法

```java
class ImplementCallable implements Callable<Integer> {

    @Override
    public Integer call() throws Exception {
        int sum = 0;
        for (int i = 0; i < 100; i++) {
            sum += i;
        }
        return sum;
    }
}

private static void runCallableThread() throws ExecutionException, InterruptedException {
    ImplementCallable implementCallable = new ImplementCallable();
    // FutureTask 用来管理多线程运行的结果
    FutureTask<Integer> futureTask = new FutureTask<>(implementCallable);
    Thread thread = new Thread(futureTask);
    Thread thread2 = new Thread(futureTask);
    thread.start();
    Integer result = futureTask.get();
    System.out.println("result = " + result);
}
```

## 线程的方法

- `void setName()`:设置线程的名字
- `String getName()`:获取线程的名字
- `static Thread currentThread()`:获取当前线程 
- `static void sleep()`:当前线程休眠
- `void setPriority()`:设置线程优先级
- `void getPriority()`:获取程优先级
- `void setDaemon`:设置线程为守护线程
- `static void yield()`:出让线程的执行权
- `final void join()`:线程加入

## 线程的生命周期

![](https://bu.dusays.com/2023/07/11/64acebd818d01.png)

- 新建:创建线程对象
- 就绪:有执行资格,没有执行权
- 运行:有执行资格,有执行权
- 阻塞:没有执行资格,没有执行权
- 死亡:线程死亡,被垃圾回收

`新建` -> `就绪`:调用 `start()` 方法

`就绪` -> `运行`:抢到 CPU 执行权

`运行` -> `就绪`:其他线程抢走 CPU 执行权

`运行` -> `阻塞`: 调用 `sleep()` 或其他阻塞方法
`阻塞` -> `就绪`:`sleep()` 方法时间结束或其他阻塞方法结束

`运行` -> `死亡`:`run()`方法执行结束

## 线程安全问题

三个窗口同时卖票问题

主程序

```java
public class SellTicket {
    public static void main(String[] args) {
        Thread thread1 = new Thread(new SellTicketRunnable(), "窗口1");
        Thread thread2 = new Thread(new SellTicketRunnable(), "窗口2");
        Thread thread3 = new Thread(new SellTicketRunnable(), "窗口3");

        thread1.start();
        thread2.start();
        thread3.start();
    }
}
```

### 同步代码块

```java
class SellTicketRunnable implements Runnable {

    private static int ticket = 100;

    @Override
    public void run() {
        while (true) {
            synchronized (SellTicketRunnable.class) {
                if (ticket > 0) {
                    System.out.println(Thread.currentThread().getName() + "正在卖第 " + ticket + " 张票");
                    ticket--;
                } else {
                    break;
                }
            }
        }
    }
}
```

### 同步方法

```java
class SellTicketRunnable implements Runnable {

    private static int ticket = 100;


    @Override
    public void run() {
        while (true) {
            if(sellTicket()) break;
        }
    }

    private synchronized boolean sellTicket() {
        if (ticket > 0) {
            System.out.println(Thread.currentThread().getName() + "正在卖第 " + ticket + " 张票");
            ticket--;
            try {
                Thread.sleep(10);
            } catch (InterruptedException e) {
                throw new RuntimeException(e);
            }
            return false;
        }
        return true;
    }
}
```

### Lock锁

```java
class SellTicketRunnable implements Runnable {

    private static int ticket = 100;

    static Lock lock = new ReentrantLock();

    @Override
    public void run() {
        while (true) {
            // 加锁
            lock.lock();
            try {
                if(sellTicket()) break;
            }finally {
                // 释放锁
                lock.unlock();
            }
        }
    }

    private synchronized boolean sellTicket() {
        if (ticket > 0) {
            System.out.println(Thread.currentThread().getName() + "正在卖第 " + ticket + " 张票");
            ticket--;
            try {
                Thread.sleep(10);
            } catch (InterruptedException e) {
                throw new RuntimeException(e);
            }
            return false;
        }
        return true;
    }
} 
```

## 等待唤醒机制

### 生产者消费者模型

#### 生产者

1. 判断缓冲区状态
2. 如果缓冲区为 0 ,则进行生产并修改缓冲区状态
3. 如果缓冲区为 1 ,则和缓冲区的锁进行链接,并唤醒消费者

```java
class Producer implements Runnable {

    @Override
    public void run() {
        while (true) {
            synchronized (Buffer.lock) {
                if (Buffer.count == 0) {
                    break;
                } else {
                    if (Buffer.flag == 0) {
                        System.out.println("生产者生产了一个产品");
                        Buffer.flag = 1;
                    } else {
                        try {
                            Buffer.lock.wait();
                        } catch (InterruptedException e) {
                            throw new RuntimeException(e);
                        }
                        Buffer.lock.notifyAll();
                    }
                }
            }
        }
    }
}
```

#### 消费者

1. 判断缓冲区状态
2. 如果缓冲区为 1 ,则进行消费并修改缓冲区状态
3. 如果缓冲区为 0 ,则和缓冲区的锁进行链接,并唤醒生产者

```java
class Consumer implements Runnable {

    @Override
    public void run() {
        while (true) {
            synchronized (Buffer.lock) {
                if (Buffer.count == 0) {
                    break;
                } else {
                    // 判断缓冲区状态
                    if (Buffer.flag == 0) {
                        try {
                            // 等待生产者生产
                            Buffer.lock.wait();
                        } catch (InterruptedException e) {
                            throw new RuntimeException(e);
                        }

                    } else {
                        Buffer.count--;
                        // 消费者消费
                        System.out.println("消费者消费了一个产品,还有 "+Buffer.count+" 个");

                        // 修改缓冲区状态
                        Buffer.flag = 0;

                        // 唤醒生产者
                        Buffer.lock.notifyAll();

                    }
                }
            }
        }
    }

}
```

#### 缓冲区

```java
class Buffer {
    // 0表示缓冲区为空，1表示缓冲区有数据
    public static int flag = 0;

    // 缓冲区大小
    public static int count = 10;

    // 锁
    public static final Object lock = new Object();

}
```



### 阻塞队列

#### 生产者

```java
class Producer extends Thread {
    ArrayBlockingQueue<String> queue;

    public Producer(ArrayBlockingQueue<String> queue) {
        this.queue = queue;
    }

    @Override
    public void run() {
        while (true){
            if (queue.isEmpty()) {
                try {
                    Thread.sleep(10);
                } catch (InterruptedException e) {
                    throw new RuntimeException(e);
                }
                queue.add("product");
                System.out.println("生产者生产了一个产品");
            }

        }
    }
}
```

#### 消费者

```java
public class Consumer extends Thread {
    ArrayBlockingQueue<String> queue;

    public Consumer(ArrayBlockingQueue<String> queue) {
        this.queue = queue;
    }

    @Override
    public void run() {
        while (true) {
            if (!queue.isEmpty()) {
                try {
                    Thread.sleep(10);
                } catch (InterruptedException e) {
                    throw new RuntimeException(e);
                }
            }
            try {
                String product = queue.take();
                System.out.println("消费者消费了一个产品: " + product);
            } catch (InterruptedException e) {
                throw new RuntimeException(e);
            }
        }
    }
}
```



## 线程池

### 实现原理

1. 创建一个空线程池
2. 提交任务时,线程池会创建新的线程对象,当任务执行完毕后,线程会还给线程池,下次再提交任务时,不需要创建新的线程,可以直接复用已有的线程
3. 如果提交任务时,线程池中没有空闲的线程,则无法创建新线程,任务会排队等待

#### 无上限线程池

```java
public class UnlimitedThreadPool {
    public static void main(String[] args) {
        // 获取线程池对象
        ExecutorService pool = Executors.newCachedThreadPool();
        // 提交任务
        pool.submit(new Job());
        // 销毁线程池
        pool.shutdown();
    }
}
```

#### 有上限线程池

```java
public class UnlimitedThreadPool {
    public static void main(String[] args) {
        // 获取线程池对象
        ExecutorService pool = Executors.newCachedThreadPool();
        // 提交任务
        pool.submit(new Job());
        pool.submit(new Job());
        pool.submit(new Job());
        pool.submit(new Job());
        pool.submit(new Job());
        pool.submit(new Job());
        // 销毁线程池
        pool.shutdown();
    }
}
```

执行结果

```shell
pool-1-thread-1
pool-1-thread-3
pool-1-thread-2
pool-1-thread-4
pool-1-thread-5
pool-1-thread-6
```

#### 任务线程

```java
class Job implements Runnable{
    @Override
    public void run() {
            System.out.println(Thread.currentThread().getName());
    }
}
```

执行结果

```java
pool-1-thread-1
pool-1-thread-2
pool-1-thread-3
pool-1-thread-3
pool-1-thread-2
```

## 自定义线程池

```java
// 核心线程数量
// 最大线程数量
// 空闲线程最大存活时间
// 时间单位 用 TimeUnit 指定
// 任务队列
// 创建线程工厂
// 任务拒绝策略
ThreadPoolExecutor pool = new ThreadPoolExecutor(
    3,
    6,
    60,
    TimeUnit.SECONDS,
    new ArrayBlockingQueue<>(10),
    Executors.defaultThreadFactory(),
    new ThreadPoolExecutor.AbortPolicy()
);
```

