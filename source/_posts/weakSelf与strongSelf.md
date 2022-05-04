---
layout: git
title: weakSelf 与 strongSelf
date: 2022-04-15 13:00:00
tags: OC
---

# weakSelf 与 strongSelf

* 避免循环引用的标准做法:weakSelf+strongSelf
* 调试方法：打印引用计数来看是否 block 会使引用计数 +1
    * `printf("retain count:%ld\n", CFGetRetainCount((__bridge CFTypeRef)(self)));`

## 写法
block 里面套 block
```
- (void)setUpModel{
    XYModel *model = [XYModel new];
   
    __weak typeof(self) weakSelf = self; // 不写这个，引用计数会 +1
    model.dataChanged = ^(NSString *title) {
        __strong typeof(self) strongSelf = weakSelf;
        strongSelf.titleLabel.text = title;
        
        __weak typeof(self) weakSelf2 = strongSelf; // 不写这个，引用计数会 +1
        strongSelf.model.dataChanged = ^(NSString *title2) {
            __strong typeof(self) strongSelf2 = weakSelf2;
            strongSelf2.titleLabel.text = title2;
        };
    };
    self.model = model;
}
```
## 原因：
### weakSelf
* 可以避免循环引用导致代码执行结束后，引用计数仍被 +1

### strongSelf
* 给 weakSelf 添加强引用，避免 weakSelf 被提前释放
* weakSelf 在block 里在执行 doSomething 还存在，但在执行 doMorething 前，可能会被释放了，故为了保证 self 在 block 执行过程里一直存在，对他强引用 strongSelf

## RAC 简便写法
```
- (void)setUpModel{
    XYModel *model = [XYModel new];

    @weakify(self) // 不写这个，引用计数会 +1
    model.dataChanged = ^(NSString *title) {
        @strongify(self)
        strongSelf.titleLabel.text = title;
        
        @weakify(self) // 不写这个，引用计数会 +1
        strongSelf.model.dataChanged = ^(NSString *title2) {
            @strongify(self)
            strongSelf2.titleLabel.text = title2;
        };
    };
    self.model = model;
}
```

[进阶 @weakify(self) @strongify(self) 链接](https://www.jianshu.com/p/9e2c36482cc2)
[参考链接](https://blog.csdn.net/u010029439/article/details/112465436?spm=1001.2101.3001.6650.1&utm_medium=distribute.pc_relevant.none-task-blog-2%257Edefault%257ECTRLIST%257Edefault-1.pc_relevant_default&depth_1-utm_source=distribute.pc_relevant.none-task-blog-2%257Edefault%257ECTRLIST%257Edefault-1.pc_relevant_default&utm_relevant_index=2)
