---
layout: default
title: vim
nav_order: 4
parent: shell
---

### 模式切换

- 命令->编辑: i、a、o、l、A、O
- 编辑->命令: ESC
- 命令->末行: shift+:

### 模式内编辑

末行模式:
- w: 保存
- q: 退出
- !: 强制

### 浏览模式:
- hjkl: 左下上右
- M: 中间位置
- L: 当前屏幕最后一行
- yy: 复制, 8yy: 表示从当前光标所在行开始复制8行
- p: 粘贴
- dd: 剪切, 8dd: 表示从当前光标所在行开始剪切8行
- u: 撤销
- ctl+r: 反撤销
- G: 跳到最后一行
- 15G: 跳转到第15行
- 1G: 跳转到第一行
- gg: 跳转到第一行
- a: append 插入光标后
- i: insert 插入光标前
- o: 插入下一行
- A: 插入当前行最后
- I: 插入当前行最前
- O: 插入上一行
- ctrl+c、ctrl+[: 快速退出命令模式=esc
- w/W移到下一个word/WORD开头，e/E下一个word/WORD
- b/B回到上一个word/WORD开头
- word指的是以非空白符分割的单词，WORD以空白符分割的单词
- 0: 移动到行首第一个字符
- ^: 移动到第一个非空白字符
- $: 移动到行尾
- g_: 移动到行尾非空白字符
- (): 在句子间移动
- {}: 在段落之间移动
- ctrl+o: 快速返回上一次光标位置
- H/M/L: 跳转到界面头、中间、尾
- ctrl+u,ctrl+f: 上下翻页
- zz: 把屏幕置为中间
- x: 删除光标选中字符
- daw: 删除单词
- r: replace
- c: change 
- s: substitute
- /: 正向搜索
- ?: 反向搜索
- n: 跳转到下一个搜索结果
- N: 跳转到上一个搜索结果
- 使用*或#进行当前单词的向前或向后匹配


搜索
- f[char]可以移动到该字符上
- t[char]可以移动到该字符前一个字符
- ;或, : 分别对应搜索下一个、上一个
- F: 搜索顺序反过来

命令模式:
- vs: vertical split
- sp: 分屏
- ctrl+w: w:循环选择窗口；hjkl: 方向选择窗口;=:窗口等宽高；
- % s/foo/bar/g 全局替换
- ctrl+h: 删除上一个字符
- ctrl+w: 删除上一个单词
- ctrl+u: 删除当前行
- ls: 输出所有当前的缓冲区，b n跳转到第n个缓冲区
- bpre,bnext,bfirst,blast
- b name: 根据文件名跳转

Tab标签页操作
- tabe[dit]{filenames}: 在标签页中打开
- ctrl+w+T: 把当前窗口移到一个新标签页
- tabc[close]: 关闭当前标签页及其中所有的窗口
- tabo[nly]: 只保留活动标签页，关闭所有其他标签页
- tabn[ext] {N}: 切换到编号为{N}的标签页
- tabn[ext]: 切换到下一个标签页
- tabp[revious]: 切换到上一个标签页

替换命令:
- :[range]s[ubstitute]{pattern}/{string}/{flags}
- range表示范围，如10,20表示10-20行，%表示全部
- pattern: 是要替换的模式，string是要替换后的文本
- flags: g表示全局范围内执行，c表示确认，可以确认或拒绝修改；n输出匹配的次数

可视模式:

选择文字
- normal模式下用v进入visual选择
- 使用V选择行
- 使用ctrl+v进行方块选择

 
