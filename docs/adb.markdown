---
layout: default
title: adb 
nav_order: 13
---

## 安装
~~~
brew cask install android-platform-tools
~~~

## 使用

- 将android手机通过USB数据线连接Mac，打开终端输入system_profiler SPUSBDataType;
- 找到对应设备的Product ID;
- 在终端输入vim ~/.android/adb_usb.ini，将设备的Product ID添加到该文件中，保存退出；
- 终端输入：adb kill-server；
- 终端输入：adb start-server / adb devices

## 命令
~~~
查看连接计算机的设备：
adb devices

获取手机系统信息（ CPU，厂商名称等）(需root权限)
adb shell "cat /system/build.prop | grep "product""

获取序列号：
adb get-serialno
or
adb -d shell getprop ro.product.model

获取CPU序列号：
adb shell cat /proc/cpuinfo

安装指定apk(路径可不用手写，直接把apk文件拖拽过来)：
adb  install <file>

卸载指定包 ：
adb uninstall <package>

连接设备 ：
adb connect [<host>[:<port>]]（默认端口号是：5555）

断开设备：
disconnect [<host>[:<port>]]

执行远程的shell：
adb shell

退出远程命令：
exit

执行远程shell命令：
adb shell <command>

拷贝文件到设备上：
adb push <local> <remote>

从设备中拷贝文件：
adb pull <remote> [<local>]

查看设备所有信息：
adb bugreport（包括 bug 报告）
~~~


