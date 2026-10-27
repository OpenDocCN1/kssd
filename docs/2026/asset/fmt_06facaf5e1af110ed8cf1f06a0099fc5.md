
`b316d082` `32c0` `xor` `al,al`                                                                                                                                                        ;`a1` = 0

`b316d084` `3bca` `cmp` `ecx,edx`                                                                                                                                                ;判断是否是空链表

`b316d086` `741f` `je` `TesSafe+0x30a7` (`b316d0a7`)                                                                        ;空链表跳转

`b316d088` `56` `push` `esi`

`b316d089` `8b7508` `mov` `esi,dword ptr [ebp+8]`                                                                                        ;`esi`=当前进程`EPROCESS`

`b316d08c` `3b71dc` `cmp` `esi,dword ptr [ecx-24h]`                                                                                ;比较目标进程是否在白名单中

`b316d08f` `7408` `je` `TesSafe+0x3099` (`b316d099`)                                                                        ;相等则跳转

`b316d091` `8b09` `mov` `ecx,dword ptr [ecx]`                                                                                                ；取下一个节点

`b316d093` `3bca` `cmp` `ecx,edx`                                                                                                                                                ;判断是否到链表结尾

`b316d095` `75f2` `jne` `TesSafe+0x3089` (`b316d089`)                                                                        ;不到则继续判断

`b316d097` `eb0d` `jmp` `TesSafe+0x30a6` (`b316d0a6`)                                                                        ;到链表结尾，而且没有在白名单中匹配

`b316d099` `8b550c` `mov` `edx,dword ptr [ebp+0Ch]`                                                                                ;`edx` =4

`b316d09c` `ff41fc` `inc` `dword ptr [ecx-4]`                                                                                                        ;[`ecx-4`]++

`b316d09f` `8551e4` `test` `dword ptr [ecx-1Ch],edx`                                                                                ;[`ecx-1ch`] 是否等于4 不等于4 则屏蔽

`b316d0a2` `7402` `je` `TesSafe+0x30a6` (`b316d0a6`)

`b316d0a4` `b001` `mov` `al,1`

`b316d0a6` `5e` `pop` `esi`

`b316d0a7` `5d` `pop` `ebp`

`b316d0a8` `c20800` `ret` `8`

在`windbg` 查看

```
lkd> dd 89a3442c-24
89a34408  8a5aa288 00000500 000fffff e8710e72
89a34418  74737953 00006d65 00000000 00000000
89a34428  00000000 88afc274 b31771a8 005c003f
89a34438  0a090007 68734c4b 89d5c8f0 89ad8778
89a34448  896ec690 00000000 00000000 897e47a8
89a34458  89d5dbf0 89ad87dc 00010000 00000001
89a34468  0001502b 00000000 e3a67220 00000008
89a34478  00000000 00000000 0a080009 e174754d
```

```
lkd> !process 8a5aa288 0
Unable to read selector for PCR for processor 0
PROCESS 8a5aa288  SessionId: none  Cid: 0004    Peb: 00000000  ParentCid: 0000
    DirBase: 0aed8020  ObjectTable: e1002e38  HandleCount: 474.
    Image: System
```

下面我找到白名单进程

`System`

`smss.exe`

`csrss.exe`

`SERVICES.EXE`

`LSASS.EXE`

`SVCHOST.EXE`

```
//=========================== 
```

判断是否在白名单2中

```
b2a510b0 8bff            mov     edi,edi
      b2a510b2 55              push    ebp
      b2a510b3 8bec            mov     ebp,esp
      b2a510b5 51              push    ecx
      b2a510b6 53              push    ebx
      b2a510b7 56              push    esi
      b2a510b8 c645ff00        mov     byte ptr [ebp-1],0                                        ;[ebp-1] = 0
      b2a510bc 32db            xor     bl,bl                                                                                                                                                                                        ;b1 = 0
      b2a510be ff150880a5b2    call    dword ptr [TesSafe+0xa008 (b2a58008)]                                      call KeGetCurrentIrql
      b2a510c4 3c02            cmp     al,2                                                                                                                                                                                                比较当前IRQL级别是否为DISPATCH_LEVEL级别
      b2a510c6 beb8b1a5b2      mov     esi,offset TesSafe+0xd1b8 (b2a5b1b8)                                                                esi = b2a5b1b8
      b2a510cb 8bce            mov     ecx,esi                                                                                                                                                                                ecx = b2a5b1b8
      b2a510cd 720a            jb      TesSafe+0x30d9 (b2a510d9)                                                                                                        ;小于DISPATCH_LEVEL级别 跳转
      b2a510cf ff157880a5b2    call    dword ptr [TesSafe+0xa078 (b2a58078)]                                                        ;call Ke386QueryIoAccessMap
      b2a510d5 fec3            inc     bl                                                                                                                                                                                                        ;b1 = 1
      b2a510d7 eb09            jmp     TesSafe+0x30e2 (b2a510e2)
      b2a510d9 ff150c80a5b2    call    dword ptr [TesSafe+0xa00c (b2a5800c)]                                                        ;call KfAcquireSpinLock
      b2a510df 8845fe          mov     byte ptr [ebp-2],al                                                                                                                          [ebp-2] 保存当前IRQL
      b2a510e2 a1b0b1a5b2      mov     eax,dword ptr [TesSafe+0xd1b0 (b2a5b1b0)]                                        ;eax = [b2a5b1b0]=8993e02c
      b2a510e7 b9b0b1a5b2      mov     ecx,offset TesSafe+0xd1b0 (b2a5b1b0)                                                                ;ecx = b2a5b1b0
      b2a510ec eb0a            jmp     TesSafe+0x30f8 (b2a510f8)
      b2a510ee 8b50dc          mov     edx,dword ptr [eax-24h]
      b2a510f1 3b5508          cmp     edx,dword ptr [ebp+8]                                                                                                                        ;白名单判断
      b2a510f4 7408            je      TesSafe+0x30fe (b2a510fe)
      b2a510f6 8b00            mov     eax,dword ptr [eax]
      b2a510f8 3bc1            cmp     eax,ecx
      b2a510fa 75f2            jne     TesSafe+0x30ee (b2a510ee)
      b2a510fc eb0f            jmp     TesSafe+0x310d (b2a5110d)
      b2a510fe 8b4d0c          mov     ecx,dword ptr [ebp+0Ch]
      b2a51101 ff40fc          inc     dword ptr [eax-4]
      b2a51104 8548e4          test    dword ptr [eax-1Ch],ecx
      b2a51107 7404            je      TesSafe+0x310d (b2a5110d)
      b2a51109 c645ff01        mov     byte ptr [ebp-1],1
      b2a5110d 84db            test    bl,bl
      b2a5110f 8bce            mov     ecx,esi
      b2a51111 7408            je      TesSafe+0x311b (b2a5111b)
      b2a51113 ff157480a5b2    call    dword ptr [TesSafe+0xa074 (b2a58074)]
      b2a51119 eb09            jmp     TesSafe+0x3124 (b2a51124)
      b2a5111b 8a55fe          mov     dl,byte ptr [ebp-2]
      b2a5111e ff151080a5b2    call    dword ptr [TesSafe+0xa010 (b2a58010)]
      b2a51124 8a45ff          mov     al,byte ptr [ebp-1]
      b2a51127 5e              pop     esi
      b2a51128 5b              pop     ebx
      b2a51129 c9              leave
      b2a5112a c20800          ret     8
```

`TXPlatform.exe`

`QQ.exe`

二 、过滤处理函数的伪代码

```
int g_globalValue;
int MyNtReadVirtualMemory(IN HANDLE ProcessHandle,
                                              IN PVOID BaseAddress,
                                              OUT PVOID Buffer,
                                               IN SIZE_T NumberOfBytesToRead,
```
