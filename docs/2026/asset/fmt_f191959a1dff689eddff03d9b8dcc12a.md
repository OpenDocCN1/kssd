
`WinDbg`在`ParentRingInterruptDpc+0x55`处成功断下，这时我们查看`rbp+0x38`位置处的数据，会发现地址`0xffff8c8f2c234338`处的4字节数据便是虚拟机驱动中`connection_id`的值。我们可以通过这种办法判断当运行到`ParentRingInterruptDpc`函数时传输着什么虚拟设备的数据，方便之后跟踪调试特定设备的数据传输流程，减小调试的难度。

言归正传，我们继续介绍`vmbkmclr!KmclpVmbusIsr`函数之后的流程。在笔者的Linux虚拟机环境中，虚拟网卡的设备是`hv_netvsc`，它发送数据时`vmbus_channel`结构体中`offermsg.connection_id`的值为`0x10008`，这个值可能在不同的机器上会不一样。下面我们要跟踪网卡数据在宿主机驱动中的流向，首先先要逆向`vmbkmclr!KmclpVmbusIsr`函数，代码如下。

```
KmclpVmbusIsr      KmclpVmbusIsr   proc near        

KmclpVmbusIsr      var_48          = qword ptr -48h

KmclpVmbusIsr      var_38          = byte ptr -38h

KmclpVmbusIsr      var_30          = dword ptr -30h

KmclpVmbusIsr      var_2C          = dword ptr -2Ch

KmclpVmbusIsr      var_20          = qword ptr -20h

KmclpVmbusIsr      arg_8           = qword ptr  10h

KmclpVmbusIsr      arg_10          = qword ptr  18h

KmclpVmbusIsr

KmclpVmbusIsr                      mov     [rsp+arg_10], rbx

KmclpVmbusIsr+5                    push    rsi

KmclpVmbusIsr+6                    push    rdi

KmclpVmbusIsr+7                    push    r14

KmclpVmbusIsr+9                    sub     rsp, 50h

KmclpVmbusIsr+D                    mov     rax, cs:__security_cookie

KmclpVmbusIsr+14                   xor     rax, rsp

KmclpVmbusIsr+17                   mov     [rsp+68h+var_20], rax

KmclpVmbusIsr+1C                   inc     qword ptr [rcx+5C0h] ; transaction_id加1

KmclpVmbusIsr+23                   lea     rsi, [rcx+40h]

KmclpVmbusIsr+27                   mov     eax, [rsi+8Ch]

KmclpVmbusIsr+2D                   mov     rbx, rcx

KmclpVmbusIsr+30                   add     eax, [rsi+48h]

KmclpVmbusIsr+33                   mov     ecx, [rsi+4Ch]

KmclpVmbusIsr+36   loc_1C0002436:

KmclpVmbusIsr+36                   mov     [rsp+68h+arg_8], rbp

KmclpVmbusIsr+3B                   sub     eax, ecx

KmclpVmbusIsr+3D                   jz      loc_1C00059B2

KmclpVmbusIsr+43                   lea     eax, [rcx+1]

KmclpVmbusIsr+46                   mov     r14b, 1

KmclpVmbusIsr+49                   mov     [rsi+4Ch], eax

KmclpVmbusIsr+4C   loc_1C000244C: ; CODE XREF: KmclpVmbusIsr+35BB

KmclpVmbusIsr+4C                   lea     rcx, [rbx+380h] ; SpinLock

KmclpVmbusIsr+53                   xor     dil, dil

KmclpVmbusIsr+56                   call    cs:__imp_KeAcquireSpinLockAtDpcLevel

KmclpVmbusIsr+5C                   cmp     dword ptr [rbx+388h], 4

KmclpVmbusIsr+63                   jnz     short loc_1C000248D

KmclpVmbusIsr+65                   mov     rax, [rbx+58h]

KmclpVmbusIsr+69                   mov     dil, 1

KmclpVmbusIsr+6C                   mov     dword ptr [rax+8], 1

KmclpVmbusIsr+73                   mov     dword ptr [rbx+388h], 1

KmclpVmbusIsr+7D                   mov     rax, gs:188h

KmclpVmbusIsr+86                   mov     [rbx+3A8h], rax

KmclpVmbusIsr+8D   loc_1C000248D: ; CODE XREF: KmclpVmbusIsr+63

KmclpVmbusIsr+8D                   lea     rcx, [rbx+380h] ; SpinLock

KmclpVmbusIsr+94                   call    cs:__imp_KeReleaseSpinLockFromDpcLevel

KmclpVmbusIsr+9A                   test    dil, dil

KmclpVmbusIsr+9D                   jz      loc_1C0002546

KmclpVmbusIsr+A3                   cmp     qword ptr [rbx+4F0h], 0FFFFFFFFFFFFFFFFh

KmclpVmbusIsr+AB                   jz      loc_1C00059C0

KmclpVmbusIsr+B1                   mov     rax, 0FFFFF78000000320h

KmclpVmbusIsr+BB                   mov     rax, [rax]

KmclpVmbusIsr+BE                   mov     rdx, rax

KmclpVmbusIsr+C1                   sub     rdx, [rbx+4E0h]

KmclpVmbusIsr+C8                   jnz     loc_1C000257B

KmclpVmbusIsr+CE   loc_1C00024CE: ; CODE XREF: KmclpVmbusIsr+1A6

KmclpVmbusIsr+CE                                ; KmclpVmbusIsr+1DA

KmclpVmbusIsr+CE                   mov     rax, [rbx+4F0h]

KmclpVmbusIsr+D5                   cmp     [rbx+4E8h], rax

KmclpVmbusIsr+DC                   jnb     loc_1C00025DF

KmclpVmbusIsr+E2                   lea     rcx, [rsp+68h+var_38]

KmclpVmbusIsr+E7                   call    cs:__imp_KeQueryDpcWatchdogInformation

KmclpVmbusIsr+ED                   test    eax, eax

KmclpVmbusIsr+EF                   js      short loc_1C000250A

KmclpVmbusIsr+F1                   imul    ecx, [rsp+68h+var_30], 1Eh

KmclpVmbusIsr+F6                   mov     eax, 51EB851Fh

KmclpVmbusIsr+FB                   mul     ecx

KmclpVmbusIsr+FD                   shr     edx, 5

KmclpVmbusIsr+100                  cmp     [rsp+68h+var_2C], edx

KmclpVmbusIsr+104                  jb      loc_1C00025DF

KmclpVmbusIsr+10A  loc_1C000250A: ; CODE XREF: KmclpVmbusIsr+EF

KmclpVmbusIsr+10A                  rdtsc

KmclpVmbusIsr+10C                  shl     rdx, 20h

KmclpVmbusIsr+110                  mov     r8d, 1Eh

KmclpVmbusIsr+116                  or      rax, rdx

KmclpVmbusIsr+119                  mov     rcx, rbx

KmclpVmbusIsr+11C                  mov     dl, 2

KmclpVmbusIsr+11E                  mov     rdi, rax

KmclpVmbusIsr+121                  call    InpFillAndProcessQueue

KmclpVmbusIsr+126                  mov     r8d, eax

KmclpVmbusIsr+129                  rdtsc

KmclpVmbusIsr+12B                  shl     rdx, 20h

KmclpVmbusIsr+12F                  or      rax, rdx

KmclpVmbusIsr+132                  sub     rax, rdi

KmclpVmbusIsr+135                  add     [rbx+4E8h], rax

KmclpVmbusIsr+13C  loc_1C000253C: ; CODE XREF: KmclpVmbusIsr+35D3

KmclpVmbusIsr+13C                  mov     dl, 1

KmclpVmbusIsr+13E                  mov     rcx, rbx

KmclpVmbusIsr+141                  call    InpTransitionRunningQueue

KmclpVmbusIsr+146  loc_1C0002546: ; CODE XREF: KmclpVmbusIsr+9D

KmclpVmbusIsr+146                               ; KmclpVmbusIsr+208

KmclpVmbusIsr+146                  mov     edx, [rbx+0C8h]

KmclpVmbusIsr+14C                  mov     rbp, [rsp+68h+arg_8]

KmclpVmbusIsr+151                  test    edx, edx

KmclpVmbusIsr+153                  jnz     sub_1C0005A00

KmclpVmbusIsr+159  loc_1C0002559: 

KmclpVmbusIsr+159                  movzx   eax, r14b

KmclpVmbusIsr+15D                  mov     rcx, [rsp+68h+var_20]

KmclpVmbusIsr+162                  xor     rcx, rsp

KmclpVmbusIsr+165                  call    __security_check_cookie

KmclpVmbusIsr+16A                  mov     rbx, [rsp+68h+arg_10]

KmclpVmbusIsr+172                  add     rsp, 50h

KmclpVmbusIsr+176                  pop     r14

KmclpVmbusIsr+178                  pop     rdi

KmclpVmbusIsr+179                  pop     rsi

KmclpVmbusIsr+17A                  retn

```

通过`WinDbg`单步跟踪，我们发现`vmbkmclr!KmclpVmbusIsr`函数会调用函数`vmbkmclr!InpFillAndProcessQueue`。继续单步跟踪函数`vmbkmclr!InpFillAndProcessQueue`发现，`vmbkmclr!InpFillAndProcessQueue`函数中会直接调用`vmswitch`模块中的函数。在`InpFillAndProcessQueue+16E`处会调用`vmswitch!VmsVmNicPvtKmclProcessPacket`函数，这个函数`vmswitch`模块用于处理发来的数据；在`InpFillAndProcessQueue+2CA`处调用`vmswitch!VmsVmNicPvtKmclProcessingComplete`函数，这个函数也同样用于处理虚拟机发来的数据。这样，便从`VMBus`总线过渡到`vmswitch`之类的虚拟设备驱动中。之后在`vmswitch`驱动中的操作便是解析数据，并对解析后的数据进行下一步操作。

以上便是`Hyper-V`将虚拟机中数据传递给`Windows`内核中`Hyper-V`驱动组件（如`vmswitch.sys`）的流程，为了便于理解，我们将以上的流程概括成图1-33。

![](https://bbs.kanxue.com/upload/attach/201711/624619_zf7hsrxujc8epl1.png)

图1-33

### 4.5. Windows应用层

这一小节内容为`Hyper-V`把虚拟机中数据传递给`Windows`用户态下的`Hyper-V`组件的流程，我们会以`hyperv_fb`设备为例子，介绍数据从`Linux`虚拟机中的`hyperv_fb`设备到`Hyper-V`用户态组件`vmuidevices.dll`的过程。这个小节名字虽为`Windows`应用层，但实际上很大部分介绍还是在`Windows`内核态下。

首先我们继续上面小节的部分，介绍`vmbkmclr!KmclpVmbusManualIsr`这个分支之后的流程。`vmbkmclr!KmclpVmbusManualIsr`代码如下。
