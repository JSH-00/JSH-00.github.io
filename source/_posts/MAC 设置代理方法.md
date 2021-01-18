---
layout: git
title: MAC 设置代理方法
date: 2021-01-18 09:00:00
tags: MacOS
---
# MAC 设置代理方法
mac 使用命令行更换代理。可根据此设计脚本，实现一键设置代理，减少开启/关闭代理流程。
## 预览 networksetup 基本功能

```
networksetup -h
```

## 查看当前 net work services

```
networksetup -listallnetworkservices
```

## 设置代理
开启
```
sudo networksetup -setwebproxy "Wi-Fi" 192.168.00.00 2900
sudo networksetup -setsecurewebproxy "Wi-Fi" 192.168.98.48 2900
sudo networksetup -setsocksfirewallproxy "Wi-Fi" 192.168.98.48 2900
```

关闭
```
sudo networksetup -setwebproxystate "Wi-Fi" off
sudo networksetup -setsecurewebproxystate "Wi-Fi" off
sudo networksetup -setsocksfirewallproxystate "Wi-Fi" off
```
## 手动方法
以上命令行可代替的手动方法如下
* 设置->网络->高级->代理
* 勾选并逐个设置代理服务器：192.168.98.48:2900
    * 网页代理（HTTP）
    * 安全网页代理（HTTPS）
    * SOCKS代理
     