---
title: "Centos7安装jdk脚本"
date: 2019-06-16T21:16:32+08:00
lastmod: 2019-06-16T21:16:32+08:00
draft: true
keywords: ["java","jdk"]
description: "一键安装jdk"
tags: ["java","jdk","sh"]
categories: ["Java"]
# author: ""
comment: false
# toc: false
contentCopyright: false
reward: false
mathjax: false
---

Centos7 安装JDK 版本jdk1.8.0_161

什么都不说，直接上代码
<!--more-->
```sh
#!/bin/bash

## 软件的安装包的地址(请根据实际版本及文件安装包名称进行修改)
JAVA_TAR=/usr/local/java/jdk-8u161-linux-x64.tar.gz
JAVA_INSTALL_PATH=/usr/local/java/

checkjdk(){
if [ ! -f "${JAVA_TAR}"  ]; then
	echo －－提示：Java安装文件或openfire安装文件不存在,请将它们拷贝过来再运行!
	exit 1;
fi

echo －－提示：－－－－－－－正在安装配置java环境变量，请稍等！－－－－－－－
sleep 1
echo －－提示：－－－－－－－第一步：检查本机是否安装了java！－－－－－－－

java_version=java -version 2>&1|grep java|awk -F '"' '{print $2}'

echo --------------"$java_version"--------  
if [ ! -n $? ]
then
    echo －－－－－－－本机已安装了java环境变量无须在安装，可以直接安装其他应用 －－－－－
	java -version
else  
    echo －－－－－－－本机未安装java,开始安装－－－－－－－
	installjdk
fi
}
installjdk(){
echo －－－－－－－创建java安装路径－－－－－－－
if [ -d "$JAVA_INSTALL_PATH" ]
then
	echo －－－－－－－已经存在java安装路径－－－－－－－
else  
   mkdir $JAVA_INSTALL_PATH
	echo －－－－－－－java安装路径创建完毕－－－－－－－ 
fi
echo －－提示：－－－－－－－开始安装java！－－－－－－－
tar -zxvf $JAVA_TAR -C $JAVA_INSTALL_PATH
echo "解压成功"
 
echo "开始修改/etc/profile中配置文件，增加新的内容"
cat >> /etc/profile << EOF
JAVA_HOME=/usr/local/java/jdk1.8.0_161
JRE_HOME=\$JAVA_HOME/
PATH=\$PATH:\$JAVA_HOME/bin:\$JRE_HOME/bin
CLASSPATH=.:\$JAVA_HOME/lib/dt.jar:\$JAVA_HOME/lib/tools.jar:\$JRE_HOME/lib
export JAVA_HOME JRE_HOME PATH CLASSPATH
EOF
echo "执行cat标准输出成功,开始更新/etc/profile文件。"
source /etc/profile
echo "jdk11 安装成功"
echo "开始测试jdk是否安装看成。看下面输出"
java -version
}

checkjdk

```