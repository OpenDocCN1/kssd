
```
.aspack:0040526A                 or      eax, eax
.aspack:0040526C                 jz      short loc_405278 ; 获得原输入表Rva
.aspack:0040526E                 add     eax, edx
.aspack:00405270                 mov     edi, eax
.aspack:00405272                 lodsw
.aspack:00405274                 stosw
.aspack:00405276                 jmp     short loc_405269
.aspack:00405278 ; ---------------------------------------------------------------------------
.aspack:00405278 loc_405278:                             ; CODE XREF: .aspack:00405265j
.aspack:00405278                                         ; .aspack:0040526Cj
.aspack:00405278                 mov     esi, 2014h      ; 获得原输入表Rva
.aspack:0040527D                 mov     edx, [ebp+422h] ; GetImageBase
.aspack:00405283                 add     esi, edx
.aspack:00405285 loc_405285:                             ; CODE XREF: .aspack:00405395j
.aspack:00405285                 mov     eax, [esi+0Ch]
.aspack:00405288                 test    eax, eax
.aspack:0040528A                 jz      loc_40539A
.aspack:00405290                 add     eax, edx        ; 获得Dll名
.aspack:00405292                 mov     ebx, eax
.aspack:00405294                 push    eax
.aspack:00405295                 call    dword ptr [ebp+0F4Dh] ; GetModuleHandle
.aspack:0040529B                 test    eax, eax        ; 判断要加载的模块是不是已经被加载
.aspack:0040529D                 jnz     short loc_4052A6
.aspack:0040529F                 push    ebx             ; 如果没有加载 就LoadLibrary
.aspack:004052A0                 call    dword ptr [ebp+0F51h]
.aspack:004052A6 loc_4052A6:                             ; CODE XREF: .aspack:0040529Dj
.aspack:004052A6                 mov     [ebp+545h], eax
.aspack:004052AC                 mov     dword ptr [ebp+549h], 0
.aspack:004052B6 loc_4052B6:                             ; CODE XREF: .aspack:0040537Fj
.aspack:004052B6                 mov     edx, [ebp+422h] ; 取出ImageBase
.aspack:004052BC                 mov     eax, [esi]      ; 取OriginalFirstThunk
.aspack:004052BE                 test    eax, eax
.aspack:004052C0                 jnz     short loc_4052C5
.aspack:004052C2                 mov     eax, [esi+10h]  ; 如果为0的话 那么就取FirstThunk
.aspack:004052C5 loc_4052C5:                             ; CODE XREF: .aspack:004052C0j
.aspack:004052C5                 add     eax, edx
.aspack:004052C7                 add     eax, [ebp+549h] ; 加上重定位数据
.aspack:004052CD                 mov     ebx, [eax]
.aspack:004052CF                 mov     edi, [esi+10h]
.aspack:004052D2                 add     edi, edx
.aspack:004052D4                 add     edi, [ebp+549h]
.aspack:004052DA                 test    ebx, ebx        ; 判断结束
.aspack:004052DC                 jz      loc_405384
.aspack:004052E2                 test    ebx, 80000000h  ; 判断是不是序号导入
.aspack:004052E8                 jnz     short loc_4052EE ; 保存ebx
.aspack:004052EA                 add     ebx, edx        ; 此为函数名调用
.aspack:004052EC                 inc     ebx
.aspack:004052ED                 inc     ebx             ; 跳过Hint 来到Name
.aspack:004052EE loc_4052EE:                             ; CODE XREF: .aspack:004052E8j
.aspack:004052EE                 push    ebx             ; 保存ebx
.aspack:004052EF                 and     ebx, 7FFFFFFFh  ; 去掉最高位
.aspack:004052F5                 push    ebx
.aspack:004052F6                 push    dword ptr [ebp+545h] ; 当前Dll的句柄
.aspack:004052FC                 call    dword ptr [ebp+0F49h] ; GetProcAddress
.aspack:00405302                 test    eax, eax
.aspack:00405304                 pop     ebx
.aspack:00405305                 jnz     short loc_405376 ; 成功就跳走
.aspack:00405307                 test    ebx, 80000000h  ; 再次判断是不是序号导入
.aspack:0040530D                 jnz     short loc_405328 ; 名字导入
.aspack:0040530F                 push    edi
.aspack:00405310                 mov     eax, [esi+0Ch]
.aspack:00405313                 add     eax, [ebp+422h]
.aspack:00405319                 push    eax
.aspack:0040531A                 push    ebx
.aspack:0040531B                 lea     eax, [ebp+475h]
.aspack:00405321                 push    eax
.aspack:00405322                 push    edi
.aspack:00405323                 jmp     loc_4053C0
.aspack:00405328 ; ---------------------------------------------------------------------------
.aspack:00405328 loc_405328:                             ; CODE XREF: .aspack:0040530Dj
.aspack:00405328                 and     ebx, 7FFFFFFFh  ; 名字导入
.aspack:0040532E                 mov     eax, [ebp+426h]
.aspack:00405334                 cmp     [ebp+545h], eax
.aspack:0040533A                 jnz     short loc_405360
.aspack:0040533C                 push    edi
.aspack:0040533D                 mov     edx, ebx
.aspack:0040533F                 dec     edx
.aspack:00405340                 shl     edx, 2
.aspack:00405343                 mov     ebx, [ebp+545h]
.aspack:00405349                 mov     edi, [ebx+3Ch]
.aspack:0040534C                 mov     edi, [ebx+edi+78h]
.aspack:00405350                 add     ebx, [ebx+edi+1Ch]
.aspack:00405354                 mov     eax, [ebx+edx]
.aspack:00405357                 add     eax, [ebp+545h]
.aspack:0040535D                 pop     edi
.aspack:0040535E                 jmp     short loc_405376 ; 填IAT
.aspack:00405360 ; ---------------------------------------------------------------------------
.aspack:00405360 loc_405360:                             ; CODE XREF: .aspack:0040533Aj
.aspack:00405360                 push    edi
.aspack:00405361                 mov     eax, [esi+0Ch]
.aspack:00405364                 add     eax, [ebp+422h]
.aspack:0040536A                 push    eax
.aspack:0040536B                 push    ebx
.aspack:0040536C                 lea     eax, [ebp+4C6h]
.aspack:00405372                 push    eax
.aspack:00405373                 push    edi
.aspack:00405374                 jmp     short loc_4053C0
.aspack:00405376 ; ---------------------------------------------------------------------------
.aspack:00405376 loc_405376:                             ; CODE XREF: .aspack:00405305j
.aspack:00405376                                         ; .aspack:0040535Ej
.aspack:00405376                 mov     [edi], eax      ; 填IAT
.aspack:00405378                 add     dword ptr [ebp+549h], 4 ; 循环填IAT
.aspack:0040537F                 jmp     loc_4052B6      ; 取出ImageBase
.aspack:00405384 ; ---------------------------------------------------------------------------
.aspack:00405384 loc_405384:                             ; CODE XREF: .aspack:004052DCj
.aspack:00405384                 mov     [esi], eax
.aspack:00405386                 mov     [esi+0Ch], eax
.aspack:00405389                 mov     [esi+10h], eax
.aspack:0040538C                 add     esi, 14h        ; 移到下一个IID地址
.aspack:0040538F                 mov     edx, [ebp+422h]
.aspack:00405395                 jmp     loc_405285
=========================================================

以下为动态修改OEP 并跳到 OEP处 此时 壳的工作已完毕

=========================================================

.aspack:0040539A loc_40539A:                             ; CODE XREF: start+2Ej
.aspack:0040539A                                         ; .aspack:0040528Aj
.aspack:0040539A                 mov     eax, 1000h
.aspack:0040539F                 push    eax
.aspack:004053A0                 add     eax, [ebp+422h]
.aspack:004053A6                 pop     ecx
.aspack:004053A7                 or      ecx, ecx
.aspack:004053A9                 mov     [ebp+3A8h], eax ; 将OEP写到Ep+0x3BB
.aspack:004053AF                 popa
.aspack:004053B0                 jnz     short loc_4053BA ;
```
