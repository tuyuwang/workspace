---
layout: default
title: Git
nav_order: 1
parent: 工具
---

## 随记

#### 提速

在开vpn后，速度提升百倍
~~~
// 关闭vpn后
git config --global http.proxy 'socks5://127.0.0.1:1086'
git config --global https.proxy 'socks5://127.0.0.1:1086'

置好代理之后去终端输入git配置命令，开启翻墙代理，电脑连手机4g热点， 命令如下


git config --global http.proxy socks5://127.0.0.1:1086
git config --global http.https://github.com.proxy socks5://127.0.0.1:1086

更新的时候，速度可达到1M/S ，更新完，可以直接通过下面的命令恢复


git config --global --unset http.proxy
git config --global --unset http.https://github.com.proxy

// 使用完后，取消上面操作
git config --global --unset http.proxy
git config --global --unset https.proxy
~~~

#### 上传文件限制

报错:
~~~
error: RPC failed; curl 18 transfer closed with outstanding read data remaining
 
fatal: The remote end hung up unexpectedly
 
fatal: early EOF
 
~~~

解决:
~~~
// 1G
git config --global http.postBuffer 1048576000
git config --list

// 压缩
git config --global core.compression 9


//或者
//拉取指定分支最近100次的提交
git clone --depth 100  --branch developer http://xxxx.git

//拉取其他分支
$ git clone --depth 1 https://github.com/dogescript/xxxxxxx.git
$ git remote set-branches origin 'remote_branch_name'
$ git fetch --depth 1 origin remote_branch_name
$ git checkout remote_branch_name
~~~

compression表示压缩，从clone的终端输出就知道，服务器会压缩目标文件，然后传输到客户端，客户端再解压。取值为 [-1, 9]，-1以zlib 为默认压缩库，0 表示不进行压缩，1..9是压缩速度与最终获得文件大小的不同程度的权衡，数字越大，压缩越慢，得到的文件会越小

#### 下载指定文件
'childfile'替换为自己要下载的文件夹名称
1. git init test && cd test     //新建仓库并进入文件夹
2. git config core.sparsecheckout true //设置允许克隆子目录
3. echo 'childfile*' >> .git/info/sparse-checkout //设置要克隆的仓库的子目录路径   //空格别漏 
4. git remote add origin git@github.com:mygithub/test.git  //这里换成你要克隆的项目和库
5. git pull origin master    //下载

更新远端分支
> git remote update origin --prune  

删除与tag重名时的分支，需要指定路径
>git push origin :refs/heads/1.1.1

>git push origin :refs/tags/1.1.1


## git 上传大文件

>brew install git-lfs

具体就是安装git-lfs，先下载，然后就是一顿操作：

2-1. 安装，我是用的是Macbook Pro，所以选择macOS用户安装方式 Homebrew 安装
brew install git-lfs
2-2. 打开终端，cd到git仓库本地路径，初始化lfs
git lfs install
2-3. 追踪单个文件
git lfs track
eg:git lfs track "*.psd"
2-4. 添加lfs追踪文件，提交仓库（此处一定要先提交追踪文件到仓库，在提交其他文件）
git add .gitattributes
git commit -m "track *.psd files using Git LFS"
git add .
git commit -m "submit other files"
2-5. 验证是否追踪大文件，如果输入后不显示则追踪不成功
git lfs ls-files
2-6. 推送至远程仓库
git push origin master


打tag
git tag 0.1.1
git push --tags
git push origin v1.5

删
本地 git tag -d 0.0.1
远端 git push origin :refs/tags/0.1.2


## 日志
git log --pretty=format:"git id: %h commit: %s" 
~~~
%H   提交对象（commit）的完整哈希字串
%h    提交对象的简短哈希字串
%T    树对象（tree）的完整哈希字串
%t    树对象的简短哈希字串
%P    父对象（parent）的完整哈希字串
%p    父对象的简短哈希字串
%an   作者（author）的名字
%ae   作者的电子邮件地址
%ad   作者修订日期（可以用 -date= 选项定制格式）
%ar   作者修订日期，按多久以前的方式显示
%cn   提交者(committer)的名字
%ce   提交者的电子邮件地址
%cd   提交日期
%cr   提交日期，按多久以前的方式显示
%s    提交说明
~~~

git log 1.1.3.. --pretty=format:"author: %an %s" | grep -E " bug:| opt:" | cat -n


## 回滚版本
git reset --hard xxxx
git push -f origin v1.x.x

1、找到提交有误的版本号
2、git revert -n 版本号
3、git commit -m “xx”
4、git push v1.x.x

## 本地合并commits
~~~
git rebase -i [startPoint] [endPoint]
~~~
其中-i的意思是–interactive，即弹出交互式的界面让用户编辑完成合并操作，[startpoint] [endpoint]则指定了一个编辑区间，如果不指定[endpoint]，则该区间的终点默认是当前分支HEAD所指向的commit(注：该区间指定的是一个前开后闭的区间)。

~~~
// 合并从当前head到15f745b(commit id)
git rebase -i 15f745b
或:
// 合并最近的两次提交
git rebase -i HEAD~2
~~~

执行这个命令后会跳到一个vi编辑器
里面的提示有：
pick：保留该commit（缩写:p）
reword：保留该commit，但我需要修改该commit的注释（缩写:r）
edit：保留该commit, 但我要停下来修改该提交(不仅仅修改注释)（缩写:e）
squash：将该commit和前一个commit合并（缩写:s）
fixup：将该commit和前一个commit合并，但我不要保留该提交的注释信息（缩写:f）
exec：执行shell命令（缩写:x）
drop：我要丢弃该commit（缩写:d）

## github慢
一、确定ip
进入网址https://github.com.ipaddress.com

查看GitHub的ip地址。

140.82.112.3 github.com


二、确定域名ip
进入网址https://fastly.net.ipaddress.com/github.global.ssl.fastly.net


199.232.69.194 github.global.ssl.fastly.net


三、确定静态资源ip
进入网址https://github.com.ipaddress.com/assets-cdn.github.com


185.199.108.153 assets-cdn.github.com

185.199.110.153 assets-cdn.github.com

185.199.111.153 assets-cdn.github.com

