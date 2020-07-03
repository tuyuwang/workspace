---
layout: default
title: go
nav_order: 4
permalink: /docs/go
has_children: true
---

- [Go 高级编程](https://chai2010.gitbooks.io/advanced-go-programming-book/content/)

### 命令
移除当前源码包里面编译生成的文件:
> go clean

该命令实际分为两步，第一步下载源码包，第二步执行go install:
> go get github.com/xxx

该命令实际分为两步，第一步是生成结果文件，第二步把结果文件移到$GOPATH/pkg或$GOPATH/bin.
> go install 

自动读取源码目录下名为*_test.go的文件，生成并运行测试用的可执行文件。
> go test

查看指定包文档
> go doc net/http

查看指定函数
> godoc fmt Printf

查看指定函数代码
> godoc -src fmt Printf

运行本地文档
> godoc -http=:8080

查看所有安装的package
> go list

#### 开源学习
- [社区网站](https://github.com/shen100/golang123)
- [gin](https://github.com/gin-gonic/gin)
- [goproxy](https://goproxy.io): 解决go get xxx失败问题
