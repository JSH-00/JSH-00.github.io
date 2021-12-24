---
layout: git
title: NSInvocation 基本用法
date: 2021-12-05 20:00:00
tags: OC
---

# NSInvocation

可以直接调用某个对象的消息方式有两种：
* performSelector:withObject；
* NSInvocation。
但是对于 >2 个的参数或者有返回值的处理，那就需要做些额外工作才能搞定。那么在这种情况下，我们就可以使用NSInvocation来进行这些相对复杂的操作。

```
- (void)viewDidLoad {
    [super viewDidLoad];
 //NSInvocation;用来包装方法和对应的对象，它可以存储方法的名称，对应的对象，对应的参数,
    /*
     NSMethodSignature：签名：再创建NSMethodSignature的时候，必须传递一个签名对象，签名对象的作用：用于获取参数的个数和方法的返回值
     */
    //创建签名对象的时候不是使用NSMethodSignature这个类创建，而是方法属于谁就用谁来创建
    NSMethodSignature*signature = [ViewController instanceMethodSignatureForSelector:@selector(sendMessageWithNumber:WithContent:)];
    //1、创建NSInvocation对象
    NSInvocation*invocation = [NSInvocation invocationWithMethodSignature:signature];
    invocation.target = self;
    //invocation中的方法必须和签名中的方法一致。
    invocation.selector = @selector(sendMessageWithNumber:WithContent:);
    /*第一个参数：需要给指定方法传递的值
           第一个参数需要接收一个指针，也就是传递值的时候需要传递地址*/
    //第二个参数：需要给指定方法的第几个参数传值
    NSString*number = @"1111";
    //注意：设置参数的索引时不能从0开始，因为0已经被self占用，1已经被_cmd占用
    [invocation setArgument:&number atIndex:2];
    NSString*number2 = @"啊啊啊";
    [invocation setArgument:&number2 atIndex:3];
    //2、调用NSInvocation对象的invoke方法
    //只要调用invocation的invoke方法，就代表需要执行NSInvocation对象中制定对象的指定方法，并且传递指定的参数
    [invocation invoke];
    
    // 获取返回值
    id res = nil;
    if(signature.methodReturnLength != 0){
        [invocation getReturnValue:&res];
    }
    NSLog(@"===>%@",(NSString *)res);
}

- (NSString *)sendMessageWithNumber:(NSString*)number WithContent:(NSString*)content{
    NSLog(@"电话号%@,内容%@",number,content);
    return @"return message";
}
```

[参考资料](https://blog.csdn.net/wzc10101415/article/details/80305840)