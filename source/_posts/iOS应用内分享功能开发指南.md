---
layout: git
title: iOS应用内多平台登录功能开发指南
date: 2023-11-18 20:00:00
tags: OC, iOS
---

# iOS应用内分享功能开发指南

本文将介绍如何在iOS应用中实现分享功能，涵盖多种社交平台的分享，包括Line、Instagram、Twitter、Facebook等。本教程将展示代码实现，并逐步解释每个实现步骤。

## 准备工作

在开始之前，确保你已经在项目中集成了以下第三方库：

- [SDWebImage](https://github.com/SDWebImage/SDWebImage)
- [YYModel](https://github.com/ibireme/YYModel)
- [FBSDKShareKit](https://developers.facebook.com/docs/sharing/ios)

## 分享工具类的实现

下面的代码展示了一个分享工具类`BLShareUtils`的实现。这个工具类支持多种分享类型，包括Line、Instagram、Twitter、Facebook和系统默认分享。

### 分享类型的定义

首先定义分享类型和一个函数，用于将分享类型转换为字符串。

```objective-c
typedef NS_ENUM(NSUInteger, BLShareType) {
    BLShareTypeNone,
    BLShareTypeLine,
    BLShareTypeTwitter,
    BLShareTypeFacebook,
    BLShareTypeInsPost,
    BLShareTypeInsStories,
    BLShareTypeCopy,
    BLShareTypeOthers
};

NSString *stringFromShareType(BLShareType shareType) {
    switch (shareType) {
        case BLShareTypeNone:
            return @"unknown";
        case BLShareTypeLine:
            return @"line";
        case BLShareTypeTwitter:
            return @"twitter";
        case BLShareTypeFacebook:
            return @"facebook";
        case BLShareTypeInsPost:
            return @"instagram_post";
        case BLShareTypeInsStories:
            return @"instagram_stories";
        case BLShareTypeCopy:
            return @"copy";
        case BLShareTypeOthers:
            return @"system";
        default:
            return @"";
    }
}
```

### 分享工具类的声明

声明一个单例类`BLShareUtils`，用于处理各种类型的分享。

```objective-c
@interface BLShareModel : NSObject
@property (nonatomic, strong) NSString *text;
@property (nonatomic, strong) NSString *link;
@property (nonatomic, strong) NSString *imageUrl;
@property (nonatomic, assign) NSUInteger feedId;
@end

@interface BLShareUtils : NSObject
@property (nonatomic, strong) BLShareModel *model;
@property (nonatomic, copy) void (^completeHandler)(BOOL success, NSUInteger feedId, BLShareType shareType, NSError *error);
@property (nonatomic, weak) UIView *supView;

+ (instancetype)sharedInstance;
- (void)shareWithLine:(BLShareModel *)model;
- (void)shareToInsPost:(BLShareModel *)model;
- (void)shareToInsStories:(BLShareModel *)model;
- (void)shareWithTwitter:(BLShareModel *)model;
- (void)shareWithFacebook:(BLShareModel *)model;
- (void)shareWithOthers:(BLShareModel *)model;
- (void)shareWithCopy:(BLShareModel *)model;
@end
```

### 实现分享功能

#### Line分享

Line的分享主要是通过URL Scheme来实现。

```objective-c
- (void)shareWithLine:(BLShareModel *)model {
    self.model = model;
    NSString *contentType = @"text";
    NSString *urlString = [NSString stringWithFormat:@"line://msg/%@/%@",
                           contentType,
                           [[NSString stringWithFormat:@"%@ %@", model.text, model.link] stringByAddingPercentEncodingWithAllowedCharacters:[NSCharacterSet URLQueryAllowedCharacterSet]]];
    NSURL *url = [NSURL URLWithString:urlString];
    if ([[UIApplication sharedApplication] canOpenURL:url]) {
        [[UIApplication sharedApplication] openURL:url options:@{} completionHandler:^(BOOL success) {
            if (self.completeHandler) {
                self.completeHandler(success, self.model.feedId, BLShareTypeLine, nil);
            }
        }];
    }
}
```

#### Instagram分享

Instagram的分享分为Post和Stories两种方式。首先下载图片，然后保存到相册，再通过URL Scheme进行分享。

**Post分享：**

```objective-c
- (void)shareToInsPost:(BLShareModel *)model {
    self.model = model;
    [[SDWebImageDownloader sharedDownloader] downloadImageWithURL:[NSURL URLWithString:self.model.imageUrl] completed:^(UIImage * _Nullable image, NSData * _Nullable data, NSError * _Nullable error, BOOL finished) {
        if (error) {
            if (self.completeHandler) {
                self.completeHandler(NO, self.model.feedId, BLShareTypeInsPost, error);
            }
        } else {
            UIImageWriteToSavedPhotosAlbum(image, self, @selector(image:didFinishSavingWithError:contextInfo:), (__bridge_retained void *)@(BLShareTypeInsPost));
        }
    }];
}
```

**Stories分享：**

```objective-c
- (void)shareToInsStories:(BLShareModel *)model {
    self.model = model;
    [[SDWebImageDownloader sharedDownloader] downloadImageWithURL:[NSURL URLWithString:self.model.imageUrl] completed:^(UIImage * _Nullable image, NSData * _Nullable data, NSError * _Nullable error, BOOL finished) {
        if (error) {
            if (self.completeHandler) {
                self.completeHandler(NO, self.model.feedId, BLShareTypeInsStories, error);
            }
        } else {
            UIImageWriteToSavedPhotosAlbum(image, self, @selector(image:didFinishSavingWithError:contextInfo:), (__bridge_retained void *)@(BLShareTypeInsStories));
        }
    }];
}
```

保存图片后的处理：

```objective-c
- (void)image:(UIImage *)image didFinishSavingWithError:(NSError *)error contextInfo:(void *)contextInfo {
    if (error) {
        // 保存图片到相册出错的处理
    } else {
        BLShareType shareType = (BLShareType)((__bridge_transfer NSNumber *)contextInfo).unsignedIntValue;
        switch (shareType) {
            case BLShareTypeInsPost:
                {
                    NSURL *instagramURL = [NSURL URLWithString:@"instagram://library?AssetPath=assets-library"];
                    [[UIApplication sharedApplication] openURL:instagramURL options:@{} completionHandler:^(BOOL success) {
                        if (self.completeHandler) {
                            self.completeHandler(success, self.model.feedId, BLShareTypeInsPost, nil);
                        }
                    }];
                }
                break;
            case BLShareTypeInsStories:
                {
                    NSString *const appIDString = @"YOUR_APP_ID";
                    NSURL *urlScheme = [NSURL URLWithString:[NSString stringWithFormat:@"instagram-stories://share?source_application=%@", appIDString]];
                    
                    if ([[UIApplication sharedApplication] canOpenURL:urlScheme]) {
                        NSArray *pasteboardItems = @[@{@"com.instagram.sharedSticker.stickerImage" : UIImagePNGRepresentation(image),
                                                       @"com.instagram.sharedSticker.backgroundTopColor" : @"#000000",
                                                       @"com.instagram.sharedSticker.backgroundBottomColor" : @"#000000"}];
                        
                        NSDictionary *pasteboardOptions = @{UIPasteboardOptionExpirationDate : [[NSDate date] dateByAddingTimeInterval:60 * 5]};
                        [[UIPasteboard generalPasteboard] setItems:pasteboardItems options:pasteboardOptions];
                        [[UIApplication sharedApplication] openURL:urlScheme options:@{} completionHandler:^(BOOL success) {
                            if (self.completeHandler) {
                                self.completeHandler(success, self.model.feedId, BLShareTypeInsStories, nil);
                            }
                        }];
                    } else {
                        if (self.completeHandler) {
                            self.completeHandler(NO, self.model.feedId, BLShareTypeInsStories, nil);
                        }
                    }
                }
                break;
            default:
                if (self.completeHandler) {
                    self.completeHandler(NO, self.model.feedId, shareType, nil);
                }
                break;
        }
    }
}
```

#### Twitter分享

Twitter分享主要是通过URL Scheme来实现。

```objective-c
- (void)shareWithTwitter:(BLShareModel *)model {
    self.model = model;
    NSString *encodedString = [NSString stringWithFormat:@"twitter://post?message=%@", [NSString stringWithFormat:@"%@\n%@", model.text, model.link]];
    encodedString = [encodedString stringByAddingPercentEncodingWithAllowedCharacters:[NSCharacterSet URLQueryAllowedCharacterSet]];
    [[UIApplication sharedApplication] openURL:[NSURL URLWithString:encodedString] options:@{} completionHandler:^(BOOL success) {
        if (self.completeHandler) {
            self.completeHandler(success, self.model.feedId, BLShareTypeTwitter, nil);
        }
    }];
}
```

#### Facebook分享

Facebook分享使用`FBSDKShareKit`来实现。

```objective-c
- (void)shareWithFacebook:(BLShareModel *)model {
    self.model = model;
    FBSDKShareLinkContent *content = [[FBSDKShareLinkContent alloc] init];
    content.contentURL = [NSURL URLWithString:model.link];
    content.quote = model.text;
    
    FBSDKShareDialog *shareDialog = [[FBSDKShareDialog alloc] initWithViewController:[UINavigationController currentViewController] content:content delegate:self];
    shareDialog.mode = FBSDKShareDialogModeNative;
    [shareDialog show];
}

#pragma mark - FBSDKSharingDelegate

- (void)sharer:(id<FBSDKSharing>)sharer didCompleteWithResults:(NSDictionary<NSString *, id> *)results {
    if (self.completeHandler) {
        self.completeHandler(YES, self.model.feedId, BLShareTypeFacebook, nil);
    }
}

- (void)sharer:(id<FBSDKSharing>)sharer didFailWithError:(NSError *)error {
    if (self.completeHandler) {
        self.completeHandler(NO, self.model.feedId, BLShareTypeFacebook, error);
    }
}

- (void)sharerDidCancel:(id<FBSDKSharing>)sharer {
    if (self.completeHandler) {
        self.completeHandler(NO, self.model.feedId, BLShareTypeFacebook, nil);
    }
}
```

#### 系统分享

系统分享使用`UIActivityViewController`来实现。

```objective-c
- (void)shareWithOthers:(BLShareModel *)model {
    self.model = model;
    
    NSMutableArray *activityItems = [NSMutableArray array];
    if (model.text.length) {
        [activityItems addObject:model.text];
    }
    
    if (model.link.length) {
        NSURL *urlToShare = [NSURL URLWithString:model.link];
        [activityItems addObject:urlToShare];
    }
    
    UIActivityViewController *activityVC = [[UIActivityViewController alloc] initWithActivityItems:activityItems applicationActivities:nil];
    UIActivityViewControllerCompletionWithItemsHandler myBlock = ^(UIActivityType activityType, BOOL completed, NSArray *returnedItems, NSError *error) {
        if (error) {
            if (self.completeHandler) {
                self.completeHandler(NO, self.model.feedId, BLShareTypeOthers, error);
            }
        } else {
            if (self.completeHandler) {
                self.completeHandler(completed, self.model.feedId, BLShareTypeOthers, nil);
            }
        }
    };
    
    activityVC.completionWithItemsHandler = myBlock;
    if ([[UIDevice currentDevice] userInterfaceIdiom] == UIUserInterfaceIdiomPad) {
        activityVC.popoverPresentationController.permittedArrowDirections = 0;
        activityVC.popoverPresentationController.sourceView = self.supView;
        activityVC.popoverPresentationController.sourceRect = CGRectMake(self.supView.frame.size.width / 2.0, self.supView.frame.size.height / 2.0, 1.0, 1.0);
        [[UINavigationController currentViewController] presentViewController:activityVC animated:YES completion:nil];
    } else {
        [[UINavigationController currentViewController] presentViewController:activityVC animated:YES completion:nil];
    }
}
```

#### 复制到剪贴板

```objective-c
- (void)shareWithCopy:(BLShareModel *)model {
    self.model = model;
    UIPasteboard *pasteboard = [UIPasteboard generalPasteboard];
    NSString *textToCopy = [NSString stringWithFormat:@"%@\n%@", model.text, model.link];
    [pasteboard setString:textToCopy];
    if (self.completeHandler) {
        self.completeHandler(YES, self.model.feedId, BLShareTypeCopy, nil);
    }
}
```

## 总结

本文详细介绍了如何在iOS应用中实现多种分享功能，包括Line、Instagram、Twitter、Facebook和系统默认分享。通过上述示例代码，你可以轻松地在自己的项目中添加分享功能。希望这篇文章对你有所帮助。