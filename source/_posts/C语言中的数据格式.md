---
title: C语言中的数据格式
date: 2020-10-10 16:22:37
tags: Programming Language
---

## struct

#### 设计初衷

既能够包含字符串又能够包含数字的数据形式，其各信息独立。

#### 基本用法

- **结构体的定义**

```c
struct book {
    char title[MAXTITLE];
    char author[MAXAUTL];
    float value;
};
```

- **创建结构变量**

```c
struct book library;
```

- **定义的同时创建变量**

```c
struct book {
    char title[MAXTITLE];
    char author[MAXAUTL];
    float value; 
} library;
```

- **初始化结构变量**

```c
struct book library = {
    "The Pious Pirate and the Devious Damsel",
    "Renee Vivotte",
    1.95
};
```

结构体相当于一个超级数组，在这个超级数组中，可以是一个元素为char类型，下一个元素为float类型，下一个元素为int类型。

- **访问结构体成员**

```c
library.title;
```

- **结构的初始化构造**

```c
struct book gift = {
	.value = 25.99,
    .author = "James Broadfool",
    .title = "Rue for the Toad"
};
```

- **结构数组的声明**

```c
struct book library[MAXBKS]; 	/* book类型结构的数组 */
```

- **结构数组的访问**

![img](file:///C:/Users/zengf/AppData/Local/Temp/msohtmlclip1/01/clip_image008.png)

```c
library[0].value;				/*访问第一个数组元素的value成员*/
library[4].title;				/*访问第五个数组元素的title成员*/
```
#### 指向结构的指针

- **声明**

```c
struct guy * him;
```

- **指向**

```c
him = &fellow[0];
```

- **访问成员**

```c
him == &fellow[0];
him->income;
fellow[0].income;		/* 此句和2是等价的*/
```

#### 向函数传递结构信息

 

- **传递结构成员**

```c
sum(stan.bankfund, stan.savefund);
```

- **传递结构地址使结构的值可以修改**

```c
modify(&stan.bankfund);
```

- **传递整个结构（的副本）**

```c
sum(stan);
```

- **其他结构特性c**

把一个结构复制给另一个结构：

```c
struct names rignt_filed = {"Ruthie", "George"};
struct names captain = right_field;			//把一个结构初始化为另一个结构
```

 

## union

#### 设计初衷

用于存储既无规律、事先也不知道顺序的混合类型。Union类型只能存储一个值。

####  基本用法

- **定义**

```c
union hold {
    int digit;
    double bigfl;
    char letter;
}
```

- **声明**

```c
union hold fit;                           // hold类型的联合变量
union hold save[10];                      // 内含10个联合变量的数组
union hold * pu;                          // 指向hold类型联合变量的指针
union hold valA;
valA.letter = 'R';
union hold valB = valA;                   // 用另一个联合来初始化
union hold valC = {88};                   //初始化联合的digit成员
union hold valD = {.bigfl = 118.2};       //指定初始化器
```

- **使用**

```c
fit.digit = 23;               // 把23存储在fit，占2字节
fit.bigfl = 2.0;              // 清除23，存储2.0，占8字节
fit.letter = 'h';             // 清除2.0，存储h，占1字节
```

## enum

#### 基本用法

- **定义**

```c
enum spectrum {red, orange, yellow, green, blue, violet};
```

- **声明**

```c
enum spectrum color;
```

默认情况下，枚举列表中的常量都被赋予0、1、2等，例如下面的代码中nippy=0，slats=1，skippy=2：

```c
enum kids {nippy, slats, skippy, nina, liz};
```

- **为枚举常量指定整数值**

```c
enum levels {low = 100, medium = 500, high = 2000};
enum feline {cat, lynx = 10, puma, tiger};         // cat=0, lynx=10, puma=11, tiger=12
```

## Typedef

#### 设计初衷

为某一类型自定义名称，其与define不同，typedef创建的符号名只受限于类型，不能用于值。

####  基本用法

- **定义**

```c
typedef unsigned char BYTE;
typedef struct {double x; double y;} rect;
```

- **声明**

```c
rect r1 = {3.0, 6.0};
rect r2;
```

其等价于

```c
struct {double x; double y;} r1 = {3.0, 6.0};
struct {double x; double y;} r2;
r2 = r1;
```

-  **复杂类型命名**

将FRPTC声明为一个函数类型，该函数返回一个指针，指针指向内含5个char类型元素的数组。

```c
typedef char (* FRPTC ()) [5];
```

-  **其他复杂的声明**

```c
int board[8][8];        //声明一个内含int数组的数组
int ** prt;             //声明一个指向指针的指针，被指向的指针指向int
int * risks[10];        //声明一个内含10个元素的指针数组，每个元素都是一个指向int的指针
int (* rusks)[10];      //声明一个指向数组的指针，该数组内含10个int类型的值
int * oof[3][4];        //声明一个3 x 4的二维数组，每个元素都是指向int类型的值
int (* uuf)[3][4];      //声明一个指向3 x 4二维数组的指针，该数组中内含int类型值
int (* uof[3])[4];      //声明一个内含3个指针元素的数组，其中每个指针都指向一个内含4个int类型元素的数组
```

- **关于*、()和[ ]的优先级**

1、数组名后面的 [ ] 和函数名后面的 ( ) 具有相同的优先级。它们比 * (解引用运算符)的优先级高，因此下面声明的risk是一个指针数组，不是指向数组的指针。

```c
int * risks[10];
```

2、[ ] 和 ( ) 的优先级相同，由于都是从左往右结合，所以下面的声明中，在应用方括号之前，* 先与rusks结合，因此rusks是一个指向数组的指针，该数组内含10个int类型的元素。

```c
int (* rusks)[10];
```

3、[ ] 和 ( ) 都是从左往右结合，因此下面声明的goods是一个由12个内含50个int类型值的数组组成的二维数组，不是一个由50个内含12个int类型的数组组成的二维数组。

```c
int goods[12][50];
```

把以上规则应用于下面的声明：

```c
int * oof[3][4];
```

由于[ ]比\*的优先级高，且从左往右结合，所以[3]先与oof结合，因此oof首先是一个内含3个元素的数组。然后再与[4]结合，所以oof的每个元素都是内含4个元素的数组。\* 说明这些元素都是指针。最后，int表明了这4个元素都是指向int的指针。因此，这条声明要表达的是：foo是一个内含3个元素的数组，其中每个元素是4个指向int的指针组成的数组，一共12个指针。

