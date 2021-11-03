---
layout: git
title: App Tracking Transparency获取IDFA
date: 2021-10-31 20:00:00
tags: OC
---
# iOS 14.5 以后 App Tracking Transparency 获取 IDFA
iOS 14.5以后的版本，想要允许其跟踪或访问其设备的广告标识符（IDFA），需使用AppTrackingTransparency
## info.plist 文件
此处自定义提示内容为必填项，也可以参考[提示模版](https://airtable.com/shrgVLgfz6tdbv3Fc/tblZVr9BgJ2Cb0ihB)
![](1.png)


## 代码判断
```
#import <AppTrackingTransparency/AppTrackingTransparency.h>
#import <AdSupport/ASIdentifierManager.h>


    if (@available(iOS 14, *)) {
        // iOS14及以上版本需要先请求权限
                ATTrackingManagerAuthorizationStatus status = ATTrackingManager.trackingAuthorizationStatus;
                switch (status) {
                    case ATTrackingManagerAuthorizationStatusDenied:
                        NSLog(@"用户拒绝");
                        break;
                    case ATTrackingManagerAuthorizationStatusAuthorized:
                    {
                        NSLog(@"用户允许");
                        NSString *idfa = [[ASIdentifierManager sharedManager].advertisingIdentifier UUIDString];
                        NSLog(@"IDFA: %@",idfa);
                        break;
                    }
                    case ATTrackingManagerAuthorizationStatusNotDetermined:
                        NSLog(@"用户未做选择或未弹窗");
                        break;
                    default:
                        break;
                }
   
        } else {
            // iOS14以下版本依然使用老方法
            // 判断在设置-隐私里用户是否打开了广告跟踪
            if ([[ASIdentifierManager sharedManager] isAdvertisingTrackingEnabled]) {
                NSString *idfa = [[ASIdentifierManager sharedManager].advertisingIdentifier UUIDString];
                NSLog(@"%@",idfa);
            } else {
                NSLog(@"请在设置-隐私-广告中打开广告跟踪功能");
            }
        }
```


[参考资料](https://www.jianshu.com/p/b4df139e959d)