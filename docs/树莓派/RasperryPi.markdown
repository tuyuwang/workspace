---
layout: default
title: RasperryPi
nav_order: 1
parent: 树莓派
---

## 文件传输

#### sftp

~~~shell
#登陆
sftp pi@192.168.2.17

#上传
put /home/fuyatao/downloads/Linuxgl.pdf /var/www/fuyatao/

#下载
get /var/www/kyu/index.php  /home/kyu/
~~~

## 随记

安装docker:
~~~
sudo curl -sSL https://get.docker.com | sh
~~~