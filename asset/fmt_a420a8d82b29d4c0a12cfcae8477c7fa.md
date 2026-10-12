
# The Lighthouse Screensaver 1.0 算法分析及算法注册机

> 原文：[`www.pediy.com/kssd/pediy11/121782.html`](https://www.pediy.com/kssd/pediy11/121782.html)

【破文标题】`The Lighthouse Screensaver 1.0 算法分析及算法注册机`

【破文作者】`zaas[PYG]`

【破解工具】`OllyICE`，`PEiD v0.94`

【破解平台】`WinXP`

【软件类别】：屏幕保护

【软件授权】：共享版

【软件介绍】：一个比较酷的夜景灯塔屏保

【破解声明】我是一只小菜鸟，偶得一点心得，愿与大家分享

--------------------------------------------------------------

【破解内容】

--------------------------------------------------------------

查壳无壳：`Microsoft Visual C++ 6.0 [Overlay]`

注册失败无提示，注册成功有提示：

`Ultra String Reference`，条目 391 `Address=00415F1D`

`Disassembly=push    0044EF30`

`Text String=registered ok , please restart the program!`

来到

```
00415DDF   .  E8 8CCD0100   call    00432B70                          ;  取假码
00415DE4   .  8B46 68       mov     eax, dword ptr [esi+68]
00415DE7   .  8D7E 68       lea     edi, dword ptr [esi+68]
00415DEA   .  8378 F8 13    cmp     dword ptr [eax-8], 13             ;  注册码 0x13 位
00415DEE   .  0F85 51010000 jnz     00415F45
00415DF4   .  51            push    ecx
00415DF5   .  8BCC          mov     ecx, esp
00415DF7   .  896424 18     mov     dword ptr [esp+18], esp
00415DFB   .  68 5CEF4400   push    0044EF5C                          ;  regseed 没有意义，大概是和其他软件共用的#define
00415E00   .  E8 9EE10100   call    00433FA3                          ;  得出固定字符串"peterpan@2977"
00415E05   .  8D4C24 14     lea     ecx, dword ptr [esp+14]
...........
00415E36   .  52            push    edx
00415E37   .  C64424 34 01  mov     byte ptr [esp+34], 1
00415E3C   .  E8 69DE0100   call    00433CAA
00415E41   .  885C24 30     mov     byte ptr [esp+30], bl
00415E45   .  E8 361BFFFF   call    00407980                          ;  关键 call
00415E4A   .  83C4 08       add     esp, 8
00415E4D   .  3BC3          cmp     eax, ebx
00415E4F   .  0F84 DF000000 je      00415F34                          ;  关键跳
00415E55   .  51            push    ecx
00415E56   .  8BCC          mov     ecx, esp
00415E58   .  896424 18     mov     dword ptr [esp+18], esp
00415E5C   .  68 50EA4400   push    0044EA50                          ;  cfyscreensaverbuilder24
00415E61   .  E8 3DE10100   call    00433FA3
00415E66   .  8B46 60       mov     eax, dword ptr [esi+60]
00415E69   .  51            push    ecx
00415E6A   .  83C0 5C       add     eax, 5C
00415E6D   .  8BCC          mov     ecx, esp
00415E6F   .  896424 20     mov     dword ptr [esp+20], esp
00415E73   .  50            push    eax
00415E74   .  C64424 34 02  mov     byte ptr [esp+34], 2
00415E79   .  E8 2CDE0100   call    00433CAA
00415E7E   .  8D4C24 14     lea     ecx, dword ptr [esp+14]
00415E82   .  885C24 30     mov     byte ptr [esp+30], bl
00415E86   .  51            push    ecx
00415E87   .  E8 1417FFFF   call    004075A0                          ;  二次加密运算出写注册表的内容
00415E8C   .  83C4 08       add     esp, 8
00415E8F   .  8D5424 10     lea     edx, dword ptr [esp+10]
00415E93   .  8BCC          mov     ecx, esp
00415E95   .  896424 1C     mov     dword ptr [esp+1C], esp
00415E99   .  52            push    edx
00415E9A   .  C64424 30 03  mov     byte ptr [esp+30], 3
00415E9F   .  E8 06DE0100   call    00433CAA
00415EA4   .  8B46 60       mov     eax, dword ptr [esi+60]
00415EA7   .  51            push    ecx
00415EA8   .  83C0 5C       add     eax, 5C
00415EAB   .  8BCC          mov     ecx, esp
00415EAD   .  896424 1C     mov     dword ptr [esp+1C], esp
00415EB1   .  50            push    eax
00415EB2   .  C64424 34 04  mov     byte ptr [esp+34], 4
00415EB7   .  E8 EEDD0100   call    00433CAA                          ;  变形出 4 组，写入注册表
00415EBC   .  51            push    ecx
00415EBD   .  8D5424 18     lea     edx, dword ptr [esp+18]
00415EC1   .  8BCC          mov     ecx, esp
00415EC3   .  896424 28     mov     dword ptr [esp+28], esp
00415EC7   .  52            push    edx
00415EC8   .  68 34EA4400   push    0044EA34                          ;  software\microsoft\windows\
00415ECD   .  51            push    ecx
...........
00415F08   >  33C0          xor     eax, eax
00415F0A   >  8B40 1C       mov     eax, dword ptr [eax+1C]
00415F0D   .  53            push    ebx                           ; /lParam
00415F0E   .  53            push    ebx                           ; |wParam
00415F0F   .  68 CE070000   push    7CE                           ; |Message = MSG(7CE)
00415F14   .  50            push    eax                           ; |hWnd
00415F15   .  FF15 20054400 call    dword ptr [<&USER32.PostMessageA>; \PostMessageA
00415F1B   .  53            push    ebx
00415F1C   .  53            push    ebx
00415F1D   .  68 30EF4400   push    0044EF30                          ;  registered ok , please restart the program!
00415F22   .  E8 4D1B0200   call    00437A74                          ;  (initial cpu selection)
```

此处可见，注册成功才会在注册表里边写内容，但是和注册码无关。

因此这里有漏洞，只要跳过去了，注册表里写的内容启动验证就能成功。

注册表里写的内容我没有分析，胡乱修改注册表的键值也可以，但 `key` 不能删掉。

因此用 `dup` 做一个注册表的 `patch` 是破解这软件的最省事儿的办法。

但是，我的目的是研究算法，

自然的，跟进 `00415E45` 看看关键 `call`：

代码:
