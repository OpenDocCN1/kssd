
# 给 Windows XP 记事本添加“自动保存”功能

> 原文：[`www.pediy.com/kssd/pediy10/81208.html`](https://www.pediy.com/kssd/pediy10/81208.html)

*   **文章作者**: `newjueqi`
*   **作者邮箱**: `zengjiansheng1@126.com`
*   **作者 QQ 号**: `190678908`
*   **作者博客**: [`http://blog.csdn.net/newjueqi`](http://blog.csdn.net/newjueqi)
*   **使用工具**: `OD`, `LordPE`, `eXeScope`
*   **操作平台**: `XP-SP2`
*   **作者声明**: 本人平时一般的文字处理都是用记事本（用 Word 好像大材小用了），电脑自从拿去大修后有时候会莫名其妙的重启，弄得经常重写（本人常常忘记保存^-^），于是想给记事本增加类似于 Word 的自动保存功能，以图个方便。失误之处敬请诸位大侠赐教!

具体的思路：其实自动保存功能就是程序隔一段时间保存一次，而记事本程序是自带了保存功能的，所以只要编程安装一个记时器，每隔一段时间模拟点击“保存菜单”，就能实现自动保存，最后退出时把记时器撤销。

首先，用 `eXeScope` 给记事本增加两个菜单项“自动保存”、“关闭自动保存”，如图 1 所示：

![](img/a7dacb1643b08463d9312df22412ec03.jpg)

**图 1**

其中，“自动保存”的菜单 ID 是 `8`, “关闭自动保存” 的菜单 ID 是 `13`。

完成后的菜单选项如图 2 所示：

![](img/83ea4075cf6c419e9695addc9fe87183.jpg)

**图 2**

本次修改要用到 3 个 window API 函数: `SetTimer`, `KillTimer`, `SendMessageA`, 这 3 个 API 没包括在记事本的输入表里，所以要用 `LordPE` 在输入表里增加这 3 个 API。

用 `LordPE` 打开记事本程序，依次选择 `Direction` -> 输入表的“…”按钮 -> 点击右键 -> `add import`，出现如图 3 的界面，就可以在输入表里增加新的 API

![](img/b3a6edec3f6662ff28016370c01edf9b.jpg)

**图 3**

完成后就新增了一个区段 `Slivana` 如图 4，放的就是新的 IID 数据，如图 4 所示

![](img/79b0759560b5598a979dc08ab9b6b0f1.jpg)

**图 4**

添加完新的 API 后又有一个新的问题了，那就是要确定新的 API 地址，这个在 PEDIY 的过程中是要自己解决的，而平时是编译器帮忙的。获得 API 地址有两个方法：

（1）直接查看新增的 IID 数据，载入程序后在命令行打入 `d 1013000`，如图 5 所示：

![](img/3c519c18fb3968c55f374f04b8959f92.jpg)

**图 5**

从 IAT 与 INT 的对应关系可得：

*   `SetTimer` 的地址=`[0101302C]`
*   `MessageBoxA` 的地址=`[01013030]`
*   `KillTimer` 的地址=`[01013034]`

(2) 在 `OD` 中，按 `Ctrl+N` 组合键，就能看到 API 地址的位置，如图 6 的红框部分所示

![](img/9bc280617e203cec9a511b42fe717ee4.jpg)

**图 6**

由于 PEDIY 一般都是在默认的窗口过程前先处理我们定义的消息，所以接下来分析一下记事本程序的流程，看看记事本的窗口过程地址。众所周知，在一个标准的窗口程序中，必须要注册窗口类，函数原型如下：

```
ATON RegisterClassEx（CONST WNDCLASSEX *Ipwcx）；
```

其中 `WNDCLASSEX *Ipwcx` 是个结构体指针，定义如下：

```
WNDCLASSEX STRUCT DWORD  
    cbSize DWORD ?  
    style DWORD ?  
    lpfnWndProc DWORD ?  
    cbClsExtra DWORD ?  
    cbWndExtra DWORD ?  
    hInstance DWORD ?  
    hIcon DWORD ?  
    hCursor DWORD ?  
    hbrBackground DWORD ?  
    lpszMenuName DWORD ?  
    lpszClassName DWORD ?  
    hIconSm DWORD ?  
WNDCLASSEX ENDS  
```

`lpfnWndProc` 就是窗口过程的地址，所以我们只需要在 `RegisterClassEx` 下中断，就能知道窗口过程的地址，在 `OD` 中如图 7 所示：

![](img/d2d94aba0167900d4a32d483b486d56f.jpg)

**图 7**

从图 7 可看到 `1003429` 就是系统默认的窗口过程的地址。

在开始写代码前我们先简单回顾一下的 Windows 的窗口过程函数和一般函数调用的过程，因为我们在后面的实际编码过程中必须要对这两个方面的知识有一定的了解。

窗口过程的原型为：

```
wondowPrco proc hwnd,uMsg, wParam, lParam
```

*   `hwnd`: 窗口的句柄，可通告 `CreateWindowEx` 函数的返回值获取
*   `uMsg`: 消息的标识，一般以 `WM` 开头
*   `wParam, lParam`: 消息的两个参数

当我们点击某个菜单后就产生了 `WM_COMMAND` 消息，对于菜单引发的 `WM_COMMAND` 消息, `lParam` 的值为 `0`，`wParam` 的低 16 位是命令 ID，高 16 位是通知码，菜单消息的通知码是 `0`。

当调用一个 `_stdcall` 函数的时候，首先是函数的参数按从右到左的顺序入栈，然后是函数的返回地址入栈，一般函数调用的开始有“`push ebp mov ebp,esp`”这句，所以我们进入窗口过程后就能得到类似于图 8 的堆栈结构

![](img/edd390f4a0aca013695a47425e363c52.jpg)

**图 8**

```
0006FDE0  /0006FE0C    ; ebp 的值  
0006FDE4  |77D18734   ； 返回到 user32.77D18734，返回地址值  
0006FDE8  |00100618    ；窗口的句柄  
0006FDEC  |00000111    ；WM_COMMAND 的 16 进制值，可查阅 WinUser.h 文件获得  
0006FDF0  |00000008    ；wParam，“自动保存”的菜单 ID 是 8  
0006FDF4  |00000000    ；lParam，  
```

由此可知，`[ebp+c]`是消息的类型，`[ebp+10]`是 `wParam` 参数，获取菜单的 ID 就需要知道 `wParam` 参数。

另外还有一个问题要解决，`SetTimer` 和 `KillTimer` 这两个 API 函数的参数中必须要传入窗口句柄作为参数，但窗口句柄怎么找呢？我们知道，在用 `CreateWindowExW` 函数创建窗口后会返回一个窗口句柄，在 `OD` 里对 `CreateWindowExW` 下断点，中断后代码如图 9：

![](img/6a13d5400a33c8b80ca220d22590ca8c.jpg)

**图 9**

从红框部分可知，窗口句柄保存在地址`[1009830]`。

我们要模拟点击“保存”菜单，可以通过 `SendMessageW` 函数向窗口过程发送菜单消息，`SendMessageW` 的原型如下：

```
LRESULT SendMessage（HWND hWnd，UINT Msg，WPARAM wParam，LPARAM IParam）；
```

参数：

*   `hWnd` 就是窗口句柄，从上面分析可知，保存在`[1009830]`地址上。
*   `Msg` 是要发送的消息类型，查阅 `WinUser.h` 文件可知，`WM_COMMAND` 为 `0x111`
*   `wParam` 的低 16 位是命令 ID，高 16 位是通知码，菜单消息的通知码是 `0`，通过 `eXeScope` 可查知“保存”的菜单 ID 为 `3`，所以 `wParam` 参数为 `3`。
*   `lParam`，对于菜单消息来说，为 `0`。

接下来就是实际的代码，本人选择在 `0100874F` 处添加代码，所以要把图 7 红框部分的 `1003429` 改为 `0100874F`。

下面是写入的代码：

```
0100874F   .  55             push    ebp  
01008750   .  8BEC          mov     ebp, esp  
01008752   .  60            pushad   
01008753   .  817D 0C 11010>cmp     dword ptr [ebp+C], 111  ；先比较消息的类型，看是否 WM_COMMAND, 如果是 WM_COMMAND 的话再检查 wParam 参数进一步确定是哪个菜单的消息  
0100875A   .  75 0C         jnz     short 01008768    ；如果不是 WM_COMMAND 消息的话就检查是否 WM_TIMER 或 WM_CLOSE 消息  
0100875C   .  837D 10 08    cmp     dword ptr [ebp+10], 8  ；比较菜单的 ID 号，看是否“自动保存”的菜单ＩＤ  
01008760   .  74 1C         je      short 0100877E    ；不是的话就跳去检查其它的消息类型  
01008762   .  837D 10 0D    cmp     dword ptr [ebp+10], 0D    ；比较菜单的 ID 号，看是否“关闭自动保存”的菜单ＩＤ  
01008766   .  74 48         je      short 010087B0    ；不是的话就跳去检查其它的消息类型  
01008768   >  817D 0C 13010>cmp     dword ptr [ebp+C], 113  ；比较消息的类型是否是 WM_TIMER 消息，响应定时器消息  
0100876F   .  74 5F         je      short 010087D0    ；不是的话就跳去函数的结束清场工作  
01008771   .  837D 0C 10    cmp     dword ptr [ebp+C], 10    ；比较消息的类型是否是 WM_CLOSE 消息，闭关时要先撤销定时器  
01008775   .  74 39         je      short 010087B0    ；不是的话就跳去函数的结束清场工作  
01008777   >  61            popad  
01008778   .  5D            pop     ebp  
01008779   .^ E9 ABACFFFF   jmp     01003429  ；跳回原来记事本的消息循环  
0100877E   >  C605 20890001>mov     byte ptr [1008920], 1  ；标记已使用自动保存功能  
01008785   .  6A 00         push    0           ; /Timerproc = NULL  
01008787   .  68 60EA0000   push    0EA60       ; |Timeout = 60000. ms  
0100878C   .  6A 01         push    1           ; |TimerID = 1  
0100878E   .  FF35 30980001 push    dword ptr [1009830]     ; |hWnd =  [1009830]，在 CreateWindowExW 后保存窗口句柄的地址  
01008794   .  FF15 2C300101 call    dword ptr [<&user32.SetTimer>]   ; \\SetTimer  
0100879A   .  6A 00         push    0             ; /Style = MB_OK|MB_APPLMODAL  
0100879C   .  68 00DB0001   push    0100DB00          ; |Title = "Note"  
010087A1   .  68 10DB0001   push    0100DB10          ; |Text = "AutoSave Start!"  
010087A6   .  6A 00         push    0           ; |hOwner = NULL  
010087A8   .  FF15 30300101 call    dword ptr [<&user32.MessageBoxA>>; \\MessageBoxA  ；提示已使用自动保存功能  
010087AE   .^ EB C7         jmp     short 01008777  ；跳去函数结束清场  
010087B0   >  803D 20890001>cmp     byte ptr [1008920], 0  ；先比较是否已启动定时器  
010087B7   .^ 74 BE         je      short 01008777    ；如果没启动定时器就跳去函数结束清场  
010087B9   .  C605 20890001>mov     byte ptr [1008920], 0  ；把启动定时器的标记恢复  
010087C0   .  6A 01         push    1             ; /TimerID = 1  
010087C2   .  FF35 30980001 push    dword ptr [1009830]    ; |hWnd，在 CreateWindowExW 后保存窗口句柄的地址  
010087C8   .  FF15 34300101 call    dword ptr [<&user32.KillTimer>]  ; \\KillTimer，撤销定时器  
010087CE   .^ EB A7         jmp     short 01008777  ；跳去函数结束清场  
010087D0   >  6A 00         push    0             ; /lParam = 0  
010087D2   .  6A 03         push    3           ; |wParam = 3 ，“保存”菜单的ＩＤ号  
010087D4   .  68 11010000   push    111           ; |Message = WM_COMMAND  
010087D9   .  FF35 30980001 push    dword ptr [1009830]      ; |hWnd ，在 CreateWindowExW 后保存窗口句柄的地址  
010087DF   .  FF15 40120001 call    dword ptr \<&USER32.SendMessageW>; \\SendMessageW  ；发送点击菜单“保存”的信号给记事本窗口，实现保存功能。  
010087E5   .^ EB 90         jmp     short 01008777    ；跳去函数结束清场  
```

在家乡的惯例中，今天年初七是“众人生日”，祝愿各位 `happy birthday`! [![](img/c72cb8eba397ac67dda6e1ac58a9b627.jpg "微笑") ![](img/c72cb8eba397ac67dda6e1ac58a9b627.jpg "微笑")]
