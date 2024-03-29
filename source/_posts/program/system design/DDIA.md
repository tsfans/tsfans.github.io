---
title: Design Data-Intensive Application
toc: true
date: 2022-05-13 09:14:04
tags: ddia
categories: system-design
---

## Chapter 1 可靠、可扩展与可维护的应用系统

- Reliable
- Scalable
- Maintainable
  - 可运维性
  - 简单性
  - 可演化性

## Chapter 2 数据模型与查询语言

### 关系模型

- 主要支持一对多
- 多对一
- 多对多
- 强关联
- 显式的写时模式(字段是定义好的)

### 文档模型

- 主要支持一对多, 或者无关联
- 弱关联
- 隐式的读时模式(由应用代码决定字段解析)

### 图状数据模型

- 支持关联关系复杂
- 复杂关联

### 数据查询语言

## Chapter 3 数据存储与检索

### 索引 - 读写, 时间空间的权衡

- 哈希索引
  - 优点: 等值查询快, 写入快
  - 缺点: 需要全部加载到内存中, 不支持区间查询

- SSTables和LSM-Tree
  - SSTables 排序字符串表
  - LSM-Tree Log-Structured Merge-Tree 日志结构的合并树
  - LSM存储引擎: 基于合并和压缩排序文件
  - 查询: 因为key有序, 因此无需加载所有key到内存中, 稀疏加载即可, 且支持范围查询
  - 写入: 内存中维护有序数据结构(红黑树,AVL树)
    - 1.写入内存中的红黑树, 并同步写磁盘日志(无需有序,用于崩溃后恢复内存表)
    - 2.内存大于阈值, 写SSTables文件
    - 3.读取顺序: 内存 -> 最新磁盘段文件 -> 次新...
    - 4.后台周期性执行合并(类似归并排序,保留最新值,丢弃旧值),压缩段文件(稀疏内存索引指向每个压缩段的开头)
  - 性能优化: 查询不存在的键
    - 布隆过滤器
    - SSTables压缩策略
      - 分层压缩(LevelDB,RocksDB): 分裂为多个更小的SSTables, 旧数据移到单独的层级, 可以逐步压缩以节省磁盘空间
      - 大小分级压缩(HBase): 合并较新,较小到较旧, 较大的SSTables中

- B-trees
  - key有序, 按页读取/写入数据
  - 拥有N个键的树高度总是为O(logN)
  - WAL(Write-ahead Log) redo log: 崩溃恢复
  - 原地覆盖更新(仅追加写入), 使用锁存器保证并发写安全
  - 优化措施
    - 写时复制: 修改的页写入不同的位置, 创建父页新版本, 指向新位置
    - B+ Tree: 只在叶子节点保存value

- 其他索引
  - 二级索引
  - 聚簇索引
  - 联合索引
  - 多维索引
    - 经纬度查询:空间索引
  - 全文搜索和模糊索引

- 内存数据库
  - 读写在内存中, 性能更高
  - 可以提供高级数据结构(Redis的List,Set等)
  - NVM(non-volatile memory) 非易失性内存

### 事务处理与分析处理

- OLTP online transaction processing 在线事务处理
  - 面向用户, 接受大量请求, 每个查询只涉及少量记录
  - 存储引擎通过索引查询数据, 磁盘寻道是瓶颈
  - 存储引擎
    - 日志结构类. 追加更新, 删除过时文件, 不会修改已写入的文件. (BitCask, SSTables, LSM-tree, LEvelDB, RocksDB, Cassandra, HBase, Lucene)
      - 核心是将随机写转化为顺序写, 可以实现更高的吞吐量
    - 原地覆盖类. 将磁盘视为可以覆盖的一组固定大小的页. (B-Tree)

- OLAP onlie analytic processing 在线分析处理
  - 面向业务分析人员, 请求量小但每次查询数据量大,磁盘带宽是瓶颈
  - 常见解决方案: 面向列的存储
  
- 数据仓库
  - OLTP的只读副本
  - 通过ETL(Extract-Transform-Load提取-转换-加载)导入
  - TODO

## Chapter 4 数据编码(序列化)与演化

### 数据编码(序列化)格式(数据传递的格式)

- 数据编码(序列化)
  - 序列化: 内存表示到字节序列
  - 反序列化: 字节序列到内存表示

- 序列化格式
  - 语言特定: java serializable
    - 缺点:
      - 绑定语言
      - 反序列化存在安全问题
      - 版本兼容性差
      - 效率较低
  - JSON XML 二进制变体
    - 优点: 可读, 模式可选, 较灵活
    - 缺点:
      - XML过于冗长
      - JSON不支持大数字(>2^53)
      - JSON, XML不支持二进制字符串
  - 二进制编码
    - Apache Thrift
    - Protocol Buffers
    - Apache Avro
    - 优点
      - 更紧凑, 节省网络带宽
      - 携带模式信息, 方便检测兼容性
    - 缺点: 不可读

### 数据流模式-多进程传递数据的方式

- 基于数据库的数据流
  - 考虑字段变动的兼容
  - 写入时编码, 读取时解码

- 基于服务调用(REST, RPC)
  - REST: 简洁, 通用, 适用于外部服务
  - RPC: 定制, 更高效, 适用于内部服务

- 基于消息队列
  - 优点
    - 消除峰值
    - 自动重试
    - 支持一对多广播
    - 服务间解耦

## PART2 分布式数据系统

- 分布式设计的目的
  - 扩展性
  - 容错与高可用性
  - 延迟考虑

- 分布式数据保存的常见方式
  - 复制: 多节点保存多份相同的数据副本
  - 分区: 将数据拆分到不同节点分区上(分片)

## Chapter 5 数据复制

  通过数据复制达到以下目的:

- 地理位置上接近用户, 降低延迟
- 部分故障时系统仍可用, 提高可用性
- 扩展多节点同时提供数据访问服务, 提高读吞吐量

### 主节点与从节点(主从复制)

- 半同步复制: 始终保持一个从节点同步复制, 其他从节点异步复制
  - 优点: 始终有两个最新的节点
  - 缺点: 影响主节点写入性能
- 全异步复制: 所有从节点都是异步复制
  - 优点: 主节点写入性能最高, 系统吞吐量大
  - 缺点: 主节点宕机, 未同步数据会丢失

配置新的从节点

- 快照主节点
- 拷贝快照到新的从节点
- 从节点连接到主节点请求快照点后的数据变更
- 追赶变更

处理节点失效

- 从节点失效
  - 追赶式恢复
- 主节点失效
  - 节点切换
  - 自动切换
    - 1.确认主节点失效. 一般基于心跳超时
    - 2.选主. 共识算法
    - 3.重新配置主节点. 更新请求路由
  - 自动切换可能的问题
    - 1.新主节点与原主节点发生写冲突, 异步复制模式下部分写请求可能未同步到新主节点上, 一般丢弃未复制的写请求
    - 2.丢弃数据风险极大, 如果数据库之外存在依赖数据库内容协同使用的系统, 可能出现意外错误
    - 3.可能遇到脑裂问题: 多个从节点都认为自己是主节点
    - 4.如何设置合适的超时时间来检测主节点失效?

主从复制复制日志的实现

- 1.基于语句的复制. 需要替换非确定性函数
  - 优点: 简单直接
  - 缺点: 部分场景不适用, 需要特殊处理(非确定性函数, 自增列, 有副作用的语句(触发器,存储过程等))
- 2.基于预写日志(WAL)传输
  - 优点: 日志结构类和Btree类存储引擎都有WAL日志, 天然支持日志同步
  - 缺点: WAL日志过于底层, 与存储引擎耦合严重, 新旧版本可能不兼容, 有可能需要停机升级 
- 3.基于行的逻辑日志复制
  - 优点: 日志格式与存储引擎解耦, 易于向后兼容, 便于外部系统获取解析日志(变更数据捕获), 例如: MySQL binlog
- 4.基于触发器的复制
  - 优点: 应用层定制逻辑, 更灵活
  - 缺点: 性能差, 容易出错

### 处理复制滞后

读自己的写read-after-write

- 问题原因：数据还未同步到从节点，导致读不到刚写入的数据
- 解决方案：
  - 1.强制从主节点读
    - 优点: 逻辑简单
    - 缺点: 只适用于部分场景, 大量应用会丧失读操作的可扩展性
  - 2.选择性读主节点
    - 优点: 满足读写一致的前提下尽可能提升读操作的可扩展性
    - 缺点: 实现逻辑较复杂, 需要处理边界条件
    - 方式
      - 记录更新时间. 小于一定阈值走主节点, 并监控从节点复制滞后程度, 避免从滞后过多的从节点多
      - 客户端上送更新时间戳. 查询获取数据时间戳不够新则走主节点
      - 副本分布在多数据中心时需要先把请求路由到主节点所在的数据中心
    - 考虑多设备的读写一致性
      - 更新时间戳需要共享
      - 不同网络接入请求路由到的数据中心不可控

单调读

- 问题原因：两次查询发生在不同的从节点，由于复制滞后导致查询结果不一致
- 解决方案
  - 每个用户从固定副本读数据，比如基于用户id取模

前缀一致读

- 对于一系列按照某个顺序发生的写请求，那么读取这些内容时也要按照当时写入的顺序
- 问题原因：分区数据经多副本复制后出现了不同程度的滞后，导致观察者先看到果后看到因

### 多主复制

### 无主复制




## 参考资料
> - []()
> - []()
