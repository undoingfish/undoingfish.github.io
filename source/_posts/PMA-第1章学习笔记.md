---
title: PMA 第1章学习笔记
date: 2021-01-08 14:20:58
tags: Practical Malware Analysis
---

#### **静态分析基础技术**

1、通过反病毒引擎扫描：https://www.virustotal.com/

2、计算恶意代码的哈希值，例如MD5值、SHA-1值。

3、使用Strings程序搜索文件中的可打印字符。

4、可使用PEiD检测恶意代码本身是否被加壳，以及被加壳的种类。

5、可使用Dependency Walker搜索恶意代码中的动态链接函数，以分析程序所使用的导入函数和导出函数。

6、使用PEview工具分析恶意程序的PE文件头部，例如在IMAGE_NT_HEADERS-->IMAGE_FILE_HEADER项中，可以查看到可执行文件的编译时间，但编译时间有可能是伪造的。另外，IMAGE_NT_HEADERS-->IMAGE_FILE_HEADER中还可以通过Subsystem描述来判断本程序是控制台程序还是图形界面程序。在IMAGE_SECTION_HEADER中，可以对比分节信息的虚拟大小和原始数据大小之间的差距来判断文件是否加壳。

7、可以用Resource Hacker工具来查看资源节（.rsrc节），并提取对应的文件。

#### **相关概念**

***PE文件格式：***

全称是Portable Executable，意为可移植的可执行的文件，常见的EXE、DLL、OCX、SYS、COM都是PE文件，PE文件是微软Windows操作系统上的程序文件（可能是间接被执行，如DLL）。

***PE文件的分节***

![](C:\Users\zengf\AppData\Roaming\Typora\typora-user-images\1.png)

***链接库与函数***

通常指导入函数，导入函数是一个程序需要用到但是存储在另一程序中的函数，例如常见的代码函数库。

***静态链接、运行时链接、动态链接***

1、静态链接是指将整个文件在编译之前链接到程序中，如果很多函数都放在一个目标文件中，很可能很多没用的函数都被一起链接，每个可执行程序中对所有需要的目标文件都要有一份副本，因此静态链接比较浪费空间。倘若目标文件有修改，则需要重新编译使得更新困难，因此静态链接比较浪费时间，是不常用的方式。

2、运行时链接是只有需要使用函数时才链接到库，而非在程序启动时就链接。运行时链接在恶意代码中比较常用。

3、动态链接解决了静态链接空间浪费和更新困难的问题，把程序按照模块拆分成各个相对独立部分，编译系统在链接阶段并不把目标文件和函数库文件链接在一起，而是等到程序在运行过程中需要使用时才链接函数库。动态链接是最常用的方法。

在PE文件头中存储了每个将被装在的库文件，以及每个会被程序使用的函数信息。在恶意代码分析的过程中对程序所使用的库与调用函数进行分析，我们可以初步猜测恶意代码的行为。

***套接字（socket)***

所谓套接字(Socket)，就是对网络中不同主机上的应用进程之间进行双向通信的端点的抽象。一个套接字就是网络上进程通信的一端，提供了应用层进程利用网络协议交换数据的机制。套接字是通信的基石，是支持TCP/IP协议的路通信的基本操作单元（逻辑上），是网络环境中进程间通信的API(现实意义上)。

#### **恶意代码中常见的Windows函数列表**

- **[accept](https://docs.microsoft.com/en-us/windows/win32/api/winsock2/nf-winsock2-accept)**

用于socket响应客户端连接请求。

```
SOCKET WSAAPI accept(
  SOCKET   s,
  sockaddr *addr,
  int      *addrlen
);
```

- **[AdjustTokenPrivileges](https://docs.microsoft.com/en-us/windows/win32/api/securitybaseapi/nf-securitybaseapi-adjusttokenprivileges)**

用于启用或禁用特定的访问权限。

```
BOOL AdjustTokenPrivileges(
  HANDLE            TokenHandle,
  BOOL              DisableAllPrivileges,
  PTOKEN_PRIVILEGES NewState,
  DWORD             BufferLength,
  PTOKEN_PRIVILEGES PreviousState,
  PDWORD            ReturnLength
);
```

- **[AttachThreadInput](https://docs.microsoft.com/en-us/windows/win32/api/winuser/nf-winuser-attachthreadinput)**

用于将一个线程处理的输入附加到另一个线程上，使得第二个线程接收到输入事件，例如鼠标和键盘事件。击键记录器和其他间谍软件会使用这个函数。

```
BOOL AttachThreadInput(
  DWORD idAttach,
  DWORD idAttachTo,
  BOOL  fAttach
);
```

- **[bind](https://docs.microsoft.com/en-us/windows/win32/api/winsock/nf-winsock-bind)**

将一个本地地址关联到socket上，用于监听客户端连接请求.

```
int bind(
  SOCKET         s,
  const sockaddr *addr,
  int            namelen
);
```

- **[Bitblt](https://docs.microsoft.com/en-us/windows/win32/api/wingdi/nf-wingdi-bitblt)**

用于将图形数据从一个设备复制到另一设备，可以使用这个函数来捕获屏幕，经常被编译器作为共享库代码添加。

```
BOOL BitBlt(
  HDC   hdc,
  int   x,
  int   y,
  int   cx,
  int   cy,
  HDC   hdcSrc,
  int   x1,
  int   y1,
  DWORD rop
);
```

- **[CallNextHookEx](https://docs.microsoft.com/en-us/windows/win32/api/winuser/nf-winuser-callnexthookex)**

将hook信息传递给当前hook链的下一个hook程序，hook程序可以在处理hook信息之前调用此函数，也可以在处理hook信息之后调用此函数。

```
LRESULT CallNextHookEx(
  HHOOK  hhk,
  int    nCode,
  WPARAM wParam,
  LPARAM lParam
);
```

- **[CertOpenSystemStore](https://docs.microsoft.com/en-us/windows/win32/api/wincrypt/nf-wincrypt-certopensystemstorea)**

用于访问本地系统中的证书。

```
HCERTSTORE CertOpenSystemStoreA(
  HCRYPTPROV_LEGACY hProv,
  LPCSTR            szSubsystemProtocol
);
```

- **[CheckRemoteDebuggerPresent](https://docs.microsoft.com/en-us/windows/win32/api/debugapi/nf-debugapi-checkremotedebuggerpresent)**

检查某个特定进程是否被调试，通常被用于反调试技术中。

```
BOOL CheckRemoteDebuggerPresent(
  HANDLE hProcess,
  PBOOL  pbDebuggerPresent
);
```

- **[CoCreateInstance](https://docs.microsoft.com/en-us/windows/win32/api/combaseapi/nf-combaseapi-cocreateinstance)**

创建并初始化一个指定CLSID（类标识）的类的实例。

```
HRESULT CoCreateInstance(
  REFCLSID  rclsid,
  LPUNKNOWN pUnkOuter,
  DWORD     dwClsContext,
  REFIID    riid,
  LPVOID    *ppv
);
```

- **[connect](https://docs.microsoft.com/en-us/windows/win32/api/winsock2/nf-winsock2-connect)**

连接一个远程socket，通常用来连接一个命令控制服务器。

```
int WSAAPI connect(
  SOCKET         s,
  const sockaddr *name,
  int            namelen
);
```

- **[ConnectNamedPipe](https://docs.microsoft.com/en-us/windows/win32/api/namedpipeapi/nf-namedpipeapi-connectnamedpipe)**

用来为进程间通信创建一个服务端管道，并等待一个客户端管道来连接。后门程序和反向shell经常使用此函数来简单地连接到一个命令控制服务器。

```
BOOL ConnectNamedPipe(
  HANDLE       hNamedPipe,
  LPOVERLAPPED lpOverlapped
);
```

- **[ControlService](https://docs.microsoft.com/en-us/windows/win32/api/winsvc/nf-winsvc-controlservice)**

发送一个控制信号到某个服务，其用意与发送的信号有关。

```
BOOL ControlService(
  SC_HANDLE        hService,
  DWORD            dwControl,
  LPSERVICE_STATUS lpServiceStatus
);
```

- **[CreateFile](https://docs.microsoft.com/en-us/windows/win32/api/fileapi/nf-fileapi-createfilea)**

创建一个新文件，或者打开一个已有文件。

```
HANDLE CreateFileA(
  LPCSTR                lpFileName,
  DWORD                 dwDesiredAccess,
  DWORD                 dwShareMode,
  LPSECURITY_ATTRIBUTES lpSecurityAttributes,
  DWORD                 dwCreationDisposition,
  DWORD                 dwFlagsAndAttributes,
  HANDLE                hTemplateFile
);
```

- **[CreateFileMapping](https://docs.microsoft.com/en-us/windows/win32/api/winbase/nf-winbase-createfilemappinga)**

创建一个映射到文件的句柄，将文件装载到内存，并使得它可以通过内存地址进行访问。启动器、装载器和注入器会使用这个函数来读取和修改PE文件。

```
HANDLE CreateFileMappingA(
  HANDLE                hFile,
  LPSECURITY_ATTRIBUTES lpFileMappingAttributes,
  DWORD                 flProtect,
  DWORD                 dwMaximumSizeHigh,
  DWORD                 dwMaximumSizeLow,
  LPCSTR                lpName
);
```

- **[CreateMutex](https://docs.microsoft.com/en-us/windows/win32/api/synchapi/nf-synchapi-createmutexa)**

**创建一个互斥对象，可以被恶意代码用来确保某个时刻在系统上只有一个实例运行。恶意代码经常使用固定名字作为互斥对象名称，因此可以作为是否感染了恶意代码的一个主机特征。**

```
HANDLE CreateMutexA(
  LPSECURITY_ATTRIBUTES lpMutexAttributes,
  BOOL                  bInitialOwner,
  LPCSTR                lpName
);
```

- **[CreateProcess](https://docs.microsoft.com/en-us/windows/win32/api/processthreadsapi/nf-processthreadsapi-createprocessa)**

创建并启动一个新进程。

```
BOOL CreateProcessA(
  LPCSTR                lpApplicationName,
  LPSTR                 lpCommandLine,
  LPSECURITY_ATTRIBUTES lpProcessAttributes,
  LPSECURITY_ATTRIBUTES lpThreadAttributes,
  BOOL                  bInheritHandles,
  DWORD                 dwCreationFlags,
  LPVOID                lpEnvironment,
  LPCSTR                lpCurrentDirectory,
  LPSTARTUPINFOA        lpStartupInfo,
  LPPROCESS_INFORMATION lpProcessInformation
);
```

- **[CreateRemoteThread](https://docs.microsoft.com/en-us/windows/win32/api/processthreadsapi/nf-processthreadsapi-createremotethread)**

**用于在另一个进程的虚拟地址空间创建一个线程。启动器和隐蔽性恶意代码经常使用此函数来将代码注入到其他进程中执行。**

```
HANDLE CreateRemoteThread(
  HANDLE                 hProcess,
  LPSECURITY_ATTRIBUTES  lpThreadAttributes,
  SIZE_T                 dwStackSize,
  LPTHREAD_START_ROUTINE lpStartAddress,
  LPVOID                 lpParameter,
  DWORD                  dwCreationFlags,
  LPDWORD                lpThreadId
);
```

- **[CreateService](https://docs.microsoft.com/en-us/windows/win32/api/winsvc/nf-winsvc-createservicea)**

创建一个可以在启动时刻运行的服务，用以持久化、隐藏或者启动内核驱动。

```
SC_HANDLE CreateServiceA(
  SC_HANDLE hSCManager,
  LPCSTR    lpServiceName,
  LPCSTR    lpDisplayName,
  DWORD     dwDesiredAccess,
  DWORD     dwServiceType,
  DWORD     dwStartType,
  DWORD     dwErrorControl,
  LPCSTR    lpBinaryPathName,
  LPCSTR    lpLoadOrderGroup,
  LPDWORD   lpdwTagId,
  LPCSTR    lpDependencies,
  LPCSTR    lpServiceStartName,
  LPCSTR    lpPassword
);
```

- **[CreateToolhelp32Snapshot](https://docs.microsoft.com/en-us/windows/win32/api/tlhelp32/nf-tlhelp32-createtoolhelp32snapshot)**

用于创建一个进程、对空间、线程和模块的快照。

```
HANDLE CreateToolhelp32Snapshot(
  DWORD dwFlags,
  DWORD th32ProcessID
);
```

- **[CryptAcquireContext](https://docs.microsoft.com/en-us/windows/win32/api/wincrypt/nf-wincrypt-cryptacquirecontexta)** 

用于获取windows加密库中某个特定密钥容器的handle。

```
BOOL CryptAcquireContextA(
  HCRYPTPROV *phProv,
  LPCSTR     szContainer,
  LPCSTR     szProvider,
  DWORD      dwProvType,
  DWORD      dwFlags
);
```

- **[DeviceIoControl](https://docs.microsoft.com/en-us/windows/win32/api/ioapiset/nf-ioapiset-deviceiocontrol)**

直接发送某个控制代码到指定的设备驱动，使得响应的设备根据控制代码执行响应操作。

```
BOOL DeviceIoControl(
  HANDLE       hDevice,
  DWORD        dwIoControlCode,
  LPVOID       lpInBuffer,
  DWORD        nInBufferSize,
  LPVOID       lpOutBuffer,
  DWORD        nOutBufferSize,
  LPDWORD      lpBytesReturned,
  LPOVERLAPPED lpOverlapped
);
```

- **[DllCanUnloadNow](https://docs.microsoft.com/en-us/windows/win32/api/combaseapi/nf-combaseapi-dllcanunloadnow)**

确定实现某个功能的DLL是否正在使用，如果不是，则调用者可以将此DLL移出内存。

```
HRESULT DllCanUnloadNow();
```

- **[DllGetClassObject](https://docs.microsoft.com/en-us/windows/win32/api/combaseapi/nf-combaseapi-dllgetclassobject)**

从DLL对象handler或者应用程序对象中检索类对象。

```
HRESULT DllGetClassObject(
  REFCLSID rclsid,
  REFIID   riid,
  LPVOID   *ppv
);
```

- **[DllInstall](https://docs.microsoft.com/en-us/windows/win32/api/shlwapi/nf-shlwapi-dllinstall)**

安装并设置一个DLL。

```
HRESULT DllInstall(
  BOOL   bInstall,
  PCWSTR pszCmdLine
);
```

- **[DllRegisterServer](https://docs.microsoft.com/en-us/windows/win32/api/olectl/nf-olectl-dllregisterserver)**

使一个进程内server为在server模块中所有受支持的class创建注册表项。

```
HRESULT DllRegisterServer();
```

- **[DllUnregisterServer](https://docs.microsoft.com/en-us/windows/win32/api/olectl/nf-olectl-dllunregisterserver)**

使一个进程内server删除掉通过DllRegisterServer创建的注册表项。

```
HRESULT DllUnregisterServer();
```

- **EnableExecuteProtectionSupport**

此函数不在API文档中，用于修改宿主机中的数据执行保护（DEP）设置，使得系统更容易被攻击。

- **[EnumProcesses](https://docs.microsoft.com/en-us/windows/win32/api/psapi/nf-psapi-enumprocesses)**

用于检索系统中每个进程的过程标识符。（此项Microsoft Docs中已404）

- **[EnumProcessModules](https://docs.microsoft.com/en-us/windows/win32/api/psapi/nf-psapi-enumprocessmodules)**

检索特定进程中的每个模块。（此项Microsoft Docs中已404）

- **FindFirstFile/FindNextFile**

[FindFirstFile](https://docs.microsoft.com/en-us/windows/win32/api/fileapi/nf-fileapi-findfirstfilea)用于在某个目录或者子目录中搜索，返回第一个匹配的文件名称。

[FindNextFile](https://docs.microsoft.com/en-us/windows/win32/api/fileapi/nf-fileapi-findnextfilea)沿袭[FindFirstFile](https://docs.microsoft.com/en-us/windows/desktop/api/fileapi/nf-fileapi-findfirstfilea), [FindFirstFileEx](https://docs.microsoft.com/en-us/windows/desktop/api/fileapi/nf-fileapi-findfirstfileexa), 以及[FindFirstFileTransacted](https://docs.microsoft.com/en-us/windows/desktop/api/winbase/nf-winbase-findfirstfiletransacteda) 的调用，继续搜索。

```
HANDLE FindFirstFileA(
  LPCSTR             lpFileName,
  LPWIN32_FIND_DATAA lpFindFileData
);

BOOL FindNextFileA(
  HANDLE             hFindFile,
  LPWIN32_FIND_DATAA lpFindFileData
);
```

- **[FindResource](https://docs.microsoft.com/en-us/windows/win32/api/winbase/nf-winbase-findresourcea)**

用于在指定模块中查找指定类型以及指定名称的资源位置。

```
HRSRC FindResourceA(
  HMODULE hModule,
  LPCSTR  lpName,
  LPCSTR  lpType
);
```

- **[FindWindow](https://docs.microsoft.com/en-us/windows/win32/api/winuser/nf-winuser-findwindowa)**

根据类名和窗口名进行桌面窗口搜索的函数，可用于反调试技术，搜索OllyDbg工具的窗口。

```
HWND FindWindowA(
  LPCSTR lpClassName,
  LPCSTR lpWindowName
);
```

- **[FtpPutFile](https://docs.microsoft.com/en-us/windows/win32/api/wininet/nf-wininet-ftpputfilea)**

用于向一个远程FTP服务器上传文件。

```
BOOLAPI FtpPutFileA(
  HINTERNET hConnect,
  LPCSTR    lpszLocalFile,
  LPCSTR    lpszNewRemoteFile,
  DWORD     dwFlags,
  DWORD_PTR dwContext
);
```

- **[GetAdaptersInfo](https://docs.microsoft.com/en-us/windows/win32/api/iphlpapi/nf-iphlpapi-getadaptersinfo)**

用于获取本地主机的网络适配器信息。也可用于获取主机的MAC地址，并且在对抗虚拟机技术中可以用来检测VMware等虚拟机。

```
IPHLPAPI_DLL_LINKAGE ULONG GetAdaptersInfo(
  PIP_ADAPTER_INFO AdapterInfo,
  PULONG           SizePointer
);
```

- **[GetAsyncKeyState](https://docs.microsoft.com/en-us/windows/win32/api/winuser/nf-winuser-getasynckeystate)**

用于确定键盘上的某个键是否被输入，可以被用来实现击键记录器。

```
SHORT GetAsyncKeyState(
  int vKey
);
```

- **[GetDC](https://docs.microsoft.com/en-us/windows/win32/api/winuser/nf-winuser-getdc)**

DC是指device context。本函数用于获取某个窗口或者整个屏幕的handle。通常被用于抓取桌面截屏。

```
HDC GetDC(
  HWND hWnd
);
```

- **[GetForegroundWindow](https://docs.microsoft.com/en-us/windows/win32/api/winuser/nf-winuser-getforegroundwindow)**

返回当前用户在桌面上的工作窗口，击键记录器会普遍使用这个函数来确定用户正在往哪个窗口输入

```
HWND GetForegroundWindow();
```

- **[gethostbyname](https://docs.microsoft.com/en-us/windows/win32/api/winsock/nf-winsock-gethostbyname)**

用于对一个特定域名进行一次DNS查询。被查询的域名可以用作恶意程序的网络检测特征码。

```
hostent * gethostbyname(
  const char *name
);
```

- **[gethostname](https://docs.microsoft.com/en-us/windows/win32/api/winsock/nf-winsock-gethostname)**

获取当前计算机的主机名。

```
int gethostname(
  char *name,
  int  namelen
);
```

- **[GetKeyState](https://docs.microsoft.com/en-us/windows/win32/api/winuser/nf-winuser-getkeystate)**

获取键盘上一个特定键的状态，通常被击键记录器所使用。

```
SHORT GetKeyState(
  int nVirtKey
);
```

- **[GetModuleFilename](https://docs.microsoft.com/en-us/windows/win32/api/libloaderapi/nf-libloaderapi-getmodulefilenamea)**

返回当前已加载的某个指定模块的绝对路径。可用于在运行进程中修改或者复制文件。

```
DWORD GetModuleFileNameA(
  HMODULE hModule,
  LPSTR   lpFilename,
  DWORD   nSize
);
```

- **[GetModuleHandle](https://docs.microsoft.com/en-us/windows/win32/api/libloaderapi/nf-libloaderapi-getmodulehandlea)**

用于获取当前已加载的某个模块的module handle。可以用来在一个装载模块中定位和修改代码，或者搜索某个合适的位置来注入代码。

```
HMODULE GetModuleHandleA(
  LPCSTR lpModuleName
);
```

- **[GetProcAddress]()**

用于获取某个被装入到内存中的DLL程序的函数地址，以便从该程序中获取需要用到的导入函数。

```
FARPROC GetProcAddress(
  HMODULE hModule,
  LPCSTR  lpProcName
);
```

- **[GetStartupInfo](https://docs.microsoft.com/en-us/windows/win32/api/processthreadsapi/nf-processthreadsapi-getstartupinfow)**

获取某个进程被创建时的[STARTUPINFO](https://docs.microsoft.com/en-us/windows/desktop/api/processthreadsapi/ns-processthreadsapi-startupinfoa)结构 。

```
void GetStartupInfoW(
  LPSTARTUPINFOW lpStartupInfo
);
```

- **[GetSystemDefaultLangId](https://docs.microsoft.com/en-us/windows/win32/api/winnls/nf-winnls-getsystemdefaultlangid)**

返回系统默认语言设置，可以用来定制系统显示语言，可以作为被感染主机的摘要信息。

```
LANGID GetSystemDefaultLangID();
```

- **[GetTempPath](https://docs.microsoft.com/en-us/windows/win32/api/fileapi/nf-fileapi-gettemppatha)**

返回某个指定的临时文件的路径。恶意代码可能通过其获取临时文件路径，读取或写入某些文件。

```
DWORD GetTempPathA(
  DWORD nBufferLength,
  LPSTR lpBuffer
);
```

- **[GetThreadContext](https://docs.microsoft.com/en-us/windows/win32/api/processthreadsapi/nf-processthreadsapi-getthreadcontext)**

返回某个指定线程的上下文结构。线程的上下文结构中存储了线程的所有信息，比如寄存器值和当前状态。

```
BOOL GetThreadContext(
  HANDLE    hThread,
  LPCONTEXT lpContext
);
```

- **[GetTickCount](https://docs.microsoft.com/en-us/windows/win32/api/sysinfoapi/nf-sysinfoapi-gettickcount)**

获取系统启动后到当前时间的微秒数，最多累计到49.7天。此函数只能提供少量的线索信息，例如在反调试技术中被用来获取时间信息。

```
DWORD GetTickCount();
```

- **[GetVersionEx](https://docs.microsoft.com/en-us/windows/win32/api/sysinfoapi/nf-sysinfoapi-getversionexa)**

本函数在Windows 8.1之后的windows版本可能不再支持。本函数返回当前运行的Windows操作系统版本系统信息。

```
NOT_BUILD_WINDOWS_DEPRECATE BOOL GetVersionExA(
  LPOSVERSIONINFOA lpVersionInformation
);
```

- **[GetWindowsDirectory](https://docs.microsoft.com/en-us/windows/win32/api/sysinfoapi/nf-sysinfoapi-getwindowsdirectorya)**

返回Windows目录的文件系统路径（通常是C:\Windows），恶意代码经常使用这个函数来确定其他恶意程序的安装路径。

```
UINT GetWindowsDirectoryA(
  LPSTR lpBuffer,
  UINT  uSize
);
```

- **[inet_addr](https://docs.microsoft.com/en-us/windows/win32/api/winsock2/nf-winsock2-inet_addr)**

对一个IP地址字符串进行转换成[IN_ADDR](https://docs.microsoft.com/en-us/windows/desktop/api/winsock2/ns-winsock2-in_addr)结构的地址，这些字符串可作为网络特征码。

```
unsigned long WSAAPI inet_addr(
  const char *cp
);
```

- **[InternetOpen](https://docs.microsoft.com/en-us/windows/win32/api/wininet/nf-wininet-internetopenw)**

初始化调用WinINet函数的程序，其参数之一User-Agent(lpszAgent)可作为网络特征码。该函数可用于寻找网络访问功能的初始位置。

```
void InternetOpenW(
  LPCWSTR lpszAgent,
  DWORD   dwAccessType,
  LPCWSTR lpszProxy,
  LPCWSTR lpszProxyBypass,
  DWORD   dwFlags
);
```

- **[InternetOpenUrl](https://docs.microsoft.com/en-us/windows/win32/api/wininet/nf-wininet-internetopenurla)**

使用FTP、HTTP或者HTTPS连接协议来打开一个特定的URL。此URL可以作为网络特征码。

```
void InternetOpenUrlA(
  HINTERNET hInternet,
  LPCSTR    lpszUrl,
  LPCSTR    lpszHeaders,
  DWORD     dwHeadersLength,
  DWORD     dwFlags,
  DWORD_PTR dwContext
);
```

- **[InternetReadFile](https://docs.microsoft.com/en-us/windows/win32/api/wininet/nf-wininet-internetreadfile)**

读取 [InternetOpenUrl](https://docs.microsoft.com/en-us/windows/desktop/api/wininet/nf-wininet-internetopenurla), [FtpOpenFile](https://docs.microsoft.com/en-us/windows/desktop/api/wininet/nf-wininet-ftpopenfilea), 或者[HttpOpenRequest](https://docs.microsoft.com/en-us/windows/desktop/api/wininet/nf-wininet-httpopenrequesta)函数打开的URL链接数据。

```
BOOLAPI InternetReadFile(
  HINTERNET hFile,
  LPVOID    lpBuffer,
  DWORD     dwNumberOfBytesToRead,
  LPDWORD   lpdwNumberOfBytesRead
);
```

- **[InternetWriteFile](https://docs.microsoft.com/en-us/windows/win32/api/wininet/nf-wininet-internetwritefile)**

向某个打开的网络文件中写数据。

```
BOOLAPI InternetWriteFile(
  HINTERNET hFile,
  LPCVOID   lpBuffer,
  DWORD     dwNumberOfBytesToWrite,
  LPDWORD   lpdwNumberOfBytesWritten
);
```

- **[IsDebuggerPresent](https://docs.microsoft.com/en-us/windows/win32/api/debugapi/nf-debugapi-isdebuggerpresent)**

检查当前进程是否被调试器调试，常用于反调试技术中。此函数经常由编译器添加并包含在许多可执行程序中。

```
BOOL IsDebuggerPresent();
```

- **[IsNTAdmin](https://source.winehq.org/WineAPI/IsNTAdmin.html)**

检查某个用户是否具有管理员权限。

```
 BOOL IsNTAdmin
 (
  DWORD   reserved,
  LPDWORD pReserved
 )
```

- **[IsWoW64Process](https://docs.microsoft.com/en-us/windows/win32/api/wow64apiset/nf-wow64apiset-iswow64process)**

用于确定某个指定进程是否运行于64位操作系统上。

```
BOOL IsWow64Process(
  HANDLE hProcess,
  PBOOL  Wow64Process
);
```

- **[LdrLoadDll](http://undocumented.ntinternals.net/index.html?page=UserMode%2FUndocumented%20Functions%2FExecutable%20Images%2FLdrLoadDll.html)**

功能同[LoadLibraryEx](https://docs.microsoft.com/en-us/windows/win32/api/libloaderapi/nf-libloaderapi-loadlibraryexa)函数，但方式更为隐蔽。

```
LdrLoadDll(
  IN PWCHAR               PathToFile OPTIONAL,
  IN ULONG                Flags OPTIONAL,
  IN PUNICODE_STRING      ModuleFileName,
  OUT PHANDLE             ModuleHandle
);
```

- **[LoadLibrary](https://docs.microsoft.com/en-us/windows/win32/api/libloaderapi/nf-libloaderapi-loadlibrarya)**

装载某个DLL程序到进程中，该DLL可能导致其他模块被加载。几乎每个Win32程序都会导入此函数。

```
HMODULE LoadLibraryA(
  LPCSTR lpLibFileName
);
```

- **[LoadResource](https://docs.microsoft.com/en-us/windows/win32/api/libloaderapi/nf-libloaderapi-loadresource)**

获取到某个指向内存中指定资源的首个字节的指针的句柄，从而装载PE文件中的资源到内存中，该资源可能是字符串、配置信息或者其他恶意文件。

```
HGLOBAL LoadResource(
  HMODULE hModule,
  HRSRC   hResInfo
);
```

- **[LsaEnumerateLogonSessions](https://docs.microsoft.com/en-us/windows/win32/api/ntsecapi/nf-ntsecapi-lsaenumeratelogonsessions)**

获取当前系统的登录会话标识符（LUIDs），往往是一个登录凭证窃取器常用的功能之一。

```
NTSTATUS LsaEnumerateLogonSessions(
  PULONG LogonSessionCount,
  PLUID  *LogonSessionList
);
```

- **[MapViewOfFile](https://docs.microsoft.com/en-us/windows/win32/api/memoryapi/nf-memoryapi-mapviewoffile)**

将一个文件映射到内存，使得文件内容可以通过内存地址访问。启动器、装载器、注入器通常使用此函数来读取和修改PE文件，从而不通过WriteFile来修改文件内容。

```
LPVOID MapViewOfFile(
  HANDLE hFileMappingObject,
  DWORD  dwDesiredAccess,
  DWORD  dwFileOffsetHigh,
  DWORD  dwFileOffsetLow,
  SIZE_T dwNumberOfBytesToMap
);
```

- **[MapVirtualKey](https://docs.microsoft.com/en-us/windows/win32/api/winuser/nf-winuser-mapvirtualkeya)**

将一个虚拟键值转换成字符值，通常被用于击键记录器。

```
UINT MapVirtualKeyA(
  UINT uCode,
  UINT uMapType
);
```

- **[MmGetSystemRoutineAddress]()**

被内核调用，仅用于获得ntoskrnl.exe和hal.dll的函数地址。

```
PVOID MmGetSystemRoutineAddress(
  PUNICODE_STRING SystemRoutineName
);
```

- **Module32First /Module32Next**

[Module32First](https://docs.microsoft.com/en-us/windows/win32/api/tlhelp32/nf-tlhelp32-module32first)可以获取到进程的第一个模块。

[Module32Next](https://docs.microsoft.com/en-us/windows/win32/api/tlhelp32/nf-tlhelp32-module32next)可以获取到进程的下一个模块

```
BOOL Module32First(
  HANDLE          hSnapshot,
  LPMODULEENTRY32 lpme
);

BOOL Module32Next(
  HANDLE          hSnapshot,
  LPMODULEENTRY32 lpme
);
```

- **[NetScheduleJobAdd](https://docs.microsoft.com/en-us/windows/win32/api/lmat/nf-lmat-netschedulejobadd)**

此函数在win 8之后不再支持。其可以指定一个程序在某个特定时刻运行。

```
NET_API_STATUS NET_API_FUNCTION NetScheduleJobAdd(
  LPCWSTR Servername,
  LPBYTE  Buffer,
  LPDWORD JobId
);
```

- **[NetShareEnum](https://docs.microsoft.com/en-us/windows/win32/api/lmshare/nf-lmshare-netshareenum)**

获取某个服务器上每个共享资源的相关信息。

```
NET_API_STATUS NET_API_FUNCTION NetShareEnum(
  LMSTR   servername,
  DWORD   level,
  LPBYTE  *bufptr,
  DWORD   prefmaxlen,
  LPDWORD entriesread,
  LPDWORD totalentries,
  LPDWORD resume_handle
);
```

- **[NtQueryDirectoryFile](https://docs.microsoft.com/en-us/windows-hardware/drivers/ddi/ntifs/nf-ntifs-ntquerydirectoryfile)**

返回一个目录中的各类文件信息。RootKit普遍使用此函数来隐藏文件。

```
__kernel_entry NTSYSCALLAPI NTSTATUS NtQueryDirectoryFile(
  HANDLE                 FileHandle,
  HANDLE                 Event,
  PIO_APC_ROUTINE        ApcRoutine,
  PVOID                  ApcContext,
  PIO_STATUS_BLOCK       IoStatusBlock,
  PVOID                  FileInformation,
  ULONG                  Length,
  FILE_INFORMATION_CLASS FileInformationClass,
  BOOLEAN                ReturnSingleEntry,
  PUNICODE_STRING        FileName,
  BOOLEAN                RestartScan
);
```

- **[NtQueryInformationProcess]()**

返回某个特定进程的各种信息。通常用于反调试技术中，其返回的信息与CheckRemoteDebuggerPresent函数相同。

```
__kernel_entry NTSTATUS NtQueryInformationProcess(
  HANDLE           ProcessHandle,
  PROCESSINFOCLASS ProcessInformationClass,
  PVOID            ProcessInformation,
  ULONG            ProcessInformationLength,
  PULONG           ReturnLength
);
```

- **[NtSetInformationProcess](http://undocumented.ntinternals.net/index.html?page=UserMode%2FUndocumented%20Functions%2FNT%20Objects%2FProcess%2FNtSetInformationProcess.html)**

用于改变一个程序的权限级别，或者用于绕过数据执行保护（DEP）。

```
NtSetInformationProcess(
  IN HANDLE               ProcessHandle,
  IN PROCESS_INFORMATION_CLASS ProcessInformationClass,
  IN PVOID                ProcessInformation,
  IN ULONG                ProcessInformationLength
);
```

- **[OleInitialize](https://docs.microsoft.com/en-us/windows-hardware/drivers/ddi/ntifs/nf-ntifs-ntquerydirectoryfile)**

用于初始化COM库，在使用COM对象的相关功能之前必须调用此函数。

```
HRESULT OleInitialize(
  LPVOID pvReserved
);
```

- **[OpenMutex](https://docs.microsoft.com/en-us/windows/win32/api/synchapi/nf-synchapi-openmutexw)**

打开一个已存在的互斥对象。通常被恶意代码用于确保在任意时间，系统中只有一个运行实例。互斥对象通常是固定名字，因此可以作为主机特征。

```
HANDLE OpenMutexW(
  DWORD   dwDesiredAccess,
  BOOL    bInheritHandle,
  LPCWSTR lpName
);
```

- **[OpenProcess](https://docs.microsoft.com/en-us/windows/win32/api/processthreadsapi/nf-processthreadsapi-openprocess)**

打开系统上其他运行中进程的句柄。此句柄可以用来向其他进程所在的运行内存中读写数据，或者注入代码。

```
HANDLE OpenProcess(
  DWORD dwDesiredAccess,
  BOOL  bInheritHandle,
  DWORD dwProcessId
);
```

- **[OpenSCManager](https://docs.microsoft.com/en-us/windows/win32/api/winsvc/nf-winsvc-openscmanagera)**

在指定计算机上建立与服务控制管理器的连接，并获取特定服务控制管理器的数据库。任何想要安装、修改或者是控制一个服务的程序，都必须先调用此函数。

```
SC_HANDLE OpenSCManagerA(
  LPCSTR lpMachineName,
  LPCSTR lpDatabaseName,
  DWORD  dwDesiredAccess
);
```

- **[OutputDebugString](https://docs.microsoft.com/en-us/windows/win32/api/debugapi/nf-debugapi-outputdebugstringw)**

输出字符串到一个附加的调试器上，可以用于反调试技术。

```
void OutputDebugStringW(
  LPCWSTR lpOutputString
);
```

- **[PeekNamedPide](https://docs.microsoft.com/en-us/windows/win32/api/namedpipeapi/nf-namedpipeapi-peeknamedpipe)**

可从命名管道中复制数据至缓冲区，复制时不删除源数据。常用于反向shell。

```
BOOL PeekNamedPipe(
  HANDLE  hNamedPipe,
  LPVOID  lpBuffer,
  DWORD   nBufferSize,
  LPDWORD lpBytesRead,
  LPDWORD lpTotalBytesAvail,
  LPDWORD lpBytesLeftThisMessage
);
```

- **Process32First/Process32Next**

[Process32First](https://docs.microsoft.com/en-us/windows/win32/api/tlhelp32/nf-tlhelp32-process32first)获取系统快照中的第一个进程信息，在调用CreateToolhelp32Snapshot函数后可以使用。

[Process32Next](https://docs.microsoft.com/en-us/windows/win32/api/tlhelp32/nf-tlhelp32-process32next)获取系统快照中的下一个进程信息。通常用于遍历以寻找到某个可注入进程。

```
BOOL Process32First(
  HANDLE           hSnapshot,
  LPPROCESSENTRY32 lppe
);

BOOL Process32Next(
  HANDLE           hSnapshot,
  LPPROCESSENTRY32 lppe
);
```

- **[QueryPerformanceCounter](https://docs.microsoft.com/en-us/windows/win32/api/profileapi/nf-profileapi-queryperformancecounter)**

用于获取基于硬件的性能计数器的值，有时可用于反调试技术以获取时间信息。此函数会被编译器添加并包含在许多可执行程序中。

```
BOOL QueryPerformanceCounter(
  LARGE_INTEGER *lpPerformanceCount
);
```

- **[QueueUserAPC](https://docs.microsoft.com/en-us/windows/win32/api/processthreadsapi/nf-processthreadsapi-queueuserapc)**

向异步过程调用（asychronous procedure call，APC）队列添加一个用户模式的APC对象。可用于向其他进程注入代码。

```
DWORD QueueUserAPC(
  PAPCFUNC  pfnAPC,
  HANDLE    hThread,
  ULONG_PTR dwData
);
```

- **[ReadProcessMemory](https://docs.microsoft.com/en-us/windows/win32/api/memoryapi/nf-memoryapi-readprocessmemory)**

从指定进程中拷贝指定范围的数据到当前进程的指定缓冲区中。

```
BOOL ReadProcessMemory(
  HANDLE  hProcess,
  LPCVOID lpBaseAddress,
  LPVOID  lpBuffer,
  SIZE_T  nSize,
  SIZE_T  *lpNumberOfBytesRead
);
```

- **[recv](https://docs.microsoft.com/en-us/windows/win32/api/winsock/nf-winsock-recv)**

从socket接受数据

```
int recv(
  SOCKET s,
  char   *buf,
  int    len,
  int    flags
);
```

- **[RegisterHotKey](https://docs.microsoft.com/en-us/windows/win32/api/winuser/nf-winuser-registerhotkey)**

用于定义一个系统范围的hot key，当用户任意时刻输入一个特定键值组合（比如Ctrl+Alt+J）时，热键句柄将会接受到消息，由于时系统范围的热键，因此用户在任何一个窗口输入键值组合都能触发。此函数通常被间谍软件使用。

```
BOOL RegisterHotKey(
  HWND hWnd,
  int  id,
  UINT fsModifiers,
  UINT vk
);
```

- **[RegOpenKey](https://docs.microsoft.com/en-us/windows/win32/api/winreg/nf-winreg-regopenkeya)**

打开某个指定的注册表键值，用以读写。修改注册表键值通常时持久化方法之一。注册表包含了完整额操作系统和应用程序配置信息。

```
LSTATUS RegOpenKeyA(
  HKEY   hKey,
  LPCSTR lpSubKey,
  PHKEY  phkResult
);
```

- **[ResumeThread](https://docs.microsoft.com/en-us/windows/win32/api/processthreadsapi/nf-processthreadsapi-resumethread)**

继续执行之前挂起的线程，常用于注入技术中。

```
DWORD ResumeThread(
  HANDLE hThread
);
```

- **[RtlCreateRegistryKey](https://docs.microsoft.com/en-us/windows-hardware/drivers/ddi/wdm/nf-wdm-rtlcreateregistrykey)**

在内核模式代码中，向注册表创建一个键值，该键值为给定的绝对路径。

```
NTSYSAPI NTSTATUS RtlCreateRegistryKey(
  ULONG RelativeTo,
  PWSTR Path
);
```

- **[RtlWriteRegistryValue](https://docs.microsoft.com/en-us/windows-hardware/drivers/ddi/wdm/nf-wdm-rtlwriteregistryvalue)**

在内核模式向注册表写入一个键值。

```
NTSYSAPI NTSTATUS RtlWriteRegistryValue(
  ULONG  RelativeTo,
  PCWSTR Path,
  PCWSTR ValueName,
  ULONG  ValueType,
  PVOID  ValueData,
  ULONG  ValueLength
);
```

- **SamIConnect**

连接安全账户管理器（SAM），以调用其他用来访问登录凭证信息的函数（在Microsoft Doc中未提供此函数的说明）。

- **SamIGetPrivateData**

从安全账户管理器数据库中查询某个特定用户的私密信息（在Microsoft Doc中未提供此函数的说明）。

- **SamQueryInformationUse**

从安全账户管理器数据库中查询某个特定用户的口令密文（在Microsoft Doc中未提供此函数的说明）。

- **[send](https://docs.microsoft.com/en-us/windows/win32/api/winsock2/nf-winsock2-send)**

向已连接的socket发送数据。

```
int WSAAPI send(
  SOCKET     s,
  const char *buf,
  int        len,
  int        flags
);
```

- **[SetFileTime](https://docs.microsoft.com/en-us/windows/win32/api/fileapi/nf-fileapi-setfiletime)**

修改某个文件或目录的创建、访问或者最后修改时间。通常被恶意程序用于隐藏恶意行为。

```
BOOL SetFileTime(
  HANDLE         hFile,
  const FILETIME *lpCreationTime,
  const FILETIME *lpLastAccessTime,
  const FILETIME *lpLastWriteTime
);
```

- **[SetThreadContext](https://docs.microsoft.com/en-us/windows/win32/api/processthreadsapi/nf-processthreadsapi-setthreadcontext)**

用于修改指定线程的上下文，可用于注入技术。

```
BOOL SetThreadContext(
  HANDLE        hThread,
  const CONTEXT *lpContext
);
```

- **[SetWindowsHookEx](https://docs.microsoft.com/en-us/windows/win32/api/winuser/nf-winuser-setwindowshookexa)**

设置一个hook函数，使其在某个特定事件触发时被调用。此函数普遍被用于击键记录器和间谍软件。此函数也可以轻易地将一个DLL程序装载到系统的所有GUI进程中。编译器又是会在编译的时候添加此函数。

```
HHOOK SetWindowsHookExA(
  int       idHook,
  HOOKPROC  lpfn,
  HINSTANCE hmod,
  DWORD     dwThreadId
);
```

- **SfcTerminatedWatcherThread**

用于禁用Windows文件保护功能并修改被保护的文件（在Microsoft Doc中未提供此函数的说明）。

- **[ShellExecute](https://docs.microsoft.com/en-us/windows/win32/api/shellapi/nf-shellapi-shellexecutea)**

用于执行另一个程序，例如创建某个新进程。

```
HINSTANCE ShellExecuteA(
  HWND   hwnd,
  LPCSTR lpOperation,
  LPCSTR lpFile,
  LPCSTR lpParameters,
  LPCSTR lpDirectory,
  INT    nShowCmd
);
```

- **[StartServiceCtrlDispatcher](https://docs.microsoft.com/en-us/windows/win32/api/winsvc/nf-winsvc-startservicectrldispatchera)**

将某个服务进程的主线程连接到服务控制管理器。任何以服务方式运行的进程必须在启动后的30秒内调用此函数。倘若在恶意程序中找到此函数，则表示此恶意程序以服务的方式运行。

```
BOOL StartServiceCtrlDispatcherA(
  const SERVICE_TABLE_ENTRYA *lpServiceStartTable
);
```

- **[SuspendThread](https://docs.microsoft.com/en-us/windows/win32/api/processthreadsapi/nf-processthreadsapi-suspendthread)**

停止某个正在运行的线程，并将其挂起。通常被用于代码注入技术中，使得某个线程被挂起，便于修改。

```
DWORD SuspendThread(
  HANDLE hThread
);
```

- **[system](https://www.tutorialspoint.com/c_standard_library/c_function_system.htm)**

C函数库中的函数，用于创建进程或者执行某条命令，在Windows操作系统中，此函数是CreateProcess的封装。

```
int system(const char *command)
```

- **Thread32First/Thread32Next**

[Thread32First](https://docs.microsoft.com/en-us/windows/win32/api/tlhelp32/nf-tlhelp32-thread32first)用于获取当前系统快照中任意进程的第一个线程信息。

[Thread32Next](https://docs.microsoft.com/en-us/windows/win32/api/tlhelp32/nf-tlhelp32-thread32next)用于获取当前系统快照中任意进程的下一个线程信息。

这两个函数通常被用于寻找可供注入的合适线程。

```
BOOL Thread32First(
  HANDLE          hSnapshot,
  LPTHREADENTRY32 lpte
);

BOOL Thread32Next(
  HANDLE          hSnapshot,
  LPTHREADENTRY32 lpte
);
```

- **[Toolhelp32ReadProcessMemory](https://docs.microsoft.com/en-us/windows/win32/api/tlhelp32/nf-tlhelp32-toolhelp32readprocessmemory)**

将其他进程的内存数据拷贝到应用支持的缓冲区中。

```
BOOL Toolhelp32ReadProcessMemory(
  DWORD   th32ProcessID,
  LPCVOID lpBaseAddress,
  LPVOID  lpBuffer,
  SIZE_T  cbRead,
  SIZE_T  *lpNumberOfBytesRead
);
```

- **[URLDownloadToFile](https://docs.microsoft.com/en-us/previous-versions/windows/internet-explorer/ie-developer/platform-apis/ms775123(v=vs.85))**

从某个网络上下载文件并存储到本地。此函数实现了下载器的所有功能。

```
HRESULT URLDownloadToFile(
             LPUNKNOWN            pCaller,
             LPCTSTR              szURL,
             LPCTSTR              szFileName,
  _Reserved_ DWORD                dwReserved,
             LPBINDSTATUSCALLBACK lpfnCB
);
```

- **[VirtualAllocEx](https://docs.microsoft.com/en-us/windows/win32/api/memoryapi/nf-memoryapi-virtualallocex)**

用于内存分配，可用于进程注入。

```
LPVOID VirtualAllocEx(
  HANDLE hProcess,
  LPVOID lpAddress,
  SIZE_T dwSize,
  DWORD  flAllocationType,
  DWORD  flProtect
);
```

- **[VIrtualProtectEx](https://docs.microsoft.com/en-us/windows/win32/api/memoryapi/nf-memoryapi-virtualprotectex)**

修改某块内存区域的保护机制，可将只读内存节改为可写内存节。

```
BOOL VirtualProtectEx(
  HANDLE hProcess,
  LPVOID lpAddress,
  SIZE_T dwSize,
  DWORD  flNewProtect,
  PDWORD lpflOldProtect
);
```

- **[WideCharToMultiByte](https://docs.microsoft.com/en-us/windows/win32/api/stringapiset/nf-stringapiset-widechartomultibyte)**

用于将一个Unicode类型字符串转化为一个ASCII类型字符串。

```
int WideCharToMultiByte(
  UINT                               CodePage,
  DWORD                              dwFlags,
  _In_NLS_string_(cchWideChar)LPCWCH lpWideCharStr,
  int                                cchWideChar,
  LPSTR                              lpMultiByteStr,
  int                                cbMultiByte,
  LPCCH                              lpDefaultChar,
  LPBOOL                             lpUsedDefaultChar
);
```

- **[WinExec](https://docs.microsoft.com/en-us/windows/win32/api/winbase/nf-winbase-winexec)**

用于执行某个指定程序。

```
UINT WinExec(
  LPCSTR lpCmdLine,
  UINT   uCmdShow
);
```

- **[WlxLoggedOnSAS](https://docs.microsoft.com/en-us/windows/win32/api/winwlx/nf-winwlx-wlxloggedonsas)**

身份认证模块，可用于图形化标识与认证模块（GINA）的劫持（Windows Server 2008 和 Windows Vista不再支持此函数）。

```
int WlxLoggedOnSAS(
  PVOID pWlxContext,
  DWORD dwSasType,
  PVOID pReserved
);
```

- **[Wow64DisableWow64FsRedirection](https://docs.microsoft.com/en-us/windows/win32/api/wow64apiset/nf-wow64apiset-wow64disablewow64fsredirection)**

禁用32位文件在64位操作系统中装在后发生的文件重定向机制。倘若一个32位应用程序在调用这个函数后向C:\Windows\System32写数据，那么它将会直接写到C:\Windows\System32，而不会重定向至C:\Windows\SysWOW64。

```
BOOL Wow64DisableWow64FsRedirection(
  PVOID *OldValue
);
```

- **[WriteProcessMemory](https://docs.microsoft.com/en-us/windows/win32/api/memoryapi/nf-memoryapi-writeprocessmemory)**

向指定进程的可写内存区域写入数据，常用于进程注入。

```
BOOL WriteProcessMemory(
  HANDLE  hProcess,
  LPVOID  lpBaseAddress,
  LPCVOID lpBuffer,
  SIZE_T  nSize,
  SIZE_T  *lpNumberOfBytesWritten
);
```

- **[WSAStartup](https://docs.microsoft.com/en-us/windows/win32/api/winsock/nf-winsock-wsastartup)**

用于初始化使用Winsock DLL的进程，此函数的调用位置可以用来定位网络相关的功能。

```
int WSAStartup(
  WORD      wVersionRequired,
  LPWSADATA lpWSAData
);
```