

# 【牛人们勿入】TLS 资料收集

> 原文：[`www.pediy.com/kssd/pediy10/81543.html`](https://www.pediy.com/kssd/pediy10/81543.html)

虽然说是牛人们 N 年前手中的玩物，我却是最近才知道有这回事![](img/e71d9210a215d5b66755d038631e0c83.jpg "恐怖!")   
  资料也不多，昨天 BAIDU+论坛搜索弄了很久，有点搜获，不敢独享，发出来和大家共享。  
\---------------------------------------------------------------------  
  话说很久很久以前我们就知道，main,WinMain,DllMain 等在一般情况下并不是应用程序或动态链接库执行的起点，所以我们经常在 crt0 上进行一些 OOXX 的事情。  
 结果现在发现，在 crt0 之前，我们还能做一些事情，这个 OOXX 的事情就是所谓的 Thread Local Storage 技术带来的 tls 回调函数？？！！  
 一个让我觉得很牛逼的东西。当然鸡肋还是有一点的，那就是非常明显地暴露了我们要执行特定意图代码的位置。  
 不管怎么说，我们小菜还是来学习下吧！  
PS.  
如果用 VC 写代码，在 TLS 回调函数中不要使用 C 库的函数，因为此时 MSVCRTXXX.DLL 还没加载，至少我的 VS2K3 是这样的。另外如果要执行可能出现异常的代码，一定要 try/catch。不然操作系统不会报错，而是直接跑到 OEP 去了。  
再 PS.  
牛人们请自觉，这个是菜鸟交流的帖子。。。。你们就不要来 OOXX 了。。。。  
\---------------------------------------------------------------------  
资料链接：  
http://www.pediy.com/bbshtml/bbs7/pediy7-667.htm  
http://bbs.pediy.com/showthread.php?p=80239  
http://bbs.pediy.com/showthread.php?threadid=31875  
http://www.codeproject.com/KB/threads/tls.aspx  
http://bbs.pediy.com/showthread.php?t=63237&highlight=tls  
http://bbs.pediy.com/showthread.php?threadid=17828   
http://bbs.pediy.com/showthread.php?threadid=17839   
http://bbs.pediy.com/showthread.php?threadid=12532   
http://bbs.pediy.com/showthread.php?threadid=18144  
\---------------------------------------------------------------------  
TLS 表结构  
typedef struct \_IMAGE\_TLS\_DIRECTORY32 {  
    DWORD   StartAddressOfRawData;  
    DWORD   EndAddressOfRawData;  
    DWORD   AddressOfIndex;             // PDWORD  
    DWORD   AddressOfCallBacks;         // PIMAGE\_TLS\_CALLBACK \*  
    DWORD   SizeOfZeroFill;  
    DWORD   Characteristics;  
} IMAGE\_TLS\_DIRECTORY32;  
\---------------------------------------------------------------------  
表里面的其他项都可以填 0，但是要求 AddressOfIndex 指向一个数组，数组存放所有 TLS 的回调函数的地址，以 NULL 结束数组  
\---------------------------------------------------------------------  
TLS 回调函数的函数原型  
extern "C" void NTAPI TLS\_callback ( PVOID DllHandle, DWORD Reason, PVOID Reserved );  
其中 Readson 的意义和 DllMain()的差不多  
\---------------------------------------------------------------------  
一个简单的使用 TLS 的实例  
来源 http://hi.baidu.com/zhanglinguo11/blog/item/5d7c9623c2b2be549922ed41.html  

代码:

```
// TLS\_CallBack\_test.cpp : Defines the entry point for the console application.
//

#include "stdafx.h"
#include <windows.h>
#include <winnt.h>
//下面这行告诉链接器在 PE 文件中要创建 TLS 目录
#pragma comment(linker, "/INCLUDE:\_\_tls\_used")
/\*这是 PIMAGE\_TLS\_CALLBACK()函数的原型，其中第一个和第三个参数保留，第二个参数决定函数在那种情况下\*/
void NTAPI my\_tls\_callback(PVOID h, DWORD reason, PVOID pv)
{
/\*一共有四个选项 DLL\_PROCESS\_ATTACH、DLL\_THREAD\_ATTACH、DLL\_THREAD\_DETACH 和 DLL\_PROCESS\_DETACH。详见微软发布的《Microsoft Portable Executable and Common Object File Format Specification v8》\*/
//仅在进程初始化创建主线程时执行的代码
if( reason == DLL\_PROCESS\_ATTACH ){
   MessageBox(NULL,L"hi,this is tls callback",L"title",MB\_OK);
}
return;
}
/\*下面这段是创建一个 tls 段
".CRT$XLB"的含义是：
.CRT 表明是使用 C RunTime 机制
$后面的 XLB 中
X 表示随机的标识
L 表示是 TLS callback section
B 可以被换成 B 到 Y 的任意一个字母，但是不能使用".CRT$XLA"和".CRT$XLZ"
因为".CRT$XLA"和".CRT$XLZ"是用于 tlssup.obj 的
\*/
#pragma data\_seg(".CRT$XLB")
/\*如果要定义多个 TLS\_CallBack 函数，可以把下面这句写成：
PIMAGE\_TLS\_CALLBACK p\_thread\_callback \[\] = {tls\_callback\_A, tls\_callback\_B, tls\_callback\_C,0};
其中 tls\_callback\_B 和 tls\_callback\_C 应该是你定义好的其他 TLS\_callBack 函数
\*/
PIMAGE\_TLS\_CALLBACK p\_thread\_callback = my\_tls\_callback;
#pragma data\_seg()

int main(void)
{
MessageBox(NULL,L"hi,this is main()",L"title",MB\_OK);
return 0;
}

```

\---------------------------------------------------------------------  
祝我们大家一起进步！