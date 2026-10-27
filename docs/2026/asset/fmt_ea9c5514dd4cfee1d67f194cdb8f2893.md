
# [原创]从mimikatz学习Windows安全之访问控制模型（一）

[TOC]

作者：Loong716@Amulab

## 0x00 前言

Mimikatz是法国安全研究员Benjamin Delpy开发的一款安全工具。渗透测试人员对mimikatz印象最深的肯定就是抓取Windows凭证，但作者对它的描述是“a tool I've made to learn C and make somes experiments with Windows security.”，其实它的功能不仅仅是抓取凭证，还包含了很多Windows安全相关的技术和知识

这里借用@daiker师傅的思维导图，mimikatz的模块大致可分为几个部分：

![](upload/attach/202108/931257_J2MR9RNSBUZQBHQ.png)

因此文章也会大致分为windows 访问控制模型，windows 凭据以及加解密，windows AD 安全，windows 进程以及服务，mimikatz 其他模块五个小系列。之前自己一直想分析mimikatz的相关功能，主要是出于以下原因：

+ mimikatz中有许多功能利用了Windows的一些机制和特性，以`changentlm`为例，其利用`MS-SAMR`协议修改用户的密码，我们再根据`MS-SAMR`或`RPC`进行知识延伸，肯定也有不少收获
+ mimikatz中涉及大量内存的操作，其中运用的内存Patch技术也被经常应用于一些安全机制的绕过（如绕过`AMSI`、`Credential Guard`等），于是自己想在分析过程中通过`windbg`学到一些调试的技巧
+ mimikatz在实战中被杀的很厉害，了解相应原理可以自己实现相应功能
+ 学习/练习C语言 :D

mimikatz中与Windows访问控制模型相关的有`privilege`、`token`、`sid`三个模块，其分别对应特权、访问令牌、安全标识符三个知识，本文主要分析`token`模块，并简要介绍Windows访问控制模型

由于mimikatz代码逻辑较为复杂，涉及大量回调，因此文中代码都是经过简化的。文章可能也会有一些技术上或者逻辑上的错误，还请师傅们指正

## 0x01 访问控制模型简介

Windows访问控制模型有两个基本组成部分：

+ 访问令牌`Access Token`：包含有关登录用户的信息
+ 安全描述符`Security Descriptor`：包含用于保护安全对象的安全信息

### 1. 访问令牌 Access Token

访问令牌`Access Token`被用来描述一个进程或线程的安全上下文，用户每次登录成功后，系统会为其创建访问令牌，该用户的所有进程也将拥有此访问令牌的副本

当线程与安全对象进行交互或尝试执行需要特权的系统任务时，系统使用访问令牌来标识用户。使用`windbg`查看进程的token，其包含信息如下图所示：

![](upload/attach/202108/931257_5ZCXRKBECRD34BJ.png)

![](upload/attach/202108/931257_QGKRXX6U8DJPQAU.png)

![](upload/attach/202108/931257_FTQB52SQ9JWNFBD.png)

### 2. 安全描述符 Security Descriptor

安全描述符`Security Descriptor`包含与安全对象有关的安全信息，这些信息规定了哪些用户/组可以对这个对象执行哪些操作，安全描述符主要由以下部分构成：

+ 所有者的`SID`
+ 组`SID`
+ 自主访问控制列表`DACL`，规定哪些用户/组可以对这个对象执行哪些操作
+ 系统访问控制列表`SACL`，规定哪些用户/组的哪些操作将被记录到安全审计日志中

在`windbg`中查看一个安全对象的安全描述符，可以清晰的看到安全描述符的组成：

![](upload/attach/202108/931257_ZH8UTVTSHDG2X57.png)

可以看到该安全描述符的`DACL`中有三条`ACE`，`ACE`的类型都是`ACCESS_ALLOWED_ACE_TYPE`，`Mask`是权限掩码，用来指定对应的权限。以第一条`ACE`为例，其表示允许`SID`为`S-1-5-32-544`的对象能够对该安全对象做`0x001fffff`对应的操作

### 3. 权限检查的过程

当某个线程尝试访问一个安全对象时，系统根据安全对象的`ACE`对照线程的访问令牌来判断该线程是否能够对该安全对象进行访问。通常，系统使用请求访问的线程的主访问令牌。但是，如果线程正在模拟其他用户，则系统会使用线程的模拟令牌

此时将在该安全对象的`DACL`中按顺序检查`ACE`，直到发生以下事件：

+ 某一条拒绝类型的`ACE`显式拒绝令牌中某个受信者的所有访问权限
+ 一条或多条允许类型的`ACE`允许令牌中列出的受信者的所有访问权限
+ 检查完所有`ACE`但没有一个权限显式允许，那么系统会隐式拒绝该访问

我们以微软文档中的图片为例，描述一下整个过程：

![](upload/attach/202108/931257_UDASW2HWF5SDB5A.png)

1. 线程A请求访问安全对象，系统读取`ACE1`，发现拒绝`Andrew`用户的所有访问权限，而线程A的访问令牌是`Andrew`，因此拒绝访问，并不再检查`ACE2`、`ACE3`
2. 线程A请求访问，系统按顺序读取`ACE`，`ACE1`不适用，读取到`ACE2`发现适用，再读取到`ACE3`也适用，因此最终该用户拥有对该安全对象的读、写、执行权限

## 0x02 Mimikatz的Token模块

Mimikatz的`token`模块共有5个功能：

+ `token::whoami`：列出当前进程/线程的token信息
+ `token::list`：列出当前系统中存在的token
+ `token::elevate`：窃取其他用户的token
+ `token::run`：利用某用户权限运行指定程序
+ `token::revert`：恢复为原来的token

### 1. token::whoami

该功能用于列出当前进程/线程的token信息

![](upload/attach/202108/931257_QE8NX5FNPEDTRYH.png)

只有一个可选参数`/full`，当指定该参数时会打印出当前token的组信息和特权信息：

![](upload/attach/202108/931257_MGDCKFXGYVTR4Y5.png)

该功能的原理大致如下：

1. 通过`OpenProcess()`获取当前进程/线程的句柄
2. 调用`GetTokenInformation()`获取token的各种信息并输出

其核心为调用`GetTokenInformation()`来获取token的各种信息，我们先来看这个`API`定义

``` cpp
BOOL GetTokenInformation(
HANDLE TokenHandle,
TOKEN_INFORMATION_CLASS TokenInformationClass,
LPVOID TokenInformation,
DWORD TokenInformationLength,
PDWORD ReturnLength
);
```

其中第二个参数是一个`TOKEN_INFORMATION_CLASS`枚举类型，我们可以通过指定它的值来获取token指定的信息

``` cpp
typedef enum _TOKEN_INFORMATION_CLASS {
TokenUser,
TokenGroups,
TokenPrivileges,
TokenOwner,
TokenPrimaryGroup,
TokenDefaultDacl,
TokenSource,
...
} TOKEN_INFORMATION_CLASS, *PTOKEN_INFORMATION_CLASS;
```

例如获取token的`SessionID`并输出，可以使用以下代码：

``` cpp
if (!GetTokenInformation(hToken, TokenSessionId, &sessionId, sizeof(TokenSessionId), &dwSize))
{
wprintf(L"[!] GetTokenInformation error: %u\n", GetLastError());
}
wprintf(L"\t%-21s: %u\n", L"Session ID", sessionId);
```

### 2. token::list

该功能是获取当前系统中所有的token，注意使用前需要先获取`SeDebugPrivilege`，否则列出的token不全

![](upload/attach/202108/931257_SASEUG7XGWFGUVA.png)

该功能原理大致如下：

1. 调用`NtQuerySystemInformation()`获取系统进程信息（如进程`PID`等）
2. 循环遍历所有进程的`PID`，使用`token::whoami`功能中的方法对指定token信息进行输出

`NtQuerySystemInformation()`用来检索指定的系统信息：

``` cpp
__kernel_entry NTSTATUS NtQuerySystemInformation(
SYSTEM_INFORMATION_CLASS SystemInformationClass,
PVOID SystemInformation,
ULONG SystemInformationLength,
PULONG ReturnLength
);
```

其第一个参数是一个`SYSTEM_INFORMATION_CLASS`枚举类型，我们同样可以指定不同参数来获取不同的系统信息

![](upload/attach/202108/931257_JGAJGKDCNJHQYDY.png)

以获取系统进程名和`PID`为例，代码如下：

``` cpp
PSYSTEM_PROCESS_INFORMATION pProcessInfo = NULL;
DWORD flag = TRUE;
pProcessInfo = (PSYSTEM_PROCESS_INFORMATION)malloc(dwSize);
ntReturn = NtQuerySystemInformation(SystemProcessInformation, pProcessInfo, dwSize, &dwSize);
while (ntReturn == STATUS_INFO_LENGTH_MISMATCH) {
free(pProcessInfo);
pProcessInfo = (PSYSTEM_PROCESS_INFORMATION)malloc(dwSize);
ntReturn = NtQuerySystemInformation(SystemProcessInformation, pProcessInfo, dwSize, &dwSize);
}
while (flag)
{
if (pProcessInfo->NextEntryOffset == 0)
flag = FALSE;
wprintf(L"%-15d", (DWORD)pProcessInfo->UniqueProcessId);
wprintf(L"%-50s", (wchar_t*)pProcessInfo->ImageName.Buffer);
pProcessInfo = (PSYSTEM_PROCESS_INFORMATION)((BYTE*)pProcessInfo + pProcessInfo->NextEntryOffset);
}
```

PS：按照该思路，理论上利用`CreateToolhelp32Snapshot()` + `Process32First()`遍历进程`PID`也可以实现该功能

### 3. token::elevate

该模块用于窃取指定用户的token，共有7个可选参数，这些参数主要用来指定要窃取的token，如果不指定参数则默认窃取`NT AUTHORITY\SYSTEM`的token

+ `/id`：指定目标token的`TokenID`
+ `/domainadmin`：窃取域管的token
+ `/enterpriseadmin`：窃取企业管理员的token
+ `/admin`：窃取本地管理员的token
+ `/localservice`：窃取`Local Service`权限的token
+ `/networkservice`：窃取`Network Service`权限的token
+ `/system`：窃取`SYSTEM`权限的token

假设我们现在在目标机器上发现的域管权限的token

![](upload/attach/202108/931257_ZSUF45FJ3PMMUK4.png)

我们可以指定目标`TokenID`，或者使用`/domainadmin`来窃取域管的token，执行成功后可以看到当前线程已经拥有域管的模拟令牌：

![](upload/attach/202108/931257_MEYFECJVAUMYRV9.png)

然后我们就可以在当前mimikatz上下文中使用域管身份执行操作了，如`DCSync`

![](upload/attach/202108/931257_MWFV3NMYXJNQRH4.png)

该功能大致过程如下：

1. 通过`OpenProcess()`获取当前进程/线程的句柄
2. 调用`OpenProcessToken()`打开与进程相关的token句柄
3. 使用`DuplicateTokenEx()`使用目标进程token创建一个新的模拟token
4. 调用`SetThreadToken()`设置当前线程的token为上一步创建的新的模拟token

由于窃取token是`Access Token`利用的重点，该过程放在本文后面分析

### 4. token::run

该功能是使用指定的token来运行程序，也可以使用`token::elevate`中的几个参数来指定运行程序的token，除此之外还有一个参数：

+ `/process`：指定要运行的程序，默认值为`whoami.exe`

![](upload/attach/202108/931257_DTU8HHW3R8YP4HB.png)

其原理前三步与`token::elevate`大致相同，区别在于使用`DuplicateTokenEx()`窃取token后，该功能使用`CreateProcessAsUser()`来使用新的`primary token`创建一个进程

``` cpp
BOOL CreateProcessAsUserA(
HANDLE hToken,
LPCSTR lpApplicationName,
LPSTR lpCommandLine,
LPSECURITY_ATTRIBUTES lpProcessAttributes,
LPSECURITY_ATTRIBUTES lpThreadAttributes,
BOOL bInheritHandles,
DWORD dwCreationFlags,
LPVOID lpEnvironment,
LPCSTR lpCurrentDirectory,
LPSTARTUPINFOA lpStartupInfo,
LPPROCESS_INFORMATION lpProcessInformation
);
```

创建进程后，利用匿名管道做进程间通信，将新创建进程的标准输出写入到匿名管道的`write`端，从管道`read`端读取数据进行回显（在`webshell`等非交互场景下很有用）
