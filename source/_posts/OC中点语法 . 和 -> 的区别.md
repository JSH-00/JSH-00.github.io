---
layout: git
title: OC 中点语法 . 和 -> 的区别
date: 2021-02-22 20:00:00
tags: OC
---
# OC 中点语法 . 和 -> 的区别
1. `->` 用于访问成员变量，但成员变量默认受保护，所以常常报错，手动设为public即可解决。
2. `.` 用于访问类的属性，本质是调用setter、getter方法。

## 成员变量不实现 getter 和 setter 方法
* 定义时需要使用 `@public` 才能被外部访问
* 外部使用 `->` 用于访问成员变量

```
// Test.h
@interface Test : NSObject
{
@public
    int temp;
}
@end
```

```
// mian 函数
    Test *test = [[Test alloc]init];
    test->temp = 100; // 注释 @public 则 temp 无法访问
    NSLog(@"%d",test->temp);
```
## 成员变量实现 getter 和 setter 方法
* 外部使用 `.` 访问 getter 和 setter 方法，类内部使用 `->`

```
// Test.h
@interface Test : NSObject
{
    int temp;
}
- (void)setTemp:(int)temp;
- (int)temp;
@end
```

```
// Test.m
@implementation Test
-(void)setTemp:(int)temp
{
    self->temp = temp;
}
-(int)temp
{
    return self->temp;
}
@end
```

```
// mian 函数
    Test *test = [[Test alloc]init];
    test.temp = 100;
    NSLog(@"%d",test.temp);
```