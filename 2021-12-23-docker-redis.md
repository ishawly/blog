---
title: 使用 docker 搭建 redis 环境
date: 2021-12-23 16:40:00
# description: redis 在项目中
categories:
- 服务器
tags:
- docker
- redis
---

# Redis 主从模式

> 参考文档: [使用docker 搭建redis的主从复制](https://cloud.tencent.com/developer/article/1693904)

使用 docker-compose 简化执行命令, 对应 docker-compose.yml 如下:

```
version: '3.7'
services:
  master:
    image: redis
    container_name: redis-master
    restart: always
    command: redis-server --port 6379 --requirepass master123  --appendonly yes
    ports:
      - 6379:6379
    volumes:
      - ./data/master:/data
  slave1:
    image: redis
    container_name: redis-slave-1
    restart: always
    command: redis-server --slaveof master 6379 --port 6380  --requirepass slave123 --masterauth master123  --appendonly yes
    ports:
      - 6380:6380
    volumes:
      - ./data/slave1:/data
  slave2:
    image: redis
    container_name: redis-slave-2
    restart: always
    command: redis-server --slaveof master 6379 --port 6381  --requirepass slave456 --masterauth master123  --appendonly yes
    ports:
      - 6381:6381
    volumes:
      - ./data/slave2:/data
```

```
其中:
--slaveof master 6379 设置从节点的关键语句，master指主节点的hostname
--requirepass slave456 设置密码
--appendonly yes 开启持久化，默认是AOF 模式
以上可以写到配置文件中
```

执行命令 `docker-compose up -d` 运行

# Redis 哨兵模式

# Redis 集群模式