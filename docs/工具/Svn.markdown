---
layout: default
title: Svn
nav_order: 5
parent: 工具
---

## 随记

~~~
//只更新部分文件:
svn co --depath=empty address --username name --password pwd

cd dir

// 输出文件目录
svn list

svn up filename


svn的隐藏文件.subversion

//回滚到r2:
svn merge -r -r10:r2 filename
~~~

