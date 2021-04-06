---
layout: default
title: Cocoapods
nav_order: 2
parent: 工具
---

## 随记

更新:
>sudo gem install -n /usr/local/bin cocoapods

新版本冲突:
>sudo gem install concurrent-ruby


## CDN提速
在profile文件头部添加: source 'https://cdn.cocoapods.org/'


## repo

推送组件到私有库中
> pod repo push jfshare message-center-lib.podspec --allow-warnings --use-libraries

移除repo
>pod repo remove jfshare

创建repo
>pod repo create jfshare

添加本地私有repo
>pod repo add jfshare https://gitee.com/jfshare/Specs.git

输出日志
> pod install --verbose

本地验证
pod lib lint

创建组件
pod lib create message-center-lib

查看
pod repo list
