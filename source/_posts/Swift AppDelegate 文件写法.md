---
layout: git
title: Swift APPDelegate 文件写法
date: 2021-01-10 17:00:00
tags: Swift
---
## 通过导航视图控制器设置根视图


AppDelegate.swift
```
import UIKit
@main
class AppDelegate: UIResponder, UIApplicationDelegate {

    var window: UIWindow?
    
    func application(_ application: UIApplication, didFinishLaunchingWithOptions launchOptions: [UIApplication.LaunchOptionsKey: Any]?) -> Bool {
        // Override point for customization after application launch.
        self.window = UIWindow(frame: UIScreen.main.bounds)
        // 把初始视图控制器压入导航视图控制器
        let navigationController = UINavigationController(rootViewController: MyRootViewController())
        // 把根视图控制器设为导航视图控制器
        self.window?.rootViewController = navigationController
        // 渲染页面
        return true
    }
}    
```