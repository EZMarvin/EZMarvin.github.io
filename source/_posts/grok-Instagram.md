---
title: grok-Instagram
date: 2021-10-27 10:44:13
tags:
---

# Instagram

## requirement

- Post [text + image] 
  - upload & download
  - search

- follow user
- update feed
  
- comment
- like 

## throughput

read heavy

photo storage,backup

---

500M User, 1M/DAY active

2M photo/day 2M / 86400 = 23 photo/second

---

storage - photo 200 KB, 4.6MB/S 400GB/day 146TB/year 1460TB-10year
cache - 400 * 0.2 = 80GB

## DB design

User - id | date | 

Follow - userID | UserID

Post - postID | userID

S3 - key - photo

- each table size 

## Component design

seperate read and write

two seperate service

## Data sharding 数据分片
- 按照 USERID - UserID % 10 分10片
  - problem
    - 某些高访问用户会集中
    - 某些用户会拥有大量照片
    - 一个用户所有照片存在1个shard(one point fail)或者分布在多个(latency)

- 按照 PHOTOID - 各自increment - shardID + photoID
  - dedicate ID DB, store all PhotoID
  - KEY generate Service
  - problem
    - 防止key 生成服务 下线，multiple service

- future growth
  - logical partition
  - one DB server with multiple instance
  - when one partation too large -> change config

## News feed - Main function

- 基本逻辑
  - 整理每个用户关注的User，然后从每个User处获得最新的100条feed
  - 汇总所有feed，用某种rank或者筛选方式得到最终结果
  - 需要pre-process， 提前处理得到每个USER 的 feed，并储存
  
- Pull
  - client 端 不断向server 请求新的数据
  - problem
    - 新feed会一直等待用户请求
    - 关注用户少，feed少的。可能会多次request都没有结果

- PUSH
  - server 主动向client端 推送feed， long polling
  - problem
    - 关注用户多，feed频繁的。会持续的push

- 混合方式 hybrid
  - 根据用户类型， 大用户就用pull, 小用户就用push
  - 或服务器会以一定频率push，对于大用户就需要自己额外去pull request

## 结合Data Shard

time in second + increment

2 ^ 31 + 2 ^ 9 = 2billion second + 512 photo

## cache & load balance
CDN - geographically cache

LRU - meta data cache

2-8 - 20% popular photo cache
