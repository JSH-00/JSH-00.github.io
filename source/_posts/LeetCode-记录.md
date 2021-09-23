---
layout: git
title: LeetCode 记录
date: 2021-09-18 20:00:00
tags: LeetCode
---
# LeetCode 记录

## 二进制特性

### 判断是否是 2 的幂次方
* n 是 2 进制的约数 （非进制方法）
```
return n > 0 && (1<<30) % n == 0
```
* `-n` 与操作
    * -n 是 n 二进制的补码 +1
    
```
return n > 0 && (n & -n) == n;
```

* `(n - 1)` 与操作
    * (n-1)
    
```
return n > 0 && (n & (n - 1)) == 0
```
    

