
```
.text:005567E9                     fnclex                  ; 清除异常 (no wait)

.text:005567EB                     jge     short loc_5567FF ; 大于或等于则跳 (SF=OF)

.text:005567ED                     push    0E0h

.text:005567F2                     push    offset dword_437A44

.text:005567F7                     push    edi

.text:005567F8                     push    eax

.text:005567F9                     call    ds:__vbaHresultCheckObj ; Indirect Call Near Procedure

.text:005567FF     loc_5567FF:                             ; CODE XREF: .text:005567EBj
.text:005567FF                     mov     ecx, [esi]

.text:00556801                     push    esi

.text:00556802                     call    dword ptr [ecx+300h] ; Indirect Call Near Procedure

.text:00556808                     lea     edx, [ebp-34h]  ; 载入有效位址

.text:0055680B                     push    eax

.text:0055680C                     push    edx

.text:0055680D                     call    ebx             ; Indirect Call Near Procedure

.text:0055680F                     mov     edi, eax

.text:00556811                     lea     ecx, [ebp-100h] ; 载入有效位址

.text:00556817                     push    ecx

.text:00556818                     push    edi

.text:00556819                     mov     eax, [edi]

.text:0055681B                     call    dword ptr [eax+0E0h] ; Indirect Call Near Procedure

.text:00556821                     test    eax, eax        ; 两操作数作与运算,逻辑比较

.text:00556823                     fnclex                  ; 清除异常 (no wait)

.text:00556825                     jge     short loc_556839 ; 大于或等于则跳 (SF=OF)

.text:00556827                     push    0E0h

.text:0055682C                     push    offset dword_437A44

.text:00556831                     push    edi

.text:00556832                     push    eax

.text:00556833                     call    ds:__vbaHresultCheckObj ; Indirect Call Near Procedure

.text:00556839     loc_556839:                             ; CODE XREF: .text:00556825j
.text:00556839                     mov     edi, 1

.text:0055683E                     xor     edx, edx        ; 异或清零

.text:00556840                     cmp     [ebp-100h], di  ; 比较二个运算单元

.text:00556847                     lea     ecx, [ebp-34h]  ; 载入有效位址

.text:0055684A                     push    ecx

.text:0055684B                     setz    dl              ; 计数器置0 (ZF=1)

.text:0055684E                     neg     edx             ; 寄存器反向互补

.text:00556850                     xor     eax, eax        ; 异或清零

.text:00556852                     cmp     [ebp-0FCh], di  ; 比较二个运算单元

.text:00556859                     setz    al              ; 计数器置0 (ZF=1)

.text:0055685C                     neg     eax             ; 寄存器反向互补

.text:0055685E                     and     edx, eax        ; 与运算

.text:00556860                     mov     edi, edx

.text:00556862                     lea     edx, [ebp-30h]  ; 载入有效位址

.text:00556865                     push    edx

.text:00556866                     push    2

.text:00556868                     call    ds:__vbaFreeObjList ; Indirect Call Near Procedure

.text:0055686E                     add     esp, 0Ch        ; 加法运算

.text:00556871                     test    di, di          ; 两操作数作与运算,逻辑比较

.text:00556874                     jz      short loc_55688F ; 等于0则跳 (ZF=1)

.text:00556876                     mov     word ptr [esi+44h], 3

.text:0055687C                     mov     dword ptr [ebp-1Ch], 40000000h

.text:00556883                     mov     dword ptr [ebp-18h], 417FCA03h

.text:0055688A                     jmp     loc_556A16      ; 跳

.text:0055688F     ; ---------------------------------------------------------------------------

.text:0055688F     loc_55688F:                             ; CODE XREF: .text:00556874j
.text:0055688F                     mov     eax, [esi]

.text:00556891                     push    esi

.text:00556892                     call    dword ptr [eax+304h] ; Indirect Call Near Procedure

.text:00556898                     lea     ecx, [ebp-30h]  ; 载入有效位址

.text:0055689B                     push    eax

.text:0055689C                     push    ecx

.text:0055689D                     call    ebx             ; Indirect Call Near Procedure

.text:0055689F                     mov     edi, eax

.text:005568A1                     lea     eax, [ebp-0FCh] ; 载入有效位址

.text:005568A7                     push    eax

.text:005568A8                     push    edi

.text:005568A9                     mov     edx, [edi]

.text:005568AB                     call    dword ptr [edx+0E0h] ; Indirect Call Near Procedure

.text:005568B1                     test    eax, eax        ; 两操作数作与运算,逻辑比较

.text:005568B3                     fnclex                  ; 清除异常 (no wait)

.text:005568B5                     jge     short loc_5568C9 ; 大于或等于则跳 (SF=OF)

.text:005568B7                     push    0E0h

.text:005568BC                     push    offset dword_437A44

.text:005568C1                     push    edi

.text:005568C2                     push    eax

.text:005568C3                     call    ds:__vbaHresultCheckObj ; Indirect Call Near Procedure

.text:005568C9     loc_5568C9:                             ; CODE XREF: .text:005568B5j
.text:005568C9                     mov     ecx, [esi]

.text:005568CB                     push    esi

.text:005568CC                     call    dword ptr [ecx+300h] ; Indirect Call Near Procedure

.text:005568D2                     lea     edx, [ebp-34h]  ; 载入有效位址

.text:005568D5                     push    eax

.text:005568D6                     push    edx

.text:005568D7                     call    ebx             ; Indirect Call Near Procedure

.text:005568D9                     mov     edi, eax

.text:005568DB                     lea     ecx, [ebp-100h] ; 载入有效位址

.text:005568E1                     push    ecx

.text:005568E2                     push    edi

.text:005568E3                     mov     eax, [edi]

.text:005568E5                     call    dword ptr [eax+0E0h] ; Indirect Call Near Procedure

.text:005568EB                     test    eax, eax        ; 两操作数作与运算,逻辑比较

.text:005568ED                     fnclex                  ; 清除异常 (no wait)

.text:005568EF                     jge     short loc_556903 ; 大于或等于则跳 (SF=OF)

.text:005568F1                     push    0E0h

.text:005568F6                     push    offset dword_437A44

.text:005568FB                     push    edi

.text:005568FC                     push    eax

.text:005568FD                     call    ds:__vbaHresultCheckObj ; Indirect Call Near Procedure

.text:00556903     loc_556903:                             ; CODE XREF: .text:005568EFj
.text:00556903                     xor     edx, edx        ; 异或清零

.text:00556905                     lea     ecx, [ebp-34h]  ; 载入有效位址

.text:00556908                     cmp     [ebp-100h], dx  ; 比较二个运算单元

.text:0055690F                     push    ecx

.text:00556910                     setz    dl              ; 计数器置0 (ZF=1)

.text:00556913                     neg     edx             ; 寄存器反向互补

.text:00556915                     xor     eax, eax        ; 异或清零

.text:00556917                     cmp     word ptr [ebp-0FCh], 1 ; 比较二个运算单元

.text:0055691F                     setz    al              ; 计数器置0 (ZF=1)

.text:00556922                     neg     eax             ; 寄存器反向互补

.text:00556924                     and     edx, eax        ; 与运算

.text:00556926                     mov     edi, edx
```
