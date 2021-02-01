---
layout: git
title: GCD dispatch
date: 2021-02-01 14:00:00
tags: OC
---
# GCD dispatch

## 同步 && 异步 && 串型 && 并发
* 同步：没有开启新线程的能力
    * 所有任务都在 begin 和 end 之间执行
    * 串型和并发效果一样，都不开启新线程
* 异步：有开启新线程的能力，但是否开启视情况而定
    * 若加入串型，所有任务都在 begin 和 end 之后逐个执行
    * 若加入并发，所有任务都在 begin 之后执行
* 串型：加入队列的逐个执行
* 并发：需要与异步配合，才能开启新线程并发执行。否则加入的队列逐个执行

## 同步执行 + 并发队列
* 无法开启新线程
* 任务加入队列后直接执行，然后再执行后面内容

## 异步执行 + 并发队列
 * 可以开启多个线程
 * 任务加入队列后直接执行，可以多个任务同时执行
 
## 异步执行 + 并发队列
（类似于同步执行 + 并发队列）
* 无法开启新线程
* 任务加入队列后直接执行，然后再执行后面内容

## dispatch_barrier
* 控制某些任务全部执行完成后再进行下面的任务
* 适用场景：异步 + 并发

```
dispatch_barrier_async(queue, ^{
        // 追加任务 barrier
        [NSThread sleepForTimeInterval:5];              // 模拟耗时操作
        NSLog(@"barrier---%@",[NSThread currentThread]);// 打印当前线程
    })

```

## dispatch_after
* 加入主队列并准备 2 秒后执行。
* 若 2s 后主线程未占用，则直接执行。若占用，则等待空闲时立即执行。

```
   dispatch_after(dispatch_time(DISPATCH_TIME_NOW, (int64_t)(2.0 * NSEC_PER_SEC)), dispatch_get_main_queue(), ^{
        // 2.0 秒后异步追加任务代码到主队列，并开始执行
        NSLog(@"after---%@",[NSThread currentThread]);  // 打印当前线程
    });
```

## dispatch_apply
* 快速迭代方法。
* 相当于 for 循环，建议用在并发队列。

```
    dispatch_apply(6, queue, ^(size_t index) {
        NSLog(@"%zd---%@",index, [NSThread currentThread]);
    });
```

## dispatch_group

### dispatch_group_notify
* 把任务都异步并发的加入 group 中，等前面的都执行完才执行 dispatch_group_notify

```
- (void)groupNotify {
    NSLog(@"currentThread---%@",[NSThread currentThread]);  // 打印当前线程
    NSLog(@"group---begin");
    
    dispatch_group_t group =  dispatch_group_create();
    
    dispatch_group_async(group, dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0), ^{
        // 追加任务 1
        [NSThread sleepForTimeInterval:2];              // 模拟耗时操作
        NSLog(@"1---%@",[NSThread currentThread]);      // 打印当前线程
    });
    
    dispatch_group_async(group, dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0), ^{
        // 追加任务 2
        [NSThread sleepForTimeInterval:2.5];              // 模拟耗时操作
        NSLog(@"2---%@",[NSThread currentThread]);      // 打印当前线程
    });
    
    dispatch_group_notify(group, dispatch_get_main_queue(), ^{
        // 等前面的异步任务 1、任务 2 都执行完毕后，回到主线程执行下边任务
        [NSThread sleepForTimeInterval:2];              // 模拟耗时操作
        NSLog(@"3---%@",[NSThread currentThread]);      // 打印当前线程
        NSLog(@"group---end");
    });
}
```
### dispatch_group_enter dispatch_group_leave 
* 通过 dispatch_group_enter、dispatch_group_leave 控制执行（其他任务开异步加入 queue 即可，不用加入 group）

```
- (void)groupEnterAndLeave {
    NSLog(@"currentThread---%@",[NSThread currentThread]);  // 打印当前线程
    NSLog(@"group---begin");
    
    dispatch_group_t group = dispatch_group_create();
    dispatch_queue_t queue = dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0);
    dispatch_group_enter(group);
    dispatch_async(queue, ^{
        // 追加任务 1
        [NSThread sleepForTimeInterval:2];              // 模拟耗时操作
        NSLog(@"1---%@",[NSThread currentThread]);      // 打印当前线程

        dispatch_group_leave(group);
    });
    
    dispatch_group_enter(group);
    dispatch_async(queue, ^{
        // 追加任务 2
        [NSThread sleepForTimeInterval:2];              // 模拟耗时操作
        NSLog(@"2---%@",[NSThread currentThread]);      // 打印当前线程
        
        dispatch_group_leave(group);
    });
    
    dispatch_group_notify(group, dispatch_get_main_queue(), ^{
        // 等前面的异步操作都执行完毕后，回到主线程.
        [NSThread sleepForTimeInterval:2];              // 模拟耗时操作
        NSLog(@"3---%@",[NSThread currentThread]);      // 打印当前线程
    
        NSLog(@"group---end");
    });
}
```
## dispatch_semaphore 信号
```
#pragma mark - 保证线程间同步
dispatch_queue_t que = dispatch_queue_create("com.vc.downloadQueue", DISPATCH_QUEUE_SERIAL);

// 设置信号量为 3
dispatch_semaphore_t semaphore = dispatch_semaphore_create(3); //
dispatch_async(que, ^{
    // 若 signal 0，则 wait forever 否则  signal - 1 并执行后面的代码
    dispatch_semaphore_wait(semaphore, DISPATCH_TIME_FOREVER);
});

// singal + 1
dispatch_semaphore_signal(self.semaphore);


#pragma mark - 为线程加锁
dispatch_queue_t queue = dispatch_get_global_queue(0, 0);
dispatch_semaphore_t semaphore = dispatch_semaphore_create(1);
        
for (int i = 0; i < 100; i++) {
     dispatch_async(queue, ^{
          // 相当于加锁
         dispatch_semaphore_wait(semaphore, DISPATCH_TIME_FOREVER);
          NSLog(@"i = %zd semaphore = %@", i, semaphore);
          // 相当于解锁
          dispatch_semaphore_signal(semaphore);
      });
}

```

## 主线程获取方法
```
dispatch_queue_t queue = dispatch_get_main_queue();
```
## 获取默认并发队列方法

* 调用时，先执行 dispatch ，后执行 end

```
dispatch_queue_t queue = dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0);

```

参考：[iOS多线程：『GCD』详尽总结
](https://juejin.cn/post/6844903566398717960#heading-17)