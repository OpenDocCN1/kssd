
# 样本的简单分析

> 原文：[`www.pediy.com/kssd/pediy12/143227.html`](https://www.pediy.com/kssd/pediy12/143227.html)

找了个样本简单分析下，希望能在看雪继续学习。

`PEID` 打开文件只有两个节未知~

输入表很少 `CreateThread` `ExitProcess` `Sleep` `WaitForSingleObject`

发现调用了 `gdi32.dll` 函数 `EnumFontsA` 上网查 是一个例举指定设备可用的字体

`user32.dll` 里的 `GetDCEx` 检索指定窗口客户区域或整个屏幕的显示设备上下文环境的句柄，在随后的 GDI 函数中可以使用该句柄在设备上下文环境中绘图。

用 `od` 看看把

代码:

```
001030EF  >  $  B8 29090000   mov     eax, 929
001030F4   .  40            inc     eax
001030F5   .  B9 58010000   mov     ecx, 158
001030FA   .  BB 3A150000   mov     ebx, 153A
001030FF   .  40            inc     eax
00103100   .  40            inc     eax
00103101   .  03C0          add     eax, eax
00103103   .  40            inc     eax
00103104   .  03C1          add     eax, ecx
00103106   .  03C3          add     eax, ebx
00103108   .  40            inc     eax
00103109   .  40            inc     eax
0010310A   >  C1E8 02       shr     eax, 2
0010310D   .^ E2  FB       loopd   short 0010310A
0010310F   .  51            push    ecx
00103110   .  68  EB2E1000  push    00102EEB
00103115   .  6A  01        push    1                                             \Flags = DCX_WINDOW
00103117   .  49            dec     ecx
00103118   .  50            push    eax                                          |hRegionClip = NULL
00103119   .  41            inc     ecx
0010311A   .  51            push    ecx                                           |hWnd = NULL
0010311B   .  E8  74FDFFFF  call    00102E94                                   Call GetDCEx
00103120   .  A3  B9191000  mov     dword ptr [1019B9], eax
00103125   .^ E9  6FFDFFFF  jmp     00102E99

00102E99   > /B9 C42E1000  mov     ecx, 00102EC4
00102E9E   . |81E9 C4080000 sub     ecx, 8C4
00102EA4   . |51            push    ecx              lParam = info.00102600
00102EA5   . |68  CD2E1000  push    00102ECD          CallbackFunc = info.00102ECD
00102EAA   . |FF35 A5191000 push    dword ptr [1019A5]  FaceName = NULL
00102EB0   . |FF35 B9191000 push    dword ptr [1019B9]  hDC = 01010058
00102EB6   . |E8  D4FFFFFF  call    00102E8F     Call EnumFontsA
00102EBB   . |B8  00000000  mov     eax, 0
00102EC0   . |8BC8         mov     ecx, eax
00102EC2   . |8BD1         mov     edx, ecx
00102EC4   . |33C0         xor     eax, eax
00102EC6   . |50            push    eax                                             ; /ExitCode => 0
00102EC7   . |E8  6A020000  call    <jmp.&kernel32.ExitProcess>         ; \ExitProcess
00102ECC   . |C3            retn
00102ECD   . |8B4424 10     mov     eax, dword ptr [esp+10]         EnumFontsA 回调函数
00102ED1   . |05  C4080000  add     eax, 8C4     清空了 102EC4 到 102ECB 的数据
00102ED6   . |C700 90909090 mov     dword ptr [eax], 90909090  上面的 exitprocess 没了，原来是骗人用的
00102EDC   . |83C0 04       add     eax, 4
00102EDF   . |C700 90909090 mov     dword ptr [eax], 90909090
00102EE5   . |B8  00000000  mov     eax, 0
00102EEA   . |C3            retn
00102EEB   . /EB  5D       jmp     short 00102F4A
```

原来上面这些操作全部……，这样的思路是不是就能过了杀软的虚拟机引擎了？还是有其他目的？

回调函数去修改原程序的执行流程~下面应该到真正的地方了吧~

代码:

```
00102F4A   > \90            nop
00102F4B   .  90            nop
00102F4C   .  90            nop
00102F4D   .  BB 00000000   mov     ebx, 0
00102F52   .  B9 04000000   mov     ecx, 4
00102F57   >  53            push    ebx                                     ; |pThreadParm
00102F58   .^ E2  FD       loopd   short 00102F57                  ; |
00102F5A   .  68  992F1000  push    00102F99                        ; |ThreadFunction = info.00102F99
00102F5F   .  53            push    ebx                                        ; |StackSize
00102F60   .  53            push    ebx                                        ; |pSecurity
00102F61   .  E8  CA010000  call    <jmp.&kernel32.CreateThread>     \CreateThread
00102F66   .  A3  A1191000  mov     dword ptr [1019A1], eax
00102F6B   .  BB 00000000   mov     ebx, 0
00102F70   .  53            push    ebx
00102F71   .  53            push    ebx                                        ; /pThreadId => NULL
00102F72   .  53            push    ebx                                        ; |CreationFlags => 0
00102F73   .  53            push    ebx                                        ; |pThreadParm => NULL
00102F74   .  68  DF2F1000  push    00102FDF                          ; |ThreadFunction = info.00102FDF
00102F79   .  53            push    ebx                                        ; |StackSize => 0
00102F7A   .  53            push    ebx                                        ; |pSecurity => NULL
00102F7B   .  E8  B0010000  call    <jmp.&kernel32.CreateThread>    \CreateThread
00102F80   .  A3  A5191000  mov     dword ptr [1019A5], eax
00102F85   .  A1  A1191000  mov     eax, dword ptr [1019A1]
00102F8A   .  6A  FF        push    -1                                        ; /Timeout = INFINITE
00102F8C   .  50            push    eax                                      ; |hObject => NULL
00102F8D   .  E8  B0010000  call    <jmp.&kernel32.WaitForSingleObje>; \WaitForSingleObject
00102F92   .  6A  00        push    0                                         ; /ExitCode = 0
00102F94   .  E8  9D010000  call    <jmp.&kernel32.ExitProcess>      \ExitProcess
```

创建了两个线程，地址：`102F99` 和 `102fDf` 看看这两个线程在干吗~

代码:

```
102F99:
00102F99    8B35 B1191000   mov     esi, dword ptr [1019B1]         ; info.0010130C 看来这个地址很重要~
00102F9F    8B0D B5191000   mov     ecx, dword ptr [1019B5]        0x691 这个数是干啥用的？一般都是次数~~
00102FA5    56              push    esi
00102FA6    8BF1            mov     esi, ecx
00102FA8    56              push    esi
00102FA9    6A 01           push    1
00102FAB    E8 8C010000     call    <jmp.&kernel32.Sleep>     
00102FB0    833D AD191000 0>cmp     dword ptr [1019AD], 2  1019ad 里面存数值来判断是否线程继续执行下去~
00102FB7  ^ 75  F0          jnz     short 00102FA9       如果 不是 2，那就处于无限的 sleep 1ms 了
00102FB9    EB 15           jmp     short 00102FD0       初始为 2 跳到 102fd0

00102FD0    59              pop     ecx        0x691 要做动作了~
00102FD1    5E              pop     esi        0010130c(8c9b66f4)
00102FD2    A1 A9191000     mov     eax, dword ptr [1019A9]  eax:674E8cEB   
00102FD7    8A1E            mov     bl, byte ptr [esi]       取出 f4
00102FD9    32D8            xor     bl, al        与 67 异或  EBX:1f
00102FDB    881E            mov     byte ptr [esi], bl       结果存到 esi:0010130c 中
00102FDD  ^ EB  DC          jmp     short 00102FBB

00102FBB    46              inc     esi        esi+1  0010130D
00102FBC    50              push    eax          
00102FBD    B8 AD191000     mov     eax, 001019AD
00102FC2    C700 00000000   mov     dword ptr [eax], 0    更该了线程的执行条件，为 0。
00102FC8    58              pop     eax
00102FC9  ^ E2  DA          loopd   short 00102FA5       看来要执行 0x691 次        
00102FCB  ^ E9  61E9FFFF    jmp     00101931       回到 sleep 了！因为之前更改了。所以一直在 sleep，估计下面的线程会更改这个值
```

这个线程来看是用来解密用的了，数据大小 `0x691`  启始地址：`0010130C` 开始~解密代码段？看来 `00101931` 这个地址是解码后的执行的地址了。

再看看另一个线程 `102fDf`

代码:
