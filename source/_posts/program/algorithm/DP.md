---
title: Dynamic Planning
toc: true
date: 2022-05-13 09:03:38
tags:
---

- 解题框架

```java
# 初始化 base case
dp[0][0][...] = base
# 进行状态转移
for 状态1 in 状态1的所有取值：
    for 状态2 in 状态2的所有取值：
        for ...
            dp[状态1][状态2][...] = 求最值(选择1，选择2...)
```



## 常见问题
