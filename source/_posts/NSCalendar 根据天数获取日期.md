---
layout: git
title: NSCalendar 根据天数获取日期
date: 2022-11-17 20:00:00
tags: OC
--- 

# NSCalendar 根据天数获取日期
* 可以用于设置 UIDatePicker 可选时间段

## 获取明日凌晨以及后面天数
```
    NSCalendar *calendar = [[NSCalendar alloc] initWithCalendarIdentifier:NSCalendarIdentifierGregorian];
    NSDate *currentDate = [NSDate date];
    NSDateComponents *comps = [calendar components:NSCalendarUnitYear|NSCalendarUnitMonth|NSCalendarUnitDay fromDate:currentDate]; // 时间取整
    NSDate *startDate = [calendar dateFromComponents:comps];
    NSDate *maxDate = [calendar dateByAddingUnit:NSCalendarUnitDay value:31 toDate:startDate options:0];
    NSDate *minDate = [calendar dateByAddingUnit:NSCalendarUnitDay value:1 toDate:startDate options:0];
```

## 获取当前时间后的30天
```
    NSCalendar *calendar = [[NSCalendar alloc] initWithCalendarIdentifier:NSCalendarIdentifierGregorian];
    NSDate *currentDate = [NSDate date];
    NSDateComponents *comps = [[NSDateComponents alloc] init];
    [comps setDay:31]; // 设置 31 天后
    NSDate *maxDate = [calendar dateByAddingComponents:comps toDate:currentDate options:0];
    [comps setDay:1]; // 设置 1 天后
    NSDate *minDate = [calendar dateByAddingComponents:comps toDate:currentDate options:0];
```

## 设置 UIDatePicker

```
- (UIDatePicker *)datePicker {
    if(!_datePicker) {
        _datePicker = [[UIDatePicker alloc]init];
        _datePicker.backgroundColor = [UIColor hho_colorWithHex:0x4AC052 alpha:0.12];
        _datePicker.tintColor =  [UIColor hho_colorWithHex:0x4AC052];
        _datePicker.locale = [NSLocale localeWithLocaleIdentifier:@"ja-Latn-hepburn"];
        _datePicker.datePickerMode = UIDatePickerModeDateAndTime;
        _datePicker.preferredDatePickerStyle = UIDatePickerStyleWheels;
        _datePicker.minuteInterval = 30;
        NSCalendar *calendar = [[NSCalendar alloc] initWithCalendarIdentifier:NSCalendarIdentifierGregorian];
        NSDate *currentDate = [NSDate date];
        NSDateComponents *comps = [calendar components:NSCalendarUnitYear|NSCalendarUnitMonth|NSCalendarUnitDay fromDate:currentDate];
        NSDate *startDate = [calendar dateFromComponents:comps];
        NSDate *maxDate = [calendar dateByAddingUnit:NSCalendarUnitDay value:31 toDate:startDate options:0];
        NSDate *minDate = [calendar dateByAddingUnit:NSCalendarUnitDay value:1 toDate:startDate options:0];
        [_datePicker setMaximumDate:maxDate];
        [_datePicker setMinimumDate:minDate];
    }
    return _datePicker;
}
```

参考链接：https://blog.csdn.net/cse110/article/details/50433972