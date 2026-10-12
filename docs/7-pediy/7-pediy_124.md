
# 【样章 1】《加密与解密(第三版)》--18.2.4 DLL 劫持技术（内存补丁技术）

> 原文：[`www.pediy.com/kssd/pediy10/60849.html`](https://www.pediy.com/kssd/pediy10/60849.html)

## 18.2.4 DLL 劫持技术

说明

我是从 `CoDe_Inject` 的帮助下才了解这个 DLL 劫持技术（或称 `HOOK`），利用这个制作内存补丁非常的好用。

`Yonsm` 在几年前，写了个工具 `AheadLib`，可以很方便地生成各类 DLL 头文件，见本帖 2 楼。

**这种补丁方法，适合制作被 `ASProtect`、`Armadillo`、`Themida` 等各类强壳保护软件的补丁。**

当一个可执行文件运行时，Windows 加载器将可执行模块映射到进程的地址空间中，加载器分析可执行模块的输入表，并设法找出任何需要的 DLL，并将它们映射到进程的地址空间中。

由于输入表中只包含 DLL 名而没有它的路径名，因此加载程序必须在磁盘上搜索 DLL 文件。首先会尝试从当前程序所在的目录加载 DLL，如果没找到，则在 Windows 系统目录查找，最后是在环境变量中列出的各个目录下查找。利用这个特点，先伪造一个系统同名的 DLL，提供同样的输出表，每个输出函数转向真正的系统 DLL。程序调用系统 DLL 时会先调用当前目录下伪造的 DLL，完成相关功能后，再跳到系统 DLL 同名函数里执行，如图 18.4。这个过程用个形象的词来描述就是系统 DLL 被劫持（`hijack`）了。

![](img/26e273eed5bf2184f6dceecd36bdb064.jpg)

利用这种方法取得控制权后，可以对主程序进行补丁。此种方法只对除 `kernel32.dll`、`ntdll.dll` 等核心系统库以外的 DLL 有效，如网络应用程序的 `ws2_32.dll`，游戏程序中的 `d3d8.dll`，还有大部分应用程序都调用的 `lpk.dll`，这些 DLL 都可被劫持。

利用 5.6.2 章提供的 `CrackMeNet.exe` 来演示一下如何利用劫持技术制作补丁，目标文件用 `Themida` `v1.9.2.0` 加壳保护。

### 1．补丁地址

去除这个 `CrackMe` 网络验证方法参考第五章，将相关补丁代码存放到函数 `PatchProcess()` 里。例如将 `401496h` 改成：

代码:

```
00401496 EB 29 jmp short 004014C1
```

补丁编程实现就是：

代码:

```
unsigned char p401496[2] = {0xEB, 0x29};
WriteProcessMemory(hProcess,(LPVOID)0x401496, p401496, 2, NULL);
```

`p401496` 这个数组的数据格式，可以用 `OllyDBG` 插件获得，或十六进制工具转换。例如 `Hex Workshop` 打开文件，执行菜单 `Edit/Copy As/Source` 即可得到相应的代码格式。

### 2．构建输出函数

查看实例 `CrackMeNet.exe` 输入表，会发现名称为 `ws2_32.dll` 的 DLL，因此构造一个同名的 DLL 来完成补丁任务。伪造的 `ws2_32.dll` 有着真实 `ws2_32.dll` 一样的输出函数，完整源码见光盘。实现时，可以利用 DLL 模块中的函数转发器来实现这个目标，其会将对一个函数的调用转至另一个 DLL 中的另一个函数。可以这样使用一个 `pragma` 指令：

代码:

```
#pragma comment(linker, "/EXPORT:SomeFunc=DllWork.someOtherFunc")
```

这个 `pragma` 告诉链接程序，被编译的 DLL 应该输出一个名叫 `SomeFunc` 的函数。但是 `SomeFunc` 函数的实现实际上位于另一个名叫 `SomeOtherFunc` 的函数中，该函数包含在称为 `DllWork.dll` 的模块中。

如要达到劫持 DLL 的目的，生成的 DLl 输出函数必须与目标 DLL 输出函数名一样，本例可以这样构造 `pragma` 指令：

代码:

```
#pragma comment(linker, "/EXPORT:WSAStartup=_MemCode_WSAStartup,@115")
```

编译后的 DLL，会有与 `ws2_32.dll` 同名的一个输出函数 `WSAStartup`，实际操作时，必须为想要转发的每个函数创建一个单独的 `pragma` 代码行，读者可以写一个工具或用其他办法，将 `ws2_32.dll` 输出函数转换成相应的 `pragma` 指令。

当应用程序调用伪装 `ws2_32.dll` 的输出函数时，必须将其转到系统 `ws2_32.dll` 中去，这部分的代码自己实现。例如 `WSAStartup` 输出函数如下构造：

代码:

```
ALCDECL MemCode_WSAStartup(void)
{
GetAddress("WSAStartup");
__asm JMP EAX;//转到系统 ws2_32.dll 的 WSAStartup 输出函数
}
```

其中 `GetAddress()` 函数的代码如下：

代码:

```
// MemCode 命名空间
namespace MemCode
{
HMODULE m_hModule = NULL; //原始模块句柄
DWORD m_dwReturn[500] = {0}; //原始函数返回地址
// 加载原始模块
inline BOOL WINAPI Load()
{
TCHAR tzPath[MAX_PATH]={0};
TCHAR tzTemp[MAX_PATH]={0};
GetSystemDirectory(tzPath, sizeof(tzPath));
strcat(tzPath,"\\\\ws2_32.dll");
m_hModule = LoadLibrary(tzPath);//加载系统系统目录下 ws2_32.dll
if (m_hModule == NULL)
{
wsprintf(tzTemp, TEXT("无法加载 %s，程序无法正常运行。"), tzPath);
MessageBox(NULL, tzTemp, TEXT("MemCode"), MB_ICONSTOP);
}
return (m_hModule != NULL); 
}
 
// 释放原始模块
inline VOID WINAPI Free()
{
if (m_hModule)
FreeLibrary(m_hModule);
}
// 获取原始函数地址
FARPROC WINAPI GetAddress(PCSTR pszProcName)
{
FARPROC fpAddress;
TCHAR szProcName[16]={0};
TCHAR tzTemp[MAX_PATH]={0};
 
if (m_hModule == NULL)
{
if (Load() == FALSE)
ExitProcess(-1);
}
 
fpAddress = GetProcAddress(m_hModule, pszProcName);
if (fpAddress == NULL)
{
if (HIWORD(pszProcName) == 0)
{
wsprintf(szProcName, "%d", pszProcName);
pszProcName = szProcName;
}
wsprintf(tzTemp, TEXT("无法找到函数 %hs，程序无法正常运行。"), pszProcName);
MessageBox(NULL, tzTemp, TEXT("MemCode"), MB_ICONSTOP);
ExitProcess(-2);
}
return fpAddress;
}
}
using namespace MemCode;
```

编译后，用 `LordPE` 查看伪造的 `ws2_32.dll` 输出函数，和真实 `ws2_32.dll` 完全一样，如图 18.5 所示。

![](img/65468bd4030e1b0858be5e8088692698.jpg)

查看伪造的 `ws2_32.dll` 中任意一个输出函数，例如 `WSACleanup`：

代码:

```
.text:10001CC0 ; int __stdcall WSACleanup()
.text:10001CC0 WSACleanup proc near
.text:10001CC0 push offset aWsacleanup ;"WSACleanup"
.text:10001CC5 call sub_10001000 ;GetAddress(WSACleanup)
.text:10001CCA jmp eax
.text:10001CCA WSACleanup endp
```

会发现输出函数 `WSACleanup()` 首先调用 `GetAddress(WSACleanup)`，获得真实 `ws2_32.dll` 中 `WSACleanup` 的地址，然后跳过去执行，也就是说 `ws2_32.dll` 各输出函数被 `HOOK` 了。

### 3．劫持输出函数

`ws2_32.dll` 有许多输出函数，经分析，程序发包或接包时，`WSAStartup` 输出函数调用的比较早，因此在这个输出函数放上补丁的代码。代码如下：

代码:

```
ALCDECL MemCode_WSAStartup(void)
{
hijack();
GetAddress("WSAStartup");
__asm JMP EAX; 
}
hijack()函数主要是判断是不是目标程序，如是就调用 PatchProcess()进行补丁。
void hijack()
{
if (isTarget(GetCurrentProcess())) //判断主程序是不是目标程序，是则补丁之
{
PatchProcess(GetCurrentProcess());
}
}
```

伪造的 `ws2_32.dll` 制作好后，放到程序当前目录下，这样当原程序调用 `WSASTartup` 函数时就调用了伪造的 `ws2_32.dll` 的 `WSASTartup` 函数，此时 `hijack()` 函数负责核对目标程序校验，并将相关数据补丁好，处理完毕后，转到系统目录下的 `ws2_32.dll` 执行。

这种补丁技术，对加壳保护的软件很有效，选择挂接的函数最好是在壳中没有被调用，当挂接函数被执行时，相关的代码己被解压，可以直接补丁了。有些情况下，必须用计数器统计挂接的函数的调用次数来接近 `OEP`。此方法巧妙地绕过了壳的复杂检测，很适合加壳程序的补丁制作。

一些木马或病毒也会利用 DLL 劫持技术搞破坏，因此当在应用程序目录下发现系统一些 DLL 文件存在时，应引起注意。

看雪软件安全

`http://www.pediy.com`

2008.3.8

上传的附件

| ![](img/c8c74a380a3dea7fa2460187d80ee5e0.jpg) | `ws2_32.rar` |
| :--- | :--- |
| ![](img/c8c74a380a3dea7fa2460187d80ee5e0.jpg) | `演示实例.rar` [此附件论坛上下载：[`bbs.pediy.com/showthread.php?t=60849`](http://bbs.pediy.com/showthread.php?t=60849)] |
| ![](img/c8c74a380a3dea7fa2460187d80ee5e0.jpg) | `lpk.rar` [解压密码：`PEDIY`] |
