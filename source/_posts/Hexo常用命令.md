---
layout: git
title: Hexo 常用命令
date: 2021-01-17 20:00:00
tags: Hexo
---
# Hexo 常用命令
## 安装 Hexo (基于node)
```
sudo npm install -g hexo-cli
```
## 清除缓存
```
hexo clean
```
## 启动服务预览
```
hexo s
```

## 生成并部署
```
hexo d -g
```
## 服务器
### 更改端口
```
hexo server -p 5000
```
### 自定义 IP

```
hexo server -i 192.168.1.1 
```

## 插入图片
### 设置站点配置
进入`_config.yml`，将`post_asset_folder: false`改为`post_asset_folder: true`
### 安装插件
`npm install https://github.com/CodeFalling/hexo-asset-image -- save`
### 生成 .md 及图片文件夹
运行`hexo n "XXXXXX"`，生成`XXXXX.md`博文时就会在`/source/_posts`目录下生成 XXXXXX 的文件夹，将你想在 XXXXX 博文中插入的照片放置到这个同名文件夹中即可，图片的命名随意。
### 添加图片
在想添加的位置写入`![](图片名字.图片格式)`，如`![](1.png)`。