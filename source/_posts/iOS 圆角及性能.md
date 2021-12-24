---
layout: git
title: iOS 圆角及性能
date: 2021-11-07 20:00:00
tags: OC
---
# iOS 圆角及性能

## 普通圆角添加方式
```
UIImageView *avatarImageView = [[UIImageView alloc]initWithFrame:CGRectMake(100, 100, 100, 100)];
avatarImageView.backgroundColor = [UIColor redColor];
avatarImageView.clipsToBounds = YES;
[avatarImageView.layer setCornerRadius:50];
[self.view addSubview:avatarImageView];
```

## 加入光栅化
* 设置光栅化，可以使离屏渲染的结果缓存到内存中存为位图，使用的时候直接使用缓存，节省了一直离屏渲染损耗的性能。
```
avatarImageView.layer.shouldRasterize = YES;  
avatarImageView.layer.rasterizationScale=[UIScreen mainScreen].scale; // UIImageView不加这句会由于光栅化产生一点模糊

```

## 图片覆盖
* 直接覆盖一张中间为圆形透明的图片（推荐使用）
* GPU计算多层的混合渲染blending也是会消耗一点性能的，但比普通圆角添加方法还是好上很多的

## UIImage drawInRect绘制圆角
* 这种方式 GPU 损耗低内存占用大，而且主要用在 UIImageView 上，可用 UIimageView 添加个点击手势当做 UIButton 使用。

## SDWebImage处理图片时Core Graphics绘制圆角
```
  //UIImage绘制为圆角
  int w = imageSize.width;
  int h = imageSize.height;
  int radius = imageSize.width/2;
  
  UIImage *img = image;
  CGColorSpaceRef colorSpace = CGColorSpaceCreateDeviceRGB();
  CGContextRef context = CGBitmapContextCreate(NULL, w, h, 8, 4 * w, colorSpace, kCGImageAlphaPremultipliedFirst);
  CGRect rect = CGRectMake(0, 0, w, h);
  
  CGContextBeginPath(context);
  addRoundedRectToPath(context, rect, radius, radius);
  CGContextClosePath(context);
  CGContextClip(context);
  CGContextDrawImage(context, CGRectMake(0, 0, w, h), img.CGImage);
  CGImageRef imageMasked = CGBitmapContextCreateImage(context);
  img = [UIImage imageWithCGImage:imageMasked];
  
  CGContextRelease(context);
  CGColorSpaceRelease(colorSpace);
  CGImageRelease(imageMasked);
```

## 用Instruments测试

第一种方法，UIImageView和UIButton都高亮为黄色。

第二种方法，UIImageView和UIButton都高亮为绿色

第三种方法，无任何高亮，说明没离屏渲染。
这种圆片覆盖的方法一般只用在底色为纯色的时候，如果圆角图片的父View是张图片的时候就没办法了，而且底色如果是多种颜色的话那要做多张不同颜色的圆片覆盖。（可以用代码取底色的颜色值给圆片着色）

第四种方法无任何高亮，说明没离屏渲染（但是CPU消耗和内存占用会很大）

第五种方法无任何高亮，说明没离屏渲染，而且内存占用也不大。(暂时感觉是最优方法)


[参考](https://www.jianshu.com/p/34189f62bfd8)