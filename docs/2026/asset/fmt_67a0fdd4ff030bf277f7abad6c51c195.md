
```nasm
.text:00413E4D mov al, byte ptr ds:\_IopLargeIrpStackLocations
.text:00413E52 cmp [ebp+StackSize], al
.text:00413E55 jg loc\_41B7DF ; 大于则跳转
.text:00413E5B cmp [ebp+ChargeQuota], dl
.text:00413E5E jnz loc\_41B7D3
.。。
.text:0041B7D3 loc\_41B7D3:
.text:0041B7D3 cmp [edi+\_KPRCB.LookasideIrpFloat], ebx
.text:0041B7D9 jg loc\_413E64
```

如果满足条件，接下来判断栈大小是否等于 1，来判断从哪个快表中分配 IRP 对象。

```nasm
.text:00413E64 loc\_413E64: ; CODE XREF: IopAllocateIrpPrivate(x,x)+79CE↓j
.text:00413E64 mov [ebp+var\_FixedSize], IRP\_ALLOCATED\_FIXED\_SIZE
.text:00413E68 cmp [ebp+StackSize], 1
.text:00413E6C jz short loc\_413E81
```

随后调用 `ExInterlockedPopEntrySList`，从 P 备份链表中分配 IRP 对象。

```nasm
.text:00413E81 loc\_413E81: ; CODE XREF: IopAllocateIrpPrivate(x,x)+61↑j
.text:00413E81 mov esi, [edi+ebx\*8+\_KPRCB.PPLookasideList.P]
.text:00413E88 inc [esi+\_GENERAL\_LOOKASIDE.TotalAllocates]
.text:00413E8B mov ecx, esi
.text:00413E8D call @ExInterlockedPopEntrySList@8 ; ExInterlockedPopEntrySList(x,x)
.text:00413E92 mov edx, eax
.text:00413E94 mov [ebp+var\_IRP], edx
.text:00413E97 test edx, edx
.text:00413E99 jz loc\_41275A
```

如果从 P 备份链表中分配失败，则尝试从 L 备份链表中分配。

```nasm
.text:0041275A loc\_41275A: ; CODE XREF: IopAllocateIrpPrivate(x,x)+8E↓j
.text:0041275A ; \_\_unwind { // \_\_SEH\_prolog
.text:0041275A inc [esi+\_GENERAL\_LOOKASIDE.anonymous\_0.AllocateMisses]
.text:0041275D mov esi, [edi+ebx\*8+\_KPRCB.PPLookasideList.L]
.text:00412764 inc [esi+\_GENERAL\_LOOKASIDE.TotalAllocates]
.text:00412767 mov ecx, esi
.text:00412769 call @ExInterlockedPopEntrySList@8 ; ExInterlockedPopEntrySList(x,x)
.text:0041276E mov edx, eax
.text:00412770 mov [ebp+var\_IRP], edx
.text:00412773 jmp loc\_413E9F
```

判断 IRP 对象是否分配成功，此时 `edx` 指向的就是从快表中分配的 IRP 对象。

```nasm
.text:00413EC6 xor eax, eax
.text:00413EC8 cmp edx, eax
.text:00413ECA jz loc\_42331E
```

如果分配不成功，则根据配额是否为 0 来分配 IRP 对象。如果配额不为 0，则调用函数 `ExAllocatePoolWithQuotaTag` 来分配 IRP 对象。

```nasm
.text:004232EF loc\_4232EF: ; CODE XREF: IopAllocateIrpPrivate(x,x)+F516↓j
.text:004232EF ; IopAllocateIrpPrivate(x,x)+F51B↓j
.text:004232EF cmp [ebp+ChargeQuota], 0
.text:004232F3 jz short loc\_423328
.text:004232F8 push ' prI' ; Tag
.text:004232FD movzx ecx, word ptr [ebp+var\_allocateSize]
.text:00423301 push ecx ; NumberOfBytes
.text:00423302 push eax ; PoolType
.text:00423303 call \_ExAllocatePoolWithQuotaTag@12 ; ExAllocatePoolWithQuotaTag(x,x,x)
.text:00423308 mov edx, eax
.text:0042330A mov [ebp+var\_IRP], edx
```

如果配额为 0，则调用 `ExAllocatePoolWithTag` 来分配 IRP 对象。

```nasm
.text:00423328 loc\_423328: ; CODE XREF: IopAllocateIrpPrivate(x,x)+F4E8↑j
.text:00423328 push ' prI' ; Tag
.text:0042332D movzx ecx, word ptr [ebp+var\_allocateSize]
.text:00423331 push ecx ; NumberOfBytes
.text:00423332 push eax ; PoolType
.text:00423333 call \_ExAllocatePoolWithTag@12 ; ExAllocatePoolWithTag(x,x,x)
.text:00423338 mov edx, eax
.text:0042333A jmp short loc\_423311
```

成功分配 IRP 对象以后，接着对 IRP 对象进行赋值。

```nasm
.text:00413EDE loc\_413EDE: ; CODE XREF: IopAllocateIrpPrivate(x,x)+F508↓j
.text:00413EDE movzx ecx, word ptr [ebp+var\_allocateSize]
.text:00413EE2 xor eax, eax
.text:00413EE4 mov edi, edx
.text:00413EE6 mov esi, ecx
.text:00413EE8 shr ecx, 2
.text:00413EEB rep stosd
.text:00413EED mov ecx, esi
.text:00413EEF and ecx, 3
.text:00413EF2 rep stosb
.text:00413EF4 mov [edx+\_IRP.Type], IO\_TYPE\_IRP
.text:00413EF9 mov ax, word ptr [ebp+var\_allocateSize]
.text:00413EFD mov [edx+\_IRP.Size], ax
.text:00413F01 mov cl, [ebp+StackSize]
.text:00413F04 mov [edx+\_IRP.StackCount], cl
.text:00413F07 mov al, cl
.text:00413F09 inc al
.text:00413F0B mov [edx+\_IRP.CurrentLocation], al
.text:00413F0E mov eax, large fs:124h
.text:00413F14 mov esi, eax
.text:00413F16 mov al, [esi+\_KTHREAD.ApcStateIndex]
.text:00413F1C mov [edx+\_IRP.ApcEnvironment], al
.text:00413F1F lea eax, [edx+\_IRP.ThreadListEntry]
.text:00413F22 mov [eax+LIST\_ENTRY.Blink], eax
.text:00413F25 mov [eax+LIST\_ENTRY.Flink], eax
.text:00413F27 movsx eax, cl
.text:00413F2A lea eax, [eax+eax\*8]
.text:00413F2D lea eax, [edx+eax\*4+(size \_IRP)]
.text:00413F31 mov [edx+\_IRP.Tail.Apc.NormalContext], eax
.text:00413F34 or bl, [ebp+var\_FixedSize]
.text:00413F37 mov [edx+\_IRP.AllocationFlags], bl
.text:00413F3A cmp [ebp+ChargeQuota], 0
```

## 3. 释放 IRP

IRP 的释放通过 `IoFreeIrp` 来实现。但是，和分配 IRP 对象一样，该函数只是用来实现函数转发，真正释放 IRP 对象是在 `IopFreeIrp` 中实现的。

```nasm
.text:00414012 ; void \_\_stdcall IoFreeIrp(PIRP Irp)
.text:00414012 public \_IoFreeIrp@4
.text:00414012 \_IoFreeIrp@4 proc near ; CODE XREF: IopCompleteRequest(x,x,x,x,x)-894B↑p
.text:00414012 ; IopfCompleteRequest(x,x)+992C↓p ...
.text:00414012
.text:00414012 Irp = dword ptr 8
.text:00414012
.text:00414012 mov edi, edi
.text:00414014 push ebp
.text:00414015 mov ebp, esp
.text:00414017 pop ebp
.text:00414018 jmp ds:\_pIoFreeIrp
.text:00414018 \_IoFreeIrp@4 endp
```

在 `IopFreeIrp` 中会对 IRP 对象的 `AllocationFlags` 进行判断。

```nasm
.text:00413F90 mov cl, [esi+\_IRP.AllocationFlags]
.text:00413F93 test cl, IRP\_ALLOCATED\_FIXED\_SIZE
.text:00413F96 jz loc\_423345
.text:00413F9C test cl, IRP\_ALLOCATED\_MUST\_SUCCEED
.text:00413F9F jnz loc\_423345
```

如果是从池内存中分配的 IRP 对象，则调用 `ExFreePoolWithTag` 来释放 IRP 对象。

```nasm
.text:00423345 push edi
.text:00423346 jmp loc\_41274F
.text:0041274F loc\_41274F: ; CODE XREF: IopFreeIrp(x)+F3E5↓j
.text:0041274F push esi ; P
.text:00412750 call \_ExFreePoolWithTag@8 ; ExFreePoolWithTag(x,x)
.text:00412755 jmp loc\_414006
```

否则，在释放内存以后，将其归还到备份链表中。

```nasm
.text:00413FFD loc\_413FFD: ; CODE XREF: IopFreeIrp(x)+F530↓j
.text:00413FFD mov edx, esi ; ListEntry
.text:00413FFF mov ecx, edi ; ListHead
.text:00414001 call @InterlockedPushEntrySList@8
```

## 4. 传递 IRP

`IoCallDriver` 函数可以发送 IRP 对象到一个指定对象相关的驱动程序。和上面的函数一样，该函数只是实现了函数转发，真正函数的实现是在 `IofCallDriver` 中实现的。

```nasm
.text:0045B800 ; int \_\_stdcall IoCallDriver(PDEVICE\_OBJECT DeviceObject, PIRP pIrp)
.text:0045B800 public \_IoCallDriver@8
.text:0045B800 \_IoCallDriver@8 proc near ; CODE XREF: IoCancelFileOpen(x,x)+B8↓p
.text:0045B800
.text:0045B800 DeviceObject = dword ptr 8
.text:0045B800 pIrp = dword ptr 0Ch
.text:0045B800
.text:0045B800 mov edi, edi
.text:0045B802 push ebp
.text:0045B803 mov ebp, esp
.text:0045B805 mov edx, [ebp+pIrp]
.text:0045B808 mov ecx, [ebp+DeviceObject]
.text:0045B80B call ds:\_pIofCallDriver
.text:0045B811 pop ebp
.text:0045B812 retn 8
.text:0045B812 \_IoCallDriver@8 endp
```

`IofCallDriver` 函数则再次转发函数，将函数指向 `IopfCallDriver`。

```nasm
.text:0040A311 ; NTSTATUS \_\_fastcall IofCallDriver(PDEVICE\_OBJECT DeviceObject, PIRP Irp)
.text:0040A311 public @IofCallDriver@8
.text:0040A311 @IofCallDriver@8 proc near ; CODE XREF: CcSetValidData(x,x)+9D↓p
.text:0040A311 ; PopPresentIrp(x,x)+47↓p ...
.text:0040A311 jmp ds:\_pIofCallDriver
.text:0040A311 @IofCallDriver@8 endp
```

`IopfCallDriver` 首先获取下一设备栈，从中获取主请求码，也就是要执行的驱动对象例程的函数下标。然后从设备对象中获取驱动对象，再从驱动对象中获取执行例程，最后根据获取的下标执行函数。

```nasm
.text:0040A31C ; \_\_fastcall IopfCallDriver(x, x)
.text:0040A31C @IopfCallDriver@8 proc near ; CODE XREF: IovCallDriver(x,x)+1C↓p
.text:0040A31C ; IovCallDriver(x,x)+9B↓p ...
.text:0040A31C
.text:0040A31C ; FUNCTION CHUNK AT .text:004492CB SIZE 0000000E BYTES
.text:0040A31C
.text:0040A31C dec [edx+\_IRP.CurrentLocation]
.text:0040A31F mov al, [edx+\_IRP.CurrentLocation]
.text:0040A322 test al, al
.text:0040A324 jle loc\_4492CB
.text:0040A32A mov eax, dword ptr [edx+\_IRP.Tail.Overlay.\_\_\_u4] ; EAX指向当前设备栈
.text:0040A32D sub eax, 24h ; 指向下一个设备栈(设备栈大小为0x24)
.text:0040A330 push esi
.text:0040A331 mov dword ptr [edx+\_IRP.Tail.Overlay.\_\_\_u4], eax ; 为当前设备栈赋值
.text:0040A334 mov [eax+\_IO\_STACK\_LOCATION.DeviceObject], ecx
.text:0040A337 movzx eax, [eax+\_IO\_STACK\_LOCATION.MajorFunction]
.text:0040A33A mov esi, [ecx+\_DEVICE\_OBJECT.DriverObject] ; 获取驱动对象
.text:0040A33D push edx ; 压入pIrp
.text:0040A33E push ecx ; 压入设备对象
.text:0040A33F call [esi+eax\*4+\_DRIVER\_OBJECT.MajorFunction]
.text:0040A343 pop esi
.text:0040A344 retn
.text:0040A344 @IopfCallDriver@8 endp
```

# 五、参考资料

> - 《Windows内核原理与实现》
> - 《Windows内核设计思想》

# 一、Windows I/O 系统

下图显示了 Windows 操作系统的 I/O 系统结构。Windows 的 I/O 系统由 5 个部分组成：

- I/O 管理器
