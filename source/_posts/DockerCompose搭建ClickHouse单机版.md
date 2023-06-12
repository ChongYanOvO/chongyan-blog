---
title: DockerCompose搭建ClickHouse单机版
categories: 环境搭建
tags: [Docker,DockerCompose,ClickHouse]
cover: 'https://bu.dusays.com/2023/06/12/64873d2155cfa.png'
abbrlink: 9491c2eb
date: 2023-06-12 23:53:07
---

## 通过 Docker Compose 搭建 ClickHouse 单机版

### docker-compose-single-clickhouse.yml

```yaml
version: '3'

services:
  clickhouse:
    image: yandex/clickhouse-server
    container_name: clickhouse
    restart: always
    networks:
      - deng
    ports:
      - "8123:8123"
      - "9000:9000"
    volumes:
      # 默认配置
      - ./data/config/docker_related_config.xml:/etc/clickhouse-server/config.d/docker_related_config.xml:rw
      - ./data/config/config.xml:/etc/clickhouse-server/config.xml:rw
      - ./data/config/users.xml:/etc/clickhouse-server/users.xml:rw
      - /etc/localtime:/etc/localtime:ro
      # 运行日志
      - ./data/log:/var/log/clickhouse-server
      # 数据持久
      - ./data:/var/lib/clickhouse:rw

networks:
  deng:
    external: true
```
