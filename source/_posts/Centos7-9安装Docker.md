---
title: Centos7.9安装Docker
abbrlink: 372c7897
date: 2023-06-12 22:52:10
categories: 环境搭建
tags: [Docker,CentOS] 
cover: https://bu.dusays.com/2023/06/12/64873d214e261.png
---

## Centos 7.9 安装 Docker

### 1. 卸载之前安装的 Docker 版本

```bash
yum remove docker \
                  docker-client \
                  docker-client-latest \
                  docker-common \
                  docker-latest \
                  docker-latest-logrotate \
                  docker-logrotate \
                  docker-selinux \
                  docker-engine-selinux \
                  docker-engine \
                  docker-ce
```

### 2. 安装 Docker

#### 2.1 安装 yum 相关依赖

```bash
yum install -y yum-utils \
           device-mapper-persistent-data \
           lvm2 --skip-broken
```

#### 2.2 更新本地 yum 镜像源

```bash
yum-config-manager \
    --add-repo \
    https://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
  
sed -i 's/download.docker.com/mirrors.aliyun.com\/docker-ce/g' /etc/yum.repos.d/docker-ce.repo
yum makecache fast
```

#### 2.3 通过 yum 安装 Docker

```bash
yum install -y docker-ce
```

### 3. 启动 Docker

**Docker 启动相关命令**

```bash
systemctl start docker  # 启动docker服务
systemctl stop docker  # 停止docker服务
systemctl restart docker  # 重启docker服务
```

### 4. 配置阿里云镜像加速

**创建文件夹**

```bash
sudo mkdir -p /etc/docker
```

**在文件夹内新建一个 `daemon.json` 文件**

```bash

sudo tee /etc/docker/daemon.json <<-'EOF'
{
  "registry-mirrors": ["https://akchsmlh.mirror.aliyuncs.com"]
}
EOF
```

**重新加载文件**

```bash
sudo systemctl daemon-reload
```

**重启docker**

```bash
sudo systemctl restart docker
```

### 5. 安装 Docker Compose

**检测是否安装 ocker Compose**

```bash
docker-compose -version
```

**下载**

```bash
curl -L https://get.daocloud.io/docker/compose/releases/download/1.26.2/docker-compose-`uname -s`-`uname -m` > /usr/local/bin/docker-compose
```

***设置文件可执行权限* **

```bash
chmod +x /usr/local/bin/docker-compose
```

**建立软连接**

```bash
sudo ln -s /usr/local/bin/docker-compose /usr/bin/docker-compose
```
