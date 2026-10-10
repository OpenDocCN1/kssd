
# 兼容 VC，Softice 快捷键标准的 ollydbg,F5,F8,F10,Ctrl+F5

> 原文：[`www.pediy.com/kssd/pediy08/pediy8-521.htm`](https://www.pediy.com/kssd/pediy08/pediy8-521.htm)

`ollydbg` 的快捷键是按 Borland 公司的调试器规则定义的，但没有让人设置，所以不太方便一向用 VC,Softice 的人的习惯使用，

为此，我决定改变一下它，让它可以比较完美的支持 VC,Softice 的快捷键风格，同时把修改过程 Post 出来，让更多人可以 DIY `ollydbg` 一些功能。

以下我们先看有那几个键和 VC，Softice 有差别：

Ollydbg      VC    Softice

Run        `F9`        `F5`      `F5`

Break      `Alt+B`      `F9`      `F9`

P          `F8`        `F10`      `F10`

T          `F7`        `F11`      `F8`

Restart  `Ctrl+F9`   `Ctrl+F5`  -----

我们的目的就是要对 `ollydbg` 进行完美的修改，让它支持 Softice 的快捷键盘，

以下是 Windows 系统 101 键盘按键的定义:

```
#define VK_F1             0x70
#define VK_F2             0x71
#define VK_F3             0x72
#define VK_F4             0x73
#define VK_F5             0x74
#define VK_F6             0x75
#define VK_F7             0x76
#define VK_F8             0x77
#define VK_F9             0x78
#define VK_F10            0x79
#define VK_F11            0x7A
#define VK_F12            0x7B
```

以下是 Windows 系统键盘按键消息的定义:

```
#define WM_KEYFIRST                     0x0100
#define WM_KEYDOWN                      0x0100
#define WM_KEYUP                        0x0101
#define WM_CHAR                         0x0102
#define WM_DEADCHAR                     0x0103
#define WM_SYSKEYDOWN                   0x0104
#define WM_SYSKEYUP                     0x0105
#define WM_SYSCHAR                      0x0106
#define WM_SYSDEADCHAR                  0x0107
#define WM_KEYLAST                      0x0108
```

现在准备好 `Ida Pro 5.0`,`Ultra Edit`,现在开始找代码修改位置，我先在 `ollydbg` 试了一下，发现 `F5` 已经给一个窗口处理过程占用了，

`F10` 也给弹出菜单过程占用,因为这两个功能都不是什么常用的东西，好，先让它们搬搬家，方法就是 `F5->F4`(好象没有给使用),

`F10->F11`(好象没有给使用）,搬家过程很简单，对着 `ollydbg` 说声，我可爱的 `ollydbg`，请你把它们移移位置吧。。。

打开 `Ida Pro`，把 `ollydbg.exe` 载入去，反汇编，直接到以下地址：

## F5->F4 搬家过程：

```
.text:00431E45                  cmp     eax, 100h
.text:00431E4A                  jnz     short loc_431EAB
.text:00431E4C                  cmp     edx, 74h                    ;把这里改为 73h(VK_F5->VK_F4)
.text:00431E4F                  jnz     short loc_431EAB
.text:00431E51                  test    ecx, ecx
.text:00431E53                  jnz     short loc_431EAB
.text:00431E55                  lea     eax, [ebp+lParam]  ; int
.text:00431E58                  push    eax             ; lParam
.text:00431E59                  push    0               ; wParam
.text:00431E5B                  push    229h            ; Msg
.text:00431E60                  mov     edx, hWndParent
.text:00431E66                  push    edx             ; hWnd
.text:00431E67                  call    SendMessageA
```

## F10->F11 搬家过程：

```
.text:004592F2 loc_4592F2:                             ; CODE XREF: _Tablefunction+AF2j
.text:004592F2                  cmp     [ebp+uMsg], 104h
.text:004592F9                  jz      short loc_459304
.text:004592FB                  cmp     [ebp+uMsg], 47Ah
.text:00459302                  jnz     short loc_459355
.text:00459304
.text:00459304 loc_459304:                             ; CODE XREF: _Tablefunction+F9Dj
.text:00459304                  cmp     [ebp+arg_C], 79h            ;把这里改为 79h(VK_F10->VK_F11,不过后来发现好象 F11 不能实现 F10 的作用，但因为这功能没有多大用作，不浪费时间研究了)
.text:00459308                  jnz     short loc_459338
.text:0045930A                  mov     dword_4BB408, 1
.text:00459314                  push    0               ; lParam
.text:00459316                  push    0               ; wParam
.text:00459318                  push    465h            ; Msg
.text:0045931D                  mov     edx, [ebp+hWnd]
.text:00459320                  push    edx             ; hWnd
.text:00459321                  call    SendMessageA
```

现在冲突的键都移除了，可以把我们要的 Softice 的按键重定义了:

## F8->F10:

```
.text:00431F80                  cmp     eax, 100h                  ;改为 104h
.text:00431F85                  jnz     short loc_431FB8
.text:00431F87                  cmp     edx, 77h                    ;改为 79h
.text:00431F8A                  jnz     short loc_431FB8
.text:00431F8C                  test    ecx, ecx
.text:00431F8E                  jz      short loc_431F9A
.text:00431F90                  push    2
.text:00431F92                  call    _Animate
```

## F7->F8:

```
.text:00431F48                  cmp     eax, 100h
.text:00431F4D                  jnz     short loc_431F80
.text:00431F4F                  cmp     edx, 76h                    ;改为 77h
.text:00431F52                  jnz     short loc_431F80
.text:00431F54                  test    ecx, ecx
.text:00431F56                  jz      short loc_431F62
.text:00431F58                  push    1
.text:00431F5A                  call    _Animate
```

## F9->F5:

```
.text:00431FB8                  cmp     eax, 100h
.text:00431FBD                  jnz     short loc_432004
.text:00431FBF                  cmp     edx, 78h                    ;改为 74h
.text:00431FC2                  jnz     short loc_432004
.text:00431FC4                  test    ecx, ecx
.text:00431FC6                  jz      short loc_431FE6
.text:00431FC8                  push    3
.text:00431FCA                  call    _Animate
.text:00431FCF                  pop     ecx
.text:00431FD0                  push    1               ; int
.text:00431FD2                  push    esi             ; int
.text:00431FD3                  push    2               ; int
.text:00431FD5                  push    0               ; int
.text:00431FD7                  push    0               ; int
.text:00431FD9                  call    _Go
```

再改：

```
.text:00432004 loc_432004:                             ; CODE XREF: sub_431B90+42Dj
.text:00432004                                         ; sub_431B90+432j
.text:00432004                  cmp     eax, 104h
.text:00432009                  jnz     short loc_43202E
.text:0043200B                  cmp     edx, 78h                    ;改为 74h
.text:0043200E                  jnz     short loc_43202E
.text:00432010                  push    5
.text:00432012                  call    _Animate
.text:00432017                  pop     ecx
.text:00432018                  push    1               ; int
.text:0043201A                  push    esi             ; int
.text:0043201B                  push    2               ; int
.text:0043201D                  push    0               ; int
.text:0043201F                  push    0               ; int
.text:00432021                  call    _Go
.text:00432026                  add     esp, 14h
.text:00432029                  jmp     loc_4322AC
```

## Ctrl+F9->Ctrl+F5:

```
.text:00431D3B                  cmp     eax, 100h
.text:00431D40                  jnz     short loc_431D5F
.text:00431D42                  cmp     edx, 71h                    ;改为 74h
.text:00431D45                  jnz     short loc_431D5F
.text:00431D47                  test    ecx, ecx
.text:00431D49                  jz      short loc_431D5F
.text:00431D4B                  push    0FFFFFFFFh
.text:00431D4D                  push    offset byte_4D5B80
.text:00431D52                  call    _OpenEXEfile
.text:00431D57                  add     esp, 8
.text:00431D5A                  jmp     loc_4322AC
.text:00431D5F loc_431D5F:                             ; CODE XREF: sub_431B90+1B0j
.text:00431D5F                                         ; sub_431B90+1B5j ...
.text:00431D5F                  cmp     eax, 104h
.text:00431D64                  jnz     short loc_431D96
.text:00431D66                  cmp     edx, 71h                    ;改为 74h
.text:00431D69                  jnz     short loc_431D96
.text:00431D6B                  push    1               ; int
.text:00431D6D                  call    sub_4758A4
.text:00431D72                  pop     ecx
```



.text:00431D73                 test    eax, eax  

.text:00431D75                 jnz     loc\_4322AC  

.text:00431D7B                 call    sub\_47540C  

.text:00431D80                 push    0  

.text:00431D82                 push    0  

.text:00431D84                 push    474h  

.text:00431D89                 call    \_Broadcast  

.text:00431D8E                 add     esp, 0Ch  

.text:00431D91                 jmp     loc\_4322AC  

F2->F9

```
.text:0041F8CB                 cmp     [ebp+c], 71h                ;改为 78h
.text:0041F8CF                 jnz     short loc_41F91D
```

再改:

```
.text:004199ED                 cmp     [ebp+arg_4], 71h            ;改为 78h
.text:004199F1                 jnz     loc_419C1B
```

这样全部设置都按 Softice 的设置了，需要还差了一些些，不过好象不过区别不大了,最后，为了完美收场，打开 VC6,修改菜单说明。

终于结局，呵呵。。。。好多年没有写文章了，这一次有机会，就顺手把操作过程写一下，不到之处还请多多见谅，因为看雪那天

问我要新年的礼物，我手头都是半成品，也给不出一个新年礼物，这里就当是一个新年礼物吧。

Cpu

2006 年 12 月 29 日

修正一些 F2->F9 的问题：

刚测试，源代码窗设置断点有问题，发现对于 F2->F9 还有好几处地方要修改：

```
.text:00494AA4                 cmp     ebx, 71h                    ;改为 78h
.text:00494AA7                 jnz     short loc_494AF9
.text:00494FD6                 push    1
.text:00494FD8                 push    71h                         ;改为 78h
.text:00494FDA                 mov     edx, [ebp+var_14]
.text:00494FDD                 push    edx
.text:00494FDE                 call    sub_419974
```

源代码菜单设置断点有问题，以下修改:

```
.text:00494FAD                 push    71h                         ;改为 78h
.text:00494FAF                 mov     eax, [ebp+var_14]
```

再改：

```
.text:0041998F                 cmp     [ebp+arg_4], 71h            ;改为 78h
.text:00419993                 jnz     short loc_4199A7
```
