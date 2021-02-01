---
layout: git
title: UISegmentedControl
date: 2021-02-01 13:00:00
tags: OC
---
# UISegmentedControl

```
    UISegmentedControl *topTitle = [[UISegmentedControl alloc]initWithItems:@[@"推荐", @"附近"]];
    topTitle.frame = CGRectMake(self.view.bounds.size.width * 0.25, self.view.safeAreaInsets.top +44, self.view.bounds.size.width * 0.5, 30);
    [topTitle setBackgroundColor:[UIColor greenColor]];
    [self.view addSubview:topTitle];
    
    // 设置文字选中样式
    NSMutableDictionary *attDicSelected = [NSMutableDictionary dictionary];
    attDicSelected[NSFontAttributeName] = [UIFont boldSystemFontOfSize:16];
    attDicSelected[NSForegroundColorAttributeName] = [UIColor greenColor];
    [topTitle setTitleTextAttributes:attDicSelected forState:UIControlStateSelected];
    
    // 设置文字未选中样式
    NSMutableDictionary *attDicNormal = [NSMutableDictionary dictionary];
    attDicNormal[NSFontAttributeName] = [UIFont boldSystemFontOfSize:16];
    attDicNormal[NSForegroundColorAttributeName] = [UIColor whiteColor];
    [topTitle setTitleTextAttributes:attDicNormal forState:UIControlStateNormal];
    
     // 事件
     topTitle.selectedSegmentIndex = 0; // 默认光标所在位置
     [titleV addTarget:self action:@selector(titleViewChange:) forControlEvents:UIControlEventValueChanged];
```