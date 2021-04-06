---
layout: default
title: docker
nav_order: 6
parent: 工具
---

## 随记

- [Docker镜像](http://mirrors.aliyun.com/docker-toolbox/mac/docker-for-mac/)
- [阿里巴巴](https://opsx.alibaba.com/mirror)


#### photoprism

图片管理web项目

docker安装:
~~~
docker run -p 2342:2342 -d --name photoprism \
  -v ~/Pictures:/home/photoprism/Pictures/Originals photoprism/photoprism

docker exec -ti photoprism photoprism index

docker stop photoprism
docker start photoprism

docker rm -f photoprism

//demo
docker run -p 2342:2342 -d --name demo photoprism/demo

//updating
docker pull photoprism/photoprism:latest
~~~
