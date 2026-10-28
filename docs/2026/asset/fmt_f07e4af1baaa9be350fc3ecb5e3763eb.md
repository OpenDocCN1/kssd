
```asm
0042EBF5  |.  8938               mov     dword ptr [eax], edi
0042EBF7  |.  8B15 14574D00      mov     edx, dword ptr [4D5714]   ; 4D5714中保存的是调试事件的代码DebugEvent.dwDebugEventCode
0042EBFD  |.  83FA 09            cmp     edx, 9                     ; Switch (cases 1..9)
0042EC00  |.  0F87 EE270000      ja      004313F4
0042EC06  |.  FF2495 0DEC4200    jmp     dword ptr [edx*4+42EC0D]  ; 这里是switch跳转
```

用来判断是何种异常：

```asm
0042EC0D  |.  F4134300           dd      OLLYDBG.004313F4  ; EXCEPTION_DEBUG_EVENT
0042EC11  |. |35EC4200           dd      OLLYDBG.0042EC35
0042EC15  |. |FF0C4300           dd      OLLYDBG.00430CFF
0042EC19  |. |D70D4300           dd      OLLYDBG.00430DD7
0042EC1D  |. |3F0F4300           dd      OLLYDBG.00430F3F
0042EC21  |. |37104300           dd      OLLYDBG.00431037
0042EC25  |. |2D114300           dd      OLLYDBG.0043112D
0042EC29  |. |B7114300           dd      OLLYDBG.004311B7
0042EC2D  |. |76124300           dd      OLLYDBG.00431276
0042EC31  |. |C7134300           dd      OLLYDBG.004313C7

0042EC35  |> \8B0D 0C364E00      mov     ecx, dword ptr [4E360C]              ; Case 1 of switch 0042EBFD
0042EC3B  |.  33C0               xor     eax, eax
0042EC3D  |.  894D EC            mov     dword ptr [ebp-14], ecx
0042EC40  |.  A3 0C364E00        mov     dword ptr [4E360C], eax
0042EC45  |.  C745 A4 20574D00   mov     dword ptr [ebp-5C], 004D5720
0042EC4C  |.  85FF               test    edi, edi        ;检查主线程中是否有当前寄存器的信息；
0042EC4E  |.  75 0D              jnz     short 0042EC5D
0042EC50  |.  8B55 A4            mov     edx, dword ptr [ebp-5C]
0042EC53  |.  33DB               xor     ebx, ebx
0042EC55  |.  8B4A 0C            mov     ecx, dword ptr [edx+C]
0042EC58  |.  894D D8            mov     dword ptr [ebp-28], ecx
0042EC5B  |.  EB 22              jmp     short 0042EC7F

0042EC5D  |>  8B47 2C            mov     eax, dword ptr [edi+2C]              ; ntdll.7C921231
0042EC60  |.  8945 D8            mov     dword ptr [ebp-28], eax              ;这一段是把异常地址赋给局部变量ebp-28
0042EC63  |.  837D EC 00         cmp     dword ptr [ebp-14], 0
0042EC67  |.  8B5F 10            mov     ebx, dword ptr [edi+10]
0042EC6A  |.  74 13              je      short 0042EC7F

0042EC6C  |.  F647 31 01         test    byte ptr [edi+31], 1
0042EC70  |.  74 0D              je      short 0042EC7F
0042EC72  |.  8167 30 FFFEFFFF   and     dword ptr [edi+30], FFFFFEFF
0042EC79  |.  C707 01000000      mov     dword ptr [edi], 1

0042EC7F  |>  8B45 A4            mov     eax, dword ptr [ebp-5C]
0042EC82  |.  8138 03000080      cmp     dword ptr [eax], 80000003          ; Int3中断
0042EC88  |.  74 07              je      short 0042EC91
0042EC8A  |.  33D2               xor     edx, edx
0042EC8C  |.  8955 DC            mov     dword ptr [ebp-24], edx
0042EC8F  |.  EB 79              jmp     short 0042ED0A

0042EC91  |>  6A 02              push    2                         ; /Arg4 = 00000002
0042EC93  |.  6A 01              push    1                         ; |Arg3 = 00000001
0042EC95  |.  8B4D D8            mov     ecx, dword ptr [ebp-28]             ; |
0042EC98  |.  49                 dec     ecx                                ; |减一是让指令码回溯
0042EC99  |.  51                 push    ecx                              ; |Arg2
0042EC9A  |.  8D45 BB            lea     eax, dword ptr [ebp-45]              ; |
0042EC9D  |.  50                 push    eax                              ; |Arg1
0042EC9E  |.  E8 69260300        call    _Readmemory                 ; \_Readmemory
0042ECA3  |.  83C4 10            add     esp, 10
0042ECA6  |.  83F8 01            cmp     eax, 1
0042ECA9  |.  74 07              je      short 0042ECB2
0042ECAB  |.  33D2               xor     edx, edx
0042ECAD  |.  8955 DC            mov     dword ptr [ebp-24], edx
0042ECB0  |.  EB 58              jmp     short 0042ED0A

0042ECB2  |>  33C0               xor     eax, eax
0042ECB4  |.  8A45 BB            mov     al, byte ptr [ebp-45]
0042ECB7  |.  3D CC000000        cmp     eax, 0CC
0042ECBC  |.  75 09              jnz     short 0042ECC7
0042ECBE  |.  C745 DC 01000000   mov     dword ptr [ebp-24], 1        ; 若Readmemory在断点地址读的指令码是CC的话，ebp-24设为1---ebp-24保存的是要回溯的指令长度
0042ECC5  |.  EB 43              jmp     short 0042ED0A

0042ECC7  |>  83F8 03            cmp     eax, 3
0042ECCA  |.  74 07              je      short 0042ECD3
0042ECCC  |.  33D2               xor     edx, edx
0042ECCE  |.  8955 DC            mov     dword ptr [ebp-24], edx
0042ECD1  |.  EB 37              jmp     short 0042ED0A

0042ECD3  |>  6A 02              push    2                         ; /Arg4 = 00000002
0042ECD5  |.  6A 01              push    1                         ; |Arg3 = 00000001
0042ECD7  |.  8B4D D8            mov     ecx, dword ptr [ebp-28]      ; |
0042ECDA  |.  83E9 02            sub     ecx, 2                     ; |指令码回溯2
0042ECDD  |.  51                 push    ecx                       ; |Arg2
0042ECDE  |.  8D45 BB            lea     eax, dword ptr [ebp-45]        ; |
0042ECE1  |.  50                 push    eax                        ; |Arg1
0042ECE2  |.  E8 25260300        call    _Readmemory              ; \_Readmemory
0042ECE7  |.  83C4 10            add     esp, 10
0042ECEA  |.  83F8 01            cmp     eax, 1
0042ECED  |.  75 0D              jnz     short 0042ECFC
0042ECEF  |.  33D2               xor     edx, edx
0042ECF1  |.  8A55 BB            mov     dl, byte ptr [ebp-45]
0042ECF4  |.  81FA CD000000      cmp     edx, 0CD
0042ECFA  |.  74 07              je      short 0042ED03

0042ECFC  |>  33C9               xor     ecx, ecx
0042ECFE  |.  894D DC            mov     dword ptr [ebp-24], ecx
0042ED01  |.  EB 07              jmp     short 0042ED0A

0042ED03  |>  C745 DC 02000000   mov     dword ptr [ebp-24], 2

0042ED0A  |>  8B45 DC            mov     eax, dword ptr [ebp-24]
0042ED0D  |.  2945 D8            sub     dword ptr [ebp-28], eax      ; 回溯指令码，才是正确的断点地址
0042ED10  |.  8B15 08574D00      mov     edx, dword ptr [4D5708]
0042ED16  |.  3B55 D8            cmp     edx, dword ptr [ebp-28]   ; ntdll.DbgBreakPoint
0042ED19  |.  75 08              jnz     short 0042ED23
0042ED1B  |.  3B1D 0C574D00      cmp     ebx, dword ptr [4D570C]
0042ED21  |.  74 16              je      short 0042ED39

0042ED23  |> \33C9          xor     ecx, ecx
0042ED25  |.  8B45 D8       mov     eax, dword ptr [ebp-28]
0042ED28  |.  890D 10574D00 mov     dword ptr [4D5710], ecx
0042ED2E  |.  A3 08574D00   mov     dword ptr [4D5708], eax       ; 把当前断点地址保存到全局变量4D5708中
0042ED33  |.  891D 0C574D00 mov     dword ptr [4D570C], ebx

0042ED39  |>  8B55 A4       mov     edx, dword ptr [ebp-5C]
0042ED3C  |.  8B0A          mov     ecx, dword ptr [edx]
```

以上找到了回溯后的断点地址。然后下面是做一些别的操作，这里暂不予考虑。

这时因为触发了`Int3`异常，被调试程序断了下来，且已经回溯了地址，这时按下`F9`的话，OD就会重新跑起来，这时OD对用户设置的`Int3`断点又做了两件事：

1. 恢复原有的指令，让被调试程序正确执行指令；
2. 走完正确指令之后，再重新设置指令为`0xCC`。

## 在OD的函数`Go`中

先通过`Findthread`获得线程结构体：

```asm
00434A45  |> \8B4D 08       mov     ecx, dword ptr [ebp+8]
00434A48  |.  51            push    ecx            ; ecx 是Go函数传入参数，为被调试线程的ID
00434A49  |.  E8 2A3F0400   call    \_Findthread      ; 返回OD中t_thread线程结构体
00434A4E  |.  59            pop     ecx
00434A4F  |.  8945 DC       mov     dword ptr [ebp-24], eax ;把t_thread线程结构体的地址赋给ebp-24
```

然后通过线程结构体获得当前要执行的指令的地址（这个地址在前面已经用回溯法修正过了）：

```asm
00434AB9  |.  8B99 18030000 mov     ebx, dword ptr [ecx+t_thread.th_reg.r_EIP]
…
00434B2A  |.  53            push    ebx       ; ebx = t_thread.th_reg.r_EIP
00434B2B  |.  E8 7C49FEFF   call    004194AC   ; 把Eip传给函数004194AC
```

在函数`004194AC`中，先获得`Int3`断点结构体数组：

```asm
004194EC  |> \56            push    esi             ;Eip
004194ED  |.  68 E17E4D00   push    004D7EE1       ; |Arg1 = 004D7EE1 ASCII "Table of breakpoints"
004194F2  |.  E8 19C00300   call    \_Findsorteddata    ; 返回Int3断点结构体数组首地址
```

OD的`Int3`断点结构体中有断点处的原始指令码，用来恢复被调试程序原先的指令码。把该地址作为参数传给函数`00418C4C`。
