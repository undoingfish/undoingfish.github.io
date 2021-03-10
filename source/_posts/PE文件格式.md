---
title: PE文件格式
date: 2021-02-16 14:51:23
tags: Reverse Engineering
---

## 基本概念

PE文件是Windows操作系统下使用的可执行文件格式，全称为Portable Executable File Format。

## PE文件种类

![](https://raw.githubusercontent.com/undoingfish/undoingfish.github.io/hexo/pic/2021/02/16/1.png)

## 几个重要的地址

基地址（ImageBase）：PE文件加载到内存的起始地址。

相对虚拟地址（RVA）：PE文件内部的地址，即未加载到内存时，某个位置相对开始位置的地址。

虚拟地址（VA）：PE文件加载到内存后各部分在内存中的实际地址。

虚拟地址=基地址+相对虚拟地址

## PE文件结构

PE头存储了文件运行需要的所有信息，例如内存加载方式、运行起点、运行所需库、运行所需堆栈大小等。这些信息以结构体形式存储，因此学习PE文件格式就是学习PE头中的结构体。

PE文件=PE头+PE体，不同的书本对此的叙述不同，例如，在《逆向工程核心原理》“第13章 PE文件格式”中，DOS头到节区头是属于PE头，剩下的是PE体；在《加密与解密（第四版）》“第11章 PE文件格式”中，DOS头不包含在PE头中。在《恶意代码分析实战》第一章节中，则未提及到DOS头的存在，另外本文所使用的文件为《恶意代码分析实战》书中的Lab-01-1.dll文件，关于PE头文件的结构体，比较重要的字段在本文中会被加粗。

![](https://raw.githubusercontent.com/undoingfish/undoingfish.github.io/hexo/pic/2021/02/16/2.png)

### DOS头

##### MAGE_DOS_HEADER

DOS头中存储的是一个名为IMAGE_DOS_HEADER的结构体，其大小为40字节，即18个word型变量和一个long型变量，其中long型变量为32位，即4个字节，word变量16位，即2个字节，18个word共36个字节。

![](https://raw.githubusercontent.com/undoingfish/undoingfish.github.io/hexo/pic/2021/02/16/3.png)

比较重要的两个字段是**e_maigic**和**e_lfanew**，前者是指DOS签名，其值为4D5A，对应的ASCII值为“MZ”，取自其设计者Mark Zbikowski名字的首字母。e_lfanew则指示NT头的偏移，使用小端存储，因此下图中的值为0x000000E0h，以及表示在0x000000E0h处开始，是NT头的开始。

![](https://raw.githubusercontent.com/undoingfish/undoingfish.github.io/hexo/pic/2021/02/16/4.png)

文件在IDA pro以PE格式加载的hex view中，不显示此字段：

![](https://raw.githubusercontent.com/undoingfish/undoingfish.github.io/hexo/pic/2021/02/16/5.png)

### DOS stub

stub意为“存根”，DOS头之后便是DOS stub，其为一个可选项，大小不固定的区域，通常由编译器或者汇编器生成，在上图的例子中此字段为00000040行至000000D0行。

![](https://raw.githubusercontent.com/undoingfish/undoingfish.github.io/hexo/pic/2021/02/16/6.png)

### NT头

##### **IMAGE_NT_HEADERS**

接下来的分节是NT头，NT头包括IMAGE_NT_HEADERS，其大小为。

![](https://raw.githubusercontent.com/undoingfish/undoingfish.github.io/hexo/pic/2021/02/16/7.png)

**Signature**：字段的值为0x50450000h，对应的是“PE00“字符；

##### **IMAGE_FILE_HEADER**

Signature字段后面是IMAGE_FILE_HEADER结构体， IMAGE_FILE_HEADER内容如下所示，其大小为4\*3+4*2=20个字节。

![](https://raw.githubusercontent.com/undoingfish/undoingfish.github.io/hexo/pic/2021/02/16/8.png)

对应文件中的各个字段：

![](https://raw.githubusercontent.com/undoingfish/undoingfish.github.io/hexo/pic/2021/02/16/9.png)

1、**Machine**：表示了每个CPU的Machine码，例如兼容Intel x86芯片的Machine码为014Ch，各类CPU的Machine码可参考winnt.h文件的定义，常见标识如下

![](https://raw.githubusercontent.com/undoingfish/undoingfish.github.io/hexo/pic/2021/02/16/10.png)

2、**NumberOfSections**：表示的是节区数，当定义的节区数与实际节区数不同是，PE文件将发生运行错误。

3、TimeDateStamp：表示文件创建的时间，其值等于自1970年1月1日以来用格林威治时间计算的秒数。

4、PointerToSymbolTable：表示COFF符号表的文件偏移位置，若COFF符号表不存在，则此值为0。

5、NumberOfSymbols：当COFF符号表存在时，表示其中的符号数目。

6、**SizeOfOptionalHeader**：***表明IMAGE_OPTIONAL_HEADER结构体的大小***。在IMAGE_NT_HEADERS中，IMAGE_OPTIONAL_HEADER紧跟IMAGE_FILE_HEADER。若当前文件是32位，则该值为00E0h，若当前文件是64位，则该值为00F0h。

7、**Characteristics**：用于标识文件属性，例如文件是否为可运行的形态、是否为DLL……，如下表所示：

![](https://raw.githubusercontent.com/undoingfish/undoingfish.github.io/hexo/pic/2021/02/16/11.png)

##### **IMAGE_OPTIONAL_HEADER**

IMAGE_OPTIONAL_HEADER结构体在IMAGE_NT_HEADERS结构体后面。此结构体是可选的，在此结构体中可以定义更多的PE文件属性，因此将IMAGE_OPTIONAL_HEADER结构与IMAGE_FILE_HEADER结构结合便是完整的PE文件头结构。IMAGE_OPTIONAL_HEADER结构是PE头结构体中最大的。

![](https://raw.githubusercontent.com/undoingfish/undoingfish.github.io/hexo/pic/2021/02/16/12.png)

实例中对应的字段如下：

![](https://raw.githubusercontent.com/undoingfish/undoingfish.github.io/hexo/pic/2021/02/16/13.png)

1、**magic**：0107h时表示文件是ROM映像，010Bh时表示文件是32位普通可执行映像，020Bh时表示文件是64位可执行映像。

2、MajorLinkerVersion：链接程序的主版本号。

3、MinorLinkerVersion：链接程序的次版本号。

4、SizeOfCode：有IMAGE_SCN_CNT_CODE属性的区块的总大小，向上取整。

5、SizeOfInitializedData：已初始化数据块的大小，即在编译时所构成的块的大小，此数据不太准确。

6、SizeOfUninitializedData：未初始化数据块的大小。

7、**AddressOfEntryPoint**：程序执行入口的相对虚拟地址（RVA），指定了程序执行的初始地址，此值非常重要，在使用OllyDBG调试时最开始定位到此位置。

![](https://raw.githubusercontent.com/undoingfish/undoingfish.github.io/hexo/pic/2021/02/16/14.png)

8、BaseOfCode：代码段的起始相对虚拟地址（RVA），在使用IDA分析文件时，最开始将自动定位到此地址。在内存中，代码段通常在PE文件头之后，数据块之前。

![](https://raw.githubusercontent.com/undoingfish/undoingfish.github.io/hexo/pic/2021/02/16/15.png)

9、BaseOfData：数据段的起始RVA。数据段通常在内存的末尾，即PE文件头和代码段之后，此值在64位可执行文件中不会出现。

10、**ImageBase**：文件在内存中的首选载入地址。例如本例中所使用DLL文件的ImageBase值为10000000。执行PE文件时，PE装载器先创建进程，再将文件载入内存，然后把EIP寄存器的值设置为ImageBase+AddressOfEntryPoint。

11、**SectionAlignment**：指定了节区在载入内存时的最小单位。默认值是目标CPU的页尺寸。例如本例中为0x00001000h=4096B=4KB。不同的CPU架构分页大小不同，例如在arm上貌似是64KB？

12、**FileAlignment**：指定了节区在磁盘文件中的最小单位，此值通常为200h或者1000h。

13、MajorOperatingSystemVersion：指定了操作系统最低版本号的主版本号，此值参考价值有限。

14、MinorOperatingSystemVersion：指定了操作系统最低版本号的次版本号。

15、MajorImageVersion：可执行文件的主版本号。

16、MinorImageVersion：可执行文件的次版本号。

17、MajorSubsystemVersion：要求最低子系统版本的主版本号。

18、MinorSubsystemVersion：要求最低子系统版本的次版本号。

19、Win32VersionValue：此字段目前从未被使用，通常为0。

20、**SizeOfImage**：映像载入内存后的总尺寸，即从ImageBase到最后一个块的大小。一般而言文件大小与加载到内存中的大小是不同的。

21、**SizeOfHeaders**：整个PE头的大小，必须是FileAlignment的整数倍。

22、CheckSum：映像的校验和，可以使用IMAGEHLP.DLL中的CheckSumMappedFile函数计算，通常为0，但是一些内核模式的驱动程序和系统DLL必须计算此校验和。

23、**Subsystem**：此字段值通常只对EXE重要，不同值对应不同的subsystem类型。

![](https://raw.githubusercontent.com/undoingfish/undoingfish.github.io/hexo/pic/2021/02/16/16.png)

24、DllCharacteristics：DllMain()函数何时被调用。

25、SizeOfStackReserve：EXE文件中为线程保留的栈的大小。

26、SizeOfStackCommit：EXE文件中最开始分配给栈的内存，默认值为1000h=4096B=4KB

27、SizeOfHeapReserve：EXE文件中为进程默认堆保留的内存，默认值为100000h=1048576B=1024*1024B

=1MB。

28、SizeOfHeapCommit：EXE文件中分配给堆的内存，默认值为1000h=4096B=4KB

29、LoaderFlags：与调试有关，默认为0。

30、**NumberOfRvaAndSizes**：数据目录的项数，自WindowsNT发布以来一直是0010h=16

31、**DataDirectory**：数据目录表，由数个相同的IMAGE_DATA_DIRECTORY结构组成的数组，值项import table、export table、resource等数据。每个IMAGE_DATA_DIRECTORY结构大小为8字节，其中重要的项为EXPORT、IMPORT、RESOURCE、TLS。

![](https://raw.githubusercontent.com/undoingfish/undoingfish.github.io/hexo/pic/2021/02/16/17.png)

PE文件在定位import table、export table、resource等重要数据时，便是从IMAGE_DATA_DIRECTORY结构开始的。

在本例中，数据目录表内容如下，若export table和import table都为0，则表示无输出表和输入表，本例中都不为0，说明有输入表和输出表：

![](https://raw.githubusercontent.com/undoingfish/undoingfish.github.io/hexo/pic/2021/02/16/18.png)

[IMAGE_DATA_DIRECTORY](https://docs.microsoft.com/en-us/windows/win32/api/winnt/ns-winnt-image_data_directory)结构体详细内容如下，8字节中，有4字节表示虚拟地址，4字节表示大小，结合本例，export table地址为0x000021B0h，大小为0x00023E16h，import table地址为0x0000205Ch，大小为0x00000050h：

![](https://raw.githubusercontent.com/undoingfish/undoingfish.github.io/hexo/pic/2021/02/16/19.png)

使用PE Tools查看PE文件信息：

![](https://raw.githubusercontent.com/undoingfish/undoingfish.github.io/hexo/pic/2021/02/16/20.png)

PEiD工具与PE Tool工具分析同一文件所得到的信息对比：

![](https://raw.githubusercontent.com/undoingfish/undoingfish.github.io/hexo/pic/2021/02/16/21.png)

### 节区头

接下来的部分，不同的书对此归纳不同，但基础层次的内容还是不变的，逆向工程核心原理将此分为节区头，节区头又分为三小节，即.text、.data、.rsrc节，加密与解密则将此节称为区块。不论何种称呼，此处分节的本质是IMAGE_SECTION_HEADER结构体组成的数组，每个结构体对应不同的节区，例如.text节区和.data节区有各自对应的IMAGE_SECTION_HEADER结构体。

##### IMAGE_SECTION_HEADER

IMAGE_SECTION_HEADER结构定义如下：

![](https://raw.githubusercontent.com/undoingfish/undoingfish.github.io/hexo/pic/2021/02/16/22.png)

本文例子中第一个IMAGE_SECTION_HEADER结构内容：

![](https://raw.githubusercontent.com/undoingfish/undoingfish.github.io/hexo/pic/2021/02/16/23.png)

1、 Name：大小为8字节的块名。

2、 **VirtualSize**：内存中实际使用的区块大小。

3、**VirtualAddress**：该区块被装载到内存时起始的相对虚拟地址，数值总是SectionAlignment的整数倍，即0x00001000h=4096B=4KB的整数倍。

4、**SizeOfRawData**：该块在磁盘中所占的空间。在PE文件中，该值会被FileAlignment调整，例如指定FileAlignment的值为200h，则若VirtualSize中的块长度为19Ah字节，此块的长度将被设置为200h。

5、**PointerToRawData**：该块在磁盘文件中的偏移，亦即磁盘文件中节区的起始位置，由FileAlignment确定。

6、PointerToRelocations：在EXE文件中无意义，在OBJ文件中表示本快重定位信息的偏移量。在OBJ文件中若该值不是0，则会值项一个IMAGE_RELOCATION结构数组。

7、PointerToLinenumbers：行号表在文件中的偏移量，属于调试信息。

8、NumberOfRelocations：在EXE文件中无意义。在OBJ文件中表示本快在重定位表中的重定位数目。

9、NumberOfLinenumbers：该块在行号表中的行号数目。

10、**Characteristics**：该字段指明块属性，例如代码/数据、可读/可写等标志，字段由下列值组合而成。

![](https://raw.githubusercontent.com/undoingfish/undoingfish.github.io/hexo/pic/2021/02/16/24.png)

 

PE文件一般至少由两个区块，一个是代码块，另一个是数据块，常见区块见下表。如果两个区块具有相似或者一致的属性，那么链接器在链接时会把它们合并成一个区块，这样便能节省磁盘和内存空间，区块的合并规则在此不赘述。

![](https://raw.githubusercontent.com/undoingfish/undoingfish.github.io/hexo/pic/2021/02/16/25.png)

此表格内容对应了第一章节中的以下表格：

![](https://raw.githubusercontent.com/undoingfish/undoingfish.github.io/hexo/pic/2021/01/08/PMA-%E7%AC%AC1%E7%AB%A0%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/1.png)

区块大小需要对齐，在PE文件中，FileAlignment定义了磁盘区块的对齐值，空白处由00h填充从而形成区块间隙，SectionAlignment定义了内存中区块的对齐值。

这一部分在两本书上写的不太一样，看的有点不明白，因此不再纠结此处，目前已经明白的是由于内存的分页机制，文件内容和加载到内存中的内容存在偏差。

ImageBase：内存基址

VA：在内存中的地址(存疑)

RVA：虚拟偏移量(存疑)

Offset：内存地址-内存基址+文件节区起始地址

### 输入表(IAT)

IAT，全称Import Address Table，又名导入地址表。其主要用于实现程序的动态调用，即程序能够调用执行代码不在自身程序中的函数，且调用方式为隐式调用，即程序开始时一同加载DLL。

##### IMAGE_IMPORT_DESCRIPTOR

IAT的主要结构为IMAGE_IMPORT_DESCRIPTOR，如下所示：

![](https://raw.githubusercontent.com/undoingfish/undoingfish.github.io/hexo/pic/2021/02/16/26.png)

1、**OriginalFirstThunk**：指向输入名称表INT的地址（RVA），INT是一个IMAGE_THUNK_DATA结构的数组，数组的每个IMAGE_THUNK_DATA结构都指向IMAGE_IMPORT_BY_NAME结构。

2、TimeDateStamp：一个32位的时间标志，可忽略。

3、ForwarderChain：转向第一个调用的API的索引。

4、**Name**：库名称字符串的地址（RVA），库名称形如“KERNEL32.DLL”、“USER32.DLL”。

5、**FirstThunk**：指向输入地址表IAT的地址（RVA）。

FirstThunk和OriginalFirstThunk相似，两者分别指向两个本质上相同的IMAGE_THUNK_DATA结构。

![](https://raw.githubusercontent.com/undoingfish/undoingfish.github.io/hexo/pic/2021/02/16/27.png)

##### IMAGE_THUNK_DATA

IMAGE_THUNK_DATA结构如下，注意其成员是一个union结构：

![](https://raw.githubusercontent.com/undoingfish/undoingfish.github.io/hexo/pic/2021/02/16/28.png)

当IMAGE_THUNK_DATA值的最高位为1是，表示函数以序号方式输入，则此时低31位表示一个函数序号。当其最高位为0时，表示函数以字符串函数名的方式输入，此时双字的值是一个RVA，指向一个IMAGE_IMPORT_BY_NAME结构。

##### IMAGE_IMPORT_BY_NAME

IMAGE_IMPORT_BY_NAME结构如下：

![](https://raw.githubusercontent.com/undoingfish/undoingfish.github.io/hexo/pic/2021/02/16/29.png)

1、Hint：表示本函数在其所驻留DLL的输出表中的序号。该域被PE装载器用来在DLL的输出表里快速查询函数，此项为可选项。

2、Name：含有输入函数的函数名，函数名是一个ASCII字符串，是一个可变尺寸域，以“NULL”结尾。

输入表在IMAGE_FILE_HEADER结构体的80h偏移位置，在本例中，其地址为E0h+80h=160h，160h处的值为5c200000h，亦即0000205c，通过PE tools知道该地址在.rdata分节，而Δk=0，Δk=Virtual Offset – Raw Offset，同样在PE tools中可以计算得本例中Δk=0，因此可以直接跳转到0000205c的位置。从0000205c开始，便是IMAGE_IMPORT_DESCRIPTOR的五个字段，每个字段四字节，最后一组IMAGE_IMPORT_DESCRIPTOR的值全为0。

![](https://raw.githubusercontent.com/undoingfish/undoingfish.github.io/hexo/pic/2021/02/16/30.png)

以第一个IMAGE_IMPORT_DESCRIPTOR为例，

1、 其Name中的地址为214E，214E中的地址便是需要导入的DLL名称。

![](https://raw.githubusercontent.com/undoingfish/undoingfish.github.io/hexo/pic/2021/02/16/31.png)

2、其OriginalFirstThunk中的地址为20AC，20AC中的地址为2116，且20AC后还有四个地址，都存储着要调用的DLL中的函数名称，如下所示。

![](https://raw.githubusercontent.com/undoingfish/undoingfish.github.io/hexo/pic/2021/02/16/32.png)

### 输出表(EAT)

与IAT对应，EAT使得其他应用程序可以调用此库文件中所提供的库函数。

##### IMAGE_EXPORT_DIRECTORY

输出表的结构体名称为IMAGE_EXPORT_DIRECTORY，其定义如下：

![](https://raw.githubusercontent.com/undoingfish/undoingfish.github.io/hexo/pic/2021/02/16/33.png)

1、 Characteristics：输出属性的标志，目前未使用，其值为0。

2、TimeDateStamp：输出表创建的时间（GMT时间）。

3、MajorVersion：输出表的主版本号，目前未使用，其值为0。

4、MinorVersion：输出表的次版本号，目前未使用，其值为0。

5、Name：指向与输出函数相关联DLL的名称字符串（例如“KERNEL32.DLL”）的地址（RVA）。

6、Base：PE文件输出表的起始序数值。

7、**NumberOfFunctions**：EAT中的条目数量。

8、**NumberOfNames**：输出函数名称表ENT里的条目数量。

9、**AddressOfFunctions**：EAT的地址（RVA）。

10、**AddressOfNames**：ENT的地址（RVA）。

11、**AddressOfNameOrdinals**：输出序数表的地址（RVA）。

类似查找导入表，导出表的位置在DataDirectory[1]中的VirtualAddress字段，其中的值为000021B0。

![](https://raw.githubusercontent.com/undoingfish/undoingfish.github.io/hexo/pic/2021/02/16/34.png)

000021B0处便是IMAGE_EXPORT_DIRECTORY结构的各项字段，本例中各字段全为0，因此无导出函数。

![](https://raw.githubusercontent.com/undoingfish/undoingfish.github.io/hexo/pic/2021/02/16/35.png)

整个PE文件实例的完整分析可见[Lab-01-1](https://github.com/undoingfish/undoingfish.github.io/blob/hexo/pic/lab01-1.pdf)。

 

 

 

 

 

 

 

![img](file:///C:/Users/zengf/AppData/Local/Temp/msohtmlclip1/01/clip_image091.jpg)