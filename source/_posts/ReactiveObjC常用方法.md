---
layout: git
title: ReactiveObjC 常用方法
date: 2022-05-03 13:00:00
tags: OC
---

# ReactiveObjC 常用方法

## 基本类型
* `RACSiganl` 信号（类）
* `RACSubscriber` 订阅者（协议）
* `RACDisposable` 用于取消订阅或者清理资源，当信号发送完成或者发送错误的时候，就会自动触发它。 当你不想监听某个信号时，可以通过它主动取消订阅信号。
* `RACSubject` 信号提供者，自己可以充当信号，又能发送信号。通常用来代替代理，有了它，就不必要定义代理了。
    * `RACReplaySubject` 重复提供信号类，RACSubject的子类。
    * RACReplaySubject可以先发送信号，在订阅信号，RACSubject就不可以。

## RACObserve

```
[RACObserve(self.userNameTextFiled, tag) subscribeNext:^(id  _Nullable x) {
    NSLog(@"self.userNameTextFiled.tag = %lu",self.userNameTextFiled.tag);
}];
```

##  RACReplaySubject

* `RACReplaySubject` 重复提供信号类，`RACSubject 的子类。
* `RACReplaySubject` 可以先发送信号，在订阅信号，`RACSubject` 不可以。

```
    RACReplaySubject *replaySubject = [RACReplaySubject subject];

    // 2.发送信号
    [replaySubject sendNext:@1];
    [replaySubject sendNext:@2];
    NSLog(@"send over");

    // 3.订阅信号
    [replaySubject subscribeNext:^(id x) {

        NSLog(@"第一个订阅者接收到的数据%@",x);
    }];

    // 订阅信号
    [replaySubject subscribeNext:^(id x) {

        NSLog(@"第二个订阅者接收到的数据%@",x);
    }];

```

## RACMulticastConnection
* RACMulticastConnection 用于当一个信号被多个订阅时，为了避免创建信号时多次调用创建信号中的block造成多次发生数据，可以使用这个该类处理。

```
    RACSignal *signal1 = [RACSignal createSignal:^RACDisposable *(id<RACSubscriber> subscriber) {
        NSLog(@"发送数据"); //执行 2 次数，因为被订阅了 2 次数
        return nil;
    }];
    
    [signal1 subscribeNext:^(id x) {
        NSLog(@"接收数据");
    }];

    [signal1 subscribeNext:^(id x) {
        NSLog(@"接收数据");
    }];
    
    // 会执行两遍 `NSLog(@"发送数据");` 每次订阅都会发送一次请求
    
    
    // RACMulticastConnection:解决重复请求问题
    RACSignal *signal = [RACSignal createSignal:^RACDisposable *(id<RACSubscriber> subscriber) {
        NSLog(@"发送请求");
        [subscriber sendNext:@"我是数据源"];
        return nil;
    }];

    // 创建连接
    RACMulticastConnection *connect = [signal publish];

    // 订阅信号，
    // 注意：订阅信号，但没有激活信号，只是保存订阅者到数组，必须通过连接,当调用连接，就会一次性调用所有订阅者的sendNext:
    [connect.signal subscribeNext:^(id x) {
        NSLog(@"订阅者一:%@", x);

    }];

    [connect.signal subscribeNext:^(id x) {
        NSLog(@"订阅者二:%@", x);
    }];

    // 连接,激活信号
    [connect connect];

```
## RAC宏绑定
```
//单向绑定,signal10中的next值将作用于view的背景色
RAC(self.view,backgroundColor) = signal10;
//双向绑定,互相作用
RACChannelTo(self.lb_name,text) = RACChannelTo(model, name);
```

## 冷热信号
* 热信号是主动的，即使没有订阅事件，仍然会时刻推送；而冷信号是被动的，只有当你订阅的时候，它才会发送消息。

### 创建冷信号
```
#import <ReactiveObjC/ReactiveObjC.h>

    RACSignal *signal = [RACSignal createSignal:^RACDisposable * _Nullable(id<RACSubscriber>  _Nonnull subscriber) {
        [subscriber sendNext:@1];
        [[RACScheduler mainThreadScheduler] afterDelay:0.5 schedule:^{
                    [subscriber sendNext:@2];
        }];
        [[RACScheduler mainThreadScheduler] afterDelay:2 schedule:^{
                    [subscriber sendNext:@3];
        }];
        return nil;
    }];
    NSLog(@"====time01====");
    [[RACScheduler mainThreadScheduler]afterDelay:5 schedule:^{
            [signal subscribeNext:^(id  _Nullable x) {
                // time01 5s 后执行
                NSLog(@"singnal1接收到信号了%@",x);
            }];
    }];
    NSLog(@"====time02====");

    [[RACScheduler mainThreadScheduler]afterDelay:10 schedule:^{
            [signal subscribeNext:^(id  _Nullable x) {
                // time01 10s 后执行
                NSLog(@"singnal2接收到信号了%@",x);
            }];
    }];
```

### 创建热信号
```
    RACSubject *subject = [RACSubject subject];

    NSLog(@"subject 发送1");
    [subject sendNext:@1];
    NSLog(@"subject 发送完成1");

    NSLog(@"==== 01");
    [[RACScheduler mainThreadScheduler]afterDelay:0.5 schedule:^{
        NSLog(@"subject 发送2");
            [subject sendNext:@2];
        NSLog(@"subject 发送完成2");
    }];
    NSLog(@"==== 02");
    [[RACScheduler mainThreadScheduler]afterDelay:2 schedule:^{
        NSLog(@"subject 发送3");
            [subject sendNext:@3];
        NSLog(@"subject 发送完成3");
    }];
    NSLog(@"==== 03");
    [[RACScheduler mainThreadScheduler]afterDelay:0.1 schedule:^{
        [subject subscribeNext:^(id  _Nullable x) {
            NSLog(@"subject 001接收到了%@",x);
        }];
    }];
    NSLog(@"==== 04");
    [[RACScheduler mainThreadScheduler]afterDelay:1 schedule:^{
        [subject subscribeNext:^(id  _Nullable x) {
            NSLog(@"subject 002接收到了%@",x);
        }];
    }];
    
```

```
RACtestDemo[51003:2926156] subject 发送1
RACtestDemo[51003:2926156] subject 发送完成1
RACtestDemo[51003:2926156] ==== 01
RACtestDemo[51003:2926156] ==== 02
RACtestDemo[51003:2926156] ==== 03
RACtestDemo[51003:2926156] ==== 04
RACtestDemo[51003:2926156] subject 发送2
RACtestDemo[51003:2926156] subject 001接收到了2
RACtestDemo[51003:2926156] subject 发送完成2
RACtestDemo[51003:2926156] subject 发送3
RACtestDemo[51003:2926156] subject 001接收到了3
RACtestDemo[51003:2926156] subject 002接收到了3
RACtestDemo[51003:2926156] subject 发送完成3
```


[参考](https://www.cnblogs.com/soliloquy/p/7920551.html)
[代理、KVO、监听事件等参考](https://www.jianshu.com/p/3db561083fef)