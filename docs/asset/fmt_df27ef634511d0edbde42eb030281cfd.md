
# Ring3 下截获 QQ2009 密码

> 原文：[`www.pediy.com/kssd/pediy12/103119.html`](https://www.pediy.com/kssd/pediy12/103119.html)

搞出来很久了，看到看雪上不少同学都很期待，干脆发出来。

本文大部分研究成果归功于 `mentaldease`，欲读此文，请遵守以下约定：

1. 绝不可利用本文所揭示的技术编写或者辅助编写盗号木马，危害中国网民。
2. 也不可将本文所揭示的技术直接或间接传授给写盗号木马的败类。
3. 违背以上两点约定者，天打雷劈，断子绝孙...(以下省略 2000 字)。

\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*

不能遵守以上三点约定者，请立即离开！

\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*

在分析 `QQ` 之前，尝试直接用低级键盘钩子截获密码。

笔者发现可以获取正确的 `QQ` 号，但是截获的 `QQ` 密码里面却有很多虚假按键：

![](img/547cd35a95bb5f85e4449443f296e408.jpg)

事实上，只要将鼠标焦点放在密码输入框里，`QQ` 就自动产生虚假按键。所以，靠低级键盘钩子不能截获正确的 `QQ` 密码。

用 `XueTr` 查看系统中的消息钩子。笔者发现只要用户双击打开 `QQ2009` 登录窗口，`QQ` 就会创建一个 `WH_KEYBOARD_LL` 低级键盘钩子，如图 2

![](img/c7c85183305046582c7ae56f42ca1860.jpg)

我们知道 `WH_KEYBOARD_LL` 钩子的优先级高于 `WH_KEYBOARD` 和 `WH_GETMESSAGE` 钩子。显然 `QQ2009` 的低级键盘钩子会比其他类型的钩子先执行，`QQ` 获取用户的真实按键后再伪造虚假按键。

笔者在成功截获 `QQ2009` 密码之后，挂钩了 `SetWindowHookEx` 观察 `QQ` 创建 `WH_KEYBOARD_LL` 钩子的过程：

![](img/ebbceb23c20c5a8a733b3e31c4e8821d.jpg)

由上图可见 `QQ2009` 每隔不到 0.2 秒就创建一个低级键盘钩子，这样确保自己的钩子始终在钩子队列的最顶层。

我们对 `SetWindowHookEx` 函数下断，但是断不下来，显然是 `QQ` 在这里有反调试。删除刚才的断点，下一个硬件断点，程序断下，执行到返回：

来到 `042437C5` 处，`tssafeed` 模块中看到如下指令：

```
042437BB    56               push    esi
042437BC    50               push    eax
042437BD    FF75 0C          push    dword ptr [ebp+C]
042437C0    E8 AB550000     call    04248D70             ; jmp 到 MSVCRT.memcmp
042437C5    8B3D 28A02404   mov     edi, dword ptr [424A028]  ; KERNEL32.GetCurrentProcess
042437CB    83C4 0C         add     esp, 0C
042437CE    85C0            test    eax, eax
042437D0    0F84 8F000000   je      04243865
```

猜测这里很可能就是用 `memcmp` 函数校验 `SetWindowsHookExA`(可能还有其他函数)的代码。在 `042437BB` 处 `F2`，再 `F9` 让程序运行，把鼠标焦点移到密码输入框时，断下来了：

![](img/b9457253a22a0b85b2d96c1c90485da1.jpg)

单步执行到 `042437C0`，看堆栈信息：

![](img/32b1208b8e7e52c9fa80bf13c6f8fe41.jpg)

果然是对 `SetWindowsHookExA` 初的内存进行校验。`memcmp` 返回 `0` 则认为该处内存没有改变。

在 `77D311D1` 处下硬件执行断点，成功断下后，堆栈信息如图：

![](img/6e8f80dba5aa6e5d1e3e1be48d930632.jpg)

`HookType` 为 `13`，即是 `WH_KEYBOARD_LL` 钩子，其 `HookProc` 为 `04260F1B`，转到该地址看看 `QQ` 的钩子回调例程：

```
04260F1B    55               push    ebp
04260F1C    8BEC             mov     ebp, esp
04260F1E    83EC 30          sub     esp, 30
04260F21    56               push    esi
04260F22    57               push    edi
04260F23    C745 FC F00C260>mov     dword ptr [ebp-4], 4260CF0
04260F2A    8D7D D0          lea     edi, dword ptr [ebp-30]
04260F2D    B9 07000000      mov     ecx, 7
04260F32    33C0             xor     eax, eax
04260F34    FC               cld
04260F35    F3:AB            rep     stos dword ptr es:[edi]
04260F37    8B45 10          mov     eax, dword ptr [ebp+10]
04260F3A    8945 F8          mov     dword ptr [ebp-8], eax
04260F3D    C745 F4 0000000>mov     dword ptr [ebp-C], 0
04260F44    E8 20000000      call    04260F69             ; 跟进

… …
```

`04260F69` 处的 `call`：

```
04260F69    5F               pop     edi
04260F6A    8B4D F4          mov     ecx, dword ptr [ebp-C]
04260F6D    833C8F 00        cmp     dword ptr [edi+ecx*4], 0
04260F71    74 1D            je      short 04260F90
04260F73    8B55 F4          mov     edx, dword ptr [ebp-C]
04260F76    8B0497           mov     eax, dword ptr [edi+edx*4]
04260F79    50               push    eax
04260F7A    8B4D FC          mov     ecx, dword ptr [ebp-4]
04260F7D    FF51 50          call    dword ptr [ecx+50]    ; USER32.GetAsyncKeyState
04260F80    0FBFD0           movsx   edx, ax
04260F83    81E2 00800000    and     edx, 8000
04260F89    75 05            jnz     short 04260F90
04260F8B    FF45 F4          inc     dword ptr [ebp-C]
04260F8E    ^ EB DA          jmp     short 04260F6A
04260F90    8B55 F8          mov     edx, dword ptr [ebp-8]
04260F93    8B02             mov     eax, dword ptr [edx]
04260F95    C745 F0 0000000>mov     dword ptr [ebp-10], 0
04260F9C    E8 3C000000      call    04260FDD              ;跟进

… …
```

这个函数中有很多花指令，这里就不仔细分析了，对 `GetForegroundWindow` 下断，只要将鼠标焦点移到 `QQ` 登录窗口，就会断下，再对 `CallNextHookEx` 下断观察。它的大概功能猜也猜得出来：`QQ` 自己获取按键之后，发送虚假按键，然后直接返回。现在就是找出 `QQ2009` 到底是用什么函数发送虚假按键。对 `keybd_event` 下硬件执行断点，没有断下，对 `sendInput` 下硬件断点，断下了：

![](img/e8c834349d6a20bb0085215016688ecf.jpg)

`Sendinput` 函数原型：

```
UINT SendInput(
  UINT nInputs,     // count of input events
  LPINPUT pInputs,  // array of input events
  int cbSize        // size of structure
);

typedef struct tagINPUT {
  DWORD   type;
  union {
      MOUSEINPUT      mi;
      KEYBDINPUT      ki;
      HARDWAREINPUT   hi;
  };
} INPUT, *PINPUT;
```

观察堆栈：

![](img/dd4dbed7b7a498fd64c7ac6e048d7cd5.jpg)

数据窗口跟随到 `0012FA94`：

![](img/423b171eeb8c273aec3a145d3f905912.jpg)

如图所示 `01` 表示的是输入类型 `INPUT_KEYBOARD`，`pInouts` 的结构如下：

```
typedef struct tagKEYBDINPUT {
  WORD      wVk;
  WORD      wScan;
  DWORD     dwFlags;
  DWORD     time;
  ULONG_PTR dwExtraInfo;
} KEYBDINPUT, *PKEYBDINPUT;
```

如图所示 `0x38` 对应的虚拟按键是数字 `8`，用我们刚才写的那个低级键盘钩子可以看到，所接收到的虚假按键正是 `8`。

![](img/a9035566d8ac3002a90c27faa4175bcb.jpg)

现在我们能够写出 `QQ` 的低级键盘钩子处理例程：

```
HWND hwnd = GetForegroundWindow();

If( hWnd == hPassEditWnd)
{
  MapVirtualKey();
  SendInput();
  return  1;
}
else
{
  return  CallNextHookEx(ncode,…);
}
```

现在 `2009` 登录窗口保护的原理已经分析清楚，获取 `QQ` 密码的方法也就有了。其关键就是绕过 `QQ` 对 `SetWindowsHookExA` 等函数的内存校验，然后 `hook` `SetWindowsHookExA`，判断如果是 `QQ` 的 `WH_KEYBOARD_LL` 钩子，就替换其回调函数，然后在 `my_KeyboardllProc` 中截获密码，下图是截获密码的效果图：

![](img/9ae20e807726a97dff60605440a67c3e.jpg)

`PS`：`QQ2009` 的软键盘也被我和 `mentaldease` 兄突破，并成功截获密码：

![](img/10408c628d04832021e41d21cea13ce3.jpg)
