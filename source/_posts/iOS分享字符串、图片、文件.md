---
layout: git
title: iOS 分享字符串、图片、文件
date: 2021-06-24 20:00:00
tags: OC
---

## UIActivityViewController
分享数据：字符串、图片、多个文件

```
NSString *path = [[NSSearchPathForDirectoriesInDomains(NSCachesDirectory, NSUserDomainMask, YES) objectAtIndex:0] stringByAppendingPathComponent:@"Logs"];
NSError *error = nil;
NSArray *arrayURLArray = [[NSFileManager defaultManager] contentsOfDirectoryAtURL:[NSURL URLWithString:path] includingPropertiesForKeys:[NSArray array] options:0 error:&error];
if (error) {
    NSLog(@"delete failed %@",error);
}

NSArray *activityItems = arrayURLArray;// 分享多个文件
//  NSData *data = [[NSData alloc] initWithContentsOfFile:path];
//  NSString *textToShare = @"这是测试字符串002";
//  UIImage *imageToShare = [UIImage imageNamed:@"动态(1)@2x.png"];
//  分享的图片不能为空
//  NSArray *activityItems = @[data, textToShare, imageToShare]; // 分享其他类型
    
UIActivityViewController *activityVc = [[UIActivityViewController alloc] initWithActivityItems:array applicationActivities:nil];
[self presentViewController:activityVc animated:YES completion:nil];
    
activityVc.completionWithItemsHandler = ^(UIActivityType  _Nullable activityType, BOOL completed, NSArray * _Nullable returnedItems, NSError * _Nullable activityError) {
    if (completed) {
        NSLog(@"分享成功");
    } else {
        NSLog(@"分享取消");
    }
};
```

## UIDocumentInteractionController
分享沙盒文件

```
@interface PLLogTool ()<UIDocumentInteractionControllerDelegate>
// 一定要定义为 strong，否则 share 时会被 release
@property (nonatomic, strong)UIDocumentInteractionController  *documentController;
@end
UIDocumentInteractionController *docCtrl = [UIDocumentInteractionController interactionControllerWithURL:[NSURL fileURLWithPath:filePath]];
_documentController = docCtrl; //
docCtrl.delegate = self;

BOOL canOpen = [docCtrl presentOpenInMenuFromRect:CGRectZero inView:vc.view animated:YES];
if (!canOpen) {
    NSLog(@"No program can open the selected file.");
}
```