---
layout: git
title: _cmd、objc_getAssociatedObject、objc_setAssociatedObject
date: 2021-12-25 20:00:00
tags: OC
---
# _cmd、objc_getAssociatedObject、objc_setAssociatedObject

## 参数含义

```
//关联对象
 objc_setAssociatedObject(self, @selector(btnAction:), block, OBJC_ASSOCIATION_RETAIN_NONATOMIC);
// self 关联的类，
//key：要保证全局唯一，key与关联的对象是一一对应关系。必须全局唯一。通常用@selector(methodName)作为key。
//value：要关联类的对象。
//policy：关联策略。有五种关联策略。
//OBJC_ASSOCIATION_ASSIGN 等价于 @property(assign)。
//OBJC_ASSOCIATION_RETAIN_NONATOMIC等价于 @property(strong, //nonatomic)。
//OBJC_ASSOCIATION_COPY_NONATOMIC等价于@property(copy, nonatomic)。
//OBJC_ASSOCIATION_RETAIN等价于@property(strong,atomic)。
//OBJC_ASSOCIATION_COPY等价于@property(copy, atomic)。

 btnBlock block= objc_getAssociatedObject(self, _cmd);
//获取关联对象的信息，_cmd 就是key

```

```
objc_setAssociatedObject 相当于 setValue:forKey 进行关联value对象

objc_getAssociatedObject 用来读取对象

objc_AssociationPolicy  属性 是设定该value在object内的属性，即 assgin, (retain,nonatomic)...等

 objc_removeAssociatedObjects 函数来移除一个关联对象，或者使用objc_setAssociatedObject函数将key指定的关联对象设置为nil。

```
```

- (void)someCategoryMethod
{
    NSString *extendVar = objc_getAssociatedObject(self, _cmd);
    if(!extendVar){
        extendVar = @"someText";
        objc_setAssociatedObject(self, _cmd, extendVar, OBJC_ASSOCIATION_COPY_NONATOMIC);
    }
}
```

关联方法
* _cmd 在 Objective-C的方法中表示当前方法的selector，正如同self表示当前方法调用的对象实例。


```
- (CustomNavigationControllerDelegate *)customDelegate
{
    return objc_getAssociatedObject(self, _cmd);
}

- (void)setCustomDelegate:(CustomNavigationControllerDelegate *)customDelegate
{
    objc_setAssociatedObject(self, @selector(customDelegate), customDelegate, OBJC_ASSOCIATION_RETAIN_NONATOMIC);
}
```

* objc_getAssociatedObject、objc_setAssociatedObject 还可以给分类添加属性，手动写 getter 和 setter 方法。


参考：
https://blog.csdn.net/songbai1211/article/details/101393934
https://www.jianshu.com/p/fdb1bc445266