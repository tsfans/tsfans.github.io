---
title: Two Pointer
toc: true
date: 2022-06-09 14:48:08
tags: two-pointer
categories: algorithm
---

## 左右指针

## 快慢指针

## 滑动窗口

```java
public void slidingWindow(String s, String t){
    Map<Character, Integer> window = new HashMap<>();
    Map<Character, Integer> target = new HashMap<>();
    for(char c : t.toCharArray()){
        target.put(c, target.getOrDefault(c, 0) + 1);
    }
    int left = 0, right = 0, match = 0;
    while(right < s.length()){
        char ac = s.charAt(right);
        right++;
        // update window

        // shrink condition
        while(match == target.size()){
            // update result
            char rc = s.charAt(left);
            left++;
            // update window
        }
    }
}
```

## 常见问题
