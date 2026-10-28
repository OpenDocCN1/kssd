
# 汇编代码分析

## CreateProcessA 调用序列
```
00477928  |>  8D95 14E4FFF>lea     edx, dword ptr [ebp-1BEC]

0047792E  |.  8D8D 04E5FFF>lea     ecx, dword ptr [ebp-1AFC]

00477934  |.  52           push    edx                           ; /pProcessInfo

00477935  |.  51           push    ecx                           ; |pStartupInfo

00477936  |.  8D85 DCFDFFF>lea     eax, dword ptr [ebp-224]          ; |

0047793C  |.  8D8D 50E7FFF>lea     ecx, dword ptr [ebp-18B0]         ; |

00477942  |.  50           push    eax                           ; |CurrentDir

00477943  |.  6A 00        push    0                             ; |pEnvironment = NULL

00477945  |.  8B55 FC      mov     edx, dword ptr [ebp-4]           ; |

00477948  |.  81CA 2200000>or      edx, 4000022                   ; |

0047794E  |.  52           push    edx                           ; |CreationFlags

0047794F  |.  6A 00        push    0                             ; |InheritHandles = FALSE

00477951  |.  6A 00        push    0                            ; |pThreadSecurity = NULL

00477953  |.  6A 00        push    0                           ; |pProcessSecurity = NULL

00477955  |.  51           push    ecx                         ; |CommandLine

00477956  |.  6A 00        push    0                           ; |ModuleFileName = NULL

00477958  |.  E8 73760300  call    <jmp.&KERNEL32.CreateProcessA>; \CreateProcessA
```

```
0047795D  |.  85C0         test    eax, eax        ;检查返回值

0047795F  |.  75 1E        jnz     short 0047797F        ;成功则跳过错误处理
```

## 打开线程的错误处理部分，返回-1并退出
```
00477961  |.  8D85 E0FEFFF>lea     eax, dword ptr [ebp-120]

00477967  |.  50           push    eax                           ; /Arg2

00477968  |.  8D96 7409000>lea     edx, dword ptr [esi+974]      ; |

0047796E  |.  52           push    edx                           ; |Arg1

0047796F  |.  E8 A8C6FDFF  call    \_Error                        ; \\_Error

00477974  |.  83C4 08      add     esp, 8

00477977  |.  83C8 FF      or      eax, FFFFFFFF        ;返回-1

0047797A  |.  E9 08010000  jmp     00477A87                ;跳转结束处
```

## 将CreateProcess中获得的PROCESS_INFORMATION的四个值赋值给全局变量
```
0047797F  |>  8B95 14E4FFF>mov     edx, dword ptr [ebp-1BEC]

00477985  |.  8915 605A4D0>mov     dword ptr [4D5A60], edx

0047798B  |.  8B8D 18E4FFF>mov     ecx, dword ptr [ebp-1BE8]

00477991  |.  890D 645A4D0>mov     dword ptr [4D5A64], ecx

00477997  |.  C705 685A4D0>mov     dword ptr [4D5A68], 6

004779A1  |.  C705 6C5A4D0>mov     dword ptr [4D5A6C], 6

004779AB  |.  8B85 1CE4FFF>mov     eax, dword ptr [ebp-1BE4]

004779B1  |.  A3 705A4D00  mov     dword ptr [4D5A70], eax

004779B6  |.  8B95 20E4FFF>mov     edx, dword ptr [ebp-1BE0]

004779BC  |.  8915 745A4D0>mov     dword ptr [4D5A74], edx

004779C2  |.  8B4D F0      mov     ecx, dword ptr [ebp-10]
```

## 设置OD的标题显示内容
```
004779C5  |.  51           push    ecx                           ; /Arg4

004779C6  |.  8D86 A503000>lea     eax, dword ptr [esi+3A5]          ; |

004779CC  |.  50           push    eax                           ; |Arg3

004779CD  |.  8D96 8E09000>lea     edx, dword ptr [esi+98E]          ; |

004779D3  |.  52           push    edx                           ; |Arg2

004779D4  |.  8D8D E0FEFFF>lea     ecx, dword ptr [ebp-120]          ; |

004779DA  |.  51           push    ecx                           ; |Arg1

004779DB  |.  E8 4CF20200  call    004A6C2C                      ; \\_sprintf

004779E0  |.  83C4 10      add     esp, 10

004779E3  |.  8D85 E0FEFFF>lea     eax, dword ptr [ebp-120]

004779E9  |.  50           push    eax                           ; /Text

004779EA  |.  8B15 7C3B4D0>mov     edx, dword ptr [4D3B7C]       ; |

004779F0  |.  52           push    edx                         ; |hWnd

004779F1  |.  E8 C87B0300  call    <jmp.&USER32.SetWindowTextA>  ; \SetWindowTextA

004779F6  |.  C705 88574D0>mov     dword ptr [4D5788], 1
```

## 设置界面部分的代码
```
00477A00  |.  33C9         xor     ecx, ecx

00477A02  |.  890D 94574D0>mov     dword ptr [4D5794], ecx

00477A08  |.  8D46 14      lea     eax, dword ptr [esi+14]

00477A0B  |.  50           push    eax                           ; /Arg3

00477A0C  |.  6A 00        push    0                             ; |Arg2 = 00000000

00477A0E  |.  6A 00        push    0                             ; |Arg1 = 00000000

00477A10  |.  E8 F72BFEFF  call    \_Addtolist                    ; \\_Addtolist

00477A15  |.  83C4 0C      add     esp, 0C

00477A18  |.  837D FC 00   cmp     dword ptr [ebp-4], 0

00477A1C  |.  74 18        je      short 00477A36

00477A1E  |.  68 805B4D00  push    004D5B80      ; /Arg3 = 004D5B80 ASCII "D:\CrackMe.exe"

00477A23  |.  8D96 9609000>lea     edx, dword ptr [esi+996]      ; |

00477A29  |.  52           push    edx                           ; |Arg2

00477A2A  |.  6A 00        push    0                             ; |Arg1 = 00000000

00477A2C  |.  E8 FF9BFBFF  call    \_Message                      ; \\_Message

00477A31  |.  83C4 0C      add     esp, 0C

00477A34  |.  EB 16        jmp     short 00477A4C

00477A36  |>  68 805B4D00  push    004D5B80      ; /Arg3 = 004D5B80 ASCII "D:\CrackMe.exe"

00477A3B  |.  8D8E A809000>lea     ecx, dword ptr [esi+9A8]      ; |

00477A41  |.  51           push    ecx                           ; |Arg2

00477A42  |.  6A 00        push    0                             ; |Arg1 = 00000000

00477A44  |.  E8 E79BFBFF  call    \_Message                      ; \\_Message

00477A49  |.  83C4 0C      add     esp, 0C

00477A4C  |>  803D 885D4D0>cmp     byte ptr [4D5D88], 0

00477A53  |.  74 16        je      short 00477A6B

00477A55  |.  68 885D4D00  push    004D5D88                      ; /Arg3 = 004D5D88

00477A5A  |.  8D86 B209000>lea     eax, dword ptr [esi+9B2]      ; |

00477A60  |.  50           push    eax                           ; |Arg2

00477A61  |.  6A 00        push    0                             ; |Arg1 = 00000000

00477A63  |.  E8 C89BFBFF  call    \_Message                      ; \\_Message

00477A68  |.  83C4 0C      add     esp, 0C
```

## 读写配置文件
```
00477A6B  |>  68 885D4D00  push    004D5D88                      ; /Arg2 = 004D5D88

00477A70  |.  68 805B4D00  push    004D5B80      ; |Arg1 = 004D5B80 ASCII "D:\CrackMe.exe"

00477A75  |.  E8 8AF3FFFF  call    00476E04                      ; \Ollydbg.00476E04

00477A7A  |.  83C4 08      add     esp, 8

00477A7D  |.  6A 03        push    3                             ; /Arg1 = 00000003

00477A7F  |.  E8 F49EFBFF  call    00431978                      ; \Ollydbg.00431978

00477A84  |.  59           pop     ecx
```

## 返回0，恢复现场，并退出
```
00477A85  |.  33C0         xor     eax, eax

00477A87  |>  5F           pop     edi

00477A88  |.  5E           pop     esi

00477A89  |.  5B           pop     ebx

00477A8A  |.  8BE5         mov     esp, ebp

00477A8C  |.  5D           pop     ebp

00477A8D  \.  C3           retn
```

OD打开被调试程序之后，进入调试循环体，在调试循环体中通过读取文件的OEP，在单步的方式下对OEP下Int3断点，断下程序后，恢复OEP处Int3断点为初始指令码。

这个流程可以在本系列前几个文档中得到验证，这里就不再加以说明了。

武汉科锐学员：driverox

2008-5-31

[[招生]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)

[#调试逆向](https://bbs.kanxue.com/forum-4-1-1.htm)
