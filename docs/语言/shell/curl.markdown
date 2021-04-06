---
layout: default
title: curl
nav_order: 3
parent: shell
grand_parent: 语言
---

## 随记

#### curl
上传文件:
~~~
curl -X POST http://localhost:8080/upload \
  -F "upload[]=@/Users/appleboy/test1.zip" \
  -F "upload[]=@/Users/appleboy/test2.zip" \
  -H "Content-Type: multipart/form-data"
~~~

请求参数
~~~
curl -X POST http://localhost:8080/login -H 'content-type: application/json' -d '{"user": "tuyw"}' 
{"error":"Key: 'Login.Password' Error:Field validation for 'Password' failed on the 'required' tag"}

//-v
curl -v -X POST http://localhost:8080/login -H 'content-type: application/json' -d '{"user": "tuyw"}'
Note: Unnecessary use of -X or --request, POST is already inferred.
*   Trying ::1...
* TCP_NODELAY set
* Connected to localhost (::1) port 8080 (#0)
> POST /login HTTP/1.1
> Host: localhost:8080
> User-Agent: curl/7.54.0
> Accept: */*
> content-type: application/json
> Content-Length: 16
> 
* upload completely sent off: 16 out of 16 bytes
< HTTP/1.1 400 Bad Request
< Content-Type: application/json; charset=utf-8
< Date: Mon, 16 Sep 2019 06:52:41 GMT
< Content-Length: 101
< 
{"error":"Key: 'Login.Password' Error:Field validation for 'Password' failed on the 'required' tag"}
* Connection #0 to host localhost left intact
~~~

下载文件:
~~~
// 以服务器文件名命名
curl -O https://opensource.apple.com/tarballs/WebKit2/WebKit2-7607.2.6.1.1.tar.gz

// 自定义命名
curl -o webkit.tar.gz https://opensource.apple.com/tarballs/WebKit2/WebKit2-7607.2.6.1.1.tar.gz
~~~