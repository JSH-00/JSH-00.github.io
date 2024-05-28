---
layout: OC
title: Siri 建议 — 使用 Objective-C 集成 Siri Shortcuts
date: 2023-07-05 20:00:00
tags: Objective-C, Siri, Shortcuts
---

# Siri 建议 — 使用 Objective-C 集成 Siri Shortcuts

## 背景

Siri Shortcuts 是 iOS 12 引入的一项功能，旨在提供应用程序与 Siri 的深度集成。通过 Siri Shortcuts，应用可以在适当的时候向用户推荐相关操作，从而提升用户体验和应用的使用率。本文将介绍如何在 Objective-C 项目中集成 Siri Shortcuts 并进行模块化处理。

## 优势

1. **提高用户体验**：通过向用户推荐相关操作，提升应用的便利性和易用性。
2. **增加应用曝光率**：用户可以在 Siri 建议中看到应用的推荐操作，从而增加应用的曝光率。
3. **简化用户操作**：用户可以通过 Siri 语音命令快速执行应用中的常用操作。

## 操作步骤

### 配置 Info.plist

首先，在 `Info.plist` 文件中添加 `NSUserActivityTypes` 键，以声明应用支持的用户活动类型：

```xml
<key>NSUserActivityTypes</key>
<array>
    <string>loying.myAppName.myUserBehaviorA</string>
</array>
```

### 上报 Siri 建议

```objc
- (void)reportUserActivity {
    NSUserActivity *userActivity = [[NSUserActivity alloc] initWithActivityType:@"loying.myAppName.myUserBehaviorA"];
    userActivity.eligibleForSearch = YES;
    userActivity.eligibleForPrediction = YES;
    userActivity.eligibleForPublicIndexing = YES;
    userActivity.title = @"BehaviorA Title";
    userActivity.persistentIdentifier = @"myUserBehaviorA";
    userActivity.userInfo = @{@"testKey" : @"testValue"};
    userActivity.webpageURL = [NSURL URLWithString:@"https://www.infoq.cn/article/lsuikfdr0eyv2lusatjt"];
    
    if (@available(iOS 14.0, *)) {
        userActivity.contentAttributeSet = [[CSSearchableItemAttributeSet alloc] initWithContentType:UTTypeItem];
        userActivity.contentAttributeSet.contentDescription = @"BehaviorA subTitle";
    }
    
    [userActivity becomeCurrent];
}
```

### 处理 Siri 建议点击后的回调

```objc
- (BOOL)application:(UIApplication *)application continueUserActivity:(NSUserActivity *)userActivity restorationHandler:(void (^)(NSArray<id<UIUserActivityRestoring>> * _Nullable))restorationHandler {
    NSLog(@"continueUserActivity");
    
    if ([userActivity.activityType isEqualToString:@"loying.LearnSiriShortcut.type"]) {
        // 执行相应的业务逻辑
    }
    
    return YES;
}
```

### 删除 Siri 建议

您可以删除指定或全部 Siri Shortcuts：

```objc
// 删除指定的 Shortcut
[NSUserActivity deleteSavedUserActivitiesWithPersistentIdentifiers:identifer completionHandler:^{
    // 处理删除完成后的逻辑
}];

// 删除全部 Shortcuts
[NSUserActivity deleteAllSavedUserActivitiesWithCompletionHandler:^{
    // 处理删除完成后的逻辑
}];
```

### 进阶的 Siri 建议

除了基础的用户活动上报，您还可以配置更多的用户活动信息，如时间、地点、联系人等，以提升 Siri 建议的智能化程度：

```objc
- (void)reportAdvancedUserActivity {
    NSUserActivity *userActivity = [[NSUserActivity alloc] initWithActivityType:@"com.example.app.viewLatestArticles"];
    userActivity.title = @"View Latest Articles";

    // 设置时间信息
    NSDate *startDate = [NSDate date];
    NSDate *endDate = [NSDate dateWithTimeIntervalSinceNow:3600];
    userActivity.startDate = startDate;
    userActivity.endDate = endDate;

    // 设置地点信息
    CLLocationCoordinate2D location = CLLocationCoordinate2DMake(37.7749, -122.4194);
    MKPlacemark *placemark = [[MKPlacemark alloc] initWithCoordinate:location];
    userActivity.mapItem = [[MKMapItem alloc] initWithPlacemark:placemark];

    // 设置联系人信息
    CNMutableContact *contact = [[CNMutableContact alloc] init];
    contact.givenName = @"John";
    contact.familyName = @"Doe";
    userActivity.contact = contact;

    // 设置提醒信息
    EKEventStore *eventStore = [[EKEventStore alloc] init];
    EKReminder *reminder = [EKReminder reminderWithEventStore:eventStore];
    reminder.title = @"Buy Groceries";
    reminder.dueDateComponents = [[NSCalendar currentCalendar] components:NSCalendarUnitYear | NSCalendarUnitMonth | NSCalendarUnitDay fromDate:[NSDate date]];
    userActivity.calendarItemIdentifier = reminder.calendarItemIdentifier;

    // 设置网页链接
    userActivity.webpageURL = [NSURL URLWithString:@"https://www.example.com"];

    // 设置附加信息
    userActivity.userInfo = @{@"key": @"value"};

    // 提交用户活动
    [userActivity becomeCurrent];
}
```

## 参考链接

- [爱奇艺 iOS 深度实践：SiriKit 详解应用篇](https://www.infoq.cn/article/lsuikfdr0eyv2lusatjt)
- [十分钟接入iOS 12新特性——Siri Shortcuts](https://cloud.tencent.com/developer/article/1351163)

通过上述步骤，您可以在 Objective-C 项目中顺利集成 Siri Shortcuts，实现对用户行为的智能推荐，从而提升用户体验和应用的使用率。