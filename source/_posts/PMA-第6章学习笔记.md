---
title: PMA 第6章学习笔记
date: 2021-02-10 14:00:04
tags: Practical Malware Analysis
---

# 汇编语言与C代码结构

 

## 全局变量和局部变量

全局变量：通过内存地址引用，类似于dword_10CF60地址来进行标记

局部变量：通过栈地址引用，类似于[ebp-4]或者[ebp+var_4]

## 函数调用约定

cdecl约定：参数从右到左的顺序入栈，需要调用者add esp清理栈

stdcall：自动清理栈。

fastcall：不涉及更多的栈操作

## 算术操作

++：add

--：sub

![](C:\Users\zengf\AppData\Roaming\Typora\typora-user-images\image-20210310140224428.png)

## If结构

每个if语句必定有一个条件跳转

 ![](C:\Users\zengf\AppData\Roaming\Typora\typora-user-images\image-20210310140245890.png)

嵌套if语句有多个跳转

 ![](C:\Users\zengf\AppData\Roaming\Typora\typora-user-images\image-20210310140539397.png)

## 循环

### for循环

1、 循环变量的初始化

2、 循环变量的递增

3、 循环变量的比较

4、 跳转回到循环开始位置

for循环内部有一处条件跳转，以便中止循环。

![](C:\Users\zengf\AppData\Roaming\Typora\typora-user-images\image-20210310140939035.png)

### While循环

循环结尾处有一个无条件跳转，循环内部有一个条件跳转，循环变量的递增可以不使用。

 ![](C:\Users\zengf\AppData\Roaming\Typora\typora-user-images\image-20210310141106667.png)

## Switch结构

1、 通过if样式

结构的开始处根据case情况进行跳转

 ![](C:\Users\zengf\AppData\Roaming\Typora\typora-user-images\image-20210310141259052.png) 

2、 通过跳转表样式

基于跳转表目标的位置，edx乘以4加上跳转表的基址进行跳转，其中跳转表中的每一项是4个字节大小的地址。

 ![](C:\Users\zengf\AppData\Roaming\Typora\typora-user-images\image-20210310141516435.png)

## 数组结构

1、 全局数组类似于全局变量，使用内存地址，即基址+偏移

2、 局部数组类似于局部变量，存储于栈中，再使用栈地址+偏移

 ![](C:\Users\zengf\AppData\Roaming\Typora\typora-user-images\image-20210310141729579.png)

 

## 结构体

全局变量结构体存储在内存，使用内存进行访问，通过IDA Pro结构体窗口可以自定义结构体，并赋给内存引用。

## 链表遍历

链表遍历使用指针结构，指针指向的是同一类型的对象，并且这些对象具有递归性，例如某个变量被赋予eax，eax来自于eax的自增，而eax的自增又来自于该变量，即形成了递归。

 ![](C:\Users\zengf\AppData\Roaming\Typora\typora-user-images\image-20210310141854178.png)

 