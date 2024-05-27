---
layout: OC
title: Siri 建议
date: 2023-07-05 20:00:00
tags: OC
---

infor.plist添加：
```
<key>NSUserActivityTypes</key> <array>
<string>loying.myAppName.myUserBehaviorA</string> </array>
```


```
- (void)reportUserActivity
{

	NSUserActivity *userActivity = [[NSUserActivity alloc] initWithActivityType:@"loying.myAppName.myUserBehaviorA"]; //字符串标识符。它对于唯一地标识您的用户活动
    userActivity.eligibleForSearch = YES; //是否应该被Siri搜索索引，YES，则表明您希望Siri能够在搜索中包含该用户活动
    userActivity.eligibleForPrediction = YES; //是否应该被Siri预测。如果设置为YES，则表明您希望Siri能够根据用户的行为模式，提供关于该用户活动的建议。
    userActivity.eligibleForPublicIndexing = YES; //是否应该被公共索引，以便其他应用程序或者服务能够检索到。如果设置为YES，则表明您希望该用户活动可以被其他应用程序访问和索引。
    userActivity.title = @"BehaviorA Title"; //用户活动的标题
    userActivity.persistentIdentifier = @"myUserBehaviorA";  //唯一地标识用户活动
    userActivity.userInfo = @{@"testKey" : @"testValue"};
    userActivity.webpageURL = [NSURL URLWithString:@"https://www.infoq.cn/article/lsuikfdr0eyv2lusatjt"];
    if (@available(iOS 14.0, *)) {
        userActivity.contentAttributeSet = [[CSSearchableItemAttributeSet alloc] initWithContentType:UTTypeItem];
        userActivity.contentAttributeSet.contentDescription = @"BehaviorA subTitle";
    }
    [userActivity becomeCurrent];
}
```

点击 Siri建议后的回调

```
- (BOOL)application:(UIApplication *)application continueUserActivity:(NSUserActivity *)userActivity restorationHandler:(void (^)(NSArray<id<UIUserActivityRestoring>> * _Nullable))restorationHandler {
    NSLog(@"continueUserActivity");
    if ([userActivity.activityType isEqualToString:@"loying.LearnSiriShortcut.type"]) {
        // 做自己的业务逻辑
    }
    return YES;
}

```

进阶的siri建议

```
// 设置用户活动的标题
userActivity.title = @"View Latest Articles";

// 设置用户活动的时间信息
NSDate *startDate = [NSDate date];
NSDate *endDate = [NSDate dateWithTimeIntervalSinceNow:3600]; // 活动结束时间
userActivity.startDate = startDate;
userActivity.endDate = endDate;

// 设置用户活动的地点信息
CLLocationCoordinate2D location = CLLocationCoordinate2DMake(37.7749, -122.4194);
MKPlacemark *placemark = [[MKPlacemark alloc] initWithCoordinate:location];
userActivity.mapItem = [[MKMapItem alloc] initWithPlacemark:placemark];

// 配置用户活动的动作
NSUserActivity *alarmActivity = [[NSUserActivity alloc] initWithActivityType:@"com.example.app.setAlarm"];
alarmActivity.title = @"Set Alarm";
alarmActivity.userInfo = @{@"alarmTime": [NSDate dateWithTimeIntervalSinceNow:3600]};
[alarmActivity becomeCurrent];

// 设置用户活动的联系人信息
CNContact *contact = [[CNContact alloc] init];
contact.givenName = @"John";
contact.familyName = @"Doe";
userActivity.contact = contact;

// 配置用户活动的提醒信息
EKEventStore *eventStore = [[EKEventStore alloc] init];
EKReminder *reminder = [EKReminder reminderWithEventStore:eventStore];
reminder.title = @"Buy Groceries";
reminder.dueDateComponents = [[NSCalendar currentCalendar] components:NSCalendarUnitYear | NSCalendarUnitMonth | NSCalendarUnitDay fromDate:[NSDate date]];
userActivity.calendarItemIdentifier = [reminder.calendarItemIdentifier copy];

// 设置用户活动的网页链接
userActivity.webpageURL = [NSURL URLWithString:@"https://www.example.com"];

// 设置用户活动的附加信息
userActivity.userInfo = @{@"key": @"value"};

// 提交用户活动
[userActivity becomeCurrent];

```