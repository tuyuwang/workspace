---
layout: default
title: Mac
nav_order: 11
---

## 随记

查看端口占用情况:
~~~
lsof -i :4000
~~~

查询进程
~~~
ps -ef | grep workspace
~~~

关闭进程
~~~
kill -9 pid
~~~


### ls

- ls 查看当前路径下的文件及文件夹名称
- -a 查看隐藏文件
- -l 列表风格显示
- -h 配合-l，显示一个合理的大小单位

### 文件

- rmdir: 删除空文件夹
- rm: 删除一个文件夹，有提示
- rm xxx -r: 递归删除文件夹，不提示
- cp a b 将a的文件夹整体复制到b文件夹下
- cp a/* b 将a文件夹下的所有内容复制到b文件夹下
- tar -cvf xxx.tar * 打包
- gzip xxx.tar 压缩
- gzip -d xxx.tar.gz 解压缩
- tar -xvf xxx.tar 解包

### grep

- -n: 显示行号
- -i: 不区分大小写
- -v: 取反，即不包含需要的内容的行

### adb
> brew cask install android-platform-tools

### 云服务器
ip: 47.107.172.52
密码: 280294
控制台: https://ecs.console.aliyun.com/?spm=5176.2020520127.aliyun_sidebar.daliyun_sidebar_ecs.81d11a78XJV9jv#/server/region/cn-shenzhen?status=Running
