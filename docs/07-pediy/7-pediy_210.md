
# 记事本功能增加方案

> 原文：[`www.pediy.com/kssd/pediy07/pediy7-425.htm`](https://www.pediy.com/kssd/pediy07/pediy7-425.htm)

作者：`peansen`  
日期：`2005-10-5`  
工具：各种常用破解工具

闲来无事看到论坛上关于记事本的帖子，就和老婆一起也弄了一个，呵呵。拿出来献丑了。

这个过程的基本思想是在记事本的消息处理之前加上我们自己的消息处理函数来达到添加功能的目的，我们知道 `windows` 的程序主要是在消息处理过程中增加功能的。加上的消息处理功能放在动态链接库中。

首先，使用 `LordPE` 的 `PE` 文件编辑功能添加链接库和函数。`NotePlug.dll`《链接库名》和 `LoadPlug`《函数名，这个函数名应该是写成 `WinProc` 之类的，由于某些原因就写成这样了》。添加的函数的 `RAV` 是 `13018`，在没有重定位下的虚拟地址是 `1013018`。

写一个简单的 `dll`

```c
#include <Windows.h>

void _cdecl LoadPlug(const DWORD reversed, HWND hWnd, UINT msg, WPARAM wParam, LPARAM lParam)
{
  switch(msg) {
  case WM_CREATE:
    MessageBox(hWnd, "琴子", "oo", 0x40);
    break;
  case WM_COMMAND:
    if(wParam == 66)//这是我自己通过资源添加的一个菜单的 id
      MessageBox(hWnd, "琴子", "oo", 0x40);
  }
}
```

`def` 文件

```text
LIBRARY  notepadplug
EXPORTS
LoadPlug     @1
```

导出这个函数，注意这个函数是 `_cdecl` 的。原因后面我会讲述。

中断 `RegisterClassExW` 函数，很容易到这里

```text
01004521  |.  8945 FC           mov dword ptr ss:[ebp-4],eax
01004524  |.  8D45 D0           lea eax,dword ptr ss:[ebp-30]
01004527  |.  50                push eax                                ; /pWndClassEx，下面的 mov 是对其进行赋值
01004528  |.  C745 F4 01000000  mov dword ptr ss:[ebp-C],1              ; |
0100452F  |.  8975 E4           mov dword ptr ss:[ebp-1C],esi           ; |
01004532  |.  C745 F8 20900001  mov dword ptr ss:[ebp-8],NOTEPAD.01009020      ; |UNICODE "Notepad"
01004539      C745 D8 29340001  mov dword ptr ss:[ebp-28],NOTEPAD.01003429//这是消息处理过程的赋值
01004540  |.  C745 F0 06000000  mov dword ptr ss:[ebp-10],6             ; |
01004547  |.  897D D4           mov dword ptr ss:[ebp-2C],edi           ; |
0100454A  |.  897D DC           mov dword ptr ss:[ebp-24],edi           ; |
0100454D  |.  897D E0           mov dword ptr ss:[ebp-20],edi           ; |
01004550  |.  FF15 D0110001     call dword ptr ds:[<&USER32.RegisterClassExW>]  ; \RegisterClassExW
```

我想这个结构在 `windows` 编程中是很重要的一部分，那个 `WndClassEx` 结构看看 `msdn` 就更清楚了。

接着我们看看消息处理过程部分

```text
01003421  \.  C2 0800           retn 8
01003424      CC                int3-------------------将从这里开始消息处理，只是一个简单的 call
01003425      CC                int3
01003426      CC                int3
01003427      CC                int3
01003428      CC                int3
01003429      8BFF              mov edi,edi-------------消息处理过程是从这里开始的
0100342B  /.  55                push ebp
0100342C  |.  8BEC              mov ebp,esp
0100342E  |.  51                push ecx
0100342F  |.  51                push ecx
01003430  |.  56                push esi
01003431  |.  8B75 0C           mov esi,dword ptr ss:[ebp+C]
01003434  |.  83FE 1C           cmp esi,1C                    ;  Switch (cases 2..8001)
```

利用 `5` 个 `int3` 和 `mov edi,edi` 共 `7` 个字节，改写一个 `call` 指令

```text
\----------------------------------------------关于 mov edi,edi--抄的

MOV EDI,EDI is indeed a 2-byte no-op that is there to enable hot-patching.  It enables the application of a hot-fix to a function without a need for a reboot, or even a restart of a running application.  Instead, at runtime, the 2-byte NOP is replaced by a short jump to a long jump instruction that jumps to the hot-fix function.  A 2-byte instruction is required so that when patching the instruction pointer will not point in a middle of an instruction.

\------------------------------------------------------------------------

call dword ptr [1013018]（共六个，后面不一个 nop）
```

然后修改 `01004539` 处的赋值 `mov dword ptr ss:[ebp-28],NOTEPAD.01003429`（改成 `01003424`）

这样子就大大的减小了代码的修改量。

现在来说一下为什么在到处函数中要添加 `_cdecl`？

因为使用 `_cdecl` 就说明函数过程不对原先压在堆栈中的变量进行处理，让调用者自行处理，而调用者知道这只是中间的一个转折无需处理这个参数，只要继续往下就行了。这样就实现了同样的参数两次应用。

```c
void _cdecl LoadPlug(const DWORD reversed, HWND hWnd, UINT msg, WPARAM wParam, LPARAM lParam)
```

大家也注意到了这个函数和一般的 `WndProc` 的基本参数还不一样，前面加了一个 `Reserved` 参数，其实这个参数是上次调用者的地址。另外还加了 `const` 修饰，是为了避免在程序中修改这些值，因为后面还要用呀。

这样就将这个 `dll` 嵌入了程序，截获了消息处理，剩下的程序用该很好写了，普通的 `windows` 编程。当我们要截获消息并且终止这个消息的时候我们可以给 `msg` 赋值 `WM_NULL`（以前还真没有体会到 `WM_NULL` 的用处，现在用上了，呵呵）

```c
#include <Windows.h>

void _cdecl LoadPlug(const DWORD reversed, HWND hWnd, UINT msg, WPARAM wParam, LPARAM lParam)
{
  switch(msg) {
  case WM_COMMAND:
    if(wParam == 26)
    {
      MessageBox(hWnd, "不让你插入时间", "呵呵", 0x40);
      msg = WM_NULL;
    }
  }
}
```

现在剩下的就是写这个消息处理过程了，大家仁者见仁，智者见智吧。

`4dd466d0e1e0cc123afbe504da2462ca`
