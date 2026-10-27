
```cpp
void TestHeavensGate() {
uint64_t kernel32 = GetKernel32();
uint64_t user32 = LoadLibrary64("user32.dll");
uint64_t CreateFile64 = GetProcAddress64(kernel32, "CreateFileA");
uint64_t WriteFile64 = GetProcAddress64(kernel32, "WriteFile");
uint64_t ReadFile64 = GetProcAddress64(kernel32, "ReadFile");
uint64_t CloseHandle64 = GetProcAddress64(kernel32, "CloseHandle");
char path[MAX_PATH];
char hacked[] = "Hacked by 34r7hm4n";
uint64_t hFile;
char buffer[100] = { 0 };
GetCurrentDirectoryA(MAX_PATH, path);
lstrcatA(path, "\\test.txt");
hFile = X64Call(CreateFile64, 7, (uint64_t)path, (uint64_t)GENERIC_WRITE, (uint64_t)NULL, (uint64_t)NULL, (uint64_t)CREATE_NEW, (uint64_t)FILE_ATTRIBUTE_NORMAL, (uint64_t)NULL);
X64Call(CloseHandle64, 1, hFile);
hFile = X64Call(CreateFile64, 7, (uint64_t)path, (uint64_t)GENERIC_WRITE, (uint64_t)NULL, (uint64_t)NULL, (uint64_t)OPEN_EXISTING, (uint64_t)FILE_ATTRIBUTE_NORMAL, (uint64_t)NULL);
X64Call(WriteFile64, 5, (uint64_t)hFile, (uint64_t)hacked, (uint64_t)lstrlenA(hacked), (uint64_t)NULL, (uint64_t)NULL);
X64Call(CloseHandle64, 1, hFile);
hFile = X64Call(CreateFile64, 7, (uint64_t)path, (uint64_t)GENERIC_READ, (uint64_t)NULL, (uint64_t)NULL, (uint64_t)OPEN_EXISTING, (uint64_t)FILE_ATTRIBUTE_NORMAL, (uint64_t)NULL);
X64Call(ReadFile64, 5, (uint64_t)hFile, (uint64_t)buffer, (uint64_t)sizeof(buffer), (uint64_t)NULL, (uint64_t)NULL);
X64Call(CloseHandle64, 1, hFile);
printf("%s\n", buffer);
system("pause");
}
```

没有检测到文件释放：

![image-20211106145630433](upload/attach/202111/910514_64DRCRG9WUEP4EW.png)

## 魔盾测试

直接崩了，无语：

![image-20211106150139913](upload/attach/202111/910514_KXX3R45HBBRZ2E9.png)

## 火绒剑测试

最终没能逃出火绒剑的魔爪：

![image-20211106150000934](upload/attach/202111/910514_KWFRWTHR8UT8ATF.png)

# 0x03. 玄学Bug

目前还有一些玄学Bug，原因不明：

1. **只能生成后双击运行**。在VS中运行或在`cmd`中运行会导致无法加载`kernel32.dll`，`LdrLoadDll("kernel32.dll")`返回`0xC0000142` (`STATUS_DLL_INIT_FAILED`)
2. **在DLL中无法加载`user32.dll`**。至少在我的环境下，`LoadLibrary64("user32.dll")`直接导致整个程序崩溃

最后，由于我在Windows和恶意代码这块还是新手，难免有理解不当的地方，如果文章内容有什么问题欢迎各位师傅指正！

# 0x04. 参考资料

1. Knockin’ on Heaven’s Gate – Dynamic Processor Mode Switching by george_nicolaou
2. Mixing x86 with x64 code by ReWolf
3. WoW64 internals by wbenny
4. Heavens-Gate-2.0 by sdadas190
5. 天堂之门技术 by Tardis
6. Rebuild The Heaven's Gate: from 32-bit Hell back to 64-bit Wonderland by Sheng-Hao Ma
7. 通过PEB结构遍历进程模块
8. PE基础2-导出表-导入表

天堂之门 (Heaven's Gate) 是一种在32位WoW64进程中执行64位代码，以及直接调用64位WIN32 API函数的技术。从安全角度看，天堂之门可以作为一种软件保护技术，用于防止静态分析以及跨进程的API Hook；从恶意代码角度看，该技术可以绕过沙盒对WIN32 API调用的检测。

本文介绍了天堂之门技术的原理及C语言实现。

# 0x00. 原理分析

天堂之门技术的最早应用已经不可考究，我找到的最早的一篇详解天堂之门的文章是2012年的Knockin’ on Heaven’s Gate – Dynamic Processor Mode Switching，内容非常详细，目前能找到的有关天堂之门的文章大多都引用了这篇。比较新的一个是Rebuild The Heaven's Gate: from 32-bit Hell back to 64-bit Wonderland，貌似是一个台湾人（《惡意程式前線戰術指南》作者馬聖豪）的讲座PPT，比较简短。

## 什么是WoW64？

简单来说WoW64是Windows x64提供的一种兼容机制，可以认为WoW64是64位Windows系统创建的一个32位的模拟环境，使得32位可执行程序能够在64位的操作系统上正常运行。

推荐一篇讲解WoW64的深度好文WoW64 internals，这里就不再赘述了。

## 32位进程的API调用过程（WoW64）

下图（来自Rebuild The Heaven's Gate: from 32-bit Hell back to 64-bit Wonderland）展示了正常情况下32位进程通过WoW64机制调用WIN32 API的过程。以`ZwOpenProcess`函数的调用为例：

1. `a.exe`首先调用32位`ntdll.dll`（以下简称`ntdll32`）中的`ZwOpenProcess`函数
2. `ntdll32`调用`wow64cpu.dll`中的`X86SwitchTo64BitMode`，顾名思义，调用该函数后进程从32位模式切换到64位模式
3. 由`wow64.dll`将32位的系统调用转化为64位
4. 调用64位`ntdll.dll`的`ZwOpenProcess`函数
5. 切换到内核态（Ring0）执行系统调用

![image-20211106105559585](https://bbs.kanxue.com/upload/attach/202111/910514_JB5CY6FJ4ZMMUJZ.png)

## 32位进程的API调用过程（天堂之门）

下图展示了通过天堂之门技术调用WIN32 API的过程。这里我们通过一些操作绕过了WoW64机制，手动切换到64位模式并调用64位下的`ZwOpenProcess`函数，大致流程如下（和图中不太一样）：

1. 将`cs`段寄存器设为`0x33`，切换到64位模式
2. 从`gs:0x60`读取64位PEB
3. 从64位PEB中定位64位`ntdll`基址
4. 遍历`ntdll64`导出表，读取`ZwOpenProcess`函数地址
5. 构造64位函数调用

如果需要调用的是`ntdll`之外的函数，以`kernel32.dll`中的`CreateFile`函数为例，还需要：

1. 遍历`ntdll64`导出表，读取`LdrLoadDll`函数地址
2. 调用`LdrLoadDll("kernel32.dll")`加载64位`kernel32.dll`
3. 从64位的`kernel32`中读取`GetProcAddress`等函数，获取`CreateFile`函数地址
4. 调用`CreateFile`函数

![image-20211106111544450](https://bbs.kanxue.com/upload/attach/202111/910514_SBBBUJSMN9DJFZ9.png)

从上述过程中我们可以发现通过天堂之门的API调用并没有调用`ntdll32`中的函数，而目前大多数沙箱在检测32位程序时仅仅会对32位函数进行Hook，通过天堂之门，我们成功绕过了沙箱的API检测：

![image-20211106114019275](https://bbs.kanxue.com/upload/attach/202111/910514_QJZA4VE5KQ288GK.png)

# 0x01. 代码实现

GitHub仓库：`bluesadi/Heavens-Gate`

实现天堂之门大概需要实现到以下几个函数：

1. `memcpy64`：在64位地址之间拷贝数据
2. `GetPEB64`：获取64位的PEB地址
3. `GetModuleHandle64`：获取64位的模块基址
4. `GetProcAddress64`：获取64位模块中的函数地址
5. `X64Call`：调用64位函数
6. `MakeUTFStr`：构造`UNICODE_STRING`结构体
7. `GetKernel32`：加载64位`kernel32.dll`及其依赖`kernelbase.dll`
8. `LoadLibrary64`：在加载`kernel32.dll`后用于加载`user32.dll`
