---
title: 微信运动hook
date: 2020-05-18 20:37:00
tags: 其它
---

​		很久之前看到的[微信运动hook repo](https://github.com/zhengmin1989/WechatSportCheat)，在这次周末花了点时间试了一下。使用的手机型号是红米Note3全网通版本。

​		由于是第一次刷机，因此整个过程比较曲折。为避免繁杂的描述，所以只挑重点总结。

​		首先刷入TWRP，下载链接见[红米NOTE3全网通|Kenzo/Kate||TWRP](https://www.xiaomi.cn/post/4126589)，手机关机之后按【电源键+音量上】，出现如下图的画面后，打开下载好的TWRP文件，运行文件中的脚本，便可以自动刷入：

![](https://raw.githubusercontent.com/undoingfish/undoingfish.github.io/hexo/2020/05/18/%E5%BE%AE%E4%BF%A1%E8%BF%90%E5%8A%A8hook/1.png)

​		待手机重启后，会自动进入TWRP系统。关于TWRP系统的介绍，见维基百科词条[**Team Win Recovery Project**（**TWRP**）](https://zh.wikipedia.org/wiki/TWRP)。刷完TWRP后，开始获取系统的root权限。获取root权限可以使用TWRP内置的SuperSU或者Magisk。于是用Magisk-v20.0获取到了root权限。

​		获取了root权限后，安装xposed框架。尝试安装EdXposed，然而安装之后重启导致卡米……尝试MIUI修改版的xposed框架，安装之后等了大概四十多分钟，还是没起来，断定卡米，怀疑是xposed框架的安装存在一定的限制，例如框架版本号和系统版本号存在不兼容的情况。因此断定此路不通，放弃此法，寻找其他出路。

​		继续搜索后，发现还是有人遇到了一样的情况，并且确实成功刷入了xposed框架，地址如下：[MIUI9 红米note3全网通（KENZO）Xposed详细安装教程](https://www.xiaomi.cn/post/4943600)。然而当前的系统是MIUI10，如果按照教程的方法需要先刷成MIUI9，便在百度贴吧搜索MIUI9的相关信息：

![](https://raw.githubusercontent.com/undoingfish/undoingfish.github.io/hexo/2020/05/18/%E5%BE%AE%E4%BF%A1%E8%BF%90%E5%8A%A8hook/2.png)

图中标红的记录，是开始出现转机的时刻，点进去：

![](https://raw.githubusercontent.com/undoingfish/undoingfish.github.io/hexo/2020/05/18/%E5%BE%AE%E4%BF%A1%E8%BF%90%E5%8A%A8hook/3.png)		

继续找：

![](https://raw.githubusercontent.com/undoingfish/undoingfish.github.io/hexo/2020/05/18/%E5%BE%AE%E4%BF%A1%E8%BF%90%E5%8A%A8hook/4.png)

MIUI9？MIUIPRO？Android7.0？？对比一下我手机里面的版本，是Android6.0版本。

接着往下看：

![](https://raw.githubusercontent.com/undoingfish/undoingfish.github.io/hexo/2020/05/18/%E5%BE%AE%E4%BF%A1%E8%BF%90%E5%8A%A8hook/5.png)

再点进去，然后往下翻，找到某个百度云盘链接：

![](https://raw.githubusercontent.com/undoingfish/undoingfish.github.io/hexo/2020/05/18/%E5%BE%AE%E4%BF%A1%E8%BF%90%E5%8A%A8hook/6.png)

链接中的文件如下图所示。下载下来，拷贝到TWRP系统里面，安装好系统，重复开始的步骤，root、安装xposed框架。

![](https://raw.githubusercontent.com/undoingfish/undoingfish.github.io/hexo/2020/05/18/%E5%BE%AE%E4%BF%A1%E8%BF%90%E5%8A%A8hook/7.png)

成功。

![](https://raw.githubusercontent.com/undoingfish/undoingfish.github.io/hexo/2020/05/18/%E5%BE%AE%E4%BF%A1%E8%BF%90%E5%8A%A8hook/8.png)

装好微信hook框架，重启后开启小米运动，摇动手机，同步微信运动，搞定：

![](https://raw.githubusercontent.com/undoingfish/undoingfish.github.io/hexo/2020/05/18/%E5%BE%AE%E4%BF%A1%E8%BF%90%E5%8A%A8hook/9.png)

​		但是步数显得比较刻意，需要改一下。尝试使用repo中的源代码编译成apk……折腾好久没有成功，遂放弃。由于repo中提供了apk源文件，因此考虑走另一条路：反编译——>修改smali文件——>重新编译成apk。

​		源代码中的步数修改是直接在手机系统自带的计数器记录的步数基础上*1000，然后上传到微信运动。repo中的代码不多，很容易就找到了这里，代码路径是：

[sourcecode](https://github.com/zhengmin1989/WechatSportCheat/tree/master/sourcecode)/[xposedwechat](https://github.com/zhengmin1989/WechatSportCheat/tree/master/sourcecode/xposedwechat)/[src](https://github.com/zhengmin1989/WechatSportCheat/tree/master/sourcecode/xposedwechat/src)/[com](https://github.com/zhengmin1989/WechatSportCheat/tree/master/sourcecode/xposedwechat/src/com)/[example](https://github.com/zhengmin1989/WechatSportCheat/tree/master/sourcecode/xposedwechat/src/com/example)/[xposedwechat](https://github.com/zhengmin1989/WechatSportCheat/tree/master/sourcecode/xposedwechat/src/com/example/xposedwechat)/**wechat.java** 

![](https://raw.githubusercontent.com/undoingfish/undoingfish.github.io/hexo/2020/05/18/%E5%BE%AE%E4%BF%A1%E8%BF%90%E5%8A%A8hook/10.png)

​		将apk文件反编译得到如下文件夹：

![](https://raw.githubusercontent.com/undoingfish/undoingfish.github.io/hexo/2020/05/18/%E5%BE%AE%E4%BF%A1%E8%BF%90%E5%8A%A8hook/11.png)

​		进入反编译后的smali文件夹：

![](https://raw.githubusercontent.com/undoingfish/undoingfish.github.io/hexo/2020/05/18/%E5%BE%AE%E4%BF%A1%E8%BF%90%E5%8A%A8hook/12.png)

​		smali文件生成貌似有一些规律，先打开两个含wechat的文件寻找。由于只是想把1000改成一个比较自然的数，因此找一下1000的十六进制0x3e8就可以了。在wechat$1.smali文件里搜到“3e8”：

![](https://raw.githubusercontent.com/undoingfish/undoingfish.github.io/hexo/2020/05/18/%E5%BE%AE%E4%BF%A1%E8%BF%90%E5%8A%A8hook/13.png)

​		将其改成0x46e，重新编译，传到手机，发现安装不上，原来是没有签名，便打上签名，安装成功。再摇手机同步微信运动：

![](https://raw.githubusercontent.com/undoingfish/undoingfish.github.io/hexo/2020/05/18/%E5%BE%AE%E4%BF%A1%E8%BF%90%E5%8A%A8hook/14.png)

编译过程：

```
apktool b files
keytool -genkey -keystore coolapk.keystore -keyalg RSA -validity 10000 -alias coolapk
#信息随便填一下就可以了，然后将apk拷贝到和签名文件同一目录
jarsigner -digestalg SHA1 -sigalg MD5withRSA -tsa -verbose -keystore coolapk.keystore -signedjar coolapk-signed.apk coolapk.apk coolapk
```

本次试验到此结束，谢谢观看。

