
# FlashBoot DIY 教程

> 原文：[`www.pediy.com/kssd/pediy10/60688.html`](https://www.pediy.com/kssd/pediy10/60688.html)

下载页面： `http://www.prime-expert.com/flashboot/`

软件大小： 5MB 以上

软件简介： `shBoot` is a tool to make USB flash disks bootable. It is highly configurable, has many options and was designed to be compatible with all USB disks and BIOSes.

**【作者声明】**：只是感兴趣，没有其他目的。失误之处敬请诸位大侠赐教，写教程很累，格式纯属抄袭。

**【DIY 目的】**：偷懒，启动后直接进入我们需要的功能页，并且调整界面，并且只点一次下一步就自动开始格式化。

**【所用工具】**：`WinXP`、`OllyDBG`、`PEiD`、`LordPE`、`Vistual Studio 2005`

**【BTW 一下】**：我 DIY 的是截至到现在官方最新版，由于我是壳盲，所以是某兄给我的脱壳后的文件，如果你也想自己试验，脱壳自想办法。

## DIY 过程

首先先看看我们需要做的第一步，自动选择功能页，软件启动后显示欢迎信息，然后点击下一步，然后选择功能，我们的第一个目标就是要省去这个步骤，直接显示我们的选项页。

首先用资源工具一看没有窗体资源，由此怀疑是动态创建，`od` 载入下段 `CreateWindowExA,W`,无果，再下段 `CreateDialogRedirectA,CreateDialogIndirectParamW` 断下来了，观察 `pTemplate` 参数，发现不在 `exe` 空间内，由此判断是动态创建，稍微跟踪下就发现是类脚本解释的东西，具体是啥，不去追究。

打开窗口记录器

![](img/fd9528fb250d746a8df304014ece1c2b.jpg)

发现很有规律，控件都是作为子窗口创建于一种类名为 `wxWindowClass` 的窗口，且所有页窗口均已创建，因此怀疑下一步只是简单的 `ShowWindow` 这些宿主窗口，由此下断 `ShowWindow`，这里我不准备 `google` `wxWindowClass` 找他使用的是开发包，因为超出我的理解能力范围了。注意要在程序运行起来后下，不然创建那么多窗体，即使用 `SW_SHOW` 过滤也有好几个。下好断点后我们点下一步，果然断下来了：

![](img/108365b5c27437f267decb55c172a99d.jpg)

注意观察堆栈，然后 `F9`，果然是简单的显示英藏宿主窗体，注意，思路非常重要，下一步只有一个，且下一步后句柄尚未变化，说明所有选项页切换都是在这个 `Button` 的回调过程中完成的，当然就包括我们现在断下来的代码，因此我们继续往下跟，多用断点测试，调试多次，我们很容易发现：

![](img/976351fdb75b71bf0dd750821c9f3e9c.jpg)

这个函数既为 `Botton` 的 `Proc`，且

![](img/3e2b2de49d371d33ce5d94a69787b581.jpg)

函数为关键的代码解释执行函数，我们在这个函数上下段发现程序启动会调用一次来显示初始化页面，而且还有一个更为关键的信息我们很容易的从代码上看出来，那就是这些代码均使用 `C++` 完成，意味着我们可以跟踪 `ECX` 的值来猜测很多代码工作。现在我们需要仔细分析下代码了，这个过程复杂，自己想办法，我这里直接给出概要

```
004527EF         55                 push    ebp

004527F0         8BEC               mov     ebp, esp

004527F2         56                 push    esi                               ; flashboo.0055EC08

004527F3         8BF1               mov     esi, ecx

004527F5         83BE 90010000 00   cmp     dword ptr [esi+190], 0 //判断当前页类指针是否为 NULL,我们由此可以得到一个很重要的信息，ESI 为一个类，他的 ESI+190 处存放当前选项页类，也就是 wxwindowClass 窗体类

004527FC         74 12              je      short 00452810

004527FE         8B8E 90010000      mov     ecx, dword ptr [esi+190]

00452804         8B01               mov     eax, dword ptr [ecx]

00452806         FF90 DC000000      call    dword ptr [eax+DC]

0045280C         84C0               test    al, al

0045280E         74 3A              je      short 0045284A

00452810         8B45 08            mov     eax, dword ptr [ebp+8]

00452813         E8 439FFFFF        call    0044C75B

00452818         3B86 A4010000      cmp     eax, dword ptr [esi+1A4]

0045281E         8B8E 90010000      mov     ecx, dword ptr [esi+190]

00452824         8B01               mov     eax, dword ptr [ecx]

00452826         0F9445 08          sete    byte ptr [ebp+8]

0045282A         807D 08 00         cmp     byte ptr [ebp+8], 0

0045282E         74 08              je      short 00452838

00452830         FF90 14020000      call    dword ptr [eax+214]//查找选项页类指针

00452836         EB 06              jmp     short 0045283E

00452838         FF90 10020000      call    dword ptr [eax+210]// 查找选项页类指针

0045283E         FF75 08            push    dword ptr [ebp+8]

00452841         8BC8               mov     ecx, eax

00452843         8BD6               mov     edx, esi                           ; flashboo.0055EC08

00452845         E8 D0FAFFFF        call    0045231A  //解释执行核心代码，成功返回 1，其他为 0

0045284A         5E                 pop     esi                                ; 0012F350

0045284B         5D                 pop     ebp                                ; 0012F350

0045284C         C2 0400            retn    4
```

好了，现在我们直接下一步到我们需要的页上，再下断点，点击下一步可以跟入 `00452830` 或者 `00452838` 查看如何获取下一页选项页指针，经过跟踪我们发现，指针也是定义于 `ESI` 指向的类中，同样我们下段刚才提到的 `45231A`，在启动时断下来反流程到

```
0040A49C         56                 push    esi

0040A49D         8DB7 F8010000      lea     esi, dword ptr [edi+1F8]

 0040A4A3         E8 27010300        call    0043A5CF

0040A4A8         FFB7 AC010000      push    dword ptr [edi+1AC] //这里取出其实页类指针，我们只需要 patch 为我们需要的页，我 DIY 的是需要到+1C0,这个是从上面提到的 00452830 或者 00452838 跟出来的便宜，因此直接 patch 为 push [edi+1C0], 第一个目标完成。
```

下一步目标：点击下一步后自动完成功能，这个有几种方法，我选择的是模拟鼠标事件，因此代码量很大，所以我们当然不能直接 `asm`，那样要死人的，拿出我们的 `vs2005` 吧，

```
#pragma comment(linker,"/BASE:0x920000")

#pragma comment(linker,"/merge:.rdata=.data")

#pragma comment(linker,"/merge:.data=.text")

#pragma comment(linker,"/ALIGN:0x200")

#pragma comment(linker,"/ENTRY:Entry")
```

并且编译属性的清单嵌入不要选，也不生成重定位，这样编译出来的代码就只有一个区段，我们直接补到 `exe` 后面，既然上面我们分析了下一步的执行过程，那么我们可以在

```
call    0045231A//解释执行代码
```

后接管代码，于是我们有了入口

```
void __declspec(naked) Entry()

{

  __asm

  {

    push  eax

    push  ecx

    mov   ecx,[esp+0x18]//取出传递过来的类，也就是主类

    mov   eax,[ecx+0x190]//取出当前窗体类指针，这里要注意，当 0045231A 执行成功后，当前窗体会被设置为下一页，我们由此判断是否执行成功

    cmp   eax,[ecx+0x1DC]//下一页的偏移，在上面提到获取方法

    pop   ecx

    pop   eax

    jnz   lab_Normal

    pushad

    mov   eax,0x4DC3D4//GetProcAddress 的导入表填充地址

    push  [eax]//取出立即地址，

    call  InitOnce//初始化我们需要使用到的函数

    call  AutoIt//进入模拟事件函数

    popad

lab_Normal:

    pop     esi

    pop     ebp

    retn    4

  }

}
```

这样自动完成和自动选择选项功能就有了。

下面我们来调整界面，在上面我们发现

```
00452716         6A 01              push    1

00452718         8BD6               mov     edx, esi

0045271A         E8 FBFBFFFF        call    0045231A//这里进入我们说的解释执行核心,创建窗体就在这里

0045271F         8B06               mov     eax, dword ptr [esi]

00452721         8BCE               mov     ecx, esi

00452723         FF90 3C020000      call    dword ptr [eax+23C]//这里既进入消息循环了，由此 45271F 成了我们 patch 的最佳地点

00452729         2D EC130000        sub     eax, 13EC

0045272E         F7D8               neg     eax

00452730         1BC0               sbb     eax, eax

00452732         40                 inc     eax

00452733         5E                 pop     esi                                ; kernel32.7C816FD7

00452734         C2 0400            retn    4
```

同样入口

```
void __declspec(naked) InitWindow()

{

  __asm

  {

    pushad

    mov   eax,0x4DC3D4

    push  [eax]

    call  InitOnce

    call  FixWindowPos//执行 Fix 核心，

    popad

    mov     eax, dword ptr [esi]

    mov     ecx, esi

    call    dword ptr [eax+0x23C]

    push  0x452729

    ret

  }

}
```

`DIY` 不难，难度在于调整控制代码上，最好一次多写代码，因为调试麻烦。

编译出 `DLL` 后，

转存数据，

注意，我们需要添加一个空区段，大小为 `0x1000` 冒充 `PE` 头，然后再把真实的区段补上去，然后修改 `patch` 点，分别跳转到接管函数入口，

和

到此我们的 `DIY` 完工，运行效果如下

，下面给出代码，

```
#pragma comment(linker,"/BASE:0x920000")

#pragma comment(linker,"/merge:.rdata=.data")

#pragma comment(linker,"/merge:.data=.text")

//#pragma comment(linker,"/ALIGN:0x200")

#pragma comment(linker,"/ENTRY:Entry")

//#include <wtypes.h>

#include <windows.h>

typedef FARPROC (WINAPI* PENTRY_GetProcAddress)( __in HMODULE hModule, __in LPCSTR lpProcName );

typedef HWND (WINAPI* PENTRY_FindWindowExA)(__in_opt HWND hWndParent, __in_opt HWND hWndChildAfter, __in_opt LPCSTR lpszClass, __in_opt LPCSTR lpszWindow);

typedef HMODULE (WINAPI* PENTRY_GetModuleHandleA)( __in_opt LPCSTR lpModuleName );

typedef DWORD (WINAPI* PENTRY_GetWindowThreadProcessId)(__in HWND hWnd, __out_opt LPDWORD lpdwProcessId);

typedef DWORD (WINAPI* PENTRY_GetCurrentProcessId)();

typedef int (WINAPI* PENTRY_GetWindowTextA)( __in HWND hWnd, __out_ecount(nMaxCount) LPSTR lpString, __in int nMaxCount);

typedef BOOL (WINAPI* PENTRY_PostMessageA)(__in_opt HWND hWnd, __in UINT Msg, __in WPARAM wParam, __in LPARAM lParam);

typedef BOOL (WINAPI* PENTRY_SendMessageA)(__in_opt HWND hWnd, __in UINT Msg, __in WPARAM wParam, __in LPARAM lParam);
```
