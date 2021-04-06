---
layout: default
title: shell
nav_order: 4
parent: 语言
has_children: true
---

## 随记

1、表格输出:
- 构建形式，约定以|为制表符
- 输入命令，将文件重定向
- column -t -s ‘|’ tmp > tmp1 
- 重命名文件
- mv tmp1 tmp

2、判断浮点数值大小
~~~
a=9.2
b=10.2
c=`echo "$a > $b" | bc`

echo $c
~~~

#### 模拟硬件输入

模拟键盘，鼠标点击:
[cliclick](https://github.com/BlueM/cliclick)

~~~
➜ ~ cliclick p
1382,71
➜ ~ cliclick c:1382,71
~~~

ls -a 显示隐藏文件

查看ssh
cat ~/.ssh/id_rsa.pub

### 生成ssl
openssl genrsa -des3 -out ssl.key 1024

然后他会要求你输入这个key文件的密码。不推荐输入。因为以后要给nginx使用。每次reload nginx配置时候都要你验证这个PAM密码的。

由于生成时候必须输入密码。你可以输入后 再删掉。
mv ssl.key xxx.key
 
openssl rsa -in xxx.key -out ssl.key
 
rm xxx.key

然后根据这个key文件生成证书请求文件
openssl req -new -key ssl.key -out ssl.csr

以上命令生成时候要填很多东西 一个个看着写吧（可以随便，毕竟这是自己生成的证书）最后根据这2个文件生成crt证书文件
openssl x509 -req -days 3650 -in ssl.csr -signkey ssl.key -out ssl.crt

### 输出重定向
针对xocde post actions:
> exec > /tmp/my_log_file.txt 2>&1

### zip
-A 调整可执行的自动解压缩文件。
-b<工作目录> 指定暂时存放文件的目录。
-c 替每个被压缩的文件加上注释。
-d 从压缩文件内删除指定的文件。
-D 压缩文件内不建立目录名称。
-f 此参数的效果和指定"-u"参数类似，但不仅更新既有文件，如果某些文件原本不存在于压缩文件内，使用本参数会一并将其加入压缩文件中。
-F 尝试修复已损坏的压缩文件。
-g 将文件压缩后附加在既有的压缩文件之后，而非另行建立新的压缩文件。
-h 在线帮助。
-i<范本样式> 只压缩符合条件的文件。
-j 只保存文件名称及其内容，而不存放任何目录名称。
-J 删除压缩文件前面不必要的数据。
-k 使用MS-DOS兼容格式的文件名称。
-l 压缩文件时，把LF字符置换成LF+CR字符。
-ll 压缩文件时，把LF+CR字符置换成LF字符。
-L 显示版权信息。
-m 将文件压缩并加入压缩文件后，删除原始文件，即把文件移到压缩文件中。
-n<字尾字符串> 不压缩具有特定字尾字符串的文件。
-o 以压缩文件内拥有最新更改时间的文件为准，将压缩文件的更改时间设成和该文件相同。
-q 不显示指令执行过程。
-r 递归处理，将指定目录下的所有文件和子目录一并处理。
-S 包含系统和隐藏文件。
-t<日期时间> 把压缩文件的日期设成指定的日期。
-T 检查备份文件内的每个文件是否正确无误。
-u 更换较新的文件到压缩文件内。
-v 显示指令执行过程或显示版本信息。
-V 保存VMS操作系统的文件属性。
-w 在文件名称里假如版本编号，本参数仅在VMS操作系统下有效。
-x<范本样式> 压缩时排除符合条件的文件。
-X 不保存额外的文件属性。
-y 直接保存符号连接，而非该连接所指向的文件，本参数仅在UNIX之类的系统下有效。
-z 替压缩文件加上注释。
-$ 保存第一个被压缩文件所在磁盘的卷册名称。
-<压缩效率> 压缩效率是一个介于1-9的数值。

zip -q -r html.zip /home/html

压缩包含隐藏文件
zip -r swift.zip * .[^.]*


#### 恢复误删文件
grep -a -B 50 -A 60 'some string in the file' /dev/sda1 > results.txt

- 关于grep的-a意为–binary-files=text，也就是把二进制文件当作文本文件。

- -B和-A的选项就是这段字符串之前几行和之后几行。

- /dev/sda1，就是硬盘设备，> results.txt，就是把结果重定向到results.txt文件中。

#### json解析工具
> brew install jq