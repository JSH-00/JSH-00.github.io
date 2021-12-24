---
layout: git
title: KVC 和 KVO
date: 2021-11-09 20:00:00
tags: OC
---
# KVC和KVO

## KVC 
* Key-Value Coding，键值编码，一种键值对间接访问机制，通过属性名称字符串间接访问属性。

* directly access a property (get set)
*  在某些情况下可以简化代码

```
#pragma mark - KVC

// 相当于 [myAccount setCurrentBalance:@(100.0)];
[myAccount setValue:@(100.0) forKey:@"currentBalance"];

[myAccount setValue:@(100.0) forKeyPath:@"currentBalance.person"];
[myAccount valueForKeyPath:@"currentBalance.person"];
[item setValuesForKeysWithDictionary:dic];
```

若该成员变量也访问不到，则会在下面方法中抛出异常。重写该方法，在内做些处理，防止程序直接崩溃。
```
- (void)setValue:(id)value forUndefinedKey:(NSString *)key{
    if ([key isEqualToString:@"id"]) {
        self.wbid = value;
        //wbid是替代的id属性
    }
}
- (id)valueForUndefinedKey:(NSString *)key {
    return @"";
}
```

## KVO
* Key-Value Observing 提供了一种观察者的机制
* 对目标对象的某属性添加观察，当该属性发生变化时，会自动的通知观察者。这里所谓的通知是触发观察者对象实现的KVO的接口方法。
* addObserver、removeObserving 属性发生改变，系统自动调 observeValueForKeyPath

```
#pragma mark - KVO

/*
 //第一个参数 observer：观察者 （这里观察self.myKVO对象的属性变化）
 //第二个参数 keyPath： 被观察的属性名称(这里观察 self.myKVO.person.num 属性值的改变)
 //第三个参数 options： 观察属性的新值、旧值等的一些配置（枚举值，可以根据需要设置，例如这里可以使用两项）
 //第四个参数 context： 上下文，可以为 KVO 的回调方法传值（例如设定为一个放置数据的字典）
 */
static void *PersonAccountBalanceContext = &PersonAccountBalanceContext;
static void *PersonAccountInterestRateContext = &PersonAccountInterestRateContext;

// 注册监听
[self.myKVO addObserver:self
             forKeyPath:@"person.num"
                options:NSKeyValueObservingOptionOld|NSKeyValueObservingOptionNew
                context:nil];

// 结束监听，不要忘记解除注册，否则会导致资源泄露
[self.myKVO removeObserver:self forKeyPath:@"person.num" context:PersonAccountBalanceContext];


- (void)observeValueForKeyPath:(NSString *)keyPath
                      ofObject:(id)object
                        change:(NSDictionary *)change
                       context:(void *)context {
 
    NSLog(@"%s",__func__);
    if (context == PersonAccountBalanceContext) {
        // Do something with the balance…
        NSLog(@"Do something with the balance…");
        NSLog(@"%@",change);
 
    } else if (context == PersonAccountInterestRateContext) {
        // Do something with the interest rate…
        NSLog(@"Do something with the interest rate…");

    } else {
        // Any unrecognized context must belong to super（可能是被父类注册了）
        [super observeValueForKeyPath:keyPath
                             ofObject:object
                               change:change
                               context:context];
    }
}

```

### KVO 本质
```
原文：
当某个类的对象第一次被观察时，系统就会在运行期动态地创建该类的一个派生类，在这个派生类中重写基类中任何被观察属性的 setter 方法。

派生类在被重写的 setter 方法实现真正的通知机制，就如前面手动实现键值观察那样。这么做是基于设置属性会调用 setter 方法，而通过重写就获得了 KVO 需要的通知机制。当然前提是要通过遵循 KVO 的属性设置方式来变更属性值，如果仅是直接修改属性对应的成员变量，是无法实现 KVO 的。

同时派生类还重写了 class 方法以“欺骗”外部调用者它就是起初的那个类。然后系统将这个对象的 isa 指针指向这个新诞生的派生类，因此这个对象就成为该派生类的对象了，因而在该对象上对 setter 的调用就会调用重写的 setter，从而激活键值通知机制。此外，派生类还重写了 dealloc 方法来释放资源。
```

即：动态地创建该类的一个派生类，并重写了setter方法，在 setter 里面，属性赋值的前后分别调用了两个方法。
```
- (void)willChangeValueForKey:(NSString *)key;
- (void)didChangeValueForKey:(NSString *)key;
```
而 `- (void)didChangeValueForKey:(NSString *)key;` 会调用

```
- (void)observeValueForKeyPath:(nullable NSString *)keyPath ofObject:(nullable id)object change:(nullable NSDictionary<NSString*, id> *)change context:(nullable void *)context;
```

参考：
[KVC和KVO的使用及原理](https://www.jianshu.com/p/66bda10168f1)
