
# 分析

0045CFE4    85C0            test    eax, eax

0045CFE6    74 75           je      short 0045D05D

0045CFE8    B3 01           mov     bl, 1

0045CFEA >  837F 04 12      cmp     dword ptr [edi+4], 12

0045CFEE    74 66           je      short 0045D056

……

0045D049    E8 06A4FAFF     call    <jmp.&USER32.TranslateMessage>

0045D04E    57              push    edi

0045D04F    E8 F89FFAFF     call    <jmp.&USER32.DispatchMessageA>

`0045CFEA`处`EDI`指向的结构是`MSG`结构体

```c
typedef struct tagMSG {
HWND hwnd;
UINT message;
WPARAM wParam;
LPARAM lParam;
DWORD time;
POINT pt; }
MSG;
```

`[edi+4]`表示的是`message`，那么`0x12`消息对应的是`WM_QUIT`，通过分析，代码中并没有对`WM_TIMER(0x113)`做处理，统一交由系统`DispatchMessageA`处理。

这里需要做的工作就是定位不同消息对应的响应函数。

```
77D1872A    8088 B40F0000 0>or      byte ptr [eax+FB4], 1
77D18731    FF55 08         call    dword ptr [ebp+8]
```

在内存区域发现

```
01110D24  E8 DB F2 FF FF 04 60 43 00 F4 44 FD 00 E8 CE F2   枸? `C.鬌?栉
01110D34  FF FF 04 60 43 00 80 EA FC 00 E8 C1 F2 FF FF 04    `C.€挈.枇?
```

对应的是处理函数

```
01110D24    E8 DBF2FFFF     call    01110004
01110D29    04 60           add     al, 60
01110D2B    43              inc     ebx
01110D2C    00F4            add     ah, dh
01110D2E    44              inc     esp
```

定位到需要的`WM_TIMER`消息响应函数在

```
01110D31    E8 CEF2FFFF     call    01110004
01110D36    04 60           add     al, 60
01110D38    43              inc     ebx
01110D39    0080 EAFC00E8   add     byte ptr [eax+E800FCEA], al
01110004    59              pop     ecx                              ; 01110D36
01110005  - E9 123D31FF     jmp     dump4-9-.00423D1C
00423D1C    55              push    ebp
00423D1D    8BEC            mov     ebp, esp
00423D1F    31C0            xor     eax, eax
00423D21    50              push    eax
00423D22    FF75 14         push    dword ptr [ebp+14]
00423D25    FF75 10         push    dword ptr [ebp+10]
00423D28    FF75 0C         push    dword ptr [ebp+C]
00423D2B    89E2            mov     edx, esp
00423D2D    8B41 04         mov     eax, dword ptr [ecx+4]
00423D30    FF11          call    dword ptr [ecx]            ; dump4-9-.00436004
```

此时`ecx`指向的相对跳转代码后的是个字节

```
01110D21  00 00 00 E8 DB F2 FF FF 04 60 43 00 F4 44 FD 00   ...枸? `C.鬌?
01110D31  E8 CE F2 FF FF 04 60 43 00 80 EA FC 00 E8 C1 F2   栉? `C.€挈.枇
01110D41  FF FF 04 60 43 00 38 EA FC 00 E8 B4 F2 FF FF 04    `C.8挈.璐?
01110D51  60 43 00 90 58 FD 00 E8 A7 F2 FF FF 04 60 43 00   `C.怷?瑙? `C.
01110D61  70 6F FD 00                                      po?铓
0012FE38  /0012FE64
0012FE3C  |77D18734  返回到 USER32.77D18734
0012FE40  |000C01AE
0012FE44  |0000C0D4
0012FE48  |00000000
0012FE4C  |00000000
0012FE50  |01110D31
0012FE54  |DCBAABCD
0012FE58  |00000000
0012FE5C  |0012FEA0
0012FE60  |01110D31
```

这样，对于每一个`TPUtilWindow`发出的每一个消息都可以对于到一个函数中，这样建立一个已知的消息响应函数表。每一个消息都可以准确的定位到自己的响应函数。

继续分析对`WM_TIMER`处理，下面的代码现实了对`TPUtilWindow`发出的每一个消息进行分发处理。

```
00436004    55              push    ebp
00436005    8BEC            mov     ebp, esp
00436007    51              push    ecx
00436008    53              push    ebx
00436009    56              push    esi
0043600A    57              push    edi
0043600B    8BDA            mov     ebx, edx
0043600D    8945 FC         mov     dword ptr [ebp-4], eax
00436010    8B33            mov     esi, dword ptr [ebx]
00436012 >  81FE 13010000   cmp     esi, 113
00436018    75 3F           jnz     short 00436059
；如果发送的是WM_TIMER消息，就去执行004039BC，否则跳转
……
0043602F    E8 88D9FCFF     call    004039BC
……
0043603C    EB 33           jmp     short 00436071
0043603E  ^ E9 5DDCFCFF     jmp     00403CA0
……
00436057    EB 18           jmp     short 00436071
00436059    8B43 08         mov     eax, dword ptr [ebx+8]
……
00436069    E8 AE0FFDFF     call    <jmp.&USER32.DefWindowProcA>
0043606E    8943 0C         mov     dword ptr [ebx+C], eax
00436071    5F              pop     edi
00436072    5E              pop     esi
00436073    5B              pop     ebx
00436074    59              pop     ecx
00436075    5D              pop     ebp
00436076    C3              retn
```

通过上面的分析，可以知道程序通过`WM_TIMER`消息设置循环校验，也使用`WM_TIMER`消息发送`WM_QUIT`消息。并通过上面的代码分发给不同的处理函数。

# 总结软件的保护

MD5对目录下文件校验

校验配置文件

未知？

## 三、破除保护机制

### 第一、破除文件MD5校验

因为校验的值在文件中可以找到，通过重新计算得数值，重写到文件中，到比如下面所示

```
UPX1:00492F8C off_0_492F8C    dd offset MD5_File_A    ;
"......."
UPX1:00492F90 off_0_492F90    dd offset MD5_File_B
"......"
```

修改关键跳转，即是在校验不对的时候也跳往正常的程序流程，不过这样做可能导致意想不到的错误。在自己修正这个程序的自校验时，由于爆破的位置不对，带来了许多问题。

### 第二、破除SetTimer消息

1）破除壳代码的校验，不会产生退出的信号

2）设置退出的时间，可能程序运行几天才可以退出，呵呵。。。

3）其它

# 参考文章

《加密解密 二版》 第五章 软件保护技术及其弱点 时间限制

标准SHA1算法识别技巧cocoruder 4daK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2^5k6X3!0U0N6i4y4Q4x3X3g2F1k6i4c8Q4x3V1k6T1j5Y4y4Q4x3V1k6A6L8X3c8W2P5q4)9J5k6i4m8Z5M7q4)9K6c8X3q4U0N6q4)9K6c8q4y4q4i4K6t1$3k6W2)9K6c8o6u0Q4x3U0k6@1i4K6y4p5y4e0R3$3y4e0u0Q4x3U0k6H3i4K6y4p5x3U0M7@1x3e0p5H3

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)

[#调试逆向](https://bbs.kanxue.com/forum-4-1-1.htm)

上传的附件：

- [md5.JPG](javascript:void(0);)

  （9.20kb，879次下载）
