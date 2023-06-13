---
title: CentOS安装MySQL
categories: 运维
tags:
  - CentOS
  - MySQL
cover: 'https://bu.dusays.com/2023/06/13/648837aab68b1.png'
abbrlink: 4ed74180
date: 2023-06-13 17:37:28
---

## 一、更换清华镜像源

### 1. 打开终端并备份当前的软件源列表

  ```Shell
sudo cp /etc/apt/sources.list /etc/apt/sources.list.backup
  ```

### 2. 使用以下命令打开 `/etc/apt/sources.list` 文件

  ```Shell
sudo vim /etc/apt/sources.list
  ```

### 3. 将文件中的内容替换为以下清华镜像源的内容

  ```text
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ focal main restricted universe multiverse
deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ focal main restricted universe multiverse

deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ focal-updates main restricted universe multiverse
deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ focal-updates main restricted universe multiverse

deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ focal-backports main restricted universe multiverse
deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ focal-backports main restricted universe multiverse

deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ focal-security main restricted universe multiverse
deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ focal-security main restricted universe multiverse
  ```

  > **注意：**上述源仅适用于 Ubuntu 20.04 LTS（Focal Fossa）版本。如果您使用的是其他版本的 Ubuntu，请在清华镜像站上查找相应的源。

### 4. 保存并关闭文件

### 5. 使用以下命令更新软件包列表

  ```Shell
sudo apt-get update
  ```

## 二、安装 MySQL

### 1. 使用以下命令安装 MySQL ：

  ```Shell
sudo apt-get install mysql-server
  ```

  在安装过程中，系统会提示您输入 MySQL root 用户的密码。请记住这个密码，因为以后可能会用到它。

### 2. 安装完成后，使用以下命令启动 MySQL 服务：

  ```Shell
sudo systemctl start mysql
  ```

### 3. 使用以下命令检查 MySQL 服务是否已启动

  ```Shell
sudo systemctl status mysql
  ```

  如果 MySQL 服务已成功启动，输出应该显示 `Active: active (running)`

## 三、改 MySQL root 用户的密码或允许远程访问 MySQL

### 1. 打开终端并使用以下命令登录到 MySQL 

  ```Shell
sudo mysql -u root -p
  ```

  系统会提示您输入 MySQL root 用户的密码。这里一开始默认没有密码，直接回车即可。

### 2. 使用以下命令修改 root 用户的密码

  ```SQL
alter USER 'root'@'localhost' identified with mysql_native_password by 'root';
  ```

  将 `root` 替换为您要设置的新密码

### 3. 使用以下命令允许从所有远程 IP 访问 MySQL 的用户

  ```SQL
create user 'root'@'%' identified with mysql_native_password by 'root';
grant all privileges on *.* to 'root'@'%';
  ```

  将 `root` 替换为您要创建的用户名，将 `root` 替换为该用户的密码。

  如果您只想允许特定的 IP 访问 MySQL，则可以将 `%` 替换为特定的 IP。

### 4. 使用以下命令刷新 MySQL 权限

  ```SQL
flush privileges;
  ```

### 5. 使用以下命令退出 MySQL

  ```SQL
exit;
  ```
