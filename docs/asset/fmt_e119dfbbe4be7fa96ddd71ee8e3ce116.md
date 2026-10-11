
# DNF 驱动对内存读取的过滤分析

> 原文：[`www.pediy.com/kssd/pediy11/125060.html`](https://www.pediy.com/kssd/pediy11/125060.html)

下面是分析`tessafe.sys`对`NtReadVirtualMemory`调用的过滤处理方法，以及应对办法。

## 一、运行 DNF 后打开 windbg 定位到 NtReadVirtualMemory 函数位置，内核汇编代码如下：

`nt!NtReadVirtualMemory:`

```
    805b52b8 b808d316b3     mov     eax,0B316D308h  ;TesSafe.sys hook 位置
    805b52bd ffe0          jmp     eax
    805b52bf e8dc78f8ff    call    nt!_SEH_prolog (8053cba0)
    805b52c4 64a124010000  mov     eax,dword ptr fs:[00000124h]
    805b52ca 8bf8          mov     edi,eax
    805b52cc 8a8740010000  mov     al,byte ptr [edi+140h]
    805b52d2 8845e0        mov     byte ptr [ebp-20h],al
    805b52d5 8b7514       mov     esi,dword ptr [ebp+14h]
    805b52d8 84c0         test    al,al
    805b52da 7466         je      nt!NtReadVirtualMemory+0x8a (805b5342)
    805b52dc 8b450c       mov     eax,dword ptr [ebp+0Ch]
    805b52df 8d1430       lea     edx,[eax+esi]
    805b52e2 3bd0         cmp     edx,eax
```

定位到`0B316D308`位置，此部分代码就是`tessafe.sys`的过滤函数。下面是对这些汇编代码的分析，有点乱

```
    b316d308 8bff         mov     edi,edi
    b316d30a 55          push    ebp
    b316d30b 8bec        mov     ebp,esp
    b316d30d 81ec00010000 sub    esp,100h
    b316d313 60          pushad
    b316d314 9c          pushfd
    b316d315 b8206f17b3  mov     eax,0B3176F20h                ;ecx = 0B3176F20h
    b316d31a 33c9        xor     ecx,ecx                       ;ecx =0
    b316d31c 41          inc     ecx                           ;ecx = 1
    b316d31d f00fc108    lock xadd dword ptr [eax],ecx         ;[0B3176F20h] += 1
    b316d321 ff154c4017b3 call   dword ptr ds:[0B317404Ch]      ;call nt!PsGetCurrentProcess
    b316d327 8945f8      mov     dword ptr [ebp-8],eax         ;[ebp-8]=current EPROCESS
    b316d32a 6a00        push    0                             ; push 0
    b316d32c 8d45fc      lea     eax,[ebp-4]
    b316d32f 50          push    eax                           ;push ptr[ebp-4]
    b316d330 6a00        push    0                             ;push 0
    b316d332 a15c4117b3  mov     eax,dword ptr ds:[B317415Ch]    ;eax = [B317415Ch]=805649b8
    b316d337 ff30        push    dword ptr [eax]                 ;push 805649b8
    b316d339 6800040000  push    400h                          ;push 0x400
    b316d33e ff7508      push    dword ptr [ebp+8]             ;push 目标进程 ProcessHandle
    b316d341 ff159c4017b3 call   dword ptr ds:[0B317409Ch]      ;call ObReferenceObjectByHandle
    b316d347 8945f4      mov     dword ptr [ebp-0Ch],eax       ;判断函数调用是否成功
    b316d34a 837df400    cmp     dword ptr [ebp-0Ch],0
    b316d34e 7d02        jge     b316d352
    b316d350 eb65        jmp     b316d3b7                      ;不成功跳转
    b316d352 8b4dfc      mov     ecx,dword ptr [ebp-4]         ;ecx = 目标进程句柄信息
    b316d355 ff15584117b3 call   dword ptr ds:[0B3174158h]      ;call ObfDereferenceObject   撤销对象的引用计数
    b316d35b 8b45fc      mov     eax,dword ptr [ebp-4]         ;eax = 目标的句柄信息
    b316d35e 3b45f8      cmp     eax,dword ptr [ebp-8]         ;比较目标进程是否是本进程
    b316d361 7502        jne     b316d365                      ;不能，继续判断
    b316d363 eb52        jmp     b316d3b7                      ;相等不进行拦截
    b316d365 6a04        push    4                             ;push 4
    b316d367 ff75fc      push    dword ptr [ebp-4]             ;push 目标进程句柄信息
    b316d36a e8c3fdffff  call    b316d132                      ;判断目标进程是否是 DNF 进程
    b316d36f 0fb6c0      movzx   eax,al
    b316d372 85c0        test    eax,eax
    b316d374 7502        jne     b316d378
    b316d376 eb3f        jmp     b316d3b7                      ;目标进程不是 DNF 进程，不进行拦截
    b316d378 6a04        push    4                             ;push 4
    b316d37a ff75f8      push    dword ptr [ebp-8]             ;当前进程 eprocess
    b316d37d e8f0fcffff  call    b316d072                      ;比较当前进程是否在白名单中
    b316d382 0fb6c0      movzx   eax,al
    b316d385 85c0        test    eax,eax
    b316d387 7402        je      b316d38b                      ;在白名单中，继续判断
    b316d389 eb2c        jmp     b316d3b7                      ;不再则跳转，将屏蔽此次调用
    b316d38b 6a04        push    4                             ;push 4
    b316d38d ff75f8      push    dword ptr [ebp-8]             ;push 目标进程结构
    b316d390 e81bfdffff  call    b316d0b0                      ;判断是否在另外一份白名单中
    b316d395 0fb6c0      movzx   eax,al
    b316d398 85c0        test    eax,eax
    b316d39a 7402        je      b316d39e                      ;在第二份白名单中
    b316d39c eb19        jmp     b316d3b7                      ;不在第二份白名单中，将屏蔽此次调用
    b316d39e b8206f17b3  mov     eax,0B3176F20h
    b316d3a3 83c9ff      or      ecx,0FFFFFFFFh
    b316d3a6 f00fc108    lock xadd dword ptr [eax],ecx         ;[0B3176F20h] += -1
    b316d3aa 9d          popfd
    b316d3ab 61          popad
    b316d3ac 8be5        mov     esp,ebp
    b316d3ae 5d          pop     ebp
    b316d3af b80d0000c0  mov     eax,0C000000Dh                ;置返回错误代码
    b316d3b4 c21400      ret     14h                           ;不在白名单中，调用失败
    b316d3b7 b8206f17b3  mov     eax,0B3176F20h                ;[0B3176F20h] += -1
    b316d3bc 83c9ff      or      ecx,0FFFFFFFFh
    b316d3bf f00fc108    lock xadd dword ptr [eax],ecx
    b316d3c3 9d          popfd
    b316d3c4 61          popad
    b316d3c5 8be5        mov     esp,ebp
    b316d3c7 5d          pop     ebp
    b316d3c8 6a1c        push    1Ch
    b316d3ca 68f0ae4d80  push    offset nt!MmClaimParameterAdjustDownTime+0x90 (804daef0)
    b316d3cf 90          nop
    b316d3d0 90          nop
    b316d3d1 90          nop
    b316d3d2 90          nop
    b316d3d3 90          nop
    b316d3d4 90          nop
    b316d3d5 90          nop
    b316d3d6 90          nop
    b316d3d7 90          nop
    b316d3d8 ff253c6e17b3 jmp    dword ptr ds:[0B3176E3Ch]     ；跳转到
    b316d3de cc          int     3
    b316d3df cc          int     3
```

`b316d132` 如果为 DNF 进程返回 0 否则返回 1

```
    b316d132 8bff        mov     edi,edi
    b316d134 55          push    ebp
    b316d135 8bec        mov     ebp,esp
    b316d137 51          push    ecx
    b316d138 53          push    ebx
    b316d139 56          push    esi
    b316d13a c645ff00    mov     byte ptr [ebp-1],0           ;[ebp-1] = 0;
    b316d13e 32db        xor     bl,bl                         ;bl = 0;
```
