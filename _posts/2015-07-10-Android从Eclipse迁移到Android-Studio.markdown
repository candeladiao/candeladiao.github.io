---
layout: post
title: Android从Eclipse迁移到Android Studio
date: 2015-07-10
comments: true
archive: false
---
Android Studio安装及项目迁移笔记

[Android Studio](https://dl.google.com/dl/android/studio/install/1.2.2.0/android-studio-bundle-141.1980579-windows.exe)发布已有一段时间，准备亲自体验一下。


首先使用[SDK Manager](http://developer.android.com/tools/help/sdk-manager.html)下载最新的SDK tools和platforms，然后下载安装[Android Studio](http://developer.android.com/sdk/index.html)。


安装过程及所会遇到的问题[这里](http://www.2cto.com/kf/201503/379738.html)已经有详细说明，总的说来就是：


1.	更改BIOS，打开"Virtualization Technology"选项。
2.	下载SDK后，打开目录sdk/extras/intel/Hardware_Accelerated_Execution_Manager，安装intelhaxm-android.exe。如果没找到，需要打开SDK Manager找到Intel x86 Emulator Accelerator(HAXM install)下载。
3.	安装Android Studio。


Android Studio导入Eclipse工程有两种方式，具体可见[http://www.cnblogs.com/ct2011/p/4183553.html](http://www.cnblogs.com/ct2011/p/4183553.html)。


因为团队有同学仍然使用Eclipse，所以我以兼容模式先将Eclipse工程导出生成Gradle编译文件。导出后工程所在的目录会增加以下文件。（导出时只选择主工程，无需选择依赖工程）


<img src="/assets/images/2015-07-10/01.png" width="630" align=center/>


打开Android Studio，选择Import project(Eclipse ADT, Gradle, etc.)，选择build.gradle导入刚才导出的工程。


<img src="/assets/images/2015-07-10/02.png" width="630" align=center/>


却提示“Gradle version 1.10 is required. Current version is 2.2.1”，解决方法[这里](http://blog.csdn.net/lamelias/article/details/45057269)有说明，其实只需要做两步操作：


1.	找到工程目录下面的build.gradle，修改classpath为“com.android.tools.build:gradle:1.1.0” 
2.	找到\gradle\wrapper\gradle-wrapper.properties，修改distributionUrl=http\://services.gradle.org/distributions/gradle-2.2.1-all.zip


编译运行，模拟器却一直运行不起来。发现是因为模拟器CPU设置成了Intel Atom (x86)，但是没有下载Intel x86 Atom System Image。打开SDK Manager，Tools->Manage AVDs...->Android Virtual Devices，选择对应模拟器，点击右边Edit，可以看到模拟器CPU设置对对应版本的target，然后下载对应target的Intel x86 Atom System Image。


<img src="/assets/images/2015-07-10/03.jpg" width="630" align=center>


解决上面问题再次编译运行，又提示类重复的错误。


<img src="/assets/images/2015-07-10/04.png" width="630" align=center>


从ViewInjector这个关键词搜索到是因为在Eclipse中使用了ButterKnife导致的。metaandroid上的[解决方法](http://www.metaandroid.com/question/compilation-error-with-butterknife-error-duplicate-class/)是让删除工程下面的.apt_generated里面生成的类，但这样固然可以解决，但会影响以后使用Eclipse打开该工程。


通过摸索，我发现更好的解决方法应该是打开主工程的build.gradle文件，从配置里面删除.apt_generated目录。


再次运行，出现Multiple dex files define错误。


<img src="/assets/images/2015-07-10/05.png" width="630" align=center>


这是因为工程里面引入的第三方库里面包含了BuildConfig，和系统重复，所以需要找到出错提示路径下的BuildConfig文件，删除即可。


再次运行，终于出现模拟器，胜利就是眼前了。


等等，这是SM。


<div style="float:left">
<img src="/assets/images/2015-07-10/06.png" width="200" align=center>
</div>
<div style="float:left">
<img src="/assets/images/2015-07-10/07.jpg" width="430" align=center>
</div>


LoadLibrary出错，看来是.so文件没找到。再次打开主工程的build.gradle文件，在main里面增加jniLibs.srcDirs = ['libs']，因为我的.so文件是放在libs下面的。


<img src="/assets/images/2015-07-10/08.png" width="200" align=center>


关于Android Studio so文件项目依赖，可移步[这里](http://zhengxiaopeng.com/2014/12/13/Android-Studio-jar%E3%80%81so%E3%80%81library%E9%A1%B9%E7%9B%AE%E4%BE%9D%E8%B5%96/)。


至此，整个项目的Android Studio迁移就完成了。以上作个备忘。