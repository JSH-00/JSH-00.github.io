---
layout: git
title: UIAlertController 弹窗
date: 2021-03-19 20:00:00
tags: OC
---
#  UIAlertController 弹窗

通过 UIAlertController 弹出弹窗（iOS 8 以上）

```
#import "ViewController.h"

@interface ViewController ()
@property (nonatomic, strong)UIAlertController *alertController;
@end

@implementation ViewController

- (void)viewDidLoad {
    [super viewDidLoad];
    [self actionSheet];
}

- (void)touchesBegan:(NSSet<UITouch *> *)touches withEvent:(UIEvent *)event {
    // 点击屏幕弹出 Alert
    [self presentViewController:self.alertController animated:YES completion:nil];
}

- (void)actionSheet {
    
    // 初始化（Alert样式）
    UIAlertController *alertController = [UIAlertController alertControllerWithTitle:@"温馨提示" message:@"您正在使用 UIAlertController" preferredStyle:UIAlertControllerStyleAlert];
    // 初始化（Sheet样式）
//    UIAlertController *alertController = [UIAlertController alertControllerWithTitle:@"温馨提示" message:@"您正在使用 UIAlertController" preferredStyle:UIAlertControllerStyleActionSheet];
    self.alertController = alertController;
    
    //创建action 添加到alertController上 可根据UIAlertActionStyleDefault创建不通的alertAction
    UIAlertAction *action1 = [UIAlertAction actionWithTitle:@"确定" style:UIAlertActionStyleDefault handler:^(UIAlertAction *action) {
        //回调
        // 模态视图，使用dismiss 隐藏
        [self.alertController dismissViewControllerAnimated:YES completion:nil];
        
    }];
    UIAlertAction *action2 = [UIAlertAction actionWithTitle:@"确定2" style:UIAlertActionStyleDefault handler:^(UIAlertAction *action) {
        //回调
        // 模态视图，使用dismiss 隐藏
        [self.alertController dismissViewControllerAnimated:YES completion:nil];
        
    }];
    UIAlertAction *action3 = [UIAlertAction actionWithTitle:@"取消" style:UIAlertActionStyleCancel handler:^(UIAlertAction *action) {
        [self.alertController dismissViewControllerAnimated:YES completion:nil];
        
    }];
        
    // 将alertAction 添加到 alertController
    [alertController addAction:action1];
    [alertController addAction:action2];
    [alertController addAction:action3];
}

@end
```