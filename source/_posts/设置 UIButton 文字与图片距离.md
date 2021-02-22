---
layout: git
title: 设置 UIButton 文字与图片距离
date: 2021-02-22 20:00:00
tags: OC
---
# 设置UIButton图片大小、文字与图片距离

```  
    CGFloat padding = 0; // 设置图片与文字的距离
    CGFloat zoom = 8; // 设置图片缩放大小
    btn.titleEdgeInsets = UIEdgeInsetsMake(0, padding/2, 0, -padding/2);
    btn.imageEdgeInsets = UIEdgeInsetsMake(zoom, -padding/2 - 16, zoom, padding/2);
    [btn.imageView setContentMode:UIViewContentModeScaleAspectFit]; // 使图片缩放后不变形
    btn.adjustsImageWhenHighlighted = NO; // 点击后button图片不变灰
```