---
title: Redis面试
date: 2018-10-08 16:03:17
tags:
categories: Redis
---
# Redis作用
1. 缓存
2. list保存最新的数据 队列系统 lpush ltrim
3. zset保存权重最大的数据 优先级队列系统 zadd zrevrange zrank
4. set数据排重
5. 清除过期数据
6. 计数
7. subscribe/publish构建实时消息系统
