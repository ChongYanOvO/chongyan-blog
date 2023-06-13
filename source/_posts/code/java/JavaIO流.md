---
title: JavaIO流
categories: 数据开发
tags:
  - Java
  - IO流
cover: https://bu.dusays.com/2023/06/13/64880f6aa0147.png
abbrlink: c51233f6
date: 2023-06-13 17:05:07
---

## Java IO流-认真学习版

### 什么是文件和文件流

文件是`保存数据的地方`
文件流：文件在程序中是以流的形式来操作的

<img src="https://chongyandocs-1304373775.cos.ap-nanjing.myqcloud.com/chongyandocs/java-io-1.png" alt="java-io-1" style="zoom:50%;" />

流：数据在数据源(文件)和程序(内存)之间的路径
输入流：数据从数据源(文件)到程序(内存)之间的路径
输出流：数据从程序(内存)到数据源(文件)之间的路径

### 常用文件操作

#### 创建文件对象相关构造器和方法

<img src="https://chongyandocs-1304373775.cos.ap-nanjing.myqcloud.com/chongyandocs/java-io-2.png" alt="java-io-2" style="zoom:50%;" />

1. `new File(String pathname)`根据路径构建一个`File`对象
2. `new File(File parent, String child)`根据父目录文件 + 子路径构建
3. `new File(String parent, String child)`根据父目录 + 子路径构建
4. `createNewFile`创建新文件

这里的`flie`对象,在 Java 程序中只是个对象,只有执行了`createNewFile`方法才会真正的在磁盘创建该文件

##### 方式一、根据路径构建一个`File`对象

```java
String filePath = "/Volumes/Code/IdeaProjects/java-IO-learn/src/main/java/com/chongyan/file/news1.txt";
File file = new File(filePath);
try {
  file.createNewFile();
  System.out.println("文件创建成功");
} catch (IOException e) {
  throw new RuntimeException(e);
}
```

##### 方式二、根据父目录文件 + 子路径构建
```java
String parentFilePath = "/Volumes/Code/IdeaProjects/java-IO-learn/src/main/java/com/chongyan/file";
String childFilePath = "news2.txt";
File parentFile = new File(parentFilePath);
File file = new File(parentFile, childFilePath);

try {
  file.createNewFile();
  System.out.println("文件创建成功");
} catch (IOException e) {
  throw new RuntimeException(e);
}
```


##### 方式三、根据父目录 + 子路径构建
```java
String parentFilePath = "/Volumes/Code/IdeaProjects/java-IO-learn/src/main/java/com/chongyan/file";
String childFilePath = "news3.txt";
File file = new File(parentFilePath, childFilePath);
try {
  file.createNewFile();
  System.out.println("文件创建成功");
} catch (IOException e) {
  throw new RuntimeException(e);
}
```

#### 获取文件相关信息

- `getName() `文件名称
- `getAbsolutePath()` 文件绝对路径
- `getParent() `文件父目录
- `length()` 文件大小(字节)
- `exists()` 文件是否存在
- `isFile() `是不是一个文件
- `isDirectory() `是不是一个目录

```java
import java.io.File;
import java.io.IOException;

public class FileInformation {
    public static void main(String[] args) {
        String filePath = "/Volumes/Code/IdeaProjects/java-IO-learn/src/main/java/com/chongyan/file/news1.txt";
        File file = new File(filePath);
        try {
            file.createNewFile();
            System.out.println("文件创建成功");
        } catch (IOException e) {
            throw new RuntimeException(e);
        }
        
        System.out.println("文件名称: " + file.getName());
        System.out.println("文件绝对路径: " + file.getAbsolutePath());
        System.out.println("文件父目录: " + file.getParent());
        System.out.println("文件大小(字节): " + file.length());
        System.out.println("文件是否存在: " + file.exists());
        System.out.println("是不是一个文件: " + file.isFile());
        System.out.println("是不是一个目录: " + file.isDirectory());
    }
}
```

#### 目录的操作和文件删除

`mkdir`创建以及目录、`mkdirs `创建多级目录、`delete` 删除空目录或文件
在 `Java`编程中,目录也被当成文件

```java
import java.io.File;

public class Directory_ {
    public static void main(String[] args) {
        String filePath = "/Volumes/Code/IdeaProjects/java-IO-learn/src/main/java/com/chongyan/file/news1.txt";
        File file = new File(filePath);
        if (file.exists()) {
            if (file.delete()) {
                System.out.println("删除成功");
            }else {
                System.out.println("删除失败");
            }
        }else {
            System.out.println("文件不存在");
        }
        
        String directoryPath = "/Volumes/Code/IdeaProjects/java-IO-learn/src/main/java/com/chongyan/file/a/b/c";
        File directory = new File(directoryPath);
        if (directory.exists()) {
            System.out.println(directoryPath + "存在...");
        }else {
            if (directory.mkdirs()){
                System.out.println(directoryPath + "创建成功...");
            }else {
                System.out.println(directoryPath + "创建失败...");
            }
        }
    }
}
```

### IO 流原理及流的分类

**`Java IO 流`原理**

1. `I/O `是` Input/Output `的缩写,`I/O 技术`是非常更实用的技术,用于处理数据传输,如读写文件,网络通讯等
2. Java 程序中,对应数据的输入/输出操作一`流(Stream)`的方式进行
3. `java.io 包`下提供了各种流类和接口,用于获取不同种类的数据,并通过方法输出或输出数据
