---
title: SystemDesign-grok-
date: 2021-09-18 11:03:55
tags:
    - System Design
---

# Grok System Design

## step by step guid

1. 需求明确
    - 即能问出关于系统的需求问题 - 功能方面
    - 例如 文件形式 图片/视频； 搜索功能； 前后端侧重；

2. 接口设计
    - 接口方法设计，方法名，参数

3. 规模预估
    - 考虑方面： **扩展，分片，负载均衡，缓存**
    - 基础数据： 用户量，访问量，数据量
    - 推断数据： 存储，带宽

4. 数据模型
    - 设计系统内部数据传输时 使用的模型
    - 影响 **数据库选择，数据分割，数据管理**
    - SQL / NoSQL
    - 
5. 高层次模块设计
   - 功能模块 要覆盖全面
  
6. 详细设计
   - 对其中2-3个模块进行详细设计
   - 多种方法解决，并说出pros & cons，以及当前场景适合的方法
   - 大数据量 -> 是否分区 （性能，容量/花费，全表连表查询，事务性需要应用层实现，负载要求高）-> 水平(分片)，垂直（字段分割），功能 
   - 特殊情况分析 - 特殊高访问数据 / 高使用度功能特殊优化(most recent post)
   - 缓存，负载均衡

7. 瓶颈识别和解决
   - single failure handling - 高可用，高可靠
   - 副本备份 - 可用，性能
   - monitor监控，performance性能，alert警告


### Summary

    需求明确 -> 规模预估（频率 + 数量） ->（存储 + 带宽 + 负载 + 性能） 瓶颈解决 （可用，稳定，监控）

    接口 -> 数据 -> 模块 -> 高频场景功能 设计

------
# 分布式系统要素

- Scalability 可扩展性
  - 便于扩展 且不影响性能
  - 系统，功能，存储，网络 层面
  - 横向 - 加数量 Cassandra Mongo/ 纵向 - 加性能 Mysql
  
- Reliability 可靠性
  - single point fail
  - 不回由于几台机器或服务的下线而影响整体功能
  - 安全性
  - redundancy & replica
  
- Availability 可用
  - 可长时间持续运行
  - maintanence
  - Reliability -> 一定 availability， Available 不一定 reliable
  
- Efficientcy 效率
  - response time / latency 延迟
  - bandwidth / throughput 吞吐 带宽
  
- Manageability Serviceability
  - 可简单 使用 管理 维护

## Load Balancing

    prevent single point failer
    1. between user and web server
    2. between internal layer and web server
    3. between internal layer and data base

    好处
    1. 防止单点失败影响用户，less downtime， higher throughput
    2. 减少wait time，便于维护和管理
    3. 智能的LB可以分析瓶颈和处理预测信息
    4. 减少服务器负载，减少服务器损耗

    算法 - 分配方式

    health check first
    持续链接
    - least connection 转到连接数最少的服务器 
    - least response time 转到连接数最少且平均回复时间最短的
    - least bandwidth 转到现在已用带宽最少的
    短快的链接
    - round robin 按顺序均分
    - weighted round robin 权重
    - IP HASH 根据client IP随机分配

## Cache 
- Application server Cache
  - 解决硬盘
  - 分布式 - distributing cache / global cache
  - 分布式 cache 对于stateful request 会有问题
- Content Distributing Network CDN
  - 解决跨区域server静态资源
  - local资源中转暂存站
  - 可用subdomain的轻型HTTP服务器暂时替代
- Cache invalid
  - 数据更新
  - 老数据过期
  - write through - 更新数据写两次，cache和DB - high latency for write
  - write around - 跳过cache，所以会有cache miss
  - write back - 先写入cache，然后disk，有丢失风险。low latency high throughput
- Cache eviction
  - FIFO
  - LIFO 后进先出
  - LRU Least recent use 
  - MRU Most recent user 时间最近
  - LFU Frequency 最不常用
  - MFU 最常用
  - RR random随机
  ### cache相关问题
  ## 1. 缓存穿透 - 某个**不存在的数据** 被大规模访问 导致不断db查询
    - 解决：布隆过滤器过滤空值要求 / 或者直接缓存空值
  ## 2. 缓存雪崩 - **某个时间** 大规模的key集体失效 导致DB访问量大增
    - 解决：缓存设置时间时 外加随机值 / 或者物理永不过期，用过期时间放在value里功能逻辑检查更新
  
    事前：

    ① 均匀过期：设置不同的过期时间，让缓存失效的时间尽量均匀，避免相同的过期时间导致缓存雪崩，造成大量数据库的访问。

    ② 分级缓存：第一级缓存失效的基础上，访问二级缓存，每一级缓存的失效时间都不同。

    ③ 热点数据缓存永远不过期。

    ④ 保证Redis缓存的高可用，防止Redis宕机导致缓存雪崩的问题。可以使用 主从+ 哨兵，Redis集群来避免 Redis 全盘崩溃的情况。

    事中：

    ① 互斥锁：在缓存失效后，通过互斥锁或者队列来控制读数据写缓存的线程数量，比如某个key只允许一个线程查询数据和写缓存，其他线程等待。这种方式会阻塞其他的线程，此时系统的吞吐量会下降

    ② 使用熔断机制，限流降级。当流量达到一定的阈值，直接返回“系统拥挤”之类的提示，防止过多的请求打在数据库上将数据库击垮，至少能保证一部分用户是可以正常使用，其他用户多刷新几次也能得到结果。

    事后：

    ① 开启Redis持久化机制，尽快恢复缓存数据，一旦重启，就能从磁盘上自动加载数据恢复内存中的数据。

  ## 3. 缓存击穿 - 某个**高访问量的key失效** 导致DB访问量大增
    - 解决：
      - 永不过期
        - 物理上不过期
        - 逻辑上更新
      - mutex锁 - 锁住load DB 操作，缓存先去fetch data
      ```java
      public String get(key) {
      String value = redis.get(key);
      if (value == null) { //代表缓存值过期
          //设置3min的超时，防止del操作失败的时候，下次缓存过期一直不能load db
		  if (redis.setnx(key_mutex, 1, 3 * 60) == 1) {  //代表设置成功
               value = db.get(key);
                      redis.set(key, value, expire_secs);
                      redis.del(key_mutex);
              } else {  //这个时候代表同时候的其他线程已经load db并回设到缓存了，这时候重试获取缓存值即可
                      sleep(50);
                      get(key);  //重试
              }
          } else {
              return value;      
          }
      ```
| 解决方案| 优点|缺点|
| ------------- |:-------------:| -----:|
|简单分布式互斥锁（mutex key）|  1. 思路简单 2. 保证一致性 |  1. 代码复杂度增大 2. 存在死锁的风险 3. 存在线程池阻塞的风险|
|“提前”使用互斥锁|遇到有value timeout，先延长，同时去load和更新新的value|
|不过期(本文)|异步构建缓存，不会阻塞线程池|不保证一致性。2. 代码复杂度增大(每个value都要维护一个timekey)。3. 占用一定的内存空间(每个value都要维护一个timekey)。|
|资源隔离组件hystrix(本文)| hystrix技术成熟，有效保证后端。2. hystrix监控强大。|部分访问降级|

  ## 4.缓存预热
  提前缓存数据，根据优先级
  ## 5.缓存降级
  缓存降级是指缓存失效或缓存服务器挂掉的情况下，不去访问数据库，直接返回默认数据或访问服务的内存数据。降级一般是有损的操作，所以尽量减少降级对于业务的影响程度。

  > 在项目实战中通常会将部分热点数据缓存到服务的内存中，这样一旦缓存出现异常，可以直接使用服务的内存数据，从而避免数据库遭受巨大压力。


### hystrix
根据依赖对线程进行管理

不同依赖有各自的线程池

