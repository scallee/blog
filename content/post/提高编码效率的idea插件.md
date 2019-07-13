---
title: "提高编码效率的idea插件"
date: 2019-07-13T14:50:02+08:00
lastmod: 2019-07-13T14:50:02+08:00
draft: false
keywords: ["idea"]
description: "idea插件"
tags: ["idea"]
categories: ["转载"]
# author: ""
# comment: false
toc: true
contentCopyright: false
reward: false
mathjax: false
---

#  提高编码效率的idea插件 

转载来源：[房上的猫](https://www.cnblogs.com/lsy131479/p/9646444.html)

因为idea自带的插件下载可能连接不上服务器而导致插件下载失败，所以这里推荐使用引入外部插件的方式

插件包也给你们准备好了（你们无需去下载啦），对照此篇博客配置即可：

<!--more-->

链接：<https://pan.baidu.com/s/1VKXJ74W3s6RZHL-3YHlaEA> 密码：z411

 ![img](/blog/image/idea-plugins/wangpan.png)

#  **首先说一下idea引入外部插件的方式：**

1.打开settings配置面板

![img](/blog/image/idea-plugins/setting.png)

2.选择plugins--》install pluginfrom disk  选择相应插件包==》点击ok

 ![img](/blog/image/idea-plugins/install.png)

最后，“**Restart**”重启idea 让它加载一下新加入的插件！！！

![img](/blog/image/idea-plugins/restart.png)

# 常用插件：

下面的所有插件都使用上面外部引用方式集成相应的插件包即可

# 1. **FindBugs-IDEA**

## **作用：**

检测代码中可能的bug及不规范的位置，

写完代码后检测下 避免低级bug，

静态分析工具承诺无需开发人员费劲就能找出代码中已有的缺陷。当然，如果有多年的编写经验，就会知道这些承诺并不是一定能兑现。

## **插件提供地址：**

<https://plugins.jetbrains.com/plugin/3847-findbugs-idea>

## **成功标志：**

重启idea后如果插件加载成功左下角出现：

![img](/blog/image/idea-plugins/findbugs-success.png)

## **实际应用：**

**![img](/blog/image/idea-plugins/findbugs-use.png)**

\----------------------

扫描完成后：

![img](/blog/image/idea-plugins/findbugs-result.png)

### **附录（网摘）：**

FindBugs 它用来查找Java代码中存在的bug。它使用静态分析方法标识出Java程序中上百种潜在的不同类型的错误。

FindBugs支持对包级别、项目级别、模块级别、单个文件级别，以及自定义范围的Bug分析。

FindBugs：特色是能够检查出空指针异常这种bug！

使用：右键单击（module、单个文件等）--->在快捷菜单中选择findbugs即可

![img](/blog/image/idea-plugins/findbugs-desc.png)

#### **附:FindBugs的Bug种类说明**

· Bad practice 坏的实践

一些不好的实践，下面列举几个： 
HE： 类定义了equals()，却没有hashCode()；或类定义了equals()，却使用Object.hashCode()；或类定义了hashCode()，却没有equals()；或类定义了hashCode()，却使用Object.equals()；类继承了equals()，却使用Object.hashCode()。 
SQL：Statement 的execute方法调用了非常量的字符串；或Prepared Statement是由一个非常量的字符串产生。 
DE： 方法终止或不处理异常，一般情况下，异常应该被处理或报告，或被方法抛出。 
Malicious code vulnerability 可能受到的恶意攻击

如果代码公开，可能受到恶意攻击的代码，下面列举几个： 
FI： 一个类的finalize()应该是protected，而不是public的。 
MS：属性是可变的数组；属性是可变的Hashtable；属性应该是package protected的。

类名小写问题：

　![img](/blog/image/idea-plugins/findbugs-bad.png)

· Correctness 一般的正确性问题

可能导致错误的代码，下面列举几个： 
NP： 空指针被引用；在方法的异常路径里，空指针被引用；方法没有检查参数是否null；null值产生并被引用；null值产生并在方法的异常路径被引用；传给方法一个声明为@NonNull的null参数；方法的返回值声明为@NonNull实际是null。 
Nm： 类定义了hashcode()方法，但实际上并未覆盖父类Object的hashCode()；类定义了tostring()方法，但实际上并未覆盖父类Object的toString()；很明显的方法和构造器混淆；方法名容易混淆。 
SQL：方法尝试访问一个Prepared Statement的0索引；方法尝试访问一个ResultSet的0索引。 
UwF：所有的write都把属性置成null，这样所有的读取都是null，这样这个属性是否有必要存在；或属性从没有被write。

· Dodgy 危险的

具有潜在危险的代码，可能运行期产生错误，下面列举几个： 
CI： 类声明为final但声明了protected的属性。 
DLS：对一个本地变量赋值，但却没有读取该本地变量；本地变量赋值成null，却没有读取该本地变量。 
ICAST： 整型数字相乘结果转化为长整型数字，应该将整型先转化为长整型数字再相乘。 
INT：没必要的整型数字比较，如X <= Integer.MAX_VALUE。 
NP： 对readline()的直接引用，而没有判断是否null；对方法调用的直接引用，而方法可能返回null。 
REC：直接捕获Exception，而实际上可能是RuntimeException。 
ST： 从实例方法里直接修改类变量，即static属性。

· Performance 性能问题

可能导致性能不佳的代码，下面列举几个： 
DM：方法调用了低效的Boolean的构造器，而应该用Boolean.valueOf(…)；用类似Integer.toString(1) 代替new Integer(1).toString()；方法调用了低效的float的构造器，应该用静态的valueOf方法。 
SIC：如果一个内部类想在更广泛的地方被引用，它应该声明为static。 
SS： 如果一个实例属性不被读取，考虑声明为static。 
UrF：如果一个属性从没有被read，考虑从类中去掉。 
UuF：如果一个属性从没有被使用，考虑从类中去掉。

· Multithreaded correctness 多线程的正确性多线程编程时，可能导致错误的代码，下面列举几个：

ESync：空的同步块，很难被正确使用。 
MWN：错误使用notify()，可能导致IllegalMonitorStateException异常；或错误的使用wait()。 
No： 使用notify()而不是notifyAll()，只是唤醒一个线程而不是所有等待的线程。 
SC： 构造器调用了Thread.start()，当该类被继承可能会导致错误。

· Internationalization 国际化 
当对字符串使用upper或lowercase方法，如果是国际的字符串，可能会不恰当的转换。

# 2. **Maven Helper**

## **作用：**

一键查看maven依赖，查看冲突的依赖，一键进行exclude依赖

对于大型项目 非常方便

## **插件提供地址：**

<https://plugins.jetbrains.com/plugin/7179-maven-helper>

## **成功标志：**

重启idea后如果插件加载成功 打开pom文件左下角出现：

![img](/blog/image/idea-plugins/maven-helper-success.png)

## **实际应用：**

切换Dependency Analyzer试图即可进行相应操作：

　　1) Conflicts（查看冲突）

　　2) All Dependencies as List（列表形式查看所有依赖）

　　3) All Dependencies as Tree（树形式查看所有依赖）

![img](/blog/image/idea-plugins/maven-helper-usr.png)

选中右键就可以 Exclude 啦，想用1.2就把其他版本的Exclude掉，想用1.1.3就把其他版本的Exclude掉，很方便。

![img](/blog/image/idea-plugins/maven-helper-result.png)

# 3. **VisualVM Launcher**

## **作用：**

运行java程序的时候启动visualvm，方便查看jvm的情况 比如堆内存大小的分配

某个对象占用了多大的内存，jvm调优必备工具

## **插件提供地址：**

<https://plugins.jetbrains.com/plugin/7115-visualvm-launcher>

## **成功标志：**

重启idea后如果插件加载成功右上角出现：

![img](/blog/image/idea-plugins/vvm-success.png)

## **实际应用：**

非常感谢有一个大佬，给我的评论，我研究了一下才知道实际应用，非常感谢哈，

![img](/blog/image/idea-plugins/vvm-import.png)

下面教大家一下实际应用：

首先需要一个客户端（这里已经为大家准备好）：

链接：<https://pan.baidu.com/s/1kIduEykNNrYyZd1CwJpntg> 密码：bxib

![img](/blog/image/idea-plugins/vvm-client.png)

将客户端解压

打开settings配置面板

配置excutable===》选择到客户端的bin目录的visualvm.exe文件

![img](/blog/image/idea-plugins/vvm-setting.png)

 然后点击ok。。apply应用一下

![img](/blog/image/idea-plugins/vvm-apply.png)

右键使用VM插件启动

![img](/blog/image/idea-plugins/vvm-run.png)

然后会自动加载程序：

![img](/blog/image/idea-plugins/vvm-start.png)

可查看jvm状态，然后就可以自行调试咯：

![img](/blog/image/idea-plugins/vvm-status.png)

具体怎么搞，就看你们自己想怎么搞咯，，本博主后期也会发布jvm调优的博客的，


# 4. **GenerateAllSetter**

## **作用：**

一键调用一个对象的所有set方法并且赋予默认值 在对象字段多的时候非常方便

## **插件提供地址：**

<https://plugins.jetbrains.com/plugin/9360-generateallsetter>

## **成功标志：**

**。。。。。。**


## **实际应用：**

通过alt +enter生成一个类所有setter方法的默认值

当两个对象转换器具有相同的字段时，为它们生成一个set方法

当returnType是List Set Map时生成默认值

![img](/blog/image/idea-plugins/setter.png)

# 5. **Rainbow Brackets**

## **作用：**

彩虹颜色的括号  看着很舒服 敲代码效率变高。。清除分清括号个数，防止括号错乱

## **插件提供地址：**

<https://plugins.jetbrains.com/plugin/10080-rainbow-brackets>

## **成功标志：**

重启idea后如果插件加载成功右下角出现弹出框

 ![img](/blog/image/idea-plugins/rainbow-brackets.png)

 

## **实际应用：**

最近括号内容高亮效果：ctrl+鼠标右键单击  

![img](/blog/image/idea-plugins/rainbow-brackets-apply.png)

 

最近括号内容外暗淡效果：alt+鼠标右键单击 

 ![img](/blog/image/idea-plugins/rainbow-brackets-result.png)

 

# 6. **Translation**

## **作用：**

最好用的翻译插件，功能很强大，界面很漂亮

 

**特征：**

 

- 多个翻译引擎。多种语言的互译。
  - 谷歌翻译。
  - 有道翻译。
  - 百度翻译。
- 
- 文字转语音。
- 自动选择字。
- 自动分词。

 

## **插件提供地址：**

<https://plugins.jetbrains.com/plugin/8579-translation>

## **成功标志：**

重启idea后如果插件加载成功右下角出现：

![img](/blog/image/idea-plugins/translation-success.png)

多翻译引擎 （可切换）

　　**·**Google翻译

　　**·** 有道翻译

　　**·** 百度翻译

## **实际应用：**

选择单词，右键：

![img](/blog/image/idea-plugins/translation-apply.png)

翻译加朗读：

![img](/blog/image/idea-plugins/translation-result.png)

# 7. **P3c**   

**Alibaba Java Coding Guidelines**

## **作用：**

阿里巴巴出品的java代码规范插件

可以扫描整个项目找到不规范的地方 并且大部分可以自动修复 

虽说检测功能没有findbugs强大，但是可以自动修复

阿里巴巴Java编码指南插件支持。

## **插件提供地址：**

<https://plugins.jetbrains.com/plugin/10046-alibaba-java-coding-guidelines>

 
## **成功标志：**

点击上方tools，出现阿里编码制约

![img](/blog/image/idea-plugins/p3c-success.png)

## **实际应用：**

右键，点击编码制约规范

![img](/blog/image/idea-plugins/p3c-apply.png)

下方出现：

　　扫描代码后，不符合规约的代码会按Blocker/Critical/Major三个等级显示在下方面板中

　　左边是扫描出的不符合规范的代码，依次点进去可以看到是代码的多少行出现了规约问题以及哪一个规约问题，右边则是规约的详细描述及实例代码。

![img](/blog/image/idea-plugins/p3c-result.png)

也可快捷键：

　　Alt+enter

![img](/blog/image/idea-plugins/p3c-cuts.png)

还可以让其自动修复代码：

![img](/blog/image/idea-plugins/p3c-modify.png)
