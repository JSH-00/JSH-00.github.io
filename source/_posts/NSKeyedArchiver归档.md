---
layout: git
title: NSKeyedArchiver 归档
date: 2021-11-03 20:00:00
tags: OC
---
# NSKeyedArchiver 归档

## NSKeyedArchiver 和 NSUserDefault 的区别

* NSUserDefault 只能存储基本数据类型，如 NSInteger NSString NSArray 但像 UIImage、自定义类就无法存入
* NSKeyedArchiver 可以将各种类型的对象存储到文件中，而不仅仅是字符串、数组和字典类型，利用NSKeyedAarchiver 类创建带 key 的档案来完成存入

## 用法
### 服从 NSSecureCoding 协议
 所存储的对象必须必须服从NSSecureCoding协议
 
### 对于已经服从的类型，如NSString、NSInteger可以直接使用
 
NSKeyedArchiver-归档
```
   // 对需要保存的数据进行编码为 NSData
    NSData *data = [NSKeyedArchiver archivedDataWithRootObject:@"ios" requiringSecureCoding:YES error:nil];

    //2.将二进制数据保存到文件
    //创建文件
    NSString *path = [NSHomeDirectory() stringByAppendingPathComponent:@"ios.plist"];
    //创建文件
    [[NSFileManager defaultManager] createFileAtPath:path contents:nil attributes:nil];
    [data writeToFile:path atomically:YES];
```

NSKeyedUnarchiver-解归档

```
    //解归档
    //获取文件路径
    NSString *path = [NSHomeDirectory() stringByAppendingPathComponent:@"ios.plist"];
    //读取文件的内容
    NSData *data = [NSData dataWithContentsOfFile:path];
    //将二进制数据转化为对应的对象类型
   NSString *str = [NSKeyedUnarchiver unarchivedObjectOfClass:[NSString class] fromData:data error:nil];
    NSLog(@"%@", str);
```

### 对于其他类型，如自己创建的模型，需要服从协议，并且实现某些方法


Person.h

```
#import <Foundation/Foundation.h>

NS_ASSUME_NONNULL_BEGIN

@interface Person : NSObject<NSSecureCoding>

@property (nonatomic, strong) NSString *name;
@property (nonatomic, assign) NSInteger age;
@end

NS_ASSUME_NONNULL_END
```

Person.m
```
#import "Person.h"

@implementation Person
// Require in NSSecureCoding protocol.
+ (BOOL)supportsSecureCoding{
    return YES;
}

//归档的时候调用
//告诉编码器该如何归档
//将这个对象哪些属性编码起来
- (void)encodeWithCoder:(NSCoder *)aCoder{
    [aCoder encodeObject:self.name forKey:@"name"];
    [aCoder encodeInteger:self.age forKey:@"age"];
}

//解归档
- (instancetype)initWithCoder:(NSCoder *)aDecoder{
    if ([super init]) {
        self.name = [aDecoder decodeObjectForKey:@"name"];
        self.age = [aDecoder decodeIntegerForKey:@"age"];
    }
    return self;
}
```

ViewController.m 归档

```
//创建Person对象
    Person *jz = [Person new];
    jz.name = @"jz";
    jz.age = 12;
    //1.对需要保存的数据进行编码 ->NSdata *
    NSData *data = [NSKeyedArchiver archivedDataWithRootObject:jz requiringSecureCoding:YES error:nil];
    
    //2.将二进制数据保存到文件
    //创建文件
    NSString *path = [NSHomeDirectory() stringByAppendingPathComponent:@"ios.plist"];
    //创建文件
    [[NSFileManager defaultManager] createFileAtPath:path contents:nil attributes:nil];
    [data writeToFile:path atomically:YES];
```

解归档

```
//解归档
    //获取文件路径
    NSString *path = [NSHomeDirectory() stringByAppendingPathComponent:@"ios.plist"];
    //读取文件的内容
    NSData *data = [NSData dataWithContentsOfFile:path];
    //将二进制数据转化为对应的对象类型
    Person *jz = [NSKeyedUnarchiver unarchivedObjectOfClass:[Person class] fromData:data error:nil];
    NSLog(@"%@", jz);
```

[参考](https://www.cnblogs.com/jianze/p/10778821.html)