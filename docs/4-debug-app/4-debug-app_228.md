

# 分析 QQ，查找聊天窗口中的聊天信息在内存中的位置

> 原文：[`www.pediy.com/kssd/pediy11/124281.html`](https://www.pediy.com/kssd/pediy11/124281.html)

\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
\*标题:【原创】【原创】分析 QQ，查找聊天窗口中的聊天信息在内存中的位置 \*  
\*作者:踏雪流云                                                         \*  
\*日期:2010 年 11 月 6 号                                                   \*  
\*声明:本文章的目的仅为技术交流讨论                                     \*  
\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
曾设想：如果暗中替换掉用户输入的聊天信息，这样不就可以利用聊天窗口暗中发送自己的信息了；于是，便有了此文。  
首先，OD 加载 QQ，运行后，打开一个聊天窗口，回到 OD，查看 ChatFrame 模块，找到如下位置：  

代码:

```
6152FAD9    3D CB000000     cmp     eax, 0CB                         ; CB 消息
6152FADE    0F85 6F0E0000   jnz     61530953
6152FAE4    33C0            xor     eax, eax                         ; RichEdit 有输入
6152FAE6    8BCE            mov     ecx, esi
6152FAE8    8379 10 0D      cmp     dword ptr \[ecx+10\], 0D           ; 判断是否是回车
6152FAEC    6A 10           push    10                               ; shift+enter 换行
6152FAEE    0F94C0          sete    al
6152FAF1    8BF8            mov     edi, eax
6152FAF3    FF15 38B85C61   call    dword ptr \[<&USER32.GetKeyState>>; user32.GetKeyState

```

这是 RichEdit 有字符输入的响应位置，在 6152FAE4 地址下断；回到聊天窗口，在空消息的情况下按回车键（发送），为什么要按回车呢？这是因为回车会将聊天信息发送出去，这就会将 RichEdit 中的信息保存到内存了，而在空信息的状态下回车，会弹出提示“发送内容不能为空，请重新输入。”，说明 QQ 会检测输入信息的长度，我们正好可以利用此方法，来找到聊天信息的保存位置。  
按下回车后，断点命中，一路 F8，直到弹出“发送内容不能为空，请重新输入。”提示；然后，步进那个函数，再一路 F8，重复上述操作，直到找到判断信息是否为空的跳转为止。最终位置在 AFCtrl 模块：  
代码:

```
02C6C4D5    8B0F            mov     ecx, dword ptr \[edi\]
02C6C4D7    8B81 5C040000   mov     eax, dword ptr \[ecx+45C\]
02C6C4DD    8D95 68FEFFFF   lea     edx, dword ptr \[ebp-198\]
02C6C4E3    52              push    edx
02C6C4E4    57              push    edi
02C6C4E5    FFD0            call    eax
02C6C4E7    8B8D 68FEFFFF   mov     ecx, dword ptr \[ebp-198\]
02C6C4ED    3BCB            cmp     ecx, ebx
02C6C4EF    7F 23           jg      short 02C6C514                   ; 消息不为空，跳转

```

02C6C4E5    FFD0            call    eax 位置，为判断信息是否为空的函数；在此处下断，回到聊天窗口输入 123456789，回车后断点命中，一路 F7，来到如下位置：  
代码:

```
30883A90    55              push    ebp
30883A91    8BEC            mov     ebp, esp
30883A93    6A FF           push    -1
30883A95    68 A0ED9A30     push    309AEDA0
30883A9A    64:A1 00000000  mov     eax, dword ptr fs:\[0\]
30883AA0    50              push    eax
30883AA1    83EC 0C         sub     esp, 0C
30883AA4    53              push    ebx
30883AA5    56              push    esi
30883AA6    57              push    edi
30883AA7    A1 E408A530     mov     eax, dword ptr \[30A508E4\]
30883AAC    33C5            xor     eax, ebp
30883AAE    50              push    eax
30883AAF    8D45 F4         lea     eax, dword ptr \[ebp-C\]
30883AB2    64:A3 00000000  mov     dword ptr fs:\[0\], eax
30883AB8    8BF9            mov     edi, ecx
30883ABA    897D EC         mov     dword ptr \[ebp-14\], edi
30883ABD    8BB7 30020000   mov     esi, dword ptr \[edi+230\]
30883AC3    33DB            xor     ebx, ebx
30883AC5    3BF3            cmp     esi, ebx
30883AC7    895D FC         mov     dword ptr \[ebp-4\], ebx
30883ACA    8975 E8         mov     dword ptr \[ebp-18\], esi
30883ACD    0F84 EA000000   je      30883BBD
30883AD3    8B06            mov     eax, dword ptr \[esi\]
30883AD5    8B48 04         mov     ecx, dword ptr \[eax+4\]
30883AD8    56              push    esi
30883AD9    FFD1            call    ecx
30883ADB    399F 94020000   cmp     dword ptr \[edi+294\], ebx
30883AE1    75 09           jnz     short 30883AEC
30883AE3    53              push    ebx
30883AE4    E8 17B1FFFF     call    3087EC00
30883AE9    83C4 04         add     esp, 4
30883AEC    8B7D 10         mov     edi, dword ptr \[ebp+10\]
30883AEF    8B4D 08         mov     ecx, dword ptr \[ebp+8\]
30883AF2    8D45 F0         lea     eax, dword ptr \[ebp-10\]
30883AF5    50              push    eax
30883AF6    895D F0         mov     dword ptr \[ebp-10\], ebx
30883AF9    8B5D 0C         mov     ebx, dword ptr \[ebp+C\]
30883AFC    8B16            mov     edx, dword ptr \[esi\]
30883AFE    8B52 20         mov     edx, dword ptr \[edx+20\]
30883B01    57              push    edi
30883B02    53              push    ebx
30883B03    51              push    ecx
30883B04    56              push    esi
30883B05    FFD2            call    edx
30883B07    8B4D 14         mov     ecx, dword ptr \[ebp+14\]
30883B0A    85C9            test    ecx, ecx
30883B0C    8B55 F0         mov     edx, dword ptr \[ebp-10\]
30883B0F    8945 10         mov     dword ptr \[ebp+10\], eax
30883B12    74 02           je      short 30883B16
30883B14    8911            mov     dword ptr \[ecx\], edx             ; 这里长度赋值
30883B16    817D 08 E104000>cmp     dword ptr \[ebp+8\], 4E1
30883B1D    75 6E           jnz     short 30883B8D
30883B1F    85D2            test    edx, edx
30883B21    74 6A           je      short 30883B8D
30883B23    85C0            test    eax, eax
30883B25    75 66           jnz     short 30883B8D
30883B27    8945 08         mov     dword ptr \[ebp+8\], eax
30883B2A    8D45 08         lea     eax, dword ptr \[ebp+8\]
30883B2D    50              push    eax
30883B2E    C645 FC 01      mov     byte ptr \[ebp-4\], 1
30883B32    FF15 F4609C30   call    dword ptr \[<&Common.Util::Data::>; Common.Util::Data::CreateTXData

```

一路 F8，并注意寄存器的值，我们要找到长度 9 的出现位置，即  
代码:

```
30883B14    8911            mov     dword ptr \[ecx\], edx             ; 这里长度赋值

```

位置，这里 edx 保存了信息的长度。  
接下来，在 30883B05    FFD2            call    edx 处下断；然后，一路跟进，时刻注意 9 的出现，最终我们发现得到长度的代码位置：  
代码:

```
39704EFB    8B81 D4000000   mov     eax, dword ptr \[ecx+D4\]          ; 长度在这个地址
39704F01    8B49 78         mov     ecx, dword ptr \[ecx+78\]
39704F04    F6C1 01         test    cl, 1
39704F07    74 0F           je      short 39704F18
39704F09    F7C1 00000800   test    ecx, 80000
39704F0F    6A 00           push    0
39704F11    59              pop     ecx
39704F12    0F95C1          setne   cl
39704F15    41              inc     ecx
39704F16    2BC1            sub     eax, ecx
39704F18    C3              retn

```

来到 39704EFB    8B81 D4000000   mov     eax, dword ptr \[ecx+D4\]          ; 代码处，数据窗口跟随 ecx+D4 地址，便找到了聊天信息的长度；这时，在数据窗口 ctrl+B，查找”123456789“字串，马上就找到了，附近位置还有此聊天窗口显示的聊天信息。  
修改聊天信息内容或信息长度，发送出去的数据会随之变化，但缺陷是本地显示的信息也会变化，说明这是发送数据和本地显示数据的共同来源。试图想找到发送数据的直接来源，而不影响本地显示数据，未果![](img/bedc2a7572d1054a9590e607b9635bdd.jpg "皱眉")。  
本人菜鸟一个，献丑了。。。![](img/9b2aec7f3d4bb041b2d419a59832192f.jpg "恐怖!")