---
title: '#Algorithm 二分总结'
date: 2021-09-01 10:18:16
tags:
---
# 二分法
---
左右边界，取中位数查看

**防止溢出**：mid = l + (r - l) // 2

```
while L < R:  # 在L = R时停止
    if mid == target:
        R = mid
    if mid < target:
        L = mid + 1 # 左边界收缩
    else:
        R = mid

```

