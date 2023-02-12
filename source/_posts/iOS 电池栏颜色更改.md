---
layout: git
title: iOS 电池栏颜色更改
date: 2022-09-10 20:00:00
tags: OC
---

# iOS 电池栏颜色更改

### 获取外观
```
if (@available(iOS 13.0, *)) {
    UIUserInterfaceStyle mode = UITraitCollection.currentTraitCollection.userInterfaceStyle;
    if (mode == UIUserInterfaceStyleDark) {
        NSLog(@"深色模式");
    } else if (mode == UIUserInterfaceStyleLight) {
        NSLog(@"浅色模式");
    } else {
        NSLog(@"未知模式");
    }
}
```

### 电池栏更改颜色

> info.plist
> View controller-based status bar appearance
> NO

> Status bar style
> 黑底白字/白底黑字
> Dark Content/Light Content