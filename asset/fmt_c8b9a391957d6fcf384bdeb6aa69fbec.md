
# 浅谈 `hookport.sys`

> 原文：[`www.pediy.com/kssd/pediy12/127820.html`](https://www.pediy.com/kssd/pediy12/127820.html)

前几天在论坛上看见一篇 `360HOOK` 框架的文章，写得不错，其实 `hookport.sys` 之前很多人都逆过了，而且教主写了一套很详细的文章。本人才疏学浅，只是补充一些科普知识，好像一些像我一样的菜鸟成长起来。这篇文章主要介绍一些 `360` 的比较精辟的应用，至于它 `HOOK` 了那些函数大家可以去网上搜索教主那篇文章。其实就是一个结构。

### 亮点一：HOOK `KiFastCallEntry`

科普一下 `KiFastCallEntry` 快速系统调用 `SYSENTER` 就是说每次系统调用的时候，最终都会通过 `KiFastCallEntry`。

我们来看看 `KiFastCallEntry` 函数比较关键代码：

```
8054257d 8bb324010000    mov     esi,dword ptr [ebx+124h] //获得 CurrentThread
80542583 ff33            push    dword ptr [ebx]
80542585 c703ffffffff    mov     dword ptr [ebx],0FFFFFFFFh
8054258b 8b6e18          mov     ebp,dword ptr [esi+18h]
8054258e 6a01            push    1
80542590 83ec48          sub     esp,48h
80542593 81ed9c020000    sub     ebp,29Ch
80542599 c6864001000001  mov     byte ptr [esi+140h],1
805425a0 3bec            cmp     ebp,esp
805425a2 758d            jne     nt!KiFastCallEntry2+0x49 (80542531)
805425a4 83652c00        and     dword ptr [ebp+2Ch],0
805425a8 f6462cff        test    byte ptr [esi+2Ch],0FFh
805425ac 89ae34010000    mov     dword ptr [esi+134h],ebp
805425b2 0f8538feffff    jne     nt!Dr_FastCallDrSave (805423f0)
805425b8 8b5d60          mov     ebx,dword ptr [ebp+60h]
805425bb 8b7d68          mov     edi,dword ptr [ebp+68h]
805425be 89550c          mov     dword ptr [ebp+0Ch],edx
805425c1 c74508000ddbba  mov     dword ptr [ebp+8],0BADB0D00h
805425c8 895d00          mov     dword ptr [ebp],ebx
805425cb 897d04          mov     dword ptr [ebp+4],edi
805425ce fb              sti
805425cf 8bf8            mov     edi,eax //eax=SSDTindex
805425d1 c1ef08          shr     edi,8  //除以 256
805425d4 83e730          and     edi,30h //要就是 0 要就是 0X10 为了判断是 SSDTSHADOW 还是 SSDT
805425d7 8bcf            mov     ecx,edi
805425d9 03bee0000000    add     edi,dword ptr [esi+0E0h] //通过 esi=KTHREAD->ServiceTable 获得当前线程使用的 ServiceTable;
805425df 8bd8            mov     ebx,eax //SSDTID
805425e1 25ff0f0000      and     eax,0FFFh
805425e6 3b4708          cmp     eax,dword ptr [edi+8]
805425e9 0f8333fdffff    jae     nt!KiBBTUnexpectedRange (80542322)
805425ef 83f910          cmp     ecx,10h //这里判断是 SSDT 还是 SSDTSHADOW
805425f2 751b            jne     nt!KiFastCallEntry+0xcf (8054260f)
805425f4 648b0d18000000  mov     ecx,dword ptr fs:[18h]
805425fb 33db            xor     ebx,ebx
805425fd 0b99700f0000    or      ebx,dword ptr [ecx+0F70h]
80542603 740a            je      nt!KiFastCallEntry+0xcf (8054260f)
80542605 52              push    edx
80542606 50              push    eax
80542607 ff1548d75580    call    dword ptr [nt!KeGdiFlushUserBatch (8055d748)]
8054260d 58              pop     eax
8054260e 5a              pop     edx
8054260f 64ff0538060000  inc     dword ptr fs:[638h] //使用计数
80542616 8bf2            mov     esi,edx
80542618 8b5f0c          mov     ebx,dword ptr [edi+0Ch]
8054261b 33c9            xor     ecx,ecx
8054261d 8a0c18          mov     cl,byte ptr [eax+ebx]
80542620 8b3f            mov     edi,dword ptr [edi]
80542622 8b1c87          mov     ebx,dword ptr [edi+eax*4]
80542625 e9d69c8d09      jmp     89e1c300   //这个就是 360 的 HOOK 了，这个就是 360 的亮点之处
```

上面 `EDI=SSDTaddr` `EBX=系统服务地址` `EAX=SSDTindex` 有了这几个寄存器省了很多麻烦了。

```
8054262a 8bfc            mov     edi,esp
8054262c 3b3534315680    cmp     esi,dword ptr [nt!MmUserProbeAddress (80563134)]
80542632 0f83a8010000    jae     nt!KiSystemCallExit2+0x9f (805427e0)
80542638 f3a5            rep movs dword ptr es:[edi],dword ptr [esi]
8054263a ffd3            call    ebx     //服务调用
8054263c 8be5            mov     esp,ebp
```

### 第二步，我们来看看 360 是怎么 HOOK 的

```
mov     edi, edi
push    ebp
mov     ebp, esp
sub     esp, 18h
push    ebx
push    esi
push    edi
push    offset aZwsetevent ; "ZwSetEvent"
lea     eax, [ebp+DestinationString]
push    eax
call    ds:RtlInitAnsiString //初始化
lea     eax, [ebp+DestinationString]
push    eax
call    sub_17A78
xor     esi, esi
cmp     eax, esi
jz      loc_195B4
mov     eax, [eax+1]  //获取 ZwSetEventINDEXID =DB
mov     ZwSetEventaddrid, eax
lea     eax, [ebp+SpinLock]
push    eax             ; SpinLock
call    ds:KeInitializeSpinLock
mov     edi, ds:KfAcquireSpinLock
lea     ecx, [ebp+SpinLock] ; SpinLock
call    edi ; KfAcquireSpinLock
mov     cl, al
push    eax
push    ebx
push    edx
cli
mov     eax, cr0
mov     [ebp+var_C], eax
and     eax, 0FFFEFFFFh
mov     cr0, eax
mov     eax, ZwSetEventaddrid //EAX=INDEXID
mov     edx, KeServiceDescriptorTableaddr//EDX=SSDTADDR
mov     edx, [edx]
mov     ebx, [edx+eax*4] //获取 ZwSetEventADDR
mov     ZwSetEventaddr, ebx //保持
mov     ebx, offset sub_192A8//HOOK
mov     [edx+eax*4], ebx
mov     eax, [ebp+var_C]
mov     cr0, eax
sti
pop     edx
pop     ebx
pop     eax
mov     dl, cl          ; NewIrql
lea     ecx, [ebp+SpinLock] ; SpinLock
call    ds:KfReleaseSpinLock
mov     eax, 288C58F1h  //这个比较关键，主要是确定是不是自己调用
push    esi             ; PreviousState
push    eax             ; EventHandle
mov     dword_1B850, eax //保存
call    ds:ZwSetEvent
mov     eax, fanhuiaddrs //调用后返回地址 8054263c
cmp     eax, esi
jnz     short loc_19510
mov     eax, P
cmp     eax, esi
jmp     loc_195AB
cmp     addbp4addr5, esi//这个是 HOOK 返回地址 8054262a
jnz     loc_19629
```

### 第三步，我们来看下它的 HOOK 函数

```
mov     edi, edi
.text:000192AA             push    ebp
.text:000192AB             mov     ebp, esp
.text:000192AD             push    ecx
.text:000192AE             push    ecx
.text:000192AF             mov     eax, [ebp+arg_0] //获取第一个参数 EventHandle
.text:000192B2             cmp     eax, dword_1B850 //看看是不是自己调用
.text:000192B8             push    ebx
.text:000192B9             push    esi
.text:000192BA             push    edi
.text:000192BB             jnz     loc_193CB   //如果不是就跳到原服务调用
.text:000192C1             call    ds:ExGetPreviousMode
.text:000192C7             test    al, al
.text:000192C9             jnz     loc_193CB//比较是不是 USERMODULE 调用是酒跳
.text:000192CF             push    206B6444h       ; Tag
.text:000192D4             push    5               ; NumberOfBytes
.text:000192D6             push    0               ; PoolType
.text:000192D8             call    ds:ExAllocatePoolWithTag //声请一块 5 字节内存
.text:000192DE             test    eax, eax
.text:000192E0             mov     P, eax    //保存到全局变量
.text:000192E5             jz      loc_193C7
.text:000192EB             mov     byte ptr [eax], 0E9h //第一个 E9 JMP
```
