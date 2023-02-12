---
layout: git
title: iOS 比较 APP 版本号
date: 2022-10-2 20:00:00
tags: OC
--- 

# iOS 比较 APP 版本号

```  
   NSString *showVersion = @"1.6.0";
   NSString *lastAppVersion =  @"1.5.9";
    BOOL isLowerThanShowVersion = ([lastAppVersion compare:showVersion options:NSNumericSearch] == NSOrderedAscending);
```