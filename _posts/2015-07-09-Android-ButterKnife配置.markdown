---
layout: post
title: Android ButterKnife配置
date: 2015-07-09
comments: true
archive: false
---
Android笔记

最近开始学习Android开发，下载我们Android产品最新版本源码运行时却报了java.lang.NullPointerException空指针异常，调试发现类型为ListView的成员变量确实为空，而且也没找到通过findViewById给该成员变量赋值的代码。


<img src="/assets/images/2015-07-09/01.png" width="630" align=center/>


但Android版本已经在线上发布，不应该有这样问题。


查看成员变量声明，发现@InjectView注解，才搞明白这里是使用了[ButterKnife View注入框架](http://stormzhang.com/openandroid/android/2014/01/12/android-butterknife/)来简化了代码。


<img src="/assets/images/2015-07-09/02.png" width="630" align=center/>


所以需要在开发环境中配置一下ButterKnife，参考[ButterKnife Eclipse Configuration](http://jakewharton.github.io/butterknife/ide-eclipse.html)需要做以下工作：


1.	在**Package Explorer**里右击工程，选择**Properties**->**Java Compiler**->**Annotation Processing**，勾上"**Enable project specific settings**"。


2.	展开**Annotation Processing**选择**Factory Path**，勾选"**Enable project specific settings**"，然后点击"**Add JARs...**"，选择工程下面**libs**文件夹里的[Butter Knife JAR](http://repository.sonatype.org/service/local/artifact/maven/redirect?r=central-proxy&g=com.jakewharton&a=butterknife&v=LATEST)文件。（工程里没有butterknife.jar的自行下载后拷贝到**libs**文件夹）。


3.	点击"**OK**"保存设置，Eclipse会问是否重新编译工程，选择"**YES**"。


4.	确保工程根目录已经有**.apt_generated**文件夹，里面包含了类似YOURACTIVITY$$ViewBinder.java的文件，如果没有，执行**Project**->**Clean**操作。这个文件夹及里面文件都不应该加入版本控制。


5.	最后需要确认的是“**Java Compiler**”选项里的**Compiler compliance level**设置成了Java版本**1.6**及以上。


正当回到我的Eclipse里面按上面步骤操作时，发现NM，**Java Compiler**居然没有**Annotation Processing**。


<img src="/assets/images/2015-07-09/03.png" width="630" align=center/>


Google一下，原来是我的Ecplise是从Android官网上下载的，因为ADT Bundle的Eclipse专注于Android方面的开发，所以Java开发的某些插件默认不安装，只要安装**Eclipse Java Development Tools**即可。


详细解决方法见[这里](http://blog.csdn.net/lpforever/article/details/40779341)


安装完成后，做一下上面5步操作，ButterKnife就完成配置了。


如果使用的是Android Studio，详细配置见[https://github.com/avast/android-butterknife-zelezny](https://github.com/avast/android-butterknife-zelezny)
