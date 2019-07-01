---
title: "Linux命令  文件简单操作"
date: 2019-07-01T20:48:03+08:00
lastmod: 2019-07-01T20:48:03+08:00
draft: false
keywords: ["linux"]
description: "linux文件操作"
tags: ["linux"]
categories: ["linux"]
# author: ""
# comment: false
toc: true
contentCopyright: false
reward: false
mathjax: false
---

### 创建文件

```shell
touch file
#修改文件的时间属性，如果文件不存在则创建文件（只能创建空文件）
cat>>catcreatefile.txt
#cat创建catcreatefile.txt文件，并接受键盘输入作为文件的内容。Ctrl+d退出并保存文件。
cat catcreatefile.txt
#查看catcreatefile.txt的内容
```
<!--more-->

![catcreatefile](/blog/image/catcreatefile.png)

### 写入

```shell
echo "new word" >> catcreatefile.txt
#向catcreatefile.txt文件中追加写入内容“new word”，原内容不变
echo "update content">catcreatefile.txt
#覆盖catcreatefile.txt的内容
#使用echo命令写入内容，如果catcreatefile.txt文件不存在则创建
cat >> newfile.txt << eof
> new word
> eof
#使用cat命令 eof要顶格，成对使用，或者使用其他字符代替
#或者结合Ctrl+d命令
cat>> newfile.txt
usr other
#使用Ctrl+d结束输入
# >> 追加内容，原内容不变； > 覆盖原内容
```

![echowritefile](/blog/image/echowritefile.png)

![catwritefile](/blog/image/catwritefile.png)



### 查询

```shell
grep -ano '关键字' file
#查找关键字所在的行号
sed -n 2p file
#打印文件的第2行
sed -n 1,3p file 
#打印文件的第1行到第3行
tail -n 3 file 
#打印文件最后3行的数据
head -n 3 file
#打印前3行的内容
```

![查找内容](/blog/image/search.png)



### Linux中变量$$,$?等的含义

```
$# 是传给脚本的参数个数
$0 是脚本本身的名字
$1 是传递给该shell脚本的第一个参数
$2 是传递给该shell脚本的第二个参数
$@ 是传给脚本的所有参数的列表
$* 是以一个单字符串显示所有向脚本传递的参数，与位置变量不同，参数可超过9个
$$ 是脚本运行的当前进程ID号
$? 是显示最后命令的退出状态，0表示没有错误，其他表示有错误
```


