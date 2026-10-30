
```
.text:00010D62                 mov     ebx, [ebp+Irp]

.text:00010D65                 mov     esi, [ebx+60h]  ; 取出CurrentStackLocation赋给esi

.text:00010D68                 mov     ecx, [esi+_IO_STACK_LOCATION.Parameters.DeviceIoControl.Type3InputBuffer]

.text:00010D6B                 mov     [ebp+var_InputBuffer], ecx

.text:00010D6E                 mov     edi, [ebx+IRP.UserBuffer] ; 将输出缓冲区的地址赋给edi

.text:00010D71                 mov     eax, [esi+_IO_STACK_LOCATION.Parameters.DeviceIoControl.InputBufferLength]

.text:00010D74                 mov     [ebp+var_InputBufferLength], eax

.text:00010D77                 mov     edx, [esi+_IO_STACK_LOCATION.Parameters.DeviceIoControl.OutputBufferLength]

.text:00010D7A                 mov     [ebp+var_OutBufferLength], edx

.text:00010D7D                 mov     eax, ds:MmUserProbeAddress

.text:00010D82                 mov     eax, [eax]

.text:00010D84                 cmp     edi, eax        ; 输出缓冲区起始地址是否在合理范围

.text:00010D86                 ja      loc_10EF2       ; 如果大于这个范围，则跳转到函数运行失败的代码执行

.text:00010D8C                 add     edx, edi        ; edx是输出缓冲区长度，加上起始地址就是输出缓冲区的结束地址

.text:00010D8E                 cmp     edx, eax        ; 输出缓冲区结束地址是否在合理范围

.text:00010D90                 ja      loc_10EF2       ; 如果大于这个范围，则跳转到函数运行失败的代码执行

                。。。。

.text:00010EF2 loc_10EF2:                              ; CODE XREF: DispatchIoCtrl+5A↑j

.text:00010EF2                                         ; DispatchIoCtrl+64↑j ...

.text:00010EF2                 mov     dword ptr [ebp-1Ch], STATUS_INVALID_PARAMETER

.text:00010EF9 ; START OF FUNCTION CHUNK FOR DispatchIoCtrl

.text:00010EF9

.text:00010EF9 loc_10EF9:                              ; CODE XREF: DispatchIoCtrl+EC↑j

.text:00010EF9                                         ; DispatchIoCtrl+F8↑j ...

.text:00010EF9                 mov     eax, [ebp+var_OutBufferLength]

.text:00010EFC                 mov     [ebx+IRP.IoStatus.Information], eax

.text:00010EFF                 mov     esi, [ebp+var_Status]

.text:00010F02                 mov     [ebx+IRP.IoStatus.anonymous_0.Status], esi

.text:00010F05                 xor     dl, dl          ; PriorityBoost

.text:00010F07                 mov     ecx, ebx        ; Irp

.text:00010F09                 call    ds:IofCompleteRequest

.text:00010F0F                 mov     eax, esi

.text:00010F11

.text:00010F11 loc_10F11:                              ; CODE XREF: DispatchIoCtrl+31↑j

.text:00010F11                 call    __SEH_epilog

.text:00010F16                 retn    8
```

接着程序会调用`ProbeForRead`和`ProbeForWrite`对输入输出缓冲区进行读写检查。

```
.text:00010D96                 and     [ebp+ms_exc.registration.TryLevel], 0

.text:00010D9A                 push    1               ; Alignment

.text:00010D9C                 push    [ebp+var_InputBufferLength] ; Length

.text:00010D9F                 push    ecx             ; Address

.text:00010DA0                 call    ds:ProbeForRead

.text:00010DA6                 push    1               ; Alignment

.text:00010DA8                 push    [ebp+var_OutBufferLength] ; Length

.text:00010DAB                 push    edi             ; Address

.text:00010DAC                 call    ds:ProbeForWrite
```

由于，在调用`ProbeForWrite`的时候，如果它的第二个参数，也就是输出缓冲区的长度如果是0的话，那么这个函数的检查就会被绕过。所以这就产生了漏洞。只要输出缓冲区的长度是0，那么就算输出缓冲区是个非法的地址，程序依然就继续执行下去。

接下来程序就会取出发送的`IOCTL`，并根据`IOCTL`的值来决定要执行的代码

```
.text:00010DB2                 or      [ebp+ms_exc.registration.TryLevel], 0FFFFFFFFh

.text:00010DB6                 mov     esi, [esi+_IO_STACK_LOCATION.Parameters.DeviceIoControl.IoControlCode]

.text:00010DB9                 cmp     esi, 83003C07h

.text:00010DBF                 jz      loc_10E91

.text:00010DC5                 cmp     esi, 83003C0Bh

.text:00010DCB                 jz      loc_10E78

.text:00010DD1                 cmp     esi, 83003C0Fh

.text:00010DD7                 jz      short loc_10E4B

.text:00010DD9                 cmp     esi, 83003C13h

.text:00010DDF                 jz      short loc_10E3A

.text:00010DE1                 cmp     esi, 83003C17h

.text:00010DE7                 jz      short loc_10E29

.text:00010DE9                 cmp     esi, 83003C1Bh

.text:00010DEF                 jz      short loc_10E1D

.text:00010DF1                 cmp     esi, 83003C1Fh

.text:00010DF7                 jnz     loc_10EF2
```

对于第一个`IOCTL`，它的最后两位是`11`(`0x3`)，那么对应的通信方式就是`METHOD_NEITHER`。该通信方式会直接对传入的输入输出地址进行操作，很可能有漏洞，跟进`loc_10E91`查看其中的内容。

```
.text:00010E91 loc_10E91:                              ; CODE XREF: DispatchIoCtrl+93↑j

.text:00010E91                 mov     ecx, Handle     ; Handle

.text:00010E97                 cmp     byte ptr [ecx+2AFCh], 0

.text:00010E9E                 jnz     short loc_10ED2

.text:00010EA0                 push    4

.text:00010EA2                 pop     esi             ; 将esi赋值为4

.text:00010EA3                 cmp     [ebp+var_InputBufferLength], esi ; 判断输入缓冲区长度是否小于4

.text:00010EA6                 jb      short loc_10ED2 ; 如果小于4则跳转失败代码进行执行

.text:00010EA8                 mov     eax, [ebp+var_InputBuffer] ; 获取输入缓冲区地址

.text:00010EAB                 push    dword ptr [eax] ; Handle

.text:00010EAD                 call    GetObjectInfo   ; 将输入缓冲区中的前4字节的内容入栈，在调用函数

.text:00010EB2                 test    eax, eax        ; 返回值是否为0

.text:00010EB4                 jnz     short loc_10EBF ; 不为0，则对输出缓冲区进行赋值

.text:00010EB6                 mov     [ebp+var_Status], STATUS_UNSUCCESSFUL

.text:00010EBD                 jmp     short loc_10EC4

.text:00010EBF ; ---------------------------------------------------------------------------

.text:00010EBF

.text:00010EBF loc_10EBF:                              ; CODE XREF: DispatchIoCtrl+188↑j

.text:00010EBF                 mov     [edi], eax      ; 将返回值赋值到输出缓冲区中

.text:00010EC1                 mov     [ebx+IRP.IoStatus.Information], esi
```

程序首先判断输入缓冲区长度是否大于等于4，如果满足条件，则把输入缓冲区前4字节的内容压入栈中并调用`GetObjectInfo`函数。函数的返回值如果是0，则会跳过对输出缓冲区的赋值，否则就会将返回值赋到输出缓冲区中。

而在`GetObjectInfo`中，函数会将传入的输入缓冲区的4字节参数作为句柄传给`ObReferenceObjectByHandle`来获得相应的对象，随后会从这个对象中取出相应的内容来给`eax`赋值。

```
.text:000105C3 loc_105C3:                              ; CODE XREF: GetObjectInfo+13↑j

.text:000105C3                 push    ebx             ; HandleInformation

.text:000105C4                 push    esi             ; Object

.text:000105C5                 push    ebx             ; AccessMode

.text:000105C6                 push    ebx             ; ObjectType

.text:000105C7                 push    10000000h       ; DesiredAccess

.text:000105CC                 push    [esp+28h+arg_Handle] ; Handle

.text:000105D0                 call    ds:ObReferenceObjectByHandle

.text:000105D6                 test    eax, eax

.text:000105D8                 jge     short loc_105E1

.text:000105DA                 xor     eax, eax

.text:000105DC                 jmp     loc_106A3

                。。。

.text:0001068B loc_1068B:                              ; CODE XREF: GetObjectInfo+C4↑j

.text:0001068B                 cmp     [esi+2B50h], ebx

.text:00010691                 jz      short loc_10652

.text:00010693                 mov     eax, [esi+2B4Ch]

.text:00010699                 mov     eax, [eax+18h]

.text:0001069C                 or      eax, [esi+2B50h]
```

# 三.漏洞利用

对上面的分析进行总结，可以得出如下的结论:

1. `IOCTL`等于`0x83003C07`的时候，用户层和内核层会采用`METHOD_NEITHER`的方式进行通信，该方式会直接对传入的输入输出的地址进行操作。

2. 程序虽然会调用`ProbeForWrite`对输出缓冲区进行检查，但是没对输出缓冲区长度进行检查，如果传入的参数是0，那么此函数的检查将失效

3. 对输出缓冲区的赋值是将输入的前4字节作为句柄进行解析，将解析到的对象的内容赋给输出缓冲区

根据这三个结论就可以知道，要利用这个漏洞只需要输入缓冲区给一个合适的句柄对象，输出缓冲区的地址给一个非法的地址且输出缓冲区的长度设置为0。那么，此时对输出缓冲区的赋值就会导致蓝屏错误，这就造成了拒绝服务攻击，具体代码如下。
