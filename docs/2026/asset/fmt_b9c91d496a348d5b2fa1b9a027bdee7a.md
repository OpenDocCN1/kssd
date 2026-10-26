
```
00401496  /$  55            push    ebp

00401497  |.  8BEC          mov     ebp, esp

00401499  |.  81EC 1C040000 sub     esp, 41C

......

004014B8  |.  C645 EC 90    mov     byte ptr [ebp-14], 90                  ;这里几处赋值操作，我们还不知道具体它想干什么

004014BC  |.  C645 ED 90    mov     byte ptr [ebp-13], 90                  ;90h，汇编指令是NOP——有点可疑

004014C0  |.  C645 EE 90    mov     byte ptr [ebp-12], 90

004014C4  |.  C645 EF 90    mov     byte ptr [ebp-11], 90

004014C8  |.  C645 F0 90    mov     byte ptr [ebp-10], 90

......

0040150D  |.  E8 3E050000   call    00401A50

00401512  |.  FF75 08       push    dword ptr [ebp+8]                      ; /<%s> "C:\Windows\System32\user32.dll"

00401515  |.  8B35 A0204000 mov     esi, dword ptr [<&USER32.wsprintfA>]   ; |USER32.wsprintfA

0040151B  |.  8D85 E4FBFFFF lea     eax, dword ptr [ebp-41C]               ; |

00401521  |.  68 8C234000   push    0040238C                               ; |takeown /f  %s

00401526  |.  50            push    eax                                    ; |s

00401527  |.  FFD6          call    esi                                    ; \wsprintfA

00401529  |.  8D85 E4FBFFFF lea     eax, dword ptr [ebp-41C]

0040152F  |.  6A 01         push    1

00401531  |.  50            push    eax                                    ;takeown /f  C:\Windows\System32\user32.dll

00401532  |.  E8 A6FDFFFF   call    004012DD                               ;有关takeown命令行的用法，详见cmd.exe

00401537  |.  FF75 08       push    dword ptr [ebp+8]

0040153A  |.  8D85 E4FBFFFF lea     eax, dword ptr [ebp-41C]

00401540  |.  68 78234000   push    00402378                              ;icacls %s /grant

00401545  |.  50            push    eax

00401546  |.  FFD6          call    esi

00401548  |.  8B3D 74204000 mov     edi, dword ptr [<&KERNEL32.lstrcatA>]  ;  kernel32.lstrcatA

0040154E  |.  83C4 44       add     esp, 44

00401551  |.  68 68234000   push    00402368                               ; /%username%:f

00401556  |.  8D85 E4FBFFFF lea     eax, dword ptr [ebp-41C]               ; |

0040155C  |.  50            push    eax                                    ; |icacls C:\Windows\System32\user32.dll /grant

0040155D  |.  FFD7          call    edi                                    ; \lstrcatA

0040155F  |.  8D85 E4FBFFFF lea     eax, dword ptr [ebp-41C]               ;icacls C:\Windows\System32\user32.dll /grant %username%:F

00401565  |.  6A 01         push    1

00401567  |.  50            push    eax

00401568  |.  E8 70FDFFFF   call    004012DD

0040156D  |.  FF75 08       push    dword ptr [ebp+8]

00401570  |.  8D85 E4FBFFFF lea     eax, dword ptr [ebp-41C]

00401576  |.  68 48234000   push    00402348                               ;icacls %s /grant *s-1-1-0:(f)

0040157B  |.  50            push    eax				                   	       ;将授予由 sid S-1-1-0 定义的用户对user32.dll文件的完全访问权限

0040157C  |.  FFD6          call    esi

0040157E  |.  8D85 E4FBFFFF lea     eax, dword ptr [ebp-41C]

00401584  |.  6A 01         push    1

00401586  |.  50            push    eax

00401587  |.  E8 51FDFFFF   call    004012DD

0040158C  |.  8B35 64204000 mov     esi, dword ptr [<&KERNEL32.lstrcpyA>]  

00401592  |.  83C4 1C       add     esp, 1C

00401595  |.  FF75 08       push    dword ptr [ebp+8]                      ; /String2   C:\Windows\System32\user32.dll

00401598  |.  8D85 D8FDFFFF lea     eax, dword ptr [ebp-228]               ; |

0040159E  |.  50            push    eax                                    ; |String1

0040159F  |.  FFD6          call    esi                                    ; \lstrcpyA

004015A1  |.  68 40234000   push    00402340                               ;.backup   icacls %s /grant *s-1-1-0:(f)

004015A6  |.  8D85 D8FDFFFF lea     eax, dword ptr [ebp-228]

004015AC  |.  50            push    eax

004015AD  |.  FFD7          call    edi

004015AF  |.  FF75 08       push    dword ptr [ebp+8]                      ;这一部分在备份User32.dll文件

004015B2  |.  8D85 DCFEFFFF lea     eax, dword ptr [ebp-124]

004015B8  |.  50            push    eax

004015B9  |.  FFD6          call    esi

004015BB  |.  68 38234000   push    00402338                               ;.tmp

004015C0  |.  8D85 DCFEFFFF lea     eax, dword ptr [ebp-124]

004015C6  |.  50            push    eax

004015C7  |.  FFD7          call    edi

004015C9  |.  68 20234000   push    00402320                               ;\nmake a temp file...

004015CE  |.  E8 5A050000   call    00401B2D

004015D3  |.  59            pop     ecx

004015D4  |.  53            push    ebx                                    ; /FailIfExists

004015D5  |.  8D85 DCFEFFFF lea     eax, dword ptr [ebp-124]               ; |C:\Windows\System32\user32.dll.tmp

004015DB  |.  50            push    eax                                    ; |NewFileName

004015DC  |.  FF75 08       push    dword ptr [ebp+8]                      ; |ExistingFileName

004015DF  |.  FF15 48204000 call    dword ptr [<&KERNEL32.CopyFileA>]      ; \CopyFileA

004015E5  |.  85C0          test    eax, eax

004015E7  |.  74 07         je      short 004015F0

004015E9  |.  68 14234000   push    00402314                               ;success.

004015EE  |.  EB 05         jmp     short 004015F5

004015F0  |>  68 0C234000   push    0040230C                               ;fail.

004015F5  |>  E8 33050000   call    00401B2D                               ;显示处理结果

004015FA  |.  59            pop     ecx

004015FB  |.  53            push    ebx                                    ; /hTemplateFile

004015FC  |.  53            push    ebx                                    ; |Attributes

004015FD  |.  6A 03         push    3                                      ; |Mode = OPEN_EXISTING

004015FF  |.  53            push    ebx                                    ; |pSecurity

00401600  |.  6A 01         push    1                                      ; |ShareMode = FILE_SHARE_READ

00401602  |.  68 000000C0   push    C0000000                               ; |Access = GENERIC_READ|GENERIC_WRITE

00401607  |.  8D85 DCFEFFFF lea     eax, dword ptr [ebp-124]               ; |C:\Windows\System32\user32.dll.tmp

0040160D  |.  50            push    eax                                    ; |FileName

0040160E  |.  FF15 40204000 call    dword ptr [<&KERNEL32.CreateFileA>]    ; \CreateFileA

00401614  |.  83F8 FF       cmp     eax, -1                                ;打开C:\Windows\System32\user32.dll.tmp文件

00401617  |.  8945 F8       mov     dword ptr [ebp-8], eax                 ;取得句柄值并保存，下面会用到

0040161A  |.  75 12         jnz     short 0040162E

0040161C  |.  68 F8224000   push    004022F8                               ;\nfail to open file.fail.

00401621  |.  E8 07050000   call    00401B2D

00401626  |.  59            pop     ecx

00401627  |.  32C0          xor     al, al

00401629  |.  E9 24010000   jmp     00401752

0040162E  |>  53            push    ebx                                    ; /pFileSizeHigh

0040162F  |.  50            push    eax                                    ; |hFile

00401630  |.  FF15 3C204000 call    dword ptr [<&KERNEL32.GetFileSize>]    ; \GetFileSize

00401636  |.  BF 00000200   mov     edi, 20000

0040163B  |.  57            push    edi

0040163C  |.  895D E0       mov     dword ptr [ebp-20], ebx

0040163F  |.  895D E8       mov     dword ptr [ebp-18], ebx

00401642  |.  E8 BE040000   call    00401B05

00401647  |.  57            push    edi

00401648  |.  8BF0          mov     esi, eax

0040164A  |.  53            push    ebx

0040164B  |.  56            push    esi

0040164C  |.  E8 FF030000   call    00401A50

00401651  |.  68 E4224000   push    004022E4                               ;\ntry to search sign\nfail to open file.fail.

00401656  |.  E8 D2040000   call    00401B2D

0040165B  |.  83C4 14       add     esp, 14

0040165E  |.  53            push    ebx                                    ; /Origin

0040165F  |.  53            push    ebx                                    ; |pOffsetHi

00401660  |.  53            push    ebx                                    ; |OffsetLo

00401661  |.  FF75 F8       push    dword ptr [ebp-8]                      ; |hFile  看到了吧，这就是上面CreateFile得到的句柄

00401664  |.  FF15 70204000 call    dword ptr [<&KERNEL32.SetFilePointer>] ; \SetFilePointer

0040166A  |.  53            push    ebx                                    ; /pOverlapped

0040166B  |.  8D45 E0       lea     eax, dword ptr [ebp-20]                ; |

0040166E  |.  50            push    eax                                    ; |pBytesRead

0040166F  |.  68 FFFF0100   push    1FFFF                                  ; |BytesToRead = 1FFFF (131071.)

00401674  |.  56            push    esi                                    ; |Buffer

00401675  |.  FF75 F8       push    dword ptr [ebp-8]                      ; |hFile

00401678  |.  FF15 6C204000 call    dword ptr [<&KERNEL32.ReadFile>]       ; \ReadFile

0040167E  |.  33FF          xor     edi, edi

00401680  |>  8D47 08       /lea     eax, dword ptr [edi+8]                ;这一段循环在检测标志

00401683  |.  807C06 F8 39  |cmp     byte ptr [esi+eax-8], 39              ;校验user32.dll是不是vista或windows7中的系统文件

00401688  |.  75 64         |jnz     short 004016EE                        ;防止误操作，看来作者编程还算严谨哦

0040168A  |.  807C37 0D 89  |cmp     byte ptr [edi+esi+D], 89

0040168F  |.  75 5D         |jnz     short 004016EE

00401691  |.  807C37 01 35  |cmp     byte ptr [edi+esi+1], 35

00401696  |.  75 56         |jnz     short 004016EE

......

004016C5  |.  68 D4224000   |push    004022D4                              ;\nfound sign!

004016CA  |.  8945 E4       |mov     dword ptr [ebp-1C], eax

004016CD  |.  E8 5B040000   |call    00401B2D

004016D2  |.  6A 05         |push    5

004016D4  |.  8D45 EC       |lea     eax, dword ptr [ebp-14]

004016D7  |.  50            |push    eax

004016D8  |.  8D4437 08     |lea     eax, dword ptr [edi+esi+8]

004016DC  |.  50            |push    eax

004016DD  |.  E8 A1030000   |call    00401A83

004016E2  |.  83C4 10       |add     esp, 10

004016E5  |.  F7D8          |neg     eax

004016E7  |.  1AC0          |sbb     al, al

004016E9  |.  FEC0          |inc     al

004016EB  |.  8845 FF       |mov     byte ptr [ebp-1], al

004016EE  |>  47            |inc     edi

004016EF  |.  81FF E2FF0100 |cmp     edi, 1FFE2

004016F5  |.^ 72 89         \jb      short 00401680

004016F7  |.  56            push    esi

004016F8  |.  E8 1C040000   call    00401B19

004016FD  |.  385D FF       cmp     byte ptr [ebp-1], bl

00401700  |.  59            pop     ecx

00401701  |.  74 54         je      short 00401757                    ;跳过去，继续patch

00401703  |.  68 9C224000   push    0040229C                          ;\nthis file had been patched, no need to patch again!\n

00401708  |.  E8 20040000   call    00401B2D

0040170D  |.  C70424 782240>mov     dword ptr [esp], 00402278         ;\n文件已经打过补丁，不需要再次进补！\nthis file had been patched, no need to patch again!\n

00401714  |.  E8 14040000   call    00401B2D

00401719  |.  59            pop     ecx

0040171A  |>  FF75 F8       push    dword ptr [ebp-8]          

0040171D  |.  FF15 34204000 call    dword ptr [<&KERNEL32.CloseHandle>] 

00401723  |.  FF75 F4       push    dword ptr [ebp-C]

00401726  |.  68 64224000   push    00402264                          ;\n\ntotal found: %d\n

0040172B  |.  E8 FD030000   call    00401B2D

00401730  |.  59            pop     ecx

00401731  |>  59            pop     ecx

00401732  |.  8D85 DCFEFFFF lea     eax, dword ptr [ebp-124]

00401738  |.  50            push    eax                                    ; /Path

00401739  |.  FF15 98204000 call    dword ptr [<&SHLWAPI.PathFileExistsA>] ; \PathFileExistsA

0040173F  |.  85C0          test    eax, eax

00401741  |.  74 0D         je      short 00401750

00401743  |.  8D85 DCFEFFFF lea     eax, dword ptr [ebp-124]

00401749  |.  50            push    eax                                    ; /FileName

0040174A  |.  FF15 44204000 call    dword ptr [<&KERNEL32.DeleteFileA>]    ; \DeleteFileA

00401750  |>  B0 01         mov     al, 1

00401752  |>  5F            pop     edi

00401753  |.  5E            pop     esi

00401754  |.  5B            pop     ebx

00401755  |.  C9            leave

00401756  |.  C3            retn

00401757  |>  837D F4 01    cmp     dword ptr [ebp-C], 1

0040175B  |.^ 75 BD         jnz     short 0040171A

0040175D  |.  68 58224000   push    00402258                               ;\n\nmatch!

00401762  |.  E8 C6030000   call    00401B2D

00401767  |.  C70424 442240>mov     dword ptr [esp], 00402244              ;\ntry to modify...

0040176E  |.  E8 BA030000   call    00401B2D

00401773  |.  59            pop     ecx

00401774  |.  53            push    ebx                                    ; /Origin

00401775  |.  53            push    ebx                                    ; |pOffsetHi

00401776  |.  FF75 E4       push    dword ptr [ebp-1C]                     ; |offset 031A7

00401779  |.  FF75 F8       push    dword ptr [ebp-8]                      ; |hFile C:\Windows\System32\user32.dll.tmp句柄

0040177C  |.  FF15 70204000 call    dword ptr [<&KERNEL32.SetFilePointer>] ; \SetFilePointer

00401782  |.  53            push    ebx                                    ; /pOverlapped

00401783  |.  8D45 E8       lea     eax, dword ptr [ebp-18]                ; |

00401786  |.  50            push    eax                                    ; |pBytesWritten

00401787  |.  6A 05         push    5                                      ; |nBytesToWrite = 5

00401789  |.  8D45 EC       lea     eax, dword ptr [ebp-14]                ; |哈哈，还记得函数开头连续几个90h的赋值操作吗

0040178C  |.  50            push    eax                                    ; |Buffer 指向一串90h

0040178D  |.  FF75 F8       push    dword ptr [ebp-8]                      ; |hFile

00401790      FF15 68204000 call    dword ptr [<&KERNEL32.WriteFile>]      ; \WriteFile 往031A7偏移处写入5字节90h

00401796  |.  FF75 F8       push    dword ptr [ebp-8]                      ; /hObject

00401799  |.  FF15 34204000 call    dword ptr [<&KERNEL32.CloseHandle>]    ; \CloseHandle

0040179F  |.  837D E8 05    cmp     dword ptr [ebp-18], 5

004017A3  |.  0F85 A1000000 jnz     0040184A

004017A9  |.  68 14234000   push    00402314                          ;success.

004017AE  |.  E8 7A030000   call    00401B2D

004017B3  |.  C70424 282240>mov     dword ptr [esp], 00402228         ;\nre-checksum the file...

004017BA  |.  E8 6E030000   call    00401B2D

004017BF  |.  8D85 DCFEFFFF lea     eax, dword ptr [ebp-124]

004017C5  |.  50            push    eax

004017C6  |.  E8 3FF9FFFF   call    0040110A

004017CB  |.  85C0          test    eax, eax

004017CD  |.  59            pop     ecx

004017CE  |.  59            pop     ecx

004017CF  |.  75 79         jnz     short 0040184A

004017D1  |.  68 1C224000   push    0040221C                          ; success!

004017D6  |.  E8 52030000   call    00401B2D

004017DB  |.  C70424 082240>mov     dword ptr [esp], 00402208         ;\nmake a backup...

004017E2  |.  E8 46030000   call    00401B2D

004017E7  |.  59            pop     ecx

004017E8  |.  53            push    ebx                               ; /FailIfExists

004017E9  |.  8D85 D8FDFFFF lea     eax, dword ptr [ebp-228]          ; |

004017EF  |.  50            push    eax                               ; |NewFileName

004017F0  |.  FF75 08       push    dword ptr [ebp+8]                 ; |ExistingFileName

004017F3      FF15 48204000 call    dword ptr [<&KERNEL32.CopyFileA>] ; \CopyFileA

004017F9  |.  85C0          test    eax, eax

004017FB  |.  74 07         je      short 00401804

004017FD  |.  68 C8214000   push    004021C8                          ;success.\noriginal file has been renamed as user32.dll.backup.

00401802  |.  EB 05         jmp     short 00401809

00401804  |>  68 0C234000   push    0040230C                          ;fail.

00401809  |>  E8 1F030000   call    00401B2D

0040180E  |.  59            pop     ecx

0040180F  |.  68 A8214000   push    004021A8                          ;\ntry to overwrite old file...

00401814  |.  E8 14030000   call    00401B2D

00401819  |.  FF75 08       push    dword ptr [ebp+8]

0040181C  |.  8D85 DCFEFFFF lea     eax, dword ptr [ebp-124]

00401822  |.  50            push    eax

00401823  |.  E8 F2F9FFFF   call    0040121A

00401828  |.  83C4 0C       add     esp, 0C

0040182B  |.  84C0          test    al, al

0040182D  |.  74 1B         je      short 0040184A

0040182F  |.  68 74214000   push    00402174                          ;success!  \n\nrestart your computer to take effect!\n

00401834  |.  E8 F4020000   call    00401B2D

00401839  |.  C70424 542140>mov     dword ptr [esp], 00402154         ;\n成功!  重新启动计算机后生效\n

00401840  |>  E8 E8020000   call    00401B2D

00401845  |.^ E9 E7FEFFFF   jmp     00401731

0040184A  |>  68 4C214000   push    0040214C                          ;fail!

0040184F  \.^ EB EF         jmp     short 00401840
```
