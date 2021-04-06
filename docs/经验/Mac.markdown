---
layout: default
title: Mac
nav_order: 1
parent: 经验
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

### 访问github.io失败

[访问github.io失败](https://www.cnblogs.com/yangzhou33/p/13973868.html)

~~~shell
ping tuyuw.github.io

PING github.io (127.0.0.1): 56 data bytes
64 bytes from 127.0.0.1: icmp_seq=0 ttl=64 time=0.030 ms
64 bytes from 127.0.0.1: icmp_seq=1 ttl=64 time=0.105 ms
64 bytes from 127.0.0.1: icmp_seq=2 ttl=64 time=0.084 ms
~~~

上面显示 ip 地址为127.0.0.1，这个就是本机地址嘛，然后我去/etc/hosts里面查看hosts文件有没有被意外修改过，没看到github.io被解析到127.0.0.1的记录，这就表示，电脑上的dns解析是完好的，但是远程的 dns 解析很可能惨遭网络运营商的污染了，从而导致解析的时候被解析到了127.0.0.1。


修改本机 dns

于是我们去修改本机的dns，加入114.114.144.114和8.8.8.8，这两个 dns 都是非商业用途的 dns，解析成功率很高，并且纯净无广告。前一个是国内移动、电信和联通通用的 DNS，是国内用户上网常用的DNS；后一个是 GOOGLE公司提供的 DNS，该全球通用的。

这里以 mac 为例，依次打开系统偏好设置->网络->高级->dns，然后加入即可。加入之后需要刷新mac缓存，命令如下（记住一定要带上sudo，否则不会生效）：
~~~shell
sudo killall -HUP mDNSResponder
~~~

生成ssh密钥：
>ssh-keygen -t rsa -C "xxxxx@xxxxx.com"  

>cat ~/.ssh/id_rsa.pub

将已有的ssh密钥复制到粘贴板中：
>pbcopy < ~/.ssh/id_rsa.pub


## 破解软件

一、提示“XXX已损坏，打不开，您应该将它移至废纸篓”的解决方法：

 

1. 打开终端，输入命令：“sudo spctl --master-disable”然后回车；

 

2. 提示 “Password:” 输入Mac系统密码（这里输入密码是不显示的）然后回车即可。 


3. 如果还不行，在终端粘贴复制输入命令：“sudo xattr -r -d com.apple.quarantine ”（注意最后有一个空格），打开 “访达”（Finder）进入 “应用程序” 目录，找到该软件图标，将图标拖到刚才的终端窗口里面会得到类似命令：“sudo xattr -r -d com.apple.quarantine /Applications/Downie\ 3.app”，回到终端窗口按回车。

 

二、打开软件提示“意外退出”、“应该移到废纸篓”等解决方法：

 

1. 点击桌面底部程序坞的 “启动台”（小火箭）图标 → “其他” → “终端” 打开终端；

 

2. 打开终端后，输入命令：“xcode-select --install”然后回车，接着点击“install”等待安装完成；

 

3. 在终端输入：“sudo codesign -f -s - ”，打开 “访达”（Finder）进入“应用程序”目录，找到打不开的软件，将该软件图标拖到刚才的终端窗口里面（例如：“codesign --force --deep --sign - /Applications/Downie\ 3.app”），回到终端窗口按回车，提示“replacing existing signature”即可。