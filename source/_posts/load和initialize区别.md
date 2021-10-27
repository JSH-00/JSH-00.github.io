---
layout: git
title: load和initialize区别
date: 2021-10-27 20:00:00
tags: OC
---

# `load`和`initialize`区别

1、load是根据函数地址直接调用
2、initialize是通过objc_msgSend调用
调用时刻
1、load是**runtime加载类**、分类的时候调用(只会调用一次)
2、initialize是类第一次接收到消息的时候调用, 每一个类只会initialize一次(如果子类没有实现initialize方法, 会调用父类的initialize方法, 所以父类的initialize方法可能会调用多次)


`load`是只要类所在文件被引用就会被调用，而`initialize`是在类或者其子类的第一个方法被调用前调用。所以如果类没有被引用进项目，就不会有`load`调用；但即使类文件被引用进来，但是没有使用，那么`initialize`也不会被调用；`load`每个类只会调用一次，`initialize`也只调用一次，但是如果子类没有实现`initialize`方法则会调用父类的方法，因此作为父类的`initialize`方法可能会调用多次。

load和initializee的调用顺序
1、load:
先调用类的load, 在调用分类的load
先编译的类, 优先调用load, 调用子类的load之前, 会先调用父类的load
先编译的分类, 优先调用load

2、initialize
先初始化分类, 后初始化子类
通过消息机制调用, 当子类没有initialize方法时, 会调用父类的initialize方法, 所以父类的initialize方法会调用多次？？？

```
+ (void)load {

    }
    
+ (void)initialize {

    }
```

![1](1.png)


`load` 父类调用顺序按照 Compile Sources 调用
`initialize` 调用顺序按照 Compile Sources 反向的顺序

![2](2.png)


[参考资料](https://www.jianshu.com/p/5e5a26c1ae15)