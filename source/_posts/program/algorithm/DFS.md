---
title: Depth First Search
toc: true
date: 2022-05-13 09:02:38
tags: DFS
categories: algorithm
---

- 解题框架

```java
result = []
def backtrack(路径, 选择列表):
    if 满足结束条件:
        result.add(路径)
        return
    
    for 选择 in 选择列表:
        做选择
        backtrack(路径, 选择列表)
        撤销选择
```



## 常见问题
