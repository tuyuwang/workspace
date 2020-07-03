---
layout: default
title: git
nav_order: 8
---

## 随记

#### 提速

在开vpn后，速度提升百倍
~~~
// 关闭vpn后
git config --global http.proxy 'socks5://127.0.0.1:1086'
git config --global https.proxy 'socks5://127.0.0.1:1086'

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
git config --global http.postBuffer 524288000
git config --list
~~~

#### 下载指定文件
'childfile'替换为自己要下载的文件夹名称
1. git init test && cd test     //新建仓库并进入文件夹
2. git config core.sparsecheckout true //设置允许克隆子目录
3. echo 'childfile*' >> .git/info/sparse-checkout //设置要克隆的仓库的子目录路径   //空格别漏 
4. git remote add origin git@github.com:mygithub/test.git  //这里换成你要克隆的项目和库
5. git pull origin master    //下载

更新远端分支
> git remote update origin --prune  
