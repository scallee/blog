---
title: "npm环境配置及使用"
date: 2019-08-05T00:48:03+08:00
lastmod: 2019-08-05T00:48:03+08:00
draft: false
keywords: ["npm"]
description: "npm安装及操作"
tags: ["npm"]
categories: ["npm"]
# author: ""
# comment: false
toc: true
contentCopyright: false
reward: false
mathjax: false
---

# npm环境配置及使用

## 安装node.js

### 下载windows版本的node.msi

安装react-native

打开cmd：

`npm install -g react-native-cli`

### 配置国内镜像

在D:\Program Files\nodejs\node_modules\npm\下的npmrc文件中配置
`registry=https://registry.npm.taobao.org`

`npm config set prefix "D:\Program Files\nodejs\node_global"`

`npm config set cache "D:\Program Files\nodejs\node_cache"`

在【系统变量】下新建【NODE_PATH】，输入【D:\Program Files\nodejs\node_global\node_modules】，
将【用户变量】下的【Path】修改为【D:\Program Files\nodejs\node_global】

```cmd
#查看基本配置 
npm config list 
#查看所有配置
npm config list -l
```

### npm本地安装与全局安装

`npm install grunt` // 本地安装，则是将模块下载到当前命令行所在目录。 

`npm install -g grunt` //全局安装，模块将被下载安装到【全局目录】中；

npm如何获取全局安装的默认目录？

`npm config get prefix`

npm如何设置全局安装的默认目录？

`npm config set prefix “directory”`

## 如何从Windows中删除Node.js：

1.从卸载程序卸载程序和功能。

2.重新启动（或者您可能会从任务管理器中杀死所有与节点相关的进程）。

3.寻找这些文件夹并删除它们（及其内容）（如果还有）。根据您安装的版本，UAC设置和CPU架构，这些可能或可能不存在：

> C:\Program Files (x86)\Nodejs
> C:\Program Files\Nodejs
> C:\Users\{User}\AppData\Roaming\npm（或%appdata%\npm）
> C:\Users\{User}\AppData\Roaming\npm-cache（或%appdata%\npm-cache）

4.检查您的%PATH%环境变量以确保没有引用Nodejs或npm存在。

5.如果仍然没有卸载，请where node在命令提示符下键入，您将看到它所在的位置 - 删除（也可能是父目录）。

6.重新启动

## 常用npm命令

```cmd
#查看npmjs服务器上包的最新版本信息
npm view <packageName> version
#查看npmjs服务器上包所有信息
npm info <packageName>
#查看本地包信息
npm ls <packageName>
#查看本地包信息 全局
npm ls <packageName> -g
#查看所有全局安装的模块
npm list -g
#查看某个模块的版本号
npm list <packageName>
#安装好后不写入package.json中
npm install <packageName>
#安装好后写入package.json的dependencies中（生产环境依赖）
npm install <packageName> --save
#安装好后写入package.json的devDepencies中（开发环境依赖）
npm install <packageName> --save-dev
#删除模块，但不删除模块留在package.json中的对应信息
npm uninstall <packageName>
#删除模块，同时删除模块留在package.json中dependencies下的对应信息
npm uninstall <packageName> --save(或者简写-S)
#删除模块，同时删除模块留在package.json中devDependencies下的对应信息
npm uninstall <packageName> --save-dev(或者简写-D)
#更新模块
npm update <packageName>
#搜索模块
npm search <packageName>
#查看包的依赖关系
npm view <packageName> dependencies
#查看当前包的安装路径
npm root
#查看全局的包的安装路径
npm root -g
#查看npm使用的所有文件夹
npm help folders
#清理缓存
npm cache clean
```

## npm i和npm install实际使用的区别点主要如下(windows下)： 

1. 用npm i安装的模块无法用npm uninstall删除，用npm uninstall i才卸载掉 
2. npm i会帮助检测与当前node版本最匹配的npm包版本号，并匹配出来相互依赖的npm包应该提升的版本号 
3. 部分npm包在当前node版本下无法使用，必须使用建议版本 
4. 安装报错时intall肯定会出现npm-debug.log 文件，npm i不一定

## NPM依赖包版本号~和^和*的区别

`~ `会匹配最近的小版本依赖包，比如~1.2.3会匹配所有1.2.x版本，但是不包括1.3.0

`^` 会匹配最新的大版本依赖包，比如^1.2.3会匹配所有1.x.x的包，包括1.3.0，但是不包括2.0.0

`*` 这意味着安装最新版本的依赖包

## npm-audit

### 语法：

```cmd
npm audit [--json|--parseable|--audit-level=(low|moderate|high|critical)]
npm audit fix [--force|--package-lock-only|--dry-run]
```

### 用法(谷歌翻译)：

扫描项目是否存在漏洞,并自动将任何兼容更新安装到易受攻击的依赖项:

`$ npm audit fix`

在不修改节点模块但仍在更新 pkglock 的情况下运行审核修复:

`$ npm audit fix --package-lock-only`

跳过更新开发依赖项:
只更新dependencies中安装的包，跳过devDependencies中的包

`$ npm audit fix --only=prod`

让审核修复程序将 semver 主要更新安装到顶级依赖项,而不仅仅是与 semver 兼容的更新:

`$ npm audit fix --force`

执行干运行以获取审核修复将做什么,以及输出 JSON 格式的安装信息:

`$ npm audit fix --dry-run --json`

扫描项目漏洞,只显示详细信息,而不修复任何内容:

`$ npm audit`

以 JSON 格式获取详细的审核报告:

`$ npm audit --json`

以纯文本结果获取详细的审核报告,由制表符分隔,以便将来在脚本或命令行后处理中重复使用,例如,选择打印的一些列:

`$ npm audit --parseable`

要分析列,可以使用例如 awk,只需打印其中一些:

`$ npm audit --parseable | awk -F $'\t' '{print $1,$4}'`

仅当结果包含级别为中等或更高的漏洞时,审核才会失败:

`$ npm audit --audit-level=moderate`

参考：https://docs.npmjs.com/cli/audit.html#synopsis

## Android SDK在线更新镜像服务器

中国科学院开源协会镜像站地址:

http://mirrors.opencas.cn 端口：80

http://mirrors.opencas.org 端口：80

http://mirrors.opencas.ac.cn 端口：80

上海GDG镜像服务器地址:

http://sdk.gdgshanghai.com 端口：8000

北京化工大学镜像服务器地址:

http://ubuntu.buct.edu.cn/ 端口：80

http://ubuntu.buct.cn/ 端口：80

http://ubuntu.buct6.edu.cn/ 端口：80

大连东软信息学院镜像服务器地址:

http://mirrors.neusoft.edu.cn 端口：80

腾讯Bugly 镜像:

http://android-mirror.bugly.qq.com 端口：8080