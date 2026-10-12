
```
.text:005569DB                 push    ecx  
.text:005569DC                 neg     edx               ; 寄存器反向互补  
.text:005569DE                 cmp     [ebp-0FCh], ax  ; 比较二个运算单元  
.text:005569E5                 setz    al                ; 计数器置 0 (ZF=1)  
.text:005569E8                 neg     eax               ; 寄存器反向互补  
.text:005569EA                 and     edx, eax          ; 与运算  
.text:005569EC                 mov     edi, edx  
.text:005569EE                 lea     edx, [ebp-30h]    ; 载入有效位址  
.text:005569F1                 push    edx  
.text:005569F2                 push    2  
.text:005569F4                 call    ds:__vbaFreeObjList ; Indirect Call Near Procedure  
.text:005569FA                 add     esp, 0Ch          ; 加法运算  
.text:005569FD                 test    di, di            ; 两操作数作与运算,逻辑比较  
.text:00556A00                 jz      short loc_556A16 ; 等于 0 则跳 (ZF=1)  
.text:00556A02                 mov     word ptr [esi+44h], 2  
.text:00556A08                 mov     dword ptr [ebp-1Ch], 80000000h  
.text:00556A0F                 mov     dword ptr [ebp-18h], 41753157h  
.text:00556A16     loc_556A16:                            ; CODE XREF: .text:0055688Aj  
.text:00556A16                            ; .text:00556950j ...  
.text:00556A16                 mov     edx, offset aC  ; "c:\\"    ★取 C 盘 ID 作为机器码运算特征码★  
.text:00556A1B                 lea     ecx, [ebp-20h]    ; 载入有效位址  
.text:00556A1E                 call    ds:__vbaStrCopy ; Indirect Call Near Procedure  
.text:00556A24                 lea     eax, [ebp-20h]    ; 载入有效位址  
.text:00556A27                 push    eax  
.text:00556A28                 call    sub_488540        ; ★机器码计算 CALL★有兴趣的朋友可以跟进看看！  
.text:00556A2D                 fabs                ; 取绝对值  
.text:00556A2F                 fadd    ds:dbl_401410     ; 准备加法运算  
.text:00556A35                 lea     ecx, [ebp-48h]    ; 载入有效位址  
.text:00556A38                 lea     edx, [ebp-58h]    ; 载入有效位址  
.text:00556A3B                 push    ecx  
.text:00556A3C                 push    edx  
.text:00556A3D                 fadd    qword ptr [ebp-1Ch] ; 准备加法运算  
.text:00556A40                 mov     dword ptr [ebp-48h], 5  
.text:00556A47                 fstp    qword ptr [ebp-40h] ; 准备储存和出栈  
.text:00556A4A                 fnstsw  ax                ; 储存写入状态 (no wait)  
.text:00556A4C                 test    al, 0Dh           ; 两操作数作与运算,逻辑比较  
.text:00556A4E                 jnz     loc_556F16        ; 不等于 0 时则跳 (ZF=0)  
.text:00556A54                 call    ds:rtcVarStrFromVar ; Indirect Call Near Procedure  
.text:00556A5A                 lea     eax, [ebp-58h]    ; 载入有效位址  
.text:00556A5D                 lea     ecx, [ebp-68h]    ; 载入有效位址  
.text:00556A60                 push    eax  
.text:00556A61                 push    ecx  
.text:00556A62                 call    ds:rtcTrimVar     ; Indirect Call Near Procedure  
.text:00556A68                 lea     edx, [ebp-68h]    ; 载入有效位址  
.text:00556A6B                 mov     dword ptr [ebp-0FCh], 0  
.text:00556A75                 push    edx  
.text:00556A76                 call    ds:__vbaStrVarMove ; Indirect Call Near Procedure  
.text:00556A7C                 mov     ebx, ds:__vbaStrMove  
.text:00556A82                 mov     edx, eax  
.text:00556A84                 lea     ecx, [ebp-24h]    ; 载入有效位址  
.text:00556A87                 call    ebx ; __vbaStrMove ; Indirect Call Near Procedure  
.text:00556A89                 lea     eax, [ebp-0FCh]   ; 载入有效位址  
.text:00556A8F                 lea     ecx, [ebp-24h]    ; 载入有效位址  
.text:00556A92                 push    eax  
.text:00556A93                 push    ecx  
.text:00556A94                 lea     edi, [esi+40h]    ; 载入有效位址  
.text:00556A97                 call    sub_488E70        ; ★注册码计算 CALL★跟进！  
.text:00556A9C                 mov     edx, eax          ; ★真码出现★(此处可以做内存注册机)  
.text:00556A9E                 lea     ecx, [ebp-28h]    ; 载入有效位址  
.text:00556AA1                 call    ebx ; __vbaStrMove ; Indirect Call Near Procedure  
.text:00556AA3                 mov     edx, eax  
.text:00556AA5                 mov     ecx, edi  
.text:00556AA7                 call    ds:__vbaStrCopy ; Indirect Call Near Procedure  
.text:00556AAD                 lea     edx, [ebp-28h]    ; 载入有效位址  
.text:00556AB0                 lea     eax, [ebp-24h]    ; 载入有效位址  
.text:00556AB3                 push    edx  
.text:00556AB4                 lea     ecx, [ebp-20h]    ; 载入有效位址  
.text:00556AB7                 push    eax  
.text:00556AB8                 push    ecx  
.text:00556AB9                 push    3  
.text:00556ABB                 call    ds:__vbaFreeStrList ; Indirect Call Near Procedure  
.text:00556AC1                 mov     ebx, ds:__vbaFreeVarList  
.text:00556AC7                 lea     edx, [ebp-68h]    ; 载入有效位址  
.text:00556ACA                 lea     eax, [ebp-58h]    ; 载入有效位址  
.text:00556ACD                 push    edx  
.text:00556ACE                 lea     ecx, [ebp-48h]    ; 载入有效位址  
.text:00556AD1                 push    eax  
.text:00556AD2                 push    ecx  
.text:00556AD3                 push    3  
.text:00556AD5                 call    ebx ; __vbaFreeVarList ; Indirect Call Near Procedure  
.text:00556AD7                 mov     edx, [esi+3Ch]  
.text:00556ADA                 mov     eax, [edi]  
.text:00556ADC                 add     esp, 20h          ; 加法运算  
.text:00556ADF                 push    edx  
.text:00556AE0                 push    eax  
.text:00556AE1                 call    ds:__vbaStrCmp    ; Indirect Call Near Procedure  
.text:00556AE7                 test    eax, eax          ; 两操作数作与运算,逻辑比较  
.text:00556AE9                 jnz     loc_556D99        ; 不等于 0 时则跳 (ZF=0)  
.text:00556AEF                 movsx   eax, word ptr [esi+44h] ; 先符号扩展,再传送  
.text:00556AF3                 dec     eax               ; 运算前自减 1  
.text:00556AF4                 jz      loc_556CAF        ; 等于 0 则跳 (ZF=1)  
.text:00556AFA                 dec     eax               ; 运算前自减 1  
.text:00556AFB                 jz      loc_556BDA        ; 等于 0 则跳 (ZF=1)  
.text:00556B01                 dec     eax               ; 运算前自减 1  
.text:00556B02                 jnz     loc_556E85        ; 不等于 0 时则跳 (ZF=0)  
.text:00556B08                 or      eax, 0FFFFFFFFh ; 或运算  
.text:00556B0B                 push    esi  
.text:00556B0C                 mov     word_58006E, ax  
.text:00556B12                 mov     word_58006C, ax  
.text:00556B18                 mov     word_580070, ax  
.text:00556B1E                 mov     ecx, [esi]  
.text:00556B20                 call    dword ptr [ecx+6F8h] ; Indirect Call Near Procedure  
.text:00556B26                 mov     edi, ds:__vbaVarDup  
.text:00556B2C                 mov     ecx, 80020004h  
.text:00556B31                 mov     [ebp-70h], ecx  
.text:00556B34                 mov     eax, 0Ah  
```
