---
title: Docker搭建一个小而美的网站流量监控——Umami
categories: 运维
tags:
  - Docker
  - DockerCompose
  - Kafka
cover: 'https://bu.dusays.com/2023/06/13/64880f6a5aaf7.png'
abbrlink: c8787dd0
date: 2023-06-13 15:42:39
---

## 通过 Docker搭建一个小而美的网站流量监控——Umami

### 创建对应的目录

```bash
cd ~
mkdir -p ~/data/docker_data/umami
cd ~/data/docker_data/umami
```

### 编写`docker-compose`配置文件

```bash
vim docker-compose.yml
```

```yaml
---
version: '3'
services:
  umami:
    image: ghcr.io/mikecao/umami:postgresql-latest
    ports:
      - "3000:3000"
    environment:
      DATABASE_URL: postgresql://umami:umami@db:5432/umami # 这里的数据库和密码要和下方你修改的相同
      DATABASE_TYPE: postgresql
      HASH_SALT: replace-me-with-a-random-string
    depends_on:
      - db
    restart: always
  db:
    image: postgres:12-alpine
    environment:
      POSTGRES_DB: umami
      POSTGRES_USER: umami # 数据库用户
      POSTGRES_PASSWORD: umami  # 数据库密码
    volumes:
      - ./sql/schema.postgresql.sql:/docker-entrypoint-initdb.d/schema.postgresql.sql:ro
      - ./umami-db-data:/var/lib/postgresql/data
    restart: always

```

### 启动`Umami`

```bash
docker-compose up -d
```

此时，通过访问http://ip:3000就可以看到