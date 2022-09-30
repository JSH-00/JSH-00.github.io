---
layout: git
title: Masonry 多控件水平/竖直布局
date: 2022-07-31 20:00:00
tags: OC
---
# Masonry 多控件水平/竖直布局

```
//水平方向宽度固定等间隔
//每个 item 水平方向宽度固定为 110，item 之间间隔自动计算，第一个item相对父view左侧大小：leadSpacing，最后一个item相对父view右侧大小：tailSpacing

[self.viewArray mas_distributeViewsAlongAxis:MASAxisTypeHorizontal withFixedItemLength:110 leadSpacing:12 tailSpacing:12];
[self.viewArray mas_makeConstraints:^(MASConstraintMaker *make) {
    make.bottom.mas_equalTo(self);
    make.height.mas_equalTo(112);
}];


//竖直方向高度固定等间隔
[self.viewArray mas_distributeViewsAlongAxis:MASAxisTypeVertical withFixedItemLength:104 leadSpacing:10 tailSpacing:10];
[self.viewArray mas_makeConstraints:^(MASConstraintMaker *make) {
    make.right.mas_equalTo(self);
    make.width.mas_equalTo(110);
}];

// 水平方向排列、控件间隔固定为50、控件长度不定
[btns mas_distributeViewsAlongAxis:MASAxisTypeHorizontal withFixedSpacing:50 leadSpacing:10 tailSpacing:10];
```