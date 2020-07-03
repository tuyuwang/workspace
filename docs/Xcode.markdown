---
layout: default
title: Xcode
nav_order: 7
---

## 随记

#### xcodebuild

单元测试:
~~~
xcodebuild test -scheme bonreeAgent -target bonreeAgent -destination 'platform=iOS Simulator,name=iPhone 8 Plus,OS=12.2'
~~~

#### plist
~~~
/usr/libexec/PlistBuddy -c "Print NSLocationWhenInUseUsageDescription" Info.plist
~~~

#### 代码段

path: cd ~/Library/Developer/Xcode/UserData/CodeSnippets

#### Templates

path: 
~~~
cd /Applications/Xcode.app/Contents/Developer/Platforms/iPhoneOS.platform/Developer/Library/Xcode/Templates/File\ Templates/Source
sudo git clone https://github.com/QMUI/QMUI_iOS_Templates.git QMUI\ Class.xctemplate
~~~

#### run 环境设置

- 展示启动时间耗时，DYLD_PRINT_STATISTICS = 1
- - 详细耗时: DYLD_PRINT_STATISTICS_DETAILS = 1