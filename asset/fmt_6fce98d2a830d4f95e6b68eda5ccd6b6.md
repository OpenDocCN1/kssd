
## 解密字符串函数

```
10001050 >/$ 55             push    ebp
10001051  |. 8BEC           mov     ebp, esp
10001053  |. 83EC 08        sub     esp, 8
10001056  |. 894D F8        mov     dword ptr [ebp-8], ecx
10001059  |. C745 FC 00000> mov     dword ptr [ebp-4], 0
10001060  |. EB 09          jmp     short 1000106B
10001062  |> 8B45 FC        /mov    eax, dword ptr [ebp-4]
10001065  |. 83C0 01        |add     eax, 1
10001068  |. 8945 FC        |mov     dword ptr [ebp-4], eax
1000106B  |> 8B4D FC        mov     ecx, dword ptr [ebp-4]
1000106E  |. 3B4D 10        |cmp     ecx, dword ptr [ebp+10]
10001071  |. 7D 16          |jge     short 10001089
10001073  |. 8B55 08        |mov     edx, dword ptr [ebp+8]
10001076  |. 0355 FC        |add     edx, dword ptr [ebp-4]
10001079  |. 0FBE02         |movsx   eax, byte ptr [edx]
1000107C  |. 3345 14        |xor     eax, dword ptr [ebp+14]
1000107F  |. 8B4D 0C        |mov     ecx, dword ptr [ebp+C]
10001082  |. 034D FC        |add     ecx, dword ptr [ebp-4]
10001085  |. 8801           |mov     byte ptr [ecx], al
10001087  |.^ EB D9         \jmp     short 10001062
10001089  |> 8BE5           mov     esp, ebp
1000108B  |. 5D            pop     ebp
1000108C  \. C2 1000       retn    10
```

上面的解密函数用于揭秘一些被加密的 `API` 函数名。继而调用 `GetApiAddressByName` 直接得到 `API` 地址。

### GetApiAddressByName

```
100011F0 >/$ 55             push    ebp
100011F1  |. 8BEC           mov     ebp, esp
100011F3  |. 83EC 08        sub     esp, 8
100011F6  |. 894D F8        mov     dword ptr [ebp-8], ecx
100011F9  |. 8B4D F8        mov     ecx, dword ptr [ebp-8]
100011FC  |. E8 BFFEFFFF    call    GetKernel32base
10001201  |. 8945 FC        mov     dword ptr [ebp-4], eax
10001204  |. 8B45 08        mov     eax, dword ptr [ebp+8]
10001207  |. 50            push    eax
10001208  |. 8B4D FC        mov     ecx, dword ptr [ebp-4]
1000120B  |. 51            push    ecx
1000120C  |. 8B55 F8        mov     edx, dword ptr [ebp-8]
1000120F  |. 8B42 04        mov     eax, dword ptr [edx+4]
10001212  |. FFD0           call    eax                          ; kernel32.GetProcAddress
10001214  |. 8BE5           mov     esp, ebp
10001216  |. 5D            pop     ebp
10001217  \. C2 0400       retn    4
```

`API` 地址填充完毕，看：

```
1000102A  |. E8 91210000   call    100031C0  这个 call.
```

这个 `call` 启动两个线程：

```
100031C0 /$ 55             push    ebp
100031C1  |. 8BEC           mov     ebp, esp
100031C3  |. 83EC 10        sub     esp, 10
100031C6  |. 894D F0        mov     dword ptr [ebp-10], ecx
100031C9  |. 50            push    eax
100031CA  |. B8 01000000   mov     eax, 1
100031CF  |. B8 04000000   mov     eax, 4
100031D4  |. B8 0A000000   mov     eax, 0A
100031D9  |. 58            pop     eax
100031DA  |. 50            push    eax
100031DB  |. B8 01000000   mov     eax, 1
100031E0  |. B8 04000000   mov     eax, 4
100031E5  |. B8 0A000000   mov     eax, 0A
100031EA  |. 58            pop     eax
100031EB  |. 50            push    eax
100031EC  |. B8 01000000   mov     eax, 1
100031F1  |. B8 04000000   mov     eax, 4
100031F6  |. B8 0A000000   mov     eax, 0A
100031FB  |. 58            pop     eax
100031FC  |. C645 F8 7E    mov     byte ptr [ebp-8], 7E
10003200  |. C645 F9 7E    mov     byte ptr [ebp-7], 7E
10003204  |. C645 FA 7E    mov     byte ptr [ebp-6], 7E
10003208  |. C645 FB 21    mov     byte ptr [ebp-5], 21
1000320C  |. C645 FC 7E    mov     byte ptr [ebp-4], 7E
10003210  |. C645 FD 7E    mov     byte ptr [ebp-3], 7E
10003214  |. C645 FE 7E    mov     byte ptr [ebp-2], 7E
10003218  |. C645 FF 00    mov     byte ptr [ebp-1], 0
1000321C  |. 8D45 F8        lea     eax, dword ptr [ebp-8]
1000321F  |. 50            push    eax
10003220  |. 6A 00         push    0
10003222  |. 6A 00         push    0
10003224  |. 8B4D F0        mov     ecx, dword ptr [ebp-10]
10003227  |. 8B51 34        mov     edx, dword ptr [ecx+34]
1000322A  |. FFD2           call    edx                          ; CreateMutex
1000322C  |. 6A 00         push    0
1000322E  |. 6A 00         push    0
10003230  |. 8B45 F0        mov     eax, dword ptr [ebp-10]
10003233  |. 50            push    eax
10003234  |. 68 80310010   push    ThreadFunc1                   ; ThreadFunc1
10003239  |. 6A 00         push    0
1000323B  |. 6A 00         push    0
1000323D  |. 8B4D F0        mov     ecx, dword ptr [ebp-10]
10003240  |. 8B51 38        mov     edx, dword ptr [ecx+38]
10003243  |. FFD2           call    edx                          ; CreateThread
10003245  |. 8945 F4        mov     dword ptr [ebp-C], eax
10003248  |. 8B45 F4        mov     eax, dword ptr [ebp-C]
1000324B  |. 50            push    eax
1000324C  |. 8B4D F0        mov     ecx, dword ptr [ebp-10]
1000324F  |. 8B91 84000000  mov     edx, dword ptr [ecx+84]
10003255  |. FFD2           call    edx                          ; CloseHandle
10003257  |. 6A 00         push    0
10003259  |. 6A 00         push    0
1000325B  |. 8B45 F0        mov     eax, dword ptr [ebp-10]
1000325E  |. 50            push    eax
1000325F  |. 68 60310010   push    ThreadFunc2
10003264  |. 6A 00         push    0
10003266  |. 6A 00         push    0
10003268  |. 8B4D F0        mov     ecx, dword ptr [ebp-10]
1000326B  |. 8B51 38        mov     edx, dword ptr [ecx+38]
1000326E  |. FFD2           call    edx                          ; CreateThread
10003270  |. 8945 F4        mov     dword ptr [ebp-C], eax
10003273  |. 6A FF         push    -1
10003275  |. 8B45 F4        mov     eax, dword ptr [ebp-C]
10003278  |. 50            push    eax
10003279  |. 8B4D F0        mov     ecx, dword ptr [ebp-10]
1000327C  |. 8B51 3C        mov     edx, dword ptr [ecx+3C]
1000327F  |. FFD2           call    edx                          ; WaitForSingleObject
10003281  |. 33C0           xor     eax, eax
10003283  |. 8BE5           mov     esp, ebp
10003285  |. 5D            pop     ebp
10003286  \. C3            retn
```

该 `call` 头部也是一堆垃圾代码。作为免杀的作用。

然后创建一个 `"~~~!~~~"` 的互斥体。继而直接开两个线程 `10003180` 和 `10003160`。

其中 `3180` 线程句柄直接 `CloseHandle` 了，`10003160` 线程句柄则被 `WaitForSingleObject` 了。这样是为了等所有动作执行完毕了才退出进程。

### 跟到 ThreadFunc1

```
10003180 >/. 55             push    ebp
10003181  |. 8BEC           mov     ebp, esp
10003183  |. 51            push    ecx
10003184  |. 8B45 08        mov     eax, dword ptr [ebp+8]
10003187  |. 8945 FC        mov     dword ptr [ebp-4], eax
1000318A  |. 8B4D FC        mov     ecx, dword ptr [ebp-4]
1000318D  |. E8 4EFFFFFF    call    _Thread_func1
10003192  |. 8BE5           mov     esp, ebp
10003194  |. 5D            pop     ebp
10003195  \. C2 0400       retn    4
```

### _Thread_func1

```
100030E0 >/$ 55             push    ebp
100030E1  |. 8BEC           mov     ebp, esp
100030E3  |. 81EC 30010000  sub     esp, 130
100030E9  |. 898D D0FEFFFF  mov     dword ptr [ebp-130], ecx
100030EF  |. 8D8D D8FEFFFF  lea     ecx, dword ptr [ebp-128]
100030F5  |. E8 86FDFFFF    call    Fill_Rubbish_Code
100030FA  |> B8 01000000   /mov    eax, 1
100030FF  |. 85C0          |test    eax, eax
10003101  |. 74 38         |je     short 1000313B                 ; 这里根本跳不出来， 死循环一个。
10003103  |. 50            |push    eax
10003104  |. B8 01000000   |mov     eax, 1
10003109  |. B8 04000000   |mov     eax, 4
1000310E  |. B8 05000000   |mov     eax, 5
10003113  |. B8 06000000   |mov     eax, 6
10003118  |. B8 0A000000   |mov     eax, 0A
```
