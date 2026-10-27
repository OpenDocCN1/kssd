
申请段（`SEGMENT`）对象的空间，将其控制区成员指向之前申请的控制区对象。并将PE相关的信息填入段对象对应的成员中。同时设置一些控制区对象的成员，将控制区的`FilePointer`成员指向传入参数`File`文件对象：

```
PAGE:004F8A86 mov [eax+_CONTROL_AREA.FilePointer], ecx
```

子节区的控制区对象成员执行申请到的控制区对象：

```
PAGE:004F8A89 mov [esi+_SUBSECTION.ControlArea], eax
```

段对象的`BasedAddress`成员设置为文件映像的基地址：

```
PAGE:004F8AB8 mov [ecx+_SEGMENT.BasedAddress], eax
```

设置子节区对象的各成员变量。

```
PAGE:004F8C65                 sub     [ebp+var_NumberOfPtes], eax
PAGE:004F8C68                 mov     eax, [ebp+var_SizeOfHeaders]
PAGE:004F8C6B                 mov     edx, [esi+_SUBSECTION.u.SubsectionFlags]
PAGE:004F8C6E                 shr     eax, 9
PAGE:004F8C71                 mov     [esi+_SUBSECTION.NumberOfFullSectors], eax
PAGE:004F8C74                 mov     eax, [ebp+var_SizeOfHeaders]
PAGE:004F8C77                 and     eax, 1FFh       ; MMSECTOR_MASK = 1ffh
PAGE:004F8C7C                 shl     eax, 14h
PAGE:004F8C7F                 and     edx, 0FFE0Eh
PAGE:004F8C85                 or      eax, edx
PAGE:004F8C87                 or      eax, 11h
PAGE:004F8C8A                 and     edi, 0FFFFFC3Fh
PAGE:004F8C90                 mov     [esi+_SUBSECTION.u.SubsectionFlags], eax
PAGE:004F8C93                 or      edi, 20h
PAGE:004F8C96                 xor     eax, eax
PAGE:004F8C98                 mov     [ecx+(_SEGMENT.SegmentPteTemplate)], edi
PAGE:004F8C9B                 cmp     [esi+_SUBSECTION.PtesInSubsection], eax
```

根据子节区中`PtesInSubsection`的值，循环设置每一个页表项的段对象参照控制区。

```
PAGE:004F8CA0 loop:     
PAGE:004F8CA0   mov     ecx, [ebp+var_ReturnStatus] ; var_ReturnStatus = 0
PAGE:004F8CA3   cmp     ecx, [ebp+var_SizeOfHeaders]
PAGE:004F8CA6   mov     edx, [ebp+arg_ppSegment] 
; arg_ppSegment = _SEGMENT.PrototypePte
PAGE:004F8CA9   sbb     ecx, ecx
PAGE:004F8CAB   add     [ebp+var_ReturnStatus], 1000h
PAGE:004F8CB2   add     [ebp+arg_ppSegment], 4 ; 指针加1
PAGE:004F8CB6   and     ecx, edi        ; edi = _SEGMENT.SegmentPteTemplate
PAGE:004F8CB8   inc     eax
PAGE:004F8CB9   mov     [edx+_SEGMENT.ControlArea], ecx
PAGE:004F8CBB   cmp     eax, [esi+_SUBSECTION.PtesInSubsection]
PAGE:004F8CBE   jb      short loop      ; var_ReturnStatus = 0
```

之后检测节区入口点地址是否为0，获得节区总数，根据每一个节区的信息设置段对象中的子节区对象，如子节区所占内存分页数、子节区所对应的控制区对象、子节区的下一个字节区对象（`NextSubsection`）。

```
PAGE:004F8F68                 mov     [esi+_SUBSECTION.NextSubsection], eax
PAGE:004F8F6B                 mov     esi, eax
PAGE:004F8F6D                 mov     eax, [ebp+var_pControlArea]
PAGE:004F8F70                 mov     [esi+_SUBSECTION.ControlArea], eax ; esi 指向下一个 _SUBSECTION
PAGE:004F8F72                 mov     [esi+_SUBSECTION.NextSubsection], ebx ; ebx = 0
PAGE:004F8F75                 mov     [esi+_SUBSECTION.UnusedPtes], ebx
PAGE:004F8F78                 mov     eax, [edx-8]    ; eax = VirtualAddress
PAGE:004F8F7B                 add     eax, [ebp+var_ImageBase]
PAGE:004F8F7E                 cmp     [ebp+var_NextVa], eax
PAGE:004F8F81                 jnz     error4
PAGE:004F8F87                 cmp     edi, ebx        ; SectionVirtualSize 是否为 0
PAGE:004F8F89                 jz      error4
PAGE:004F8F8F                 mov     eax, [ebp+var_ImageAlignment]
PAGE:004F8F92                 lea     eax, [edi+eax-1]
PAGE:004F8F96                 cmp     eax, edi        ; edi = SectionVirtualSize
PAGE:004F8F98                 jbe     error5
PAGE:004F8F9E                 shr     eax, 0Ch
PAGE:004F8FA1                 and     eax, [ebp+var_B0] ; var_B0 = fffff
PAGE:004F8FA7                 cmp     eax, [ebp+var_NumberOfPtes] ; 54 < 6a
PAGE:004F8FAA                 mov     [esi+_SUBSECTION.PtesInSubsection], eax
PAGE:004F8FAD                 ja      error6
PAGE:004F8FB3                 sub     [ebp+var_NumberOfPtes], eax
PAGE:004F8FB6                 mov     [esi+_SUBSECTION.u.LongFlags], ebx
PAGE:004F8FB9                 mov     ecx, [edx]      ; PointerToRawData
PAGE:004F8FBB                 mov     ebx, [ebp+var_FileAlignment] ; FileAlignment = fff
PAGE:004F8FC1                 shr     ecx, 9          ; 9 == MMSECTOR_SHIFT
PAGE:004F8FC4                 mov     [esi+_SUBSECTION.StartingSector], ecx
PAGE:004F8FC7                 mov     edi, [edx]      ; edi = PointerToRawData
PAGE:004F8FC9                 mov     eax, [edx-4]    ; SizeOfRawData
PAGE:004F8FCC                 add     eax, edi
PAGE:004F8FCE                 add     eax, [ebp+var_FileAlignment]
PAGE:004F8FD4                 not     ebx
PAGE:004F8FD6                 and     eax, ebx
PAGE:004F8FD8                 cmp     eax, edi        ; eax = EndingAddress 
PAGE:004F8FDA                 jb      Error_36
PAGE:004F8FE0                 mov     edi, eax
PAGE:004F8FE2                 and     eax, 1FFh       ; MMSECTOR_MASK = 1ffh
PAGE:004F8FE7                 shl     eax, 14h
PAGE:004F8FEA                 shr     edi, 9
PAGE:004F8FED                 mov     dword ptr [esi+_SUBSECTION.u.Flags], eax
PAGE:004F8FF0                 sub     edi, ecx
PAGE:004F8FF2                 mov     [ebp+arg_pfobFile], eax
PAGE:004F8FF5                 mov     eax, [ebp+arg_ppSegment]
PAGE:004F8FF8                 mov     [esi+_SUBSECTION.SubsectionBase], eax
PAGE:004F8FFB                 mov     [esi+_SUBSECTION.NumberOfFullSectors], edi
```

对应关系建立完毕后，最后调用`ExFreePoolWithTag`函数和`MiReleaseSystemPtes`函数进行资源的释放工作。

各主要对象之间的关系：

![](https://bbs.kanxue.com/upload/attach/201006/180677_68wjlssljfbt2k9.jpg)

参考资料：

| 参考资料 | 作者 |
| --- | --- |
| 加密与解密三 | 段钢（编著） |
| Windows 内核原理与实现 | 潘爱民 |
| Windows 内核情景分析 | 毛德操 |
| 深入解析Windows操作系统 | 潘爱民（译） |

一点心得体会：

内核中最重要的就是数据结构和数据关系，数据关系决定了数据结构的内容，所以分析内核首先要从数据结构和数据关系着手。

由于时间仓促，加之笔者水平有限，分析的只是一个梗概，还有很多相关的内容没有分析到，也恳请高手对文中的错漏给予指正和补充。

科锐5期学员 孙年忠

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)

上传的附件：

- [对象关系.jpg](javascript:void(0);)（274.83kb，815次下载）
- [流程图.jpg](javascript:void(0);)（94.97kb，878次下载）
