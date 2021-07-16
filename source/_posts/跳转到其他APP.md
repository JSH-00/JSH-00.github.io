---
layout: git
title: 跳转到其他APP (苹果地图 & 高德地图 & phone)
date: 2021-06-14 20:00:00
tags: OC
---
## 苹果地图 && [高德地图](https://lbs.amap.com/api/amap-mobile/guide/ios/route)

```
- (void)jumpToMap {
    // 跳转到地图页面
    self.mapActionController = [UIAlertController alertControllerWithTitle:@"选择跳转的地图" message:nil preferredStyle:UIAlertControllerStyleActionSheet];
    UIAlertAction * phoneNum1 = [UIAlertAction actionWithTitle:@"苹果地图" style:UIAlertActionStyleDefault handler:^(UIAlertAction * _Nonnull action) {
        // 添加跳转到苹果地图
        
    }];

    UIAlertAction * phoneNum2 = [UIAlertAction actionWithTitle:@"高德地图" style:UIAlertActionStyleDefault handler:^(UIAlertAction * _Nonnull action) {
        // 添加跳转到高德地图
        // 路径规划 URL：iosamap://path?sourceApplication=applicationName&sid=BGVIS1&slat=39.92848272&slon=116.39560823&sname=A&did=BGVIS2&dlat=39.98848272&dlon=116.47560823&dname=B&dev=0&m=0&t=0
//        NSString * urlString = [NSString stringWithFormat:@"iosamap://path?sourceApplication=%@&dlat=%@dlon=%@&dname=B&dev=0&m=0&t=0",@"CityMi", @30.26211, @120.17571];

        NSString * urlString = [NSString stringWithFormat:@"iosamap://navi?sourceApplication=%@&backScheme=%@&lat=%@&lon=%@&dev=0&style=2",@"CityMi",@"MyCityMi", @30.26211, @120.17571];
        if ([[UIApplication sharedApplication] respondsToSelector:@selector(openURL:options:completionHandler:)]) {
            [[UIApplication sharedApplication] openURL:[NSURL URLWithString:urlString] options:@{}
                                     completionHandler:^(BOOL success) {
                NSLog(@"Open 高德地图: %d",success);
            }];
        } else {
            BOOL success = [[UIApplication sharedApplication] openURL:[NSURL URLWithString:urlString]];
            NSLog(@"Open 高德地图: %d",success);
        }
    }];

    
    UIAlertAction * cancel = [UIAlertAction actionWithTitle:@"取消" style:UIAlertActionStyleCancel handler:^(UIAlertAction * _Nonnull action) {
        [self.mapActionController dismissViewControllerAnimated:YES completion:nil];
    }];

    [self.mapActionController addAction:phoneNum1];
    [self.mapActionController addAction:phoneNum2];
    [self.mapActionController addAction:cancel];
    [self presentViewController:self.mapActionController animated:YES completion:nil];
}
```
## 打电话
```
[[UIApplication   sharedApplication] openURL:[NSURL URLWithString:@"tel://10010"] options:@{} completionHandler:nil];

```
