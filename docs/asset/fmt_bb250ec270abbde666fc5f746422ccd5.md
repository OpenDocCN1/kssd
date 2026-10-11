
```
cmp     ObpTraceEnabled, 0
jz      short ObCreateObjectEnd
test    ebx, ebx
jl      short ObCreateObjectEnd
push    edi            ; NewIrql
call    ObpRegisterObject ; 如果上面的那个条件都成功，执行下面的二个函数
push    1
push    edi
call    ObpPushStackInfo
jmp     short ObCreateObjectEnd
```

最后再返回。

整个函数的功能都比较简单。就是单纯的分配空间,然后就是目标对象本身的初始化，就完成目标对象的创建。

我们先来看看对象的内存结构:

![](img/d0cd8987518aa500523822f30a997bc0.jpg)

好了，现在我们来看 `ObpAllocateObject` 函数的实现吧！

这个函数有 6 个参数，参数列表如下:

代码:

```
POBJECT_CREATE_INFORMATION ObjectCreateInfo,
KPROCESSOR_MODE OwnershipMode,
POBJECT_TYPE ObjectType OPTIONAL,
PUNICODE_STRING ObjectName,
ULONG ObjectBodySize,
POBJECT_HEADER *ReturnedObjectHeader
```

在函数体的实现中，有些参数只用了一次，后面被编译器优化用于局部变量。

下面我们来看函数的实现：代码:

```
mov     eax, [ebp+arg_ObjectCreateInfo]
push    ebx
push    esi
mov     esi, [ebp+arg_HandleInfoSize]  
xor     ecx, ecx
cmp     eax, ecx        ; 这里判断一下 ObjectCreateInfo 参数是否为空
                        ; 空的话使用默认值,空的话也说明是一个类型对像
                        ; 默认有 OBJECT_HEADER_NAME_INFO、OBJECT_HEADER_NAME_INFO 可选头部
push    edi
mov     edi, [ebp+CreatorInfoSize] ; 这里原来是 ObjectName 参数，用来编译器优化，用于它用了,后来用来做变量了
jnz     short cmpDefaultQuota 
push    10h
pop     ebx            ; CreatorInfoSize 等于 10
mov     [ebp+var_QuotaInfoSize], ecx
mov     [ebp+arg_HandleInfoSize], ecx
mov     [ebp+CreatorInfoSize], ebx
jmp     short ComputerHeaderSize:
```

这里判断 `ObjectCreateInfo` 是否为空，空的话也说明是一个类型对像，然后使用默认值填充，默认是有 `OBJECT_HEADER_NAME_INFO`、`OBJECT_HEADER_NAME_INFO` 可选头部。

如果不为空的话，我们继续往下看:

代码:

```
mov     edx, [eax+_OBJECT_CREATE_INFORMATION.PagedPoolCharge] 
cmp     edx, [esi+OBJECT_TYPE.TypeInfo.DefaultPagedPoolCharge]
jnz     short cmpPsInitialSystemProcess
mov     edx, [eax+_OBJECT_CREATE_INFORMATION.NonPagedPoolCharge]
cmp     edx, [esi+OBJECT_TYPE.TypeInfo.DefaultNonPagedPoolCharge] jnz     short cmpPsInitialSystemProcess
cmp     [eax+_OBJECT_CREATE_INFORMATION.SecurityDescriptorCharge], 800h ; SE_DEFAULT_SECURITY_QUOTA
jbe     short loc_525BD5 ; OBJ_EXCLUSIVE
                        ; 判断是否允许别的进程打开

cmpPsInitialSystemProcess: ; 
mov     edx, large fs:124h
mov     edx, [edx+38h]
cmp     edx, PsInitialSystemProcess ; 判断是不是系统初始化进程
jnz     short HaveQuotaInfo ; 给 QuotaInfo 留空间

loc_525BD5:
test    byte ptr [eax+_OBJECT_CREATE_INFORMATION.Attributes], 20h 
                         ; 判断是否允许别的进程打开
mov     [ebp+var_QuotaInfoSize], ecx   ; ecx 为零值
jz      short loc_525BE4
HaveQuotaInfo:
mov     [ebp+var_QuotaInfoSize], 10h ; 给 QuotaInfo 留空间
```

首先是判断 `ObjectCreateInfo`，`PagedPoolCharge` 是否是使用默认值，如果不是的话，就有必要为 `OBJECT_HEADER_QUOTA_INFO` 留空间，因为不是默认值的话，需要空间来保存相关的内存消耗信息，如果是使用认值的话再比较 `NonPagedPoolCharge` 与 `SecurityDescriptorCharge`，这三个比较都是与操作，只要一个为真，就成立，这个成立之后跳去 `cmpPsInitialSystemProcess`。

这个判断是不是系统初始化进程，是的话不成立，即不用为 `QuotaInfoSize` 留空间。

代码:

```
mov     al, [esi+OBJECT_TYPE.TypeInfo.MaintainHandleCount] ; 对象的类型决定了是否需要 OBJECT_HEADER_HANDLE_INFO 结构
mov     ebx, [edi+UNICODE_STRING.Buffer] 
neg     al
sbb     eax, eax
and     eax, 8
neg     ebx
mov     [ebp+arg_HandleInfoSize], eax
mov     al, [esi+OBJECT_TYPE.TypeInfo.MaintainTypeList]
sbb     ebx, ebx
and     ebx, 10h  
neg     al
sbb     eax, eax
and     eax, 10h
mov     [ebp+CreatorInfoSize], eax
```

这里是二个判断，判断是否需要 `OBJECT_HEADER_HANDLE_INFO` 和 `OBJECT_HEADER_CREATOR_INFO`、`OBJECT_HEADER_NAME_INFO`。

这些有一些很有意思的编译器优化。

代码:

```
mov     ebx, [edi+UNICODE_STRING.Buffer]
neg     ebx
sbb     ebx, ebx
and     ebx, 10h
```

首先取得值给 `ebx`,再对 `ebx` 做一个求补的运算，如果 `ebx` 为 0 的话，结果也为 0,再一个带符号的减法，如果 `ebx` 原来为零，值的话也一直为 0，如果原来有值的话，经过这步之后，寄存器的值变成了全 F,然后再和 10 做个与运算(因为结构体的大小是 10)，这样运算完的结果可以直接用于赋值，而不用跳转了，同时编译器还做了一些流水线的优化。

好了，我们继续回到代码上面来，现在到了计算头大小的时候了。

代码:

```
mov     eax, [ebp+CreatorInfoSize]
mov     ecx, [ebp+var_QuotaInfoSize]
add     eax, ebx        ; 这里开始对头的大小进么计算
                        ; ebx 为 NameInfoSize
add     eax, [ebp+arg_HandleInfoSize]
xor     edx, edx
cmp     esi, edx 
lea     ecx, [eax+ecx+18h]
```

这里将 `CreatorInfoSize`、`QuotaInfoSize`、`HandleInfoSize`、`NameInfoSize` 与 `0x18` 相加(`_OBJECT_HEADER` 除去 `BODY` 的 size)。这些做了一个小优化，通过 `lea` 来进行算术运算。

接着上面的代码，我们继续分析。

代码:

```
jz      short loc_525C26
cmp     [esi+OBJECT_TYPE.TypeInfo.PoolType], edx ; NonPagedPool
jz      short loc_525C26
inc     edx            ; edx 为分页类型
loc_525C26
test    esi, esi
mov     eax, 546A624Fh  ; 字符串 TjbO
jz      short loc_525C35
; (ObjectType==NULL?'TjbO':ObjectType->Key)|PROTECTED_POOL
mov     eax, [esi+OBJECT_TYPE.Key]
loc_525C35:
or      eax, 80000000h 
; (ObjectType==NULL?'TjbO':ObjectType->Key)|PROTECTED_POOL
push    eax            ; Tag
mov     eax, [ebp+arg_ObjectBodySize]
add     ecx, eax       ; HeaderSize + ObjectBodySize
push    ecx            ; NumberOfBytes
push    edx            ; PoolType
call    ExAllocatePoolWithTag
```

在之前的代码中已经判断 `ObjectType` 是否为空了，如果不为空再判断 `ObjectType->TypeInfo.PoolType` 是否等于 `NonPagedPool`，等于的话跳过那个 `inc`,因为 `NonPagedPool` 等于 0,`PagedPool` 等于 1，分页类型用于 `ExAllocatePoolWithTag` 的最后一个参数。因为 `ObjectType` 有可能是空，所以要采用上面的方法传参。

接着下面又再次判断 `ObjectType` 是否为空，其实我觉得这里可以优化一下的，因为刚刚才比较过的，如果为空的话把 Tag`"TjbO"`赋给 `eax`,否则将 `OBJECT_TYPE.Key` 值给 `eax`,最后再把 `eax` 和 `80000000h`(`PROTECTED_POOL`)做个与的运算。

然后先最后一个参数压进栈之后，再计算一个申请空间的大小，因为我们刚刚计算的头大小中，并没有算上 `Body` 的，所以这里他做了一个加法，最后将其它的二个参数压进栈，完成函数的调用。

继续往下看代码:

代码:

```
test    eax, eax        ; 这里判断有没有申请成功
jnz     short AllocObjectSuccess
mov     eax, 0C000009Ah ; STATUS_INSUFFICIENT_RESOURCES
jmp     FunctionEnd
```

这里判断有申请空间是否成功，如果不成功的话，返回一个 `STATUS_INSUFFICIENT_RESOURCES`。如果成功了就跳走。我们再看成功的代码。

成功的代码比较长，但是比较简单，都是一些相关结构体内成员的赋值。代码:

```
mov     ecx, [ebp+var_QuotaInfoSize]
test    ecx, ecx
jz      short cmpHandleInfoSize
mov     edx, [ebp+arg_ObjectCreateInfo]
mov     edx, [edx+_OBJECT_CREATE_INFORMATION.PagedPoolCharge]
mov     [eax+_OBJECT_HEADER_QUOTA_INFO.PagedPoolCharge], edx
mov     edx, [ebp+arg_ObjectCreateInfo]
mov     edx, [edx+_OBJECT_CREATE_INFORMATION.NonPagedPoolCharge]
mov     [eax+_OBJECT_HEADER_QUOTA_INFO.NonPagedPoolCharge], edx 
mov     edx, [ebp+arg_ObjectCreateInfo]
mov    edx, [edx+_OBJECT_CREATE_INFORMATION.SecurityDescriptorCharge]
and     dword ptr [eax+0Ch], 0 ; QuotaInfo->ExclusiveProcess
mov     [eax+_OBJECT_HEADER_QUOTA_INFO.SecurityDescriptorCharge], edx
add     eax, 10h
```

这段代码是判断有没有`_OBJECT_HEADER_QUOTA_INFO` 可选头，有的话，把内存消耗的信息复制到`_OBJECT_HEADER_QUOTA_INFO` 中去。

注意后面的 `add eax, 10h`。这有助于我们了解他的内存结构。

代码:

```
cmp     [ebp+arg_HandleInfoSize], 0
jz      short cmpNameInfoSize
and     [eax+_OBJECT_HEADER_HANDLE_INFO.anonymous_0.SingleEntry.HandleCount], 0 add     eax, 8
```

这里判断有没有 `OBJECT_HEADER_HANDLE_INFO` 可选头，有的话将数据结构中的 `HandleCount` 设置为 0。

这里后面是加 8 了，说明 `OBJECT_HEADER_HANDLE_INFO` 的大小为 8，排列在`_OBJECT_HEADER_QUOTA_INFO` 头部后面。

代码:

```
test    ebx, ebx  ; ebx 为 NameInfoSize,上面一直没有改过
jz      short cmpCreatorInfoSize
mov     edx, [edi]  ; edi 指向 ObjectName
mov     dword ptr [eax+_OBJECT_HEADER_NAME_INFO.Name.Length], ed 
mov     edx, [edi+_UNICODE_STRING.Buffer] 
and     [eax+_OBJECT_HEADER_NAME_INFO.Directory], 0 
cmp     [ebp+arg_OwnershipMode], 0 ;这里判断是不是内核模式
mov     edi, [ebp+arg_ObjectCreateInfo]
mov     [eax+_OBJECT_HEADER_NAME_INFO.Name.Buffer], edx 
mov     [eax+_OBJECT_HEADER_NAME_INFO.QueryReferences], 1
jnz     short loc_525CBD
test    edi, edi
jz      short loc_525CBD
test    byte ptr [edi+_UNICODE_STRING.MaximumLength], 1  
jz      short loc_525CBD
mov     [eax+_OBJECT_HEADER_NAME_INFO.QueryReferences], 40000001h

loc_525CBD:
add     eax, 10h
jmp     short cmp_CreatorInfoSize
```

这一段代码的作用就是将对象名复制到 `OBJECT_HEADER_NAME_INFO` 数据结构中中去,同时将 `NameInfo->Directory` 设为 `NULL`, `NameInfo->QueryReferences` 设为 1,然后再判断所属模式(`OwnershipMode`)是否是内核模式成功的话接着判断 `ObjectCreateInfo` 是否为空，不为空再判断属性(`Attributes`)是否具有采用内核句柄表属性。三个都成立的话，将 `QueryReferences` 成员和 `40000001h`(`OBP_NAME_KERNEL_PROTECTED`)做个或运算，最后 `eax` 再加 `10h`。
