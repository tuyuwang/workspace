---
layout: default
title: Xcode
nav_order: 4
parent: 经验
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

#### block编译问题
build settings -> other c flags -> 
>-Xclang -fcompatibility-qualified-id-block-type-checking

#### 查看Mach-O
使用 $ xcrun size -lm <binary-path> 指令可以查看 Mach-O 文件 Data 部分的结构和各 Segment/Section 的大小信息（该 Mach-O 文件由 Xcode 的 iOS App 模板工程构建而来）。在不需要更详细的信息时，这条命令很方便。
