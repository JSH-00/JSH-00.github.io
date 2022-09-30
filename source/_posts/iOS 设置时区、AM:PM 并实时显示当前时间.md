---
layout: git
title: iOS 设置时区、AM/PM 并实时显示当前时间
date: 2022-06-05 20:00:00
tags: OC
---

# iOS 设置时区、AM/PM 并实时显示当前时间

## 完整代码

```
#import "ViewController.h"

@interface ViewController ()
@property (nonatomic,strong) NSTimer *liveTimer;
@property (nonatomic,strong) UILabel *liveBigLocalDateLabel;
@property (nonatomic,strong) UILabel *liveBigLocalTimeLabel;
@end

@implementation ViewController

- (void)viewDidLoad {
    [super viewDidLoad];
    // 设置定时器每秒更新 label
    self.liveTimer = [NSTimer scheduledTimerWithTimeInterval:1.0f target:self selector:@selector(timerFunc) userInfo:nil repeats:YES];
    
    [self.view addSubview:self.liveBigLocalTimeLabel];
    [self.view addSubview:self.liveBigLocalDateLabel];
    
}

- (void)timerFunc {
    NSDate *date = [NSDate date];
    // 设置 Time Zone ID，具体时区可以查阅表格（见参考链接）
    NSTimeZone *zone = [NSTimeZone timeZoneWithAbbreviation:@"Asia/Hong_Kong"];

    NSDateFormatter *formatterDate = [[NSDateFormatter alloc]init];
    [formatterDate setDateFormat:@"MM月dd日"];
    formatterDate.timeZone = zone;

    NSString *timestampDate = [formatterDate stringFromDate:date];
    [self.liveBigLocalDateLabel setText:timestampDate];
    
    NSDateFormatter *formatterTime = [[NSDateFormatter alloc]init];
    [formatterTime setDateFormat:@"HH:mm:ss a"];
    // 若不设置，默认显示 AM PM 和 AP P语言有关（中文显示 上午/下午）
    formatterTime.AMSymbol = @"AM";
    formatterTime.PMSymbol = @"PM";
    formatterTime.timeZone = zone;

    NSString *timestampTime = [formatterTime stringFromDate:date];
    [self.liveBigLocalTimeLabel setText:timestampTime];
}

- (UILabel *)liveBigLocalTimeLabel {
    if (_liveBigLocalTimeLabel == nil) {
        _liveBigLocalTimeLabel = [[UILabel alloc]initWithFrame:CGRectMake(20, 182, 350, 42)];
        _liveBigLocalTimeLabel.textColor = [UIColor purpleColor];
        [_liveBigLocalTimeLabel setFont:[UIFont fontWithName:@"HiraginoSans-W6" size:35]];
    }
    return _liveBigLocalTimeLabel;
}

- (UILabel *)liveBigLocalDateLabel {
    if (_liveBigLocalDateLabel == nil) {
        _liveBigLocalDateLabel = [[UILabel alloc]initWithFrame:CGRectMake(20, 130, 200, 30)];
        _liveBigLocalDateLabel.textColor = [UIColor purpleColor];
        [_liveBigLocalDateLabel setFont:[UIFont fontWithName:@"HiraginoSans-W6" size:35]];
    }
    return _liveBigLocalDateLabel;
}

@end
```

参考：
[Time Zone ID 查询](https://docs.oracle.com/middleware/12212/wcs/tag-ref/MISC/TimeZones.html)