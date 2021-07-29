---
layout: git
title: OC 黑魔法
date: 2021-07-29 20:00:00
tags: OC
---
# 黑魔法
使用 category 对类及其子类的某方法进行重写。可用于页面数量统计等需求。

## 举例
重写 viewWillAppear 方法

UIViewController+swizzling.h
```
#import "UIViewController+swizzling.h"
#import <objc/runtime.h>
@implementation ViewController (swizzling)
+ (void)load {
//    [super load];
    // 通过class_getInstanceMethod()函数从当前对象中的method list获取method结构体，如果是类方法就使用class_getClassMethod()函数获取。
    Method fromMethod = class_getInstanceMethod([self class], @selector(viewWillAppear:));
    Method toMethod = class_getInstanceMethod([self class], @selector(swizzlingViewWillAppear));
    /**
     *  我们在这里使用class_addMethod()函数对Method Swizzling做了一层验证，如果self没有实现被交换的方法，会导致失败。
     *  而且self没有交换的方法实现，但是父类有这个方法，这样就会调用父类的方法，结果就不是我们想要的结果了。
     *  所以我们在这里通过class_addMethod()的验证，如果self实现了这个方法，class_addMethod()函数将会返回NO，我们就可以对其进行交换了。
     */

    if (!class_addMethod([self class], @selector(viewWillAppear:), method_getImplementation(toMethod), method_getTypeEncoding(toMethod))) {
        method_exchangeImplementations(fromMethod, toMethod);
    }

}
- (void)swizzlingViewWillAppear {
    NSString *str = [NSString stringWithFormat:@"%@", self.class];
    // 我们在这里加一个判断，将系统的UIViewController的对象剔除掉
//    if(![str containsString:@"UI"]){
        NSLog(@"统计打点 : %@，%@", self.class,str);
//    }
    [self swizzlingViewWillAppear];
    NSLog(@"End");
}
@end
```


其中有一个有趣的点。我在 ViewController 中未实现 viewWillAppear，则会造成死循环，toMethod 反复调用自身。
最后排查，由于 if 中判断的语句（如下）执行后导致
`class_addMethod([self class], @selector(viewWillAppear:), method_getImplementation(toMethod), method_getTypeEncoding(toMethod))`

原因：当父类方法 0 调用了父类方法 1，子类未重写方法1，子类的 category 却对方法1进行了黑魔法，则category 中的 `class_addMethod` 和 `[self swizzlingViewWillAppear];` 配合，造成了死循环。

例如： 父类：UIViewContoller
    方法 0：viewDidLoad:
    方法 1：viewWillAppear:
