---
layout: git
title: OC正则匹配
date: 2021-11-30 20:00:00
tags: OC
---

# OC 正则匹配


```
NSError* error = NULL;
NSRegularExpression* regex = [NSRegularExpression regularExpressionWithPattern:@"(encoding=\")[^\"]+(\")" options:0 error:&error];
NSString* sample = @"<xml encoding=\"abc\"></xml><xml encoding=\"def\"></xml><xml encoding=\"ttt\"></xml>";
NSLog(@"Start:%@",sample);
NSString* result = 
    [regex stringByReplacingMatchesInString:sample
                                    options:0
                                      range:NSMakeRange(0, sample.length)                 
                               withTemplate:@"$1utf-8$2"];
NSLog(@"Result:%@", result);
```
``` 
输出：
Start:<xml encoding="abc"></xml><xml encoding="def"></xml><xml encoding="ttt"></xml>
Result:<xml encoding="utf-8"></xml><xml encoding="utf-8"></xml><xml encoding="utf-8"></xml>
```

* 通过正则表达式 `encoding=\"[^\"]+\"` 匹配
* `[^\"]+` 匹配任意数量的非`"`字符
* 加上 `()` 用来后续 withTemplate 的匹配，`$1` `$2` 可用于匹配对应的第几个括号（从 1 开始）
* 共匹配到三处，并修改