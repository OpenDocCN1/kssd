
```
.text:100013C0 ; int __cdecl MainThread(HANDLE hThread_Sleep, int buffer, int buflen)
.text:100013C0 MainThread      proc near                 ; CODE XREF: MainProc+115p
.text:100013C0
.text:100013C0 Pass_Content    = byte ptr -80h
.text:100013C0 hThread_Sleep   = dword ptr  8
.text:100013C0 buffer          = dword ptr  0Ch
.text:100013C0 buflen          = dword ptr  10h
.text:100013C0
.text:100013C0                 push    ebp           ; 此函数处理 21 号端口接收到的信息
.text:100013C1                 mov     ebp, esp
.text:100013C3                 sub     esp, 80h
.text:100013C9                 push    ebx
.text:100013CA                 push    esi
.text:100013CB                 mov     esi, [ebp+buffer]
.text:100013CE                 xor     eax, eax        ; eax=0
.text:100013D0                 mov     al, [esi]       ; al=接受的信息的第一个 byte
.text:100013D2                 mov     ecx, eax        ; ecx=接受的信息的第一个 byte
.text:100013D4                 and     ecx, 1111b      ; ecx=buffer 第一个 byte 的低 4 位
.text:100013D7                 cmp     byte ptr [esi+9], 6 ; 判断接收信息的第十个 byte 是否为 6
.text:100013DB                 lea     ebx, [esi+ecx*4] ; ebx = STRUCT2
.text:100013DE                 jnz     loc_10001561
.text:100013E4                 and     al, 11110000b
.text:100013E6                 cmp     al, 1000000b
.text:100013E8                 jnz     loc_10001561
.text:100013EE                 movzx   eax, byte ptr [ebx+0Ch] ; eax = buffer 中的(buffer 第一个 byte 的低 4 位 * 4 + 0x0C)位
.text:100013F2                 shr     eax, 4          ; eax = eax / 16
.text:100013F5                 add     eax, ecx        ; eax = eax + 接收信息中第一个 byte 的低 4 位
.text:100013F7                 push    edi
.text:100013F8                 mov     edi, [ebp+buflen] ; edi = 接收信息长度
.text:100013FB                 shl     eax, 2          ; eax = eax * 4  eax = 数据包头大小
.text:100013FE                 sub     edi, eax        ; edi = edi - eax
.text:10001400                 cmp     edi, 5          ; edi = 数据包 - 数据包头后剩余的信息长度
.text:10001403                 lea     ecx, [eax+esi]
.text:10001406                 mov     [ebp+buffer], ecx ; 在第一个 DWORD 中保存实体数据的指针
.text:10001409                 jbe     INFO_less_equal_5
.text:1000140F                 cmp     edi, 1500       ; 如果实体数据长度不小于 1500
.text:10001415                 jnb     INFO_less_equal_5
.text:1000141B                 push    5               ; 如果 5<实体数据长度<1500 则来到这里
.text:1000141D                 push    offset aUser    ; "USER "
.text:10001422                 push    ecx             ; buffer + eax
.text:10001423                 call    VerifyInfo
.text:10001428                 add     esp, 0Ch
.text:1000142B                 test    eax, eax
.text:1000142D                 push    5
.text:1000142F                 jz      short loc_10001492
.text:10001431                 pop     ecx             ; ecx = 5
.text:10001432                 cmp     edi, ecx
.text:10001434                 jbe     short loc_10001444
.text:10001436
.text:10001436 loc_10001436:                     ; CODE XREF: MainThread+82j
.text:10001436                 mov     eax, [ebp+buffer]
.text:10001439                 cmp     byte ptr [ecx+eax], ' '
.text:1000143D                 jnz     short loc_10001444
.text:1000143F                 inc     ecx
.text:10001440                 cmp     ecx, edi
.text:10001442                 jb      short loc_10001436
.text:10001444
.text:10001444 loc_10001444:                     ; CODE XREF: MainThread+74j
.text:10001444                         ; MainThread+7Dj
.text:10001444                 xor     edx, edx
.text:10001446                 jmp     short loc_10001465
.text:10001448 ; ---------------------------------------------------------------------------
.text:10001448
.text:10001448 loc_10001448:                     ; CODE XREF: MainThread+A7j
.text:10001448                 cmp     edx, 127
.text:1000144B                 jge     short loc_10001469
.text:1000144D                 mov     eax, [ebp+buffer]
.text:10001450                 mov     al, [ecx+eax]
.text:10001453                 cmp     al, 13          ; 不是回车
.text:10001455                 jz      short loc_10001469
.text:10001457                 cmp     al, 10          ; 不是换行
.text:10001459                 jz      short loc_10001469
.text:1000145B                 test    al, al          ; 空格则直接跳转
.text:1000145D                 jz      short loc_10001469
.text:1000145F                 mov     [ebp+edx+Pass_Content], al ; 填充内容
.text:10001463                 inc     ecx
.text:10001464                 inc     edx
.text:10001465
.text:10001465 loc_10001465:                     ; CODE XREF: MainThread+86j
.text:10001465                 cmp     ecx, edi
.text:10001467                 jb      short loc_10001448
.text:10001469
.text:10001469 loc_10001469:                     ; CODE XREF: MainThread+8Bj
.text:10001469                         ; MainThread+95j ...
.text:10001469                 lea     eax, [ebp+Pass_Content]
.text:1000146C                 push    eax             ; lpString2
.text:1000146D                 xor     eax, eax
.text:1000146F                 mov     ax, [ebx]       ; ax=STRUCT2.w1
.text:10001472                 mov     [ebp+edx+Pass_Content], 0 ; 字符串后补 0
.text:10001477                 push    eax             ; STRUCT2.w1
.text:10001478                 xor     eax, eax
.text:1000147A                 mov     ax, [ebx+2]     ; ax=STRUCT2.w2
.text:1000147E                 push    eax             ; STRUCT2.w2
.text:1000147F                 push    dword ptr [esi+0Ch] ; STRUCT1.dw1
.text:10001482                 push    dword ptr [esi+10h] ; STRUCT1.dw2
.text:10001485                 call    SaveToStruct    ; 保存数据到全局链表_APP_DATA 的最后一个节点中
.text:10001485                         ; 并从链表中剔除此节点，然后用 OFFSET_3160 结构记录此节点
.text:1000148A                 add     esp, 14h
.text:1000148D                 jmp     loc_10001560
.text:10001492 ; ---------------------------------------------------------------------------
.text:10001492
.text:10001492 loc_10001492:                     ; CODE XREF: MainThread+6Fj
.text:10001492                 push    offset aPass    ; "PASS "
.text:10001497                 push    [ebp+buffer]
.text:1000149A                 call    VerifyInfo
.text:1000149F                 add     esp, 0Ch
.text:100014A2                 test    eax, eax
.text:100014A4                 jz      loc_10001560
.text:100014AA                 push    5
.text:100014AC                 pop     ecx
.text:100014AD                 cmp     edi, ecx
.text:100014AF                 jbe     short loc_100014BF
.text:100014B1
.text:100014B1 loc_100014B1:                     ; CODE XREF: MainThread+FDj
.text:100014B1                 mov     eax, [ebp+buffer]
.text:100014B4                 cmp     byte ptr [ecx+eax], ' '
.text:100014B8                 jnz     short loc_100014BF
.text:100014BA                 inc     ecx
.text:100014BB                 cmp     ecx, edi
.text:100014BD                 jb      short loc_100014B1
.text:100014BF
.text:100014BF loc_100014BF:                     ; CODE XREF: MainThread+EFj
.text:100014BF                         ; MainThread+F8j
.text:100014BF                 xor     edx, edx
.text:100014C1                 jmp     short loc_100014E0
.text:100014C3 ; ---------------------------------------------------------------------------
.text:100014C3
.text:100014C3 loc_100014C3:                     ; CODE XREF: MainThread+122j
.text:100014C3                 cmp     edx, 127        ; 限制长度为 127
.text:100014C6                 jge     short loc_100014E4
.text:100014C8                 mov     eax, [ebp+buffer]
.text:100014CB                 mov     al, [ecx+eax]   ; 取密码
.text:100014CE                 cmp     al, 13          ; 不是回车
.text:100014D0                 jz      short loc_100014E4
.text:100014D2                 cmp     al, 10          ; 不是换行
.text:100014D4                 jz      short loc_100014E4
.text:100014D6                 test    al, al          ; 空格，则直接跳转
.text:100014D8                 jz      short loc_100014E4
.text:100014DA                 mov     [ebp+edx+Pass_Content], al ; 填充内容
.text:100014DE                 inc     ecx
.text:100014DF                 inc     edx
.text:100014E0
.text:100014E0 loc_100014E0:                     ; CODE XREF: MainThread+101j
.text:100014E0                 cmp     ecx, edi
.text:100014E2                 jb      short loc_100014C3
.text:100014E4
.text:100014E4 loc_100014E4:                     ; CODE XREF: MainThread+106j
.text:100014E4                         ; MainThread+110j ...
.text:100014E4                 xor     eax, eax
.text:100014E6                 mov     ax, [ebx]       ; eax=STRUCT2.w1
.text:100014E9                 mov     [ebp+edx+Pass_Content], 0 ; 末尾补 0
.text:100014EE                 push    eax
.text:100014EF                 xor     eax, eax
.text:100014F1                 mov     ax, [ebx+2]     ; eax=STRUCT2.w2
.text:100014F5                 push    eax
.text:100014F6                 push    dword ptr [esi+0Ch] ; STRUCT1.dw1
.text:100014F9                 push    dword ptr [esi+10h] ; STRUCT1.dw2
.text:100014FC                 call    VerifyData      ; 如果输入的是密码，则进行指令匹配
.text:10001501                 mov     edi, eax        ; 返回对应的节点指针
.text:10001503                 add     esp, 10h
.text:10001506                 test    edi, edi
.text:10001508                 jz      short loc_10001560
.text:1000150A                 lea     eax, [ebp+Pass_Content]
.text:1000150D                 push    eax             ; 输入的 PASS 内容
.text:1000150E                 lea     eax, [edi+18h]
.text:10001511                 push    eax             ; 输入的 USER 内容
.text:10001512                 xor     eax, eax
.text:10001514                 mov     ax, [ebx+2]
.text:10001518                 push    eax             ; STRUCT2.w2
.text:10001519                 push    dword ptr [esi+10h] ; STRUCT1.dw2
.text:1000151C                 push    [ebp+hThread_Sleep] ; hThread_Sleep
.text:1000151F                 call    PostUserPass
.text:10001524                 push    edi
.text:10001525                 call    MoveNodeToLast  ; 移动节点到链表最后
.text:1000152A                 add     esp, 18h
.text:1000152D                 jmp     short loc_10001560
.text:1000152F ; ---------------------------------------------------------------------------
.text:1000152F
.text:1000152F INFO_less_equal_5:                 ; CODE XREF: MainThread+49j
.text:1000152F                         ; MainThread+55j
.text:1000152F                 mov     al, [ebx+13]
.text:10001532                 test    al, 4           ; 该位不能是 4
.text:10001534                 jz      short loc_10001560
.text:10001536                 test    al, 1           ; 该位不能是 1
.text:10001538                 jz      short loc_10001560
.text:1000153A                 xor     eax, eax
.text:1000153C                 mov     ax, [ebx]
.text:1000153F                 push    eax
.text:10001540                 xor     eax, eax
.text:10001542                 mov     ax, [ebx+2]
.text:10001546                 push    eax
.text:10001547                 push    dword ptr [esi+0Ch]
.text:1000154A                 push    dword ptr [esi+10h]
.text:1000154D                 call    VerifyData      ; 验证指令
.text:10001552                 add     esp, 10h
.text:10001555                 test    eax, eax
.text:10001557                 jz      short loc_10001560
.text:10001559                 push    eax
.text:1000155A                 call    MoveNodeToLast
.text:1000155F                 pop     ecx
.text:10001560
.text:10001560 loc_10001560:                     ; CODE XREF: MainThread+CDj
.text:10001560                         ; MainThread+E4j ...
.text:10001560                 pop     edi
.text:10001561
.text:10001561 loc_10001561:                     ; CODE XREF: MainThread+1Ej
.text:10001561                         ; MainThread+28j
.text:10001561                 pop     esi
.text:10001562                 pop     ebx
.text:10001563                 leave
.text:10001564                 retn
.text:10001564 MainThread      endp
```
