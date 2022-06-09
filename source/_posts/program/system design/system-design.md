---
title: System Design
toc: true
date: 2022-06-09 15:14:08
tags:
---

## 设计流程

### 1.需求澄清

- 1.1 功能性需求
  - use case
    - 提炼实体对象
    - 划分业务模块
- 1.2 非功能性需求
  - 高性能
  - 高可扩展性
  - 高可靠
  - 高可维护性

### 2.架构设计

- data model
- api
- 架构图diagram

### 3.深入重点

- 选择关键模块进行阐述
- 与面试官确认关注的重点

### 4.找到系统瓶颈

- 一般是针对非功能性需求进行阐述,选择最重要的一点进行阐述
- 阐述系统可扩展性,如何支撑未来的用户增长

### 5.总结阐述

- 阐述做出当前设计的考量
- 是否还有其他可选的设计

## 常见问题

- 设计一个海量的评论系统
- 设计一个短链接生成系统？数据如何存储？高并发如何处理？
- 设计一个秒杀系统
- 设计一个微信朋友圈系统，列出主要的表结构，只需要实现一些基础的功能，比如聊天列表等
- 设计一个海量日志写入系统(可参考 cat)
- 设计一个酒店预订系统

## 参考资料

- 1.DDIA:强烈推荐读一读尤其是一些重点章节可以考虑多读几遍楼主读了两遍很多东西依然没有完全理解
- 2.YoutubeScottShi视频
- 3.AlexXuSystemDesignInterviewbook:最大的收获是过了一边各种题型和学到了一些基本技巧
- 4.Techblog:对不同的系统可以搜一搜对应的techlog往往可以得到很多启发而且经过production考验的设计往往比自己空想出来的设计更加solid
- 5.Paper:我对paper的建议是如果不是面infra职位其实看infra相关的paper未必是性价比最高的如果面infra时间允许以及自己也很有兴趣想深入学习一下仔细研读一下paper收获挺大的一些infra相关的比较不错的paper：MapreduceBigtableGooglefilesystemFacebookmemcacheFacebookTAOGoogleSpannerKafkaZookeeperAmazondynamoAmazonAurora
