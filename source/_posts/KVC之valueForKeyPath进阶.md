---
layout: git
title: KVC之valueForKeyPath进阶
date: 2022-12-10 20:00:00
tags: OC
--- 

# KVC之valueForKeyPath进阶

## KVC 常用方法
`setValue:forKey:`
`setValue: forKeyPath:`
`valueForKey:`
`valueForKeyPath:`
`setValuesForKeysWithDictionary:`

注：`valueForKey`会自动把基本类型转成NSNumber或NSValue中包装成对象,同样,动态设置`setValue: forKey:`的属性也必须先包装成NSNumber对象类型才可以.


## 高阶用法

### 把字典中的key单独生成数组

```
    NSDictionary *dic1 = @{@"city":@"北京",@"count":@"22"};
    NSDictionary *dic2 = @{@"city":@"上海",@"count":@"18"};
    NSDictionary *dic3 = @{@"city":@"深圳",@"count":@"17"};
    
    NSArray *arr = @[dic1,dic2,dic3];
    
    NSLog(@"city:%@",[arr valueForKeyPath:@"city"]);
    NSLog(@"count:%@",[arr valueForKeyPath:@"count"]);
    
```  

```
输出结果为：
    city:(
    "北京",
    "上海",
    "深圳"c4
    )   

    count:(
    22,
    18,
    17
    )
```

### 把数组中的数取整
```
    NSArray *array = [NSArray arrayWithObjects:@"10.11",@"20.22", nil];
    NSArray *resultArray = [array valueForKeyPath:@"doubleValue.intValue"];
    NSLog(@"resultArray:%@", resultArray);
```

```
输出结果为：
    resultArray:(
    10,
    20
    )
```

### valueForKeyPath 运算符
在路径中,可以引用一下运算符 `@xxx` 来进行一些运算,例如获取一组值得平均值,最值或者总数.

1. 常规操作符：`@avg` `@count` `@max` `@min` `@sum`
```
    NSMutableArray *personsArray = [[NSMutableArray alloc] initWithCapacity:5];
    for (NSInteger i = 0; i < 5; i ++) {
        NSString *tempName = [NSString stringWithFormat:@"people%ld",(long)i];
        NSDictionary *personInfoDictionary = @{
                                               @"name" : tempName,
                                               @"age" : @(10 + i),
                                               @"school" : @"Hist"
                                               };
        Person *tempPerson = [Person yy_modelWithDictionary:personInfoDictionary];
        [personsArray addObject:tempPerson];
    }
    NSNumber *count = [personsArray valueForKeyPath:@"@count"];
    NSNumber *sumAge = [personsArray valueForKeyPath:@"@sum.age"];
    NSNumber *avgAge = [personsArray valueForKeyPath:@"@avg.age"];
    NSNumber *maxAge = [personsArray valueForKeyPath:@"@max.age"];
    NSNumber *minAge = [personsArray valueForKeyPath:@"@min.age"];
    NSLog(@"%@  %@  %@  %@  %@",count,sumAge,avgAge,maxAge,minAge);
```
```
输出结果为：
    5  60  12  14  10
```

2. 对象操作符：`@distinctUnionOfObjects`(去掉重复) `@unionOfObjects`(不去重复)，都返回数组
```
NSArray *values = [object valueForKeyPath:@"@distinctUnionOfObjects.value"];
```
* 注：`@distinctUnionOfObjects` 操作符返回被操作对象指定属性的集合并做去重操作，而`@unionOfObjects`则允许重复。如果其中任何涉及的对象为`nil`，则抛出异常。

3. Array和Set操作符: `@distinctUnionOfArrays`、`@unionOfArrays`对象是嵌套型的集合对象

* 普通去重
```
NSArray *array = @[@"name", @"w", @"aa", @"zxp", @"aa"];
 //返回的是一个新的数组
 NSArray *newArray = [array valueForKeyPath:@"@distinctUnionOfObjects.self"];
 NSLog(@"%@", newArray);
```
* 单独取出字典中的 key/value 并去重
```
NSArray myArray = @[@{@300:@"5 min"},
                    @{@900:@"15 min"},
                    @{@1800:@"30 min"},
                    @{@3600:@"1 hour"}];
                    
NSArray *values = [myArray valueForKeyPath: @"@unionOfArrays.@allValues"];
NSArray *keys   = [myArray valueForKeyPath: @"@unionOfArrays.@allKeys"];
```

```
// 相当于如下代码
values = @[@"5 min",@"15 min",@"30 min",@"1 hour"];
keys = @[@300, @900, @1800, @3600];

```



参考链接：
[iOS监听模式之KVO、KVC的高阶应用](https://blog.51cto.com/u_15894905/5896194)
[valueForKeyPath 妙用](https://blog.csdn.net/chusi3843/article/details/100617629)
