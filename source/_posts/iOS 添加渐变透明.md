---
layout: git
title: iOS 添加渐变透明
date: 2022-12-04 20:00:00
tags: OC
--- 

# iOS 添加渐变透明

```
[self addAlphaMask:self.view.layer]

- (void)addAlphaMask:(CALayer *) targetLayer
{
    // 设置顶部渐隐层
    CAGradientLayer *gradientLayer = [[CAGradientLayer alloc] init];
    gradientLayer = [CAGradientLayer layer];
    gradientLayer.startPoint = CGPointMake(0, 0); //渐变色起始位置
    gradientLayer.endPoint = CGPointMake(0, 0.4); //渐变色终止位置
    gradientLayer.colors = @[(__bridge id)[UIColor.clearColor colorWithAlphaComponent:0].CGColor, (__bridge id)
    [UIColor.clearColor colorWithAlphaComponent:1.0].CGColor];
    gradientLayer.locations = @[@(0), @(1.0)]; // 对应的位置（分割线）
    gradientLayer.frame = targetLayer.bounds;
    targetLayer.mask = gradientLayer;
}

```