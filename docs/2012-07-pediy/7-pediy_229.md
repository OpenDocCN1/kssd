
# 给任务管理器增加显示程序完整路径功能

> 原文：[`www.pediy.com/kssd/pediy10/82582.html`](https://www.pediy.com/kssd/pediy10/82582.html)

**【文章标题】**: 给任务管理器增加显示程序完整路径功能  
**【文章作者】**: stalker  
**【软件名称】**: `taskmgr.exe`  
**【下载地址】**: 系统目录  
**【使用工具】**: `OllyDbg`, `LordPE`, `ResHacker`  
**【操作平台】**: `Windows XP Sp2`

---

## 详细过程

Windows 自带的任务管理器无法显示程序所在的完整路径，我一直觉得这是一个缺陷，今天我们就自己来给它增加这一功能。

获取一个进程完整路径的方法有很多，我们使用 `CreateToolhelp32Snapshot` & `Module32First` 的方法，示例代码如下：

```
invoke CreateToolhelp32Snapshot,TH32CS_SNAPMODULE,dwTargetProcessID
mov MD32.dwSize,sizeof MD32           ;MD32 为一个 MODULEENTRY32 结构体变量
invoke Module32First,eax,offset MD32
```

调用上面的代码之后 `MD32.szExePath` 中就是目标进程的完整路径了。

下面正式开始对任务管理器的修改，大致可以分为 3 个步骤：

1. 增加菜单
2. 定位程序响应菜单项点击的代码位置
3. 加入对我们新增菜单的响应代码

### 1. 增加菜单

首先使用资源编辑工具（这里我使用的是 `ResHacker`）给程序增加一个菜单项 "显示完整路径(&S)"，如图 1

![](img/f6ae7f65e5c2df9f5dc7031a661804bd.jpg)

我们新增的菜单项与 "结束进程(&E)" 处于同一组中，我们就通过它来定位任务管理器对菜单项响应代码的位置，从图 1 我们看

到它的 ID 为 40028，转换为 16 进制就是 `9C5C`（记下来）

### 2. 定位代码位置

关于定位，首先我想到的是以前使用的 “查找所有分支法”，无奈没有成功（看来这办法并不是[屡试不爽](http://baike.baidu.com/view/68337.htm)=_=）

于是另寻他法，通过行为来定位，`OD` 载入任务管理器，对 `TerminateProcess` 这个函数下断，运行之，然后随便运行一个程序，

再在被调试的任务管理器中右键结束它，此时 `OD` 断下了任务管理器对 `TerminateProcess` 函数的调用，如图 2

![](img/62d3a9252c7506e3f1d98206c8b8ff80.jpg)

从 `0100C226`，一直向上找到过程首

```
0100C17D  /$  8BFF          mov     edi, edi
0100C17F  |.  55          push    ebp
0100C180  |.  8BEC        mov     ebp, esp
0100C182  |.  83EC 20     sub     esp, 20
......省略中间若干代码
0100C223  |.  6A 01       push    1                             ; /ExitCode = 1
0100C225  |.  56          push    esi                           ; |hProcess
0100C226  |.  FF15 C0100001 call    dword ptr <&KERNEL32.TerminateP>; \TerminateProcess
```

单击 `0100C17D` 这一行，可以在信息窗口看到如图 3 所示信息

![

有三个地方调用了这个过程，我们要寻找究竟哪个地方对菜单项进行了判断处理，一个一个前往查看下，过程省略，最终找到

下面的地方

```
0100CF29   > \BA 5C9C0000   mov     edx, 9C5C       ;9C5C，你还记得吗？
0100CF2E   >  3BCA          cmp     ecx, edx
0100CF30   .^ 74 AD        je      short 0100CEDF
0100CF32   .^ 7E BB        jle     short 0100CEEF
0100CF34   .  81F9 629C0000 cmp     ecx, 9C62
0100CF3A   .  7E 30        jle     short 0100CF6C
0100CF3C   .  81F9 779C0000 cmp     ecx, 9C77
0100CF42   .  74 18        je      short 0100CF5C
0100CF44   .  81F9 A99C0000 cmp     ecx, 9CA9
0100CF4A   .^ 75 A3        jnz     short 0100CEEF
0100CF4C   .  85C0        test    eax, eax
0100CF4E   .^ 74 9F        je      short 0100CEEF
```

### 3. 加入响应代码

对菜单项 id 的处理判断找到了，我们直接从 `0100CF29` 这里跳走，加入我们自己的代码，然后再跳回来继续执行

再写代码之前我们应当知道：

1. 一个 `MODULEENTRY32` 结构的大小为 `224H`
2. 结构成员 `szExePath` 的相对于结构体起始地址的偏移为 `120H`
3. 要成功获取进程的完整路径，我们要获得该进程的 `PID` 以及寻找一个内存块来存放一个 `MODULEENTRY32` 结构体

关键是第 3 点，如何获取目标进程的 `PID`，我们知道，通过 `TerminateProcess` 来结束一个进程，首先得使用 `OpenProcess` 函数

来打开这个进程以获取进程句柄，而 `OpenProcess` 又需要进程的 `PID` 做为参数。我们回到开始任务管理器结束进程的地方看看

```
0100C17D  /$  8BFF          mov     edi, edi
0100C17F  |.  55          push    ebp
0100C180  |.  8BEC        mov     ebp, esp
0100C182  |.  83EC 20     sub     esp, 20
......
0100C210  |.  FF75 08     push    dword ptr [ebp+8]                   ; /ProcessId
0100C213  |.  6A 00       push    0                             ; |Inheritable = FALSE
0100C215  |.  6A 01       push    1                             ; |Access = TERMINATE
0100C217  |.  FF15 B4100001 call    dword ptr [<&KERNEL32.OpenProces>; \OpenProcess
0100C21D  |.  8BF0        mov     esi, eax
0100C21F  |.  85F6        test    esi, esi
0100C221  |.  74 28       je      short 0100C24B
0100C223  |.  6A 01       push    1                             ; /ExitCode = 1
0100C225  |.  56          push    esi                           ; |hProcess
0100C226  |.  FF15 C0100001 call    dword ptr [<&KERNEL32.TerminateP>; \TerminateProcess
```

可以看到 `OpenProcess` 的第三个参数为 `[ebp+8]`，即当前过程(`0100C17D`)的第一个参数，调用此过程的代码如下

```
0100CEE3   .  6A 00       push    0
0100CEE5   .  FF70 08     push    dword ptr [eax+8]    ;[eax+8]与上面的[ebp+8]相对应，里面就是目标进程 pid
0100CEE8   .  8BCE        mov     ecx, esi            ;调试过程中可以通过命令 d eax+8 来验证
0100CEEA   .  E8 8EF2FFFF   call    0100C17D
```

上面这段调用代码，与开始我们找到的判断菜单 id 进行处理的代码位于同一过程，因此我们的代码可以直接使用 `[eax+8]` 来

获取目标进程的 pid

最后一个问题就是，从哪里找地方来存放我们的 `MODULEENTRY32` 结构体

首先使用 `OD` 载入运行任务管理器（一定要运行），然后 `Alt+M` 找到其 `.data` 段的位置，如图 4

![](img/d0fc7aae7d40482d93337c482f66a6b8.jpg)

双击该地址前往，然后会弹出来一个 dump 窗口，里面显示的是内存数据，一直往下拖，直到出现大片空白为止

然后随便找一个地址作为结构体的起始地址（但要确保其后大小不小于 `224H`），我使用的是 `01016A00` 这个地址

到现在，基本所有的问题都解决了，剩下的就是找地方写代码

由于 `taskmgr.exe` 并没有导入 `CreateToolhelp32Snapshot` 以及 `Module32First` 这两个函数，写入代码之前首先用 `LordPE` 导入它们，

除此之外，我还导入了 `MessageBoxA`，因为 `taskmgr.exe` 只导入了 `MessageBoxW`，而结构体 `MODULEENTRY32` 的 `szExePath` 成员并不是

一个 `Unicode String`。当然，你也可以先将它转换为 `Unicode String`，然后再调用 `taskmgr.exe` 导入的 `MessageBoxW`。不过我想

这样也许会更麻烦，因此我采用了增加导入 `MessageBoxA` 的简单方法=_=

写入代码如下

```
01014F00   > \60          pushad
01014F01   .  BA 989C0000  mov     edx, 9C98
01014F06   .  3BCA         cmp     ecx, edx
01014F08   .  75 37        jnz     short 01014F41
01014F0A   .  FF70 08      push    dword ptr [eax+8]                   ; /ProcessID
01014F0D   .  6A 08        push    8                             ; |Flags = TH32CS_SNAPMODULE
01014F0F   .  FF15 38100201 call    dword ptr [<&kernel32.CreateTool>; \CreateToolhelp32Snapshot
01014F15   .  C705 006A0101>mov     dword ptr [1016A00], 224
01014F1F   .  68 006A0101   push    01016A00                    ; /pModuleentry = taskmgr3.01016A00
01014F24   .  50          push    eax                           ; |hSnapshot
01014F25   .  FF15 3C100201 call    dword ptr <&kernel32.Module32Fi>; \Module32First
01014F2B   .  FF15 7C130001 call    dword ptr [<&USER32.GetForegroun>; [GetForegroundWindow
01014F31   .  6A 00       push    0                             ; /Style = MB_OK|MB_APPLMODAL
01014F33   .  6A 00       push    0                             ; |Title = NULL
01014F35   .  68 206B0101   push    01016B20                    ; |Text = ""
01014F3A   .  50          push    eax                           ; |hOwner
01014F3B   .  FF15 5D100201 call    dword ptr [<&user32.MessageBoxA>>; \MessageBoxA
01014F41   >  61          popad
01014F42   .  BA 5C9C0000  mov     edx, 9C5C
01014F47   .^ E9 E27FFFFF  jmp     0100CF2E
```

### 修改

修改

```
0100CF29   > \BA 5C9C0000   mov     edx, 9C5C    
0100CF2E   >  3BCA          cmp     ecx, edx
```

为

```
0100CF29   > \E9 D27F0000   jmp     01014F00
0100CF2E   >  3BCA          cmp     ecx, edx
```

然后保存所有修改，至此，大功告成

附上胜利截图一张以及修改过的 `taskmgr.exe`

![    -----------------------------------------------------------------   【经验总结】   我的目的基本还算达到了，原来发现不认识的进程时，想找出它来看看，却发现任务管理器居然不支持显示完整路径。   但不知道为什么，获取一些系统进程的路径的时候，没有成功，显示出来是乱码。    该问题已经解决，权限问题（我自己测试的时候使用了一段在其他程序中正常的 bug 提权代码，于是没有发现）再次感谢 安摧 同学，希望大家写程序的时候一定仔细![](img/f43c1e514327696dccbb0da997496b2a.jpg "扮鬼脸")

看

![](img/576cc8298f117c6f5cbcb8f85aa5dc74.jpg)

现在附件中是上传的新的 `taskmgr.exe`

---

2009 年 02 月 22 日 下午 16:34

上传的附件

| ![](img/c8c74a380a3dea7fa2460187d80ee5e0.jpg) || `mytaskmgr.rar` |

权限问题~~~

在打开一个用户名为 `SYSTEM` 的进程的时候，因为权限不够，`CreateToolhelp32Snapshot` 返回了 `-1(INVALID_HANDLE_VALUE)`。

引用: `01014F00   > \60          pushad`

`01014F01   .  BA 989C0000  mov     edx, 9C98`

`01014F06   .  3BCA         cmp     ecx, edx`

`01014F08   .  75 37        jnz     short 01014F41`

`01014F0A   .  FF70 08      push    dword ptr [eax+8]                            ; /ProcessID`



```
01014F0D   .  6A 08          push    8                                                         ; |Flags = TH32CS_SNAPMODULE
01014F0F   .  FF15 38100201 call    dword ptr [<&kernel32.CreateToolhelp32Snapshot>]      ; \CreateToolhelp32Snapshot

//这个返回失败！
01014F15   .  C705 006A0101>mov     dword ptr [1016A00], 224
01014F1F   .  68 006A0101   push    01016A00                                                  ; /pModuleentry = mytaskmg.01016A00
01014F24   .  50            push    eax                                                       ; |hSnapshot
01014F25   .  FF15 3C100201 call    dword ptr [<&USER32.GetForegroundWindow>]                 ; \Module32First
01014F2B   .  FF15 7C130001 call    dword ptr [<&USER32.GetForegroundWindow>]                 ; [GetForegroundWindow
01014F31   .  6A 00         push    0                                                         ; /Style = MB_OK|MB_APPLMODAL
01014F33   .  6A 00         push    0                                                         ; |Title = NULL
01014F35   .  68 206B0101   push    01016B20                                                  ; |Text = "?",06,"??",06,"?,B6,"",B2,"",92,"??",06,"",B2,"",92,"?",06,"",B2,"",92,"?"
01014F3A   .  50            push    eax                                                       ; |hOwner
01014F3B   .  FF15 5D100201 call    dword ptr [<&user32.MessageBoxA>]                         ; \MessageBoxA
01014F41   >  61            popad
01014F42   .  BA 5C9C0000   mov     edx, 9C5C
01014F47   .^ E9 E27FFFFF   jmp     0100CF2E
```

提升权限的代码：

```cpp
BOOL EnableDebugPrivileges()
{
    /*********************************************/
    //提升进程自身权限
    BOOL bRet;
    HANDLE hToken;
    bRet = ::OpenProcessToken(GetCurrentProcess(), TOKEN_ADJUST_PRIVILEGES, &hToken);
    if(!bRet)
    {
        return FALSE;
    }
    TOKEN_PRIVILEGES tp;
    tp.PrivilegeCount = 1;
    ::LookupPrivilegeValue(NULL, SE_DEBUG_NAME, &tp.Privileges[0].Luid);
    tp.Privileges[0].Attributes = SE_PRIVILEGE_ENABLED;
    ::AdjustTokenPrivileges(hToken, FALSE, &tp, sizeof(tp), NULL, NULL);
    if(GetLastError() != ERROR_SUCCESS)
    {
        return FALSE;
    }
    return TRUE;
    /*********************************************/
}
```
