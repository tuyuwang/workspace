---
layout: default
title: crontab
nav_order: 2
parent: shell
---

crontab命令:

- e: 编辑任务
- l: 展示任务
- r: 删除任务

格式:

~~~
* * * * * command

分 时 日 月 周 执行的命令

0-59 0-23 0-31 1-11 0-7
~~~


执行结果失败查询方式: 
~~~
查询本机邮件:
mail 
~~~