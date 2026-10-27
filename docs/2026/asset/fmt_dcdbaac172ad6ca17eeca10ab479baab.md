
# [原创]科锐三阶段项目---CreateProcess流程分析之二：ObCreateObject分析报告

这个是武汉科锐 第三阶段 的分析项目，每个同学负责不同的模块。在此我将我所分析的模块发出来。

声明：由于本人能力有限，文中必然有错漏之处，恳请读者不吝赐教。

因为这个函数比较简单，都是直线下来的，所以没有画流程图！

`ObCreateObject`它的作用是创建指定类型`OBJECT_TYPE`的对象示例。

堆是系统数据动态申请及释放的地方，WinNT把堆叫做`pool`。根据是否可以换出,分为`PagedPool`和`NonPagedPool`。

如果系统频繁的申请和释放内存,开销很大.于是WinNT利用`LookasideList`做缓冲,内存释放时并不是直接到`pool`,而是放到`LookasideList`中,等申请内存时,先检查表中是否有合适大小的快,若有就直接使用.系统定时检查表项的数目,以保持`pool`有足够的可用内存。

可以将`Lookaside`对象想像成一个内存容器。在初始的时候，它先向Windows申请了一块比较大的内存。以后程序员每次申请内存的时候，不是直接向Windows申请内存，而是向`Lookaside`对象申请内存。`Lookaside`对象会智能地避免产生内存“空洞”。如果`Lookaside`对象内部的内存不够用时，它会向操作系统申请更多的内存。当`Lookaside`对象内部有大量的末使用的内存时，它会自动让Windows回收一部分内存。总之，`Lookaside`是一个自动的内存分配容器，通过对`Lookaside`对象申请内存，效率要高于直接向Windows申请内存。

## 工作流程概述

看实现代码之前我先大概的描述一下这个函数的工作流程吧！

`ObCreateObject`内部通过调用 `ObpAllocateObject`为对象申请空间。在调用`ObpAllocateObject`调用之前`ObCreateObject`函数做一些相关信息的收集以及判断，`ObpAllocateObject`函数返回通过参数返回其起始地址`即&Header->Body`，同时函数里面还做一些对象本身的初始化工作，申请成功后，如果要求对象成为“永久对象”，即具有`OB_FLAG_PERMANENT_OBJECT`标志的话，再检查是否为系统的安全机制所允许，如果不允话的话，还要把刚刚申请的空间释放掉，同时返回一个错误代码`STATUS_PRIVILEGE_NOT_HELD`。

整个函数就是完成一个目标对象的创建工作。

## ObCreateObject函数参数

下面先来看`ObCreateObject`函数

这个函数有9个参数，但是有一个没有使用，所以IDA里面没有标识出来。参数如下:

```
ProbeMode       = byte ptr  8

ObjectType      = dword ptr  0Ch

ObjectAttributes= dword ptr  10h

ObjectHeader    = byte ptr  14h

ObjectBodySize  = dword ptr  1Ch

PagedPoolCharge = dword ptr  20h

NonPagedPoolCharge= dword ptr  24h

arg_Object          = dword ptr  28h
```

可能有些人在想，你是如何知道他有几个参数的，第一，看参数调用之前压了多少个参数，然后看里面都用了哪些，还有注意看下有没有通过寄存器传参，最后再看函数结束的堆栈平衡了多少个参数。

凡是创建对像的系统调用，都要提供至少二个输入参数。其中这定是`DesiredAccess`，说明所创建对象的访问模式，例如"只读"等。另一个是`ObjectAttributes`,这是一个`OBJECT_ATTRIBUTES`结构指针，`OBJECT_ATTRIBUTES`是一个很重要的数据结构。

```
0:000> dt _OBJECT_ATTRIBUTES

ntdll!_OBJECT_ATTRIBUTES

+0x000 Length           : Uint4B

+0x004 RootDirectory    : Ptr32 Void

+0x008 ObjectName       : Ptr32 _UNICODE_STRING

+0x00c Attributes       : Uint4B

+0x010 SecurityDescriptor : Ptr32 Void

+0x014 SecurityQualityOfService : Ptr32 Void
```

对象总是创建在某一目录中，这个目录必须事先已经打开，当前进程必须拥有这个已经打开目录的句柄。这里的`RootDirectory`就是这个目录节点的句柄。

另一个字段`Attributes`是一些表示各种属性的标志位，这些标志位连同这里别的字段最后都进入目标对像的`OBJECT_CREATE_INFORMATION`数据结构。

继续回到`ObCreateObject`函数上来。

```
mov    ebx, large fs:20h ; 取得_KPRCB地址

push    esi

mov    esi, [ebx+5F8h] ; 取得_KPRCB的PPLookasideList[3]中指向后一个节点的值;_PP_LOOKASIDE_LIST

inc	    [esi+GENERAL_LOOKASIDE.TotalAllocates] ; _GENERAL_LOOKASIDE.TotalAllocates

push   edi            ; 保存环镜，还不是传参

mov   ecx, esi        ; 通过寄存器ecx传参

call    @ExInterlockedPopEntrySList@8 ; 删除单向链表互锁操作

mov   edi, eax        ; eax的值为指向previously _GENERAL_LOOKASIDE

test    edi, edi

jnz     short FillObjectCreateInformation

inc     [esi+GENERAL_LOOKASIDE.anonymous_0.AllocateHits]

mov   esi, [ebx+5FCh] ; 取得_KPRCB的PPLookasideList[3]中指向前一个节点的值;_PP_LOOKASIDE_LIST

inc    [esi+GENERAL_LOOKASIDE.TotalAllocates]

mov   ecx, esi

call    @ExInterlockedPopEntrySList@8 ; 删除单向链表互锁操作

mov   edi, eax

test    edi, edi

jnz     short FillObjectCreateInformation

push    [esi+GENERAL_LOOKASIDE.Tag] ; ULONG

inc     dword ptr [esi+GENERAL_LOOKASIDE.anonymous_0]

push    [esi+GENERAL_LOOKASIDE.Size] ; SIZE_T

push    [esi+GENERAL_LOOKASIDE.Type] ; POOL_TYPE

call  [esi+GENERAL_LOOKASIDE.Allocate] ;通常放 ExAllocatePoolWithTag

                        ; 还有一个Free释放函数,通常放 ExFreePool

mov     edi, eax        ; 这里判断有没有申请空间成功

test    edi, edi

jnz     short FillObjectCreateInformation

mov     ebx, 0C000009Ah ; STATUS_INVALID_PARAMETER

                        ; 如果没有申请成功，跳到函数末

                        ; 尾部有个将ebx给eax的操作,ebx为返回值

jmp     ObCreateObjectEnd
```

由上的代码中我们可以发现，首先系统先在`LookasideList`为`_OBJECT_CREATE_INFORMATION`结构体申请空间，如果申请不成功，再去非分页池中申请。

上段反汇编识别来自`ObpAllocateObjectCreateInfoBuffer`,但是在编译之后已经内联进来了。

如果申请空间成功的话，这里再通过调用`ObpCaptureObjectCreateInformation`函数，做一个填充结构体的工作。

```
mov     esi, [ebp+ObjectType]

push    0

push    edi             ; ObjectCreateInfo

lea     eax, [ebp+CapturedObjectName]

push    eax

push    [ebp+ObjectAttributes]

push    dword ptr [ebp+ObjectHeader]

push    dword ptr [ebp+ProbeMode]

push    esi             ; ObjectType

call    ObpCaptureObjectCreateInformation
```

## ObpCaptureObjectCreateInformation

下面跟进去看下`ObpCaptureObjectCreateInformation`函数。

`ObpCaptureObjectCreateInformation`有7个参数，但是第一个忽略了，所以IDA中略过了。

```
ProbeMode       = byte ptr  0Ch

CreatorMode     = byte ptr  10h

ObjectAttributes= dword ptr  14h

CapturedObjectName= dword ptr  18h

ObjectCreateInfo= dword ptr  1Ch

arg_18          = dword ptr  20h
```

下面是`ObpCaptureObjectCreateInformation`函数内部:

```
xor     eax, eax

mov     ebx, [ebp+ObjectCreateInfo]

mov     edi, ebx

rep stosd               ; 将ObjectCreateInformation结构体清零
```

函数一开始先对`ObjectCreateInformation`结构体置为0。然后结构成员根据参数给值。

```
mov     eax, [ebp+ObjectAttributes]

cmp     eax, esi
```

判断`ObjectAttributes`是否为空，不为空填写`ObjectCreateInformaint`相关的项，同时里面还有一些检查，比如检查结构体是否可读，是否是内核模式`KernelMode`的操作,然后再给相就的成员赋值。

后面还有判断`ObjectName`参数是否为空，不为空的话，调用`ObpCaptureObjectName`函数给相关成员赋值，否则直接赋空值。

在`ObpCaptureObjectName`中，有一系列的判断，比如`Unicode`字符串的长度不是偶数倍数或者不是以零结束，最后再通过`ObpAllocateObjectNameBuffer`分配空间，申请成功再调用`RtlCopyMemory`将字符串拷备过来。

`ObpAllocateObjectNameBuffer`函数的第二个参数给申请空间一些建议，如果值为真，首先从`Lookaside`列表中申请，如果这里申请不成功，再去非分页中申请，如果这个参数为假，或者是申请的长度过大，则直接通过`ExAllocatePoolWithTag`函数去非分页中申请。

好了，继续回`ObCreateObject`函数上面来。

## 对象内存分配与检查

当上面的二个操作成功之后，就到了我们真正为对象分配空间的时候了。

首先前面是一些检查。

```
mov     eax, [esi+OBJECT_TYPE.TypeInfo.InvalidAttributes]

test    [edi+_OBJECT_CREATE_INFORMATION.Attributes], eax
```

`ObjecrCreateInfo`中的`Attributes`字段来自用户空间的对象属性块，是一些表示属性要求的标志位。但是这些属性要求对于具体的目标对象类型是否可以接受，为此，对象类型的数据结构中有个字段`InvalidAttributes`，这也是一些标志位，表示对本类型没有意义或不可接受的属性。这样，如果这两个位图有交集，就说明对象属性块中有非分之想，所以加以拒绝，否则就过不了这一关。

此外，还需要在`ObjectCreateInfo`中补充收集一些附加的信息，其实只是目标对像对于内存的消耗，包括`PagedPoolCharge`和`NonPagedPoolCharge`两项。前者是对于可换出页面的消耗，后者是对于不可换出页面的消耗。

```
mov     ecx, [ebp+PagedPoolCharge] ;属性没有冲突，就填写Charge

test    ecx, ecx        ; 判断是否为0

jnz     short FillNonPagedPoolCharge

mov     ecx, [esi+OBJECT_TYPE.TypeInfo.DefaultPagedPoolCharge] ; 

FillNonPagedPoolCharge:

mov     eax, [ebp+NonPagedPoolCharge]

test    eax, eax 

jnz     short AllocObjectCreateInformation

mov     eax, [esi+OBJECT_TYPE.TypeInfo.DefaultNonPagedPoolCharge]

AllocObjectCreateInformation:

mov     [edi+_OBJECT_CREATE_INFORMATION.NonPagedPoolCharge], eax
```

首先判断`PagedPoolCharge`的值是否为0，为0的话用默认值代替，再判断`NonPagedPoolCharge`是否为0,是的话也用默认值代替，最后再将值赋给`ObjectCreateInfo`相关的成员。

接着调用`ObpAllocateObjec`为对象真正的分配空间。这里我们先跳过这个函数将着往后面看。

调用完`ObpAllocateObjec`之后，判断一下是否申请对象成功。

```
call    ObpAllocateObject

mov     ebx, eax

test    ebx, ebx

jl      ReleaseObjectCreateInformation
```

如果对像申请不成功，把之前申请的结构全部释放掉。

目标对象的数据结构分配成功以后，通过参数`Object`返回其起始地址，即`&Header->Body`。最后如果要求目标对象成为“永久对象”，即`Header->Flags`中的标志位`OB_FLAG_PERMANENT_OBJECT`为1，则还要通过`SeSinglePrivilegeCheck()`检查是否为系统的安全机制所允许，如果不允许的话就要”退赔”，就是把刚分配的数据结构释放掉。并返回`STATUS_PRIVILEGE_NOT_HELD`错误代码。

```
mov     edi, dword ptr [ebp+ObjectHeader]

mov     esi, [ebp+ arg_Object]

lea     eax, [edi+_OBJECT_CREATE_INFORMATION.SecurityDescriptorCharge]

mov    [esi], eax ;将申请的空间地址给传出参数

test    byte ptr [edi+(_OBJECT_CREATE_INFORMATION+0Fh)], 10h ; ProbeMode是一个char的宽度

jz      short CheckPrivilege

push    dword ptr [ebp+ProbeMode] ;

push    ds:SeCreatePermanentPrivilege.HighPart

push    ds:SeCreatePermanentPrivilege.LowPart ; PrivilegeValue

call    SeSinglePrivilegeCheck ; 如果固定的对像已经创建，则检查是否为系统的安全机制所允许

test    al, al

jnz     short CheckPrivilege

mov     ecx, [esi]

call    ObpFreeObject

mov     ebx, 0C0000061h ; 060000061h == STATUS_PRIVILEGE_NOT_HELD
```

如果`SeSinglePrivilegeCheck()`成功之后，比较`ObpTraceEnabled`标志位，如果为真的话，调用`ObpRegisterObject`和`ObpPushStackInfo`。

```
cmp     ObpTraceEnabled, 0

jz      short ObCreateObjectEnd

test    ebx, ebx

jl      short ObCreateObjectEnd

push    edi             ; NewIrql

call    ObpRegisterObject ; 如果上面的那个条件都成功，执行下面的二个函数

push    1

push    edi

call    ObpPushStackInfo

jmp     short ObCreateObjectEnd
```

最后再返回。

整个函数的功能都比较简单。就是单纯的分配空间,然后就是目标对象本身的初始化，就完成目标对象的创建。

我们先来看看对象的内存结构:

![](https://bbs.kanxue.com/upload/attach/201006/25325_ut0q1reuoc9a72v.jpg)

好了，现在我们来看`ObpAllocateObject`函数的实现吧！
