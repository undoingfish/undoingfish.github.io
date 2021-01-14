---
title: PMA 第3章学习笔记
date: 2021-01-14 14:30:31
tags: Practical Malware Analysis
---

#### **动态分析基础技术**

1、使用沙箱运行恶意代码。沙箱能够提供容易理解的输出结果，对初始诊断非常有用。但是沙箱只能简单的运行可执行程序，没有命令行选项。并且沙箱不能记录所有事件，且由于不同的恶意代码有不同的特征，沙箱有很多局限性。

2、使用进程监视器Process Monitor来监控注册表、文件系统、网络、进程和线程行为。

3、使用进程浏览器Process Explorer来查看当前系统运行的所有进程状态，例如被进程载入到内存的DLL文件、活跃的网络连接、可执行程序的路径、进程监听的端口等等。进程浏览器还可以通过镜像标签中的验证功能来对磁盘上的镜像文件进行微软的签名认证，但是对使用进程替换技术的程序失效。

4、可以比较进程属性窗口的字符串来区分进程是否被替换。

5、使用依赖遍历器Dependency Walker来判断某个DLL是否被加载到进程。

6、使用Regshot来进行注册表在程序执行前后的变化比较。

7、使用ApateDNS来伪造DNS响应，从而分析出恶意代码的DNS请求信息。

8、使用Netcat监听某个指定端口，从而对恶意代码的流量数据进行监视。

9、使用Wireshark进行数据包监听。

10、使用INetSim伪装成某台服务器并模拟各种服务，从而分析恶意代码的网络行为。

####  关于DLL文件

- ##### 查询DLL文件是32位还是64位

**方法一**：

1、打开VS中的开发者命令提示符工具（Developer Command Prompt for VS**)

![](C:\Users\zengf\AppData\Roaming\Typora\typora-user-images\image-20210114145641006.png)

2、运行命令：dumpbin /headers "***.dll"

为避免绝对路径中的空格影响，可将绝对路径带上双引号。如下图所示，该dll为32位dll。

![image-20210114152940662](C:\Users\zengf\AppData\Roaming\Typora\typora-user-images\image-20210114152940662.png)

**方法二**

使用记事本打开对应文件，搜索“PE”关键字，出现“PE L”则表示该dll为32位dll，若出现PE d则是64位。

![image-20210114150550994](C:\Users\zengf\AppData\Roaming\Typora\typora-user-images\image-20210114150550994.png)

- ##### 关于32位文件和64位文件运行的问题：

32位的系统不能运行64位程序，但是64位的系统可以运行32位的程序。

- ##### 加载dll文件

1、使用PEView或者PE Explorer工具来查看导出函数列表。

2、执行如下命令（ Export arguments是导出函数列表中的一个）：

```
C:\>rundll32.exe DLLname, Export arguments
```

此处的rundll32.exe是64位程序：

![image-20210114152821278](C:\Users\zengf\AppData\Roaming\Typora\typora-user-images\image-20210114152821278.png)