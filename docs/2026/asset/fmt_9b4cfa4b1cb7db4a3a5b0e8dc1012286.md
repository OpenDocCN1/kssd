
`lPararm`：指定一个传递给回调函数的应用程序定义值。

回调函数原型

`BOOL CALLBACK EnumWindowsProc(HWND *hwnd*,LPARAM *lParam*);`

参数：

`hwnd`：顶层窗口的句柄

`lparam`：应用程序定义的一个值(即`EnumWindows`中`lParam`)

### 10. PostMessage

用于将一条消息放入到消息队列中。消息队列里的消息通过调用`GetMessage`和`PeekMessage`取得。

该函数将一个消息放入（寄送）到与指定窗口创建的线程相联系消息队列里，不等待线程处理消息就返回，是异步消息模式。消息队列里的消息通过调用`GetMessage`和`PeekMessage`取得。

函数原型：

`BOOL WINAPI PostMessage(HWND hWnd,UINT Msg,WPARAM wParam,LPARAM lParam);`

参数：

`hWnd`：其窗口程序接收消息的窗口的句柄。可取有特定含义的两个值：

* `HWND_BROADCAST`：消息被寄送到系统的所有顶层窗口，包括无效或不可见的非自身拥有的窗口、 被覆盖的窗口和弹出式窗口。消息不被寄送到子窗口
* `NULL`：此函数的操作和调用参数`dwThread`设置为当前线程的标识符`PostThreadMessage`函数一样

`Msg`：指定被寄送的消息。

`wParam`：指定附加的消息特定的信息。

`LParam`：指定附加的消息特定的信息。

返回值：

如果函数调用成功，返回非零，否则函数调用返回值为零

## (二)：源码分析

```c
#include "stdio.h"
#include "windows.h"
#include "shlobj.h"
#include "Wininet.h"
#include "tchar.h"
#pragma comment(lib, "Wininet.lib")
#define DEF_BUF_SIZE (4096)
#define DEF_URL L"http://www.google.com/index.html"
#define DEF_INDEX_FILE L"index.html"
HWND g_hWnd = NULL;
#ifdef __cplusplus
extern "C" {
#endif
__declspec(dllexport) void dummy() //设置这个函数只是为了后面填充IID的时候满足形式上的完整性
{
return;
}
#ifdef __cplusplus
}
#endif
BOOL DownloadURL(LPCTSTR szURL, LPCTSTR szFile)
{
BOOL bRet = FALSE; //判断函数是否正确被执行
HINTERNET hInternet = NULL, hURL = NULL; //InternetOpen和InternetOpenUrl的返回值
BYTE pBuf[DEF_BUF_SIZE] = { 0, }; //缓冲区
DWORD dwBytesRead = 0; //实际读取了多少字节量
FILE *pFile = NULL; //文件指针
errno_t err = 0; //errno_t是_tfopen_s的返回值类型
hInternet = InternetOpen(L"ReverseCore",
INTERNET_OPEN_TYPE_PRECONFIG,
NULL, //使用InternetOpen函数创建一个HINTERNET根句柄，从而使网络连接生效
NULL,
0);
if (NULL == hInternet) //判断是否返回了正确的句柄
{
OutputDebugString(L"InternetOpen() failed!");
return FALSE;
}
hURL = InternetOpenUrl(hInternet, //通过一个HTTP网址打开资源
szURL,
NULL,
0,
INTERNET_FLAG_RELOAD,
0);
if (NULL == hURL) //判断是否返回了正确的句柄
{
OutputDebugString(L"InternetOpenUrl() failed!");
goto _DownloadURL_EXIT;
}
if (err = _tfopen_s(&pFile, szFile, L"wt")) //_tfopen_s打开文件成功返回的值是0
{
OutputDebugString(L"fopen() failed!");
goto _DownloadURL_EXIT;
}
while (InternetReadFile(hURL, pBuf, DEF_BUF_SIZE, &dwBytesRead)) //读取hURL指定的资源到pBuf中,DEF_BUF_SIZE是准备读的字节,&dwBytesReads是真正读取的字节
{
if (!dwBytesRead)
break;
fwrite(pBuf, dwBytesRead, 1, pFile); //从pBuf写入到pFile中，内容的单位是dwBytesRead，共1个内容
}
bRet = TRUE;
_DownloadURL_EXIT: //退出代码
if (pFile)
fclose(pFile);
if (hURL)
InternetCloseHandle(hURL);
if (hInternet)
InternetCloseHandle(hInternet);
return bRet;
}
BOOL CALLBACK EnumWindowsProc(HWND hWnd, LPARAM lParam) //第一个参数默认是屏幕上顶层窗口的句柄，第二个参数是EnumWindows函数传入的dwPID
{
DWORD dwPID = 0;
GetWindowThreadProcessId(hWnd, &dwPID); //通过GetWindowThreadProcessId来通过窗口句柄来获取窗口对应进程的PID
if (dwPID == (DWORD)lParam) //判断该窗口是否是我们想要找的PID
{
g_hWnd = hWnd; //是真的话，则将该窗口句柄赋给全局变量g_hWnd
return FALSE;
}
return TRUE;
}
HWND GetWindowHandleFromPID(DWORD dwPID) //定义一个函数根据进程PID获取进程对应的窗口句柄
{
EnumWindows(EnumWindowsProc, dwPID); //EnumWindows函数枚举所有屏幕上的顶层窗口
return g_hWnd;
}
BOOL DropFile(LPCTSTR wcsFile)
{
HWND hWnd = NULL; //窗口句柄
DWORD dwBufSize = 0; //存储DROPFILES和szFile的缓冲区
BYTE *pBuf = NULL; //指向缓冲区的指针
DROPFILES *pDrop = NULL; //定义一个DROPFILES结构体指针
char szFile[MAX_PATH] = { 0, }; //存储文件名
HANDLE hMem = 0; //获取的内存的句柄(首地址)
WideCharToMultiByte(CP_ACP, 0, wcsFile, -1,
szFile, MAX_PATH, NULL, NULL); //将我们输入的字符串从宽字节转换成ANSI编码,CP_ACP指定代码页为ANSI
dwBufSize = sizeof(DROPFILES) + strlen(szFile) + 1; //计算缓冲区的大小(DROPFILES结构体加上路径需要的空间)
if (!(hMem = GlobalAlloc(GMEM_ZERO INIT, dwBufSize))) //从堆中分配dwBufSize的内存并初始化为0
{
OutputDebugString(L"GlobalAlloc() failed!!!");
return FALSE;
}
pBuf = (LPBYTE)GlobalLock(hMem); //让pBuf指向分配的内存
pDrop = (DROPFILES*)pBuf; //缓冲区的句柄就是结构体开始的地址，让pDrop指向DROPFILES结构体存储的地方
pDrop->pFiles = sizeof(DROPFILES); //到要拖拽的文件名字符串存储区的偏移
strcpy_s((char*)(pBuf + sizeof(DROPFILES)), strlen(szFile) + 1, szFile); //给存储区赋值
GlobalUnlock(hMem);
if (!(hWnd = GetWindowHandleFromPID(GetCurrentProcessId()))) //获取当前进程的PID并传入GetWindowHandleFromPID获取对应的窗口句柄
{
OutputDebugString(L"GetWndHandleFromPID() failed!!!");
return FALSE;
}
PostMessage(hWnd, WM_DROPFILES, (WPARAM)pBuf, NULL);
return TRUE;
}
DWORD WINAPI ThreadProc(LPVOID lParam)
{
TCHAR szPath[MAX_PATH] = { 0, };
TCHAR *p = NULL;
OutputDebugString(L"ThreadProc() start...");
GetModuleFileName(NULL, szPath, sizeof(szPath));
if (p = _tcsrchr(szPath, L'\\'))
{
_tcscpy_s(p + 1, wcslen(DEF_INDEX_FILE) + 1, DEF_INDEX_FILE);
OutputDebugString(L"DownloadURL()");
if (DownloadURL(DEF_URL, szPath))
{
OutputDebugString(L"DropFlie()");
DropFile(szPath);
}
}
OutputDebugString(L"ThreadProc() end...");
return 0;
}
BOOL WINAPI DllMain(HINSTANCE hinstDLL, DWORD fdwReason, LPVOID lpvReserved)
{
switch (fdwReason)
{
case DLL_PROCESS_ATTACH:
CloseHandle(CreateThread(NULL, 0, ThreadProc, NULL, 0, NULL)); //CreateThread创建线程执行ThreadProc
break;
}
return TRUE;
}
```

## (三)修改程序的PE，让程序一开始便加载我们的dll

其实我们修改的就是(IDT) IMPORT DIRECTORY TABLE（`image_import_descriptor`，即IID结构体）组成的数组，只要将我们的dll添加到这个数组的结尾就可以让程序运行时自动加载我们的dll了。

### 1. 查看IDT是否有足够空间

我们从`Image_Optional_Header`的`DataDirectory`的第二项`IMPORT TABLE`得到的就是这个结构体数组的`RVA`和`Size`。

![图片描述](upload/attach/202104/921830_4NQEFC5T6F3Q77Y.png)

这里再详细的讲解一下RVA和文件偏移的转化，其实很简单，记住每个节区有个差值是不变的，什么差值呢， 就是每个节区`RVA` - 对应的文件偏移是不变的，记住这一点，我们找到节区的起始`RVA`和起始文件偏移，相减就可以得到这个差值，

通过查看`image_section_header`，发现`RVA` `84CC`处于`rdata`区域

![图片描述](upload/attach/202104/921830_JC6FZBRV5NM5P9D.png)

然后我们就可以得出差值：`0x6000`-`0x5200` = `0xE00`

然后再用我们知道的`RVA`，即`0x84CC` - 差值`0xE00` = `0x76CC`

得到的`0x76CC`就是文件偏移。

下面再遇见RVA和文件偏移的转化不再过多赘述

![图片描述](upload/attach/202104/921830_CXNBJKB2U2XKC3H.png)

然后我们使用`winhex`打开可以发现在我们的IDT之后紧贴着其它数据，我们没有足够的空间来添加一个`0x14`字节的结构体进去。(?)

![图片描述](upload/attach/202104/921830_9WPT8K9UHYM3DT5.png)

在这种情况之下，我们只有移动我们的IDT到文件的其它地方

### 2. 移动IDT

主要有三种方法：①查找文件的空白区域。②增加文件最后一个节区的大小。③在文件末尾添加新的节区。

常用的就是第一种，因为由于内存对齐的原因，一般都会导致节区的最后会有一段`Null-padding`区域。

查看`IMAGE_NT_Headers`对应的各节区头的信息

![图片描述](upload/attach/202104/921830_Y5N3RX2GYSB4SKD.png)

查看`rdata`的，发现这个节区在文件中的大小为`2E00`，但是`Virtual Size`只有`2C56`，相减得到`1AA`，足够我们放下(`0x14 * 6 = 0x78`)字节的数据。(只有分析节区头的这个差值我们才能存放我们的代码，如果是为了对齐而在内存中添加的填充区域，我们是不能存放代码的)

所以我们的新的IDT的`RVA`就可以从(`0x6000`+`0x2C56`)之后选取一段`0x78`(`0x14 * 6`)字节的大小来存放我们的IDT

我们从`0x8C80`开始存放我们的IDT(转化为文件偏移为`0x7E80`)(新IDT: `0x7E80`到(`0x7E80`+`0x78`)）

直接将原先的IDT(`76CC~772F`)复制到新区域

![图片描述](upload/attach/202104/921830_5VH6UHUFFAKQB3Y.png)

### 3. 修改DataDirectory的第二项IMPORT TABLE的值

原先的值：

![图片描述](upload/attach/202104/921830_8RKZFT32SGW4THH.png)

根据文件偏移，我们找到那个位置，更改为我们的新的IDT的`RVA`和`Size`

![图片描述](upload/attach/202104/921830_49H9KU5RXCQRP55.png)

### 4. 删除绑定导入表

![图片描述](upload/attach/202104/921830_3R3FD7H73FXHPVQ.png)

这里可以发现为0，就直接可以不用管了，这是一种提高dll加载速度的技术，是个可选项，如果不存在，无影响，如果存在，又发生了错误，那么就会导致程序运行时发生错误。

### 5. 填写我们新增的IMAGE_IMPORT_DESCRIPTOR

```
typedef struct _IMAGE_IMPORT_DESCRIPTOR {
union {
DWORD Characteristics;
DWORD OriginalFirstThunk; //INT(Import Name Table) address (RVA)
};
DWORD TimeDateStamp;
DWORD ForwarderChain;
DWORD Name; //library name string address (RVA)
DWORD FirstThunk; //IAT(Import Address Table) address (RVA)
}IMAGE_IMPORT_DESCRIPTOR;
```

首先我们设置`OriginalFirstThunk`，这是一个INT的RVA，我们使用`8D00`，处于`1AA`的那个区域之内都可以，`TimeDateStamp`和`ForwarderChain`都直接设置为0即可(8字节)，`Name`是用来存放导入模块的名称字符串的，`8D10`处存放，`FirstThunk`是IAT的RVA，我们设置为`8D20`.

如下：

![图片描述](upload/attach/202104/921830_N3SN3MTDPVSM6ZA.png)

然后我们去设置对应的INT和IAT和Name的值

最后如下：

![图片描述](upload/attach/202104/921830_CGP3ZF45Q9PPWEU.png)

其实就是INT和IAT都是`IMAGE_IMPORT_BY_NAME`（hint+函数名）的RVA

在动态链接器刚完成映射没有开始重定位和符号解析的时候，IAT中的元素就是这个，但是如果Windows的动态链接器在完成该模块的链接时，元素值会被动态链接器改为该符号的真正地址。（类似于elf的got）。

### 6. 设置IAT节区的属性值

原因：加载IAT的时候，PE装载器会修改IAT，写入函数的实际地址，所以对应的节区必须有写的属性。

直接修改`image_section_header`中的`characteristic`的值

![图片描述](upload/attach/202104/921830_SJHFV9MZRDG34EE.png)

加上`IMAGE_SCN_MEM_WRITE`(80000000)的值
