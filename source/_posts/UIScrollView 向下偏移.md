---
layout: git
title: UIScrollView 向下偏移
date: 2021-03-15 20:00:00
tags: OC
---

# UIScrollView 向下偏移
## 遇到问题
* 当 scrollView 是其父视图上的第一个子视图，且 navigationBar 不隐藏的情况下，添加到 scrollView 里的视图，都会默认下移64个像素。
* 继承 UIScrollview 的 UITableview 也会出现这个问题。

## 解决方法

```
if (@available(iOS 11.0, *)){
    self.automaticallyAdjustsScrollViewInsets = NO;
}
```

参考：https://blog.csdn.net/u013196181/article/details/51131625
