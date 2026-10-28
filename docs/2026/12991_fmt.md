
# [原创]一个SMC补丁

一个SMC补丁

**软件：** `ClockTip`（电子提醒薄）

利用SMC能修改自身代码的特点，可以对加壳程序直接进行补丁。加壳程序执行时，都有一个将数据解压并将其写入原始映像基址处的过程，在代码刚恢复还没运行前，在外壳里插入一段补丁代码，对刚解压出来的数据补丁。<<加密与解密(第三版)>>

这个软件我以前改过一次，不过改动太大，所以决定用`smc`补丁。

用`PEID`显示`ASPack 2.12` -> `Alexey Solodovnikov`，用`ASP`定理很容易脱掉，不过既然用`SMC`，就不用脱壳，直接运行。用`OD`查找字符串，可以找到“未购买用户”，双击来到汇编代码处：

```
004E5CF7  |. /EB 0A         jmp     short 004E5D03
004E5CF9  |> |E8 36F5FFFF   call    004E5234
004E5CFE  |. |E8 2DF3FFFF   call    004E5030
004E5D03  |> \C605 ACCE4E00>mov     byte ptr [4ECEAC], 0             ;对地址4ECEAC赋值为0，也就是我要修改的地方
004E5D0A  |.  8D45 DC       lea     eax, dword ptr [ebp-24]
004E5D0D  |.  E8 02F4FFFF   call    004E5114
004E5D12  |.  8D45 DC       lea     eax, dword ptr [ebp-24]
004E5D15  |.  BA 885F4E00   mov     edx, 004E5F88                    ;  ASCII "Reg.dat"
004E5D1A  |.  E8 29EBF1FF   call    00404848
004E5D1F  |.  8B45 DC       mov     eax, dword ptr [ebp-24]
004E5D22  |.  E8 8132F2FF   call    00408FA8
004E5D27  |.  84C0          test    al, al
004E5D29  |.  74 66         je      short 004E5D91
004E5D2B  |.  8B83 E4030000 mov     eax, dword ptr [ebx+3E4]
004E5D31  |.  8B10          mov     edx, dword ptr [eax]
004E5D33  |.  FF92 DC000000 call    dword ptr [edx+DC]
004E5D39  |.  8D45 D8       lea     eax, dword ptr [ebp-28]
004E5D3C  |.  E8 D3F3FFFF   call    004E5114
004E5D41  |.  8D45 D8       lea     eax, dword ptr [ebp-28]
004E5D44  |.  BA 885F4E00   mov     edx, 004E5F88                    ;  ASCII "Reg.dat"
004E5D49  |.  E8 FAEAF1FF   call    00404848
004E5D4E  |.  8B55 D8       mov     edx, dword ptr [ebp-28]
004E5D51  |.  8B83 E4030000 mov     eax, dword ptr [ebx+3E4]
004E5D57  |.  8B80 20020000 mov     eax, dword ptr [eax+220]
004E5D5D  |.  8B08          mov     ecx, dword ptr [eax]
004E5D5F  |.  FF51 68       call    dword ptr [ecx+68]
004E5D62  |.  8D4D F4       lea     ecx, dword ptr [ebp-C]
004E5D65  |.  8B83 E4030000 mov     eax, dword ptr [ebx+3E4]
004E5D6B  |.  8B80 20020000 mov     eax, dword ptr [eax+220]
004E5D71  |.  33D2          xor     edx, edx
004E5D73  |.  8B30          mov     esi, dword ptr [eax]
004E5D75  |.  FF56 0C       call    dword ptr [esi+C]
004E5D78  |.  8D4D F0       lea     ecx, dword ptr [ebp-10]
004E5D7B  |.  8B83 E4030000 mov     eax, dword ptr [ebx+3E4]
004E5D81  |.  8B80 20020000 mov     eax, dword ptr [eax+220]
004E5D87  |.  BA 01000000   mov     edx, 1
004E5D8C  |.  8B30          mov     esi, dword ptr [eax]
004E5D8E  |.  FF56 0C       call    dword ptr [esi+C]
004E5D91  |>  803D ACCE4E00>cmp     byte ptr [4ECEAC], 0             ；如果4ECEAC的值不为0，就注册成功
004E5D98  |.  75 27         jnz     short 004E5DC1                   ；关键跳转
004E5D9A  |.  8D55 D4       lea     edx, dword ptr [ebp-2C]
004E5D9D  |.  A1 A4CE4E00   mov     eax, dword ptr [4ECEA4]
004E5DA2  |.  E8 C904F6FF   call    00446270
004E5DA7  |.  8D45 D4       lea     eax, dword ptr [ebp-2C]
004E5DAA  |.  BA 985F4E00   mov     edx, 004E5F98                    ;   - 未购买用户
004E5DAF  |.  E8 94EAF1FF   call    00404848
004E5DB4  |.  8B55 D4       mov     edx, dword ptr [ebp-2C]
004E5DB7  |.  A1 A4CE4E00   mov     eax, dword ptr [4ECEA4]
004E5DBC  |.  E8 DF04F6FF   call    004462A0
004E5DC1  |>  68 80000000   push    80                               ; /NewValue = 80
004E5DC6  |.  6A EC         push    -14                              ; |Index = GWL_EXSTYLE
004E5DC8  |.  A1 4CB94E00   mov     eax, dword ptr [4EB94C]          ; |
004E5DCD  |.  8B00          mov     eax, dword ptr [eax]             ; |
004E5DCF  |.  8B40 30       mov     eax, dword ptr [eax+30]          ; |
004E5DD2  |.  50            push    eax                              ; |hWnd
004E5DD3  |.  E8 CC17F2FF   call    004075A4                         ; \SetWindowLongA
```

只要将`004E5D09`地址的值改为`1`，就可以了。

这段代码在磁盘上是加密的，不能直接修改，只能在这段代码被解密后动态的修改。所以必须找到解密`004E5D03`处的代码，在`004E5D03`处下硬件写断点，再运行：

```
005AB163   /EB 08           jmp     short 005AB16D
005AB165   |0000            add     byte ptr [eax], al
005AB167   |0000            add     byte ptr [eax], al
005AB169   |0000            add     byte ptr [eax], al
005AB16B   |0000            add     byte ptr [eax], al
005AB16D   \8BC8            mov     ecx, eax
005AB16F    8B3E            mov     edi, dword ptr [esi]
005AB171    03BD 22040000   add     edi, dword ptr [ebp+422]
005AB177    8BB5 52010000   mov     esi, dword ptr [ebp+152]
005AB17D    C1F9 02         sar     ecx, 2
005AB180    F3:A5           rep     movs dword ptr es:[edi], dword ptr [esi] ;中断再这，这段代码是对各区块解密
005AB182    8BC8            mov     ecx, eax
005AB184    83E1 03         and     ecx, 3
005AB187    F3:A4           rep     movs byte ptr es:[edi], byte ptr [esi]
005AB189    5E              pop     esi
005AB18A    E9 83150000     jmp     005AC712
005AB18F    6A 00           push    0
005AB197    FF95 51050000   call    dword ptr [ebp+551]
005AB19D    83C6 08         add     esi, 8
005AB1A0    833E 00         cmp     dword ptr [esi], 0
005AB1A3  ^ 0F85 1EFFFFFF   jnz     005AB0C7   ；走出循环，解密完毕
005AB1A9    68 00800000     push    8000
005AB1AE    6A 00           push    0
```

在`005AB1A9`处修改，跳转到我自己的代码执行，这时要保证代码区段可读写且有足够的空间存放加入的代码。用`LoadPE`查看，块属性为`C0000040`，即可读，可写，包含已初始化的数据且在块结尾有足够的空间来容纳我的代码。我们就是把代码放在`005AC712`处。

万事具备，再就开始改代码了。将

`005AB1A9    68 00800000     push    8000`

改为`jmp 005AC712`，刚好也是5字节。下面就到`005AC712`处加入我的代码了：

```
005AC712    C605 095D4E00 0>mov     byte ptr [4E5D09], 1
005AC719    68 00800000     push    8000
005AC71E  ^ E9 6CEAFFFF     jmp     005AB18F
005AC723    90              nop
```

不要因为到这里就完了，我们还要看看在`005AB1A9`处的代码是否也加密了，如果加密了就要进行多层`SMC`补丁技术。把`005AB1A9`虚拟地址转换成文件偏移地址，你可以用工具也可以自己动手，转换后为`000799A9`，用`WinHex`打开来到`000799A9`处，用此处代码与`OD`执行到此处的代码进行比较，并没有加密。

最后将修改的代码保存到磁盘文件，就成功了。运行一下，已是注册版了。

[[招生]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)

[#调试逆向](https://bbs.kanxue.com/forum-4-1-1.htm)
