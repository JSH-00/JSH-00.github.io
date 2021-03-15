---
layout: git
title: UITableViewCell 注册重用
date: 2021-03-13 20:00:00
tags: OC
---
# UITableViewCell 注册重用

```
static NSString * cellID = @"TcellID";

# pragma mark 注册 GestureTableViewCell
[gestureTable registerClass:[GestureTableViewCell class] forCellReuseIdentifier:@"TcellID"];

# pragma mark 根据注册的 cellID 重用已注册的 cell
GestureTableViewCell *cell = [self.gestureTable dequeueReusableCellWithIdentifier:cellID];

# pragma mark 若 cell 未定义，则重用已注册的TcellID
if (nil == cell) {
    cell = [[GestureTableViewCell alloc] initWithStyle:UITableViewCellStyleSubtitle reuseIdentifier:cellID];
}
```

## 未重用导致的问题：
* UITableViewCell 异常
* 明明按照 index 返回不同的 cell，但多个 cell 内容都重叠在一起
* 点击 cell 后，上面的 label 等内容会消失
* 上下滑动后，上面的 label 等内容会消失

## 示例
### MyTableViewCell.m

```
+ (instancetype)cellWithTableView:(UITableView *)tableView CellModel:(MyCellModel *)myCellModel {
    static NSString *identifier = @"myFirstCell";
    MyTableViewCell *cell = [tableView dequeueReusableCellWithIdentifier:identifier];
    if (cell == nil) {
        cell = [[MyTableViewCell alloc]initWithStyle:UITableViewCellStyleSubtitle reuseIdentifier:identifier];
        [cell setUI];
    }
    cell.model = myCellModel;
    return cell;
}
```

### MyTableViewController.m

```
- (UITableViewCell *)tableView:(UITableView *)tableView cellForRowAtIndexPath:(NSIndexPath *)indexPath {
    return [MyTableViewCell cellWithTableView:tableView CellModel:model];
}
```



