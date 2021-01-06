---
title: 微信运动hook
date: 2020-05-18 20:37:00
tags: 其它
---

​		很久之前，看到大佬的[微信运动hook repo](https://github.com/zhengmin1989/WechatSportCheat)，但是一直没有时间和精力去尝试，于是上个周末花了点时间试了一下。

​		手机型号：红米Note3全网通版本

​		由于是第一次刷机，因此整个过程踩了N+的坑，但是为了避免繁杂的描述，因此挑重点描述一下，当作整个过程的小结。

​		首先刷入TWRP，下载链接见[红米NOTE3全网通|Kenzo/Kate||TWRP](https://www.xiaomi.cn/post/4126589)，手机关机之后按【电源键+音量上】，出现如下图的画面后，打开下载好的TWRP文件，运行文件中的脚本，便可以自动刷入：

![](https://raw.githubusercontent.com/undoingfish/undoingfish.github.io/hexo/pic/image-20200518205459056.png)

​		待手机重启后，会自动进入TWRP系统。关于这个系统的介绍，见维基百科词条[**Team Win Recovery Project**（**TWRP**）](https://zh.wikipedia.org/wiki/TWRP)。刷完TWRP后，便是获取系统的root权限。获取系统的root权限，可以使用TWRP内置的SuperSU或者Magisk。我选择了后者的Magisk-v20.0版本，于是迈向了深坑……坑里面有啥，请接着往下看……

​		获取了root权限后，就是安装xposed框架啦。在这里，坑开始出现了，由于对安卓系统不熟悉，什么MIUI版本啊、安卓系统版本啊，搞得稀里糊涂的，一开始看到MIUI10，还以为是Android10……趟入深坑，以为要安装EdXposed，于是安装之后重启导致卡米……后来好不容易知道了两者的不同之处，又根据搜索结果下载了MIUI修改版的xposed框架，安装之后，心想这下不会卡米了吧……然而等了大概四十多分钟，还是没起来，于是终于可以断定又卡米了……

​		正当本人一筹莫展之际，百无聊赖地搜索，发现好多人遇到了和我一样的情况，但是确实是有人成功刷入了xposed框架的，地址如下：[MIUI9 红米note3全网通（KENZO）Xposed详细安装教程](https://www.xiaomi.cn/post/4943600)，咦？我的是MIUI10，难道MIUI9可以？再结合原帖的步骤和本人自己的步骤，也没错啊，那问题出在哪里了呢？那要不要用MIUI9来试一下？得，死马当做活马医了，但是，MIUI9在哪里可以下载到呢？于是我想到了百度贴吧……搜索历史如下：

![](https://raw.githubusercontent.com/undoingfish/undoingfish.github.io/hexo/pic/image-20200518213242590.png)

图中标红的记录，是开始出现转机的时刻，点进去：

![](https://raw.githubusercontent.com/undoingfish/undoingfish.github.io/hexo/pic/image-20200518213438245.png)		

再点进去：

![](https://raw.githubusercontent.com/undoingfish/undoingfish.github.io/hexo/pic/image-20200518213542109.png)

MIUI9？MIUIPRO？Android7.0？？等等，我手机里面的版本是啥？Android6.0？？

接着往下看：

![](https://raw.githubusercontent.com/undoingfish/undoingfish.github.io/hexo/pic/image-20200518213738363.png)

再点进去，然后往下翻（链接/s/15m9uIY1ORBURK4NpvNOGbQ码 5qe9）：

![](https://raw.githubusercontent.com/undoingfish/undoingfish.github.io/hexo/pic/image-20200518213840706.png)

nice，点进去一看，瞬间惊呆了……真是踏破铁鞋无觅处啊。下载下来，拷贝到TWRP系统里面，怀着忐忑的心情安装好系统，再结合之前的步骤，root、安装xposed框架。

![](https://raw.githubusercontent.com/undoingfish/undoingfish.github.io/hexo/pic/image-20200518214226165.png)

成功了。

![](https://raw.githubusercontent.com/undoingfish/undoingfish.github.io/hexo/pic/image-20200518215445271.png)

于是装好微信hook框架，重启后开启小米运动，摇几下，再同步微信运动，搞定：

![](https://raw.githubusercontent.com/undoingfish/undoingfish.github.io/hexo/pic/image-20200518215829915.png)

​		但是美中不足，这样的步数一看就知道，太假了，得改一下。由于不会安卓开发，因此大佬即使给了源代码也不会编译成apk……实在是太菜了，折腾了好久，终于放弃了。只好走另一条路：反编译——>修改——>再重新编译。

​		源代码中的步数修改是直接在手机系统自带的计数器记录的步数基础上*1000，然后上传到微信运动（但是为什么我会知道这个，应该是之前搜到的文章里面有提到过，再加上代码不多，因此很容易就找到了这里，代码路径是[sourcecode](https://github.com/zhengmin1989/WechatSportCheat/tree/master/sourcecode)/[xposedwechat](https://github.com/zhengmin1989/WechatSportCheat/tree/master/sourcecode/xposedwechat)/[src](https://github.com/zhengmin1989/WechatSportCheat/tree/master/sourcecode/xposedwechat/src)/[com](https://github.com/zhengmin1989/WechatSportCheat/tree/master/sourcecode/xposedwechat/src/com)/[example](https://github.com/zhengmin1989/WechatSportCheat/tree/master/sourcecode/xposedwechat/src/com/example)/[xposedwechat](https://github.com/zhengmin1989/WechatSportCheat/tree/master/sourcecode/xposedwechat/src/com/example/xposedwechat)/**wechat.java** ）：

![](https://raw.githubusercontent.com/undoingfish/undoingfish.github.io/hexo/pic/image-20200518220357451.png)

​		于是将apk文件反编译一下，得到如下文件夹：

![](https://raw.githubusercontent.com/undoingfish/undoingfish.github.io/hexo/pic/image-20200518220647035.png)

​		再从网上搜一下，了解到反编译得到的代码是在smali文件夹中（smali文件就相当于x86系统下的汇编）。那就好办了，凭着感觉，找到这个路径（说是感觉，其实是凭着路径文件的命名来找的，有一些相似之处）：

![](https://raw.githubusercontent.com/undoingfish/undoingfish.github.io/hexo/pic/image-20200518220944635.png)

​		看来smali文件生成还是有规律的，那肯定是先打开两个含wechat.smali的文件看一看咯。由于只是想把1000改成一个比较自然的数，因此找一下1000的十六进制0x3e8就可以了。打开两个文件，在下面的文件里没搜到“3e8”，又在上面的文件里搜了一下，哎？还真就有：

![](https://raw.githubusercontent.com/undoingfish/undoingfish.github.io/hexo/pic/image-20200518221510257.png)

​		从上下文的关键字来看，就是它了，改成0x46e，重新编译，传到手机，发现安装不上，发现是没有签名，于是又打上签名，安装成功了！再摇几下手机：

![](https://raw.githubusercontent.com/undoingfish/undoingfish.github.io/hexo/pic/image-20200518221917122.png)

编译过程：

```
apktool b files
keytool -genkey -keystore coolapk.keystore -keyalg RSA -validity 10000 -alias coolapk
#信息随便填一下就可以了，然后将apk拷贝到和签名文件同一目录
jarsigner -digestalg SHA1 -sigalg MD5withRSA -tsa -verbose -keystore coolapk.keystore -signedjar coolapk-signed.apk coolapk.apk coolapk
```

本次试验到此结束，谢谢观看。

