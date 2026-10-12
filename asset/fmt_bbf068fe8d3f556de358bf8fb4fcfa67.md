
```asm
mov     eax, large fs:20h ; 取得_KPRCB 地址
                                ; 这里的这些操作和 ObCreateObect 相对应
mov     ecx, [eax+5F8h] ; 取得_KPRCB 的 PPLookasideList[3]中指向后一个节点的值;_PP_LOOKASIDE_LIST
mov     edx, [esi+OBJECT_HEADER.anonymous_1.ObjectCreateInfo]
mov     di, [ecx+GENERAL_LOOKASIDE.ListHead.anonymous_0.Depth]
inc     [ecx+GENERAL_LOOKASIDE.TotalFrees]
cmp     di, [ecx+GENERAL_LOOKASIDE.Depth]
jb      short loc_525E89
inc     [ecx+_PAGED_LOOKASIDE_LIST.L.anonymous_1.FreeHits]
mov     ecx, [eax+5FCh]
mov     ax, [ecx+GENERAL_LOOKASIDE.ListHead.anonymous_0.Depth]
inc     [ecx+GENERAL_LOOKASIDE.TotalFrees]
cmp     ax, [ecx+GENERAL_LOOKASIDE.Depth]
jb      short loc_525E89
inc     [ecx+_PAGED_LOOKASIDE_LIST.L.anonymous_1.FreeHits]
push    edx                 ; PVOID
call    [ecx+_PAGED_LOOKASIDE_LIST.L.Free]
jmp     short loc_525EC7
loc_525E89:
call    @InterlockedPushEntrySList@8 
jmp     short loc_525EC7
NotNewObject:
mov     edi, [esi+OBJECT_HEADER.anonymous_1.QuotaBlockCharged]
```

这上面的代码是不是很眼熟呢，和我们申请空间中的一一对应！怎么样申请的，就怎么样释放。

这下面都是在为 `PsReturnSharedPoolQuota` 准备参数的一些判断。

代码：

```asm
test    edi, edi        ;判断一下 QuotaBlockCharged 是否有值，
jz      short loc_525ECB 
test    eax, eax 
jz      short loc_525EA5
mov     ecx, [eax+_OBJECT_HEADER_QUOTA_INFO.SecurityDescriptorCharge]
add     ecx, [eax+_OBJECT_HEADER_QUOTA_INFO.PagedPoolCharge] 
mov     eax, [eax+_OBJECT_HEADER_QUOTA_INFO.NonPagedPoolCharge]
jmp     short loc_525EBF
loc_525EA5:
test    dl, 20h    ; dl 是 flags;OB_FLAG_DEFAULT_SECURITY_QUOTA
mov     ecx, [ecx+OBJECT_TYPE.TypeInfo.DefaultPagedPoolCharge] ; ecx 指向 ObjectType
jz      short loc_525EB6
add     ecx, 800h
loc_525EB6:
mov     eax, [ebp+var_ObjectType]
mov     eax, [eax+OBJECT_TYPE.TypeInfo.DefaultNonPagedPoolCharge]
loc_525EBF:
push    eax                 ; int
push    ecx                 ; int
push    edi                 ; P
call    PsReturnSharedPoolQuota ;
```

这段代码将指定池类型的池配额返回给指定进程。

代码：

```asm
and     [esi+OBJECT_HEADER.anonymous_1.ObjectCreateInfo], 0
loc_525ECB: 
test    ebx, ebx        ; ebx 指向 OBJECT_HEADER_HANDLE_INFO
pop     edi
jz      short loc_525EE2
test    [esi+OBJECT_HEADER.Flags], 40h ; OB_FLAG_SINGLE_HANDLE_ENTRY
jnz     short loc_525EE2
push    0                ; Tag
push [ebx+_OBJECT_HEADER_HANDLE_INFO.anonymous_0.HandleCountDataBase]
call    ExFreePoolWithTag
```

这里将申请的堆空间释放。

代码：

```asm
And     [ebx+_OBJECT_HEADER_HANDLE_INFO.anonymous_0.HandleCountDataBase], 0
loc_525EE2: j
mov     eax, [ebp+var_NameInfo]
test    eax, eax
pop     ebx
jz      short loc_525F00
mov     eax, [eax+_OBJECT_HEADER_NAME_INFO.Name.Buffer]
```

有 `OBJECT_HEADER_NAME_INFO` 头的话再判断一个 `Buffer`，看下是否有值，有值的话代表申请了空间，要释放之。

代码：

```asm
test    eax, eax
jz      short loc_525F00
push    0                ; Tag
push    eax              ; P
call    ExFreePoolWithTag 
mov     eax, [ebp+var_NameInfo]
and     [eax+_OBJECT_HEADER_NAME_INFO.Name.Buffer], 0 
loc_525F00:
mov     eax, [ebp+var_ObjectType] ;  看下有没有 OBJECT_YTPE 结构。
test    eax, eax
mov     [esi+OBJECT_HEADER.Type], 0BAD0B0B0h
```

`ExFreePoolWithTag` 函数的第二个参数是一个 `Tag`，将上面的转成字符串，可以知道是“ TjbO”，这个我们在申请的时候已经见到过了。

代码：

```asm
pop     esi
jnz     short loc_525F16
mov     eax, 546A624Fh
jmp     short loc_525F1C
loc_525F16:
mov     eax, [eax+OBJECT_TYPE.Key]
```

如果 `Type` 不为空的话，将 `key` 的值给 `eax`，`eax` 将作为 `ExFreePoolWithTag` 的第二个参数，也就是 `Tag`。

代码：

```asm
or      eax, 80000000h
```

最后这二个参数都要与 `PROTECTED_POOL` 做一个或的运算。在申请的时候也做了这一步。

代码：

```asm
push    eax              ; Tag
push    [ebp+P]          ; P
call    ExFreePoolWithTag
```

我们知道`[ebp+P]`一直保持的是对象的起始地址，现在真正的把对象的空间给释放了。

### 平台环镜

`Windows XP SP2` + `VM6.5` + `WRK1.2` + `Windbg` + `IDA5.5`

### 参考文献

`[1]` 毛德操 著 《Windows 内核情景分析--采用开源代码 ReactOS》 `[M]`电子工业出版社 2009 年 5 月

`[2]` 张帆 史彩成 著《Windows 驱动开发技术详解》 `[M]`电子工业出版社  2008 年 7 月

还有几天就要毕业了，在此多谢钱方师，方老师，岳老师，赵老师，在这将近一年的时间里，对我们的教导，不仅仅是技术上，还有为人处事等等，同时还有多谢科锐的同学，是你们的陪伴，让我在这期间过得很充实，同时还有在专业学习上的交流以及指导。非常感谢。

### 上传的附件

| ![](img/668feaf58c8e7ddb6fbdbf839a5a19db.jpg) || `ObCreateObject.pdf`
|
