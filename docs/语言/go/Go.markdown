---
layout: default
title: Go
nav_order: 4
parent: 语言
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

docker rm -f photoprism  
docker stop photoprism
docker start photoprism

~~~
docker run -d \
  --name photoprism \
  --security-opt seccomp=unconfined \
  --security-opt apparmor=unconfined \
  -p 2342:2342 \
  -e PHOTOPRISM_UPLOAD_NSFW="true" \
  -e PHOTOPRISM_ADMIN_PASSWORD="please-change" \
  -v /photoprism/storage \
  -v ~/Pictures:/photoprism/originals \
  -v ~/Movies/Videos:/photoprism/originals/Videos \
  photoprism/photoprism
~~~


docker-compose up -d --no-deps photoprism
docker-compose start photoprism
docker-compose up -d
docker-compose pull photoprism
docker-compose stop photoprism
docker-compose exec photoprism photoprism index

### 配置apache2反向代理
sudo apt-get install apache2
cd /etc/apache2/sites-available
cp -r 000-default.conf ./photoprism.conf
sudo vim photoprism.conf

~~~
sudo a2ensite linuxidc.com.conf
systemctl reload apache2
sudo a2dissite 000-default.conf

sudo apache2ctl configtest
sudo systemctl status apache2
sudo systemctl restart apache2
~~~

RewriteEngine命令需要rewrite mod的支持，
~~~
cd /etc/apache2/mods-enabled
sudo ln -s ../mods-available/rewrite.load rewrite.load 
sudo /etc/init.d/apache2 restart
~~~

#### 开源学习
- [社区网站](https://github.com/shen100/golang123)
- [gin](https://github.com/gin-gonic/gin)
- [goproxy](https://goproxy.io): 解决go get xxx失败问题
