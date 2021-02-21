---
title: qtcreator-安卓环境搭建
date: 2021-02-21 11:44:43
tags: 安卓环境搭建
categories: qtcreator使用
---
本文仅在Ubuntu16.04下测试通过，其它版本没有验证。
 <!-- more -->

一、Java JDK
=====
文件：jdk-8u201-linux-x64.tar.gz

描述：jdk的是java development kit的缩写，是java语言的软件开发工具包，主要用于移动设备、嵌入式设备上的java应用程序，jdk是整个java开发的核心，它包含了java的运行环境，java工具和java基础的类库。

安装：

>sudo tar -zxvf jdk-8u201-linux-x64.tar.gz

>sudo chown make:make jdk1.8.0/

>sudo chmod -R 777 jdk1.8.0/

>sudo mv jdk1.8.0/ jdk

>cd /usr/local

>sudo vi /etc/profile（添加以下代码到文件尾端）

    export JAVA_HOME=/usr/local/jdk
    export CLASSPATH=$CLASSPATH:$JAVA_HOME/lib/tools.jar:$JAVA_HOME/lib/dt.jar
    export PATH=$PATH:$JAVA_HOME/bin:$JAVA_HOME/jre/bin

>source /etc/profile

二、Android SDK&NDK
=====
文件：android-sdk-r24.4.1 & android-ndk-r19c

描述：android sdk（Android Software Development Kit,即Android软件开发工具包）,可以说只要你使用java去开发Android这个东西就必须用到，它包含了SDK Manager 和 AVD Manage。而ndk（Native Development Kit）跟sdk差不多的是他也是一个开发工具包，用它开发c/c++是很方便的。为什么会有一个ndk？很早以前android是只有sdk的，并没有ndk。这就意味着一旦android的开发者要使用c/c++的三方库或者需要用到c/c++就必须使用非官方的法子。用java的jni去调用c/c++。耍小聪明走后门一样。而ndk的出现就意味着jni调用的这种方法转正了变成官方了以后你不需要再走后面大路正面随你走。可是这样还是没有说到为什么要有ndk啊。是的我只想说的就是如果你要操作底层直接操作内存。操作地址那你不得不去使用c/c++因为java这块想做这些。那恐怕有点困难。所以ndk是必须需要出现的。而这个sdk和ndk并不是完全不相溶的2门语言。对于android来说是同种语言的2种不同时期的必须品。

安装：下载后解压到自己指定的路径下，我的如下

    /usr/Android/android-sdk-linux
    /usr/Android/android-ndk-r19c
    
>sudo gedit /etc/profile（添加以下代码到文件尾端）    

	NDK_HOME=/usr/Android/android-ndk-r19c
	export NDK_HOME
	export PATH=$PATH:$NDK_HOME
	ANDROID_SDK_ROOT=/usr/Android/android-sdk-linux
	export ANDROID_SDK_ROOT
	export ANDROID_HOME=/usr/Android/android-sdk-linux
	export PATH=${PATH}:${ANDROID_HOME}/tools
	export PATH=${PATH}:${ANDROID_HOME}/platform-tools

三、qtcreator配置
=====
进入 **工具->选项->Android** 选项卡：

&emsp;&emsp;分别指定JDK、Android SDK和Android NDK的路径，如果报错不通过则进入sdk目录，使用**android**命令（SDK管理）更新tools和sdk即可。