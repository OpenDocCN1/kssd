
```
.text:0041E89E ; ----------------------------------------------------------------------------  
.text:0041E89E  
.text:0041E89E loc_41E89E:                              ; CODE XREF: sub_41E480+412j  
.text:0041E89E                cmp     al, 4  
.text:0041E8A0                jnz     short loc_41E8AC  
.text:0041E8A2                push    offset unk_5913A4  
.text:0041E8A7                jmp     loc_41F3B3  
.text:0041E8AC ; ----------------------------------------------------------------------------  
.text:0041E8AC  
.text:0041E8AC loc_41E8AC:                              ; CODE XREF: sub_41E480+420j  
.text:0041E8AC                cmp     al, 6  
.text:0041E8AE                jnz     short loc_41E8DC  
.text:0041E8B0                push    offset unk_59136C  
.text:0041E8B5                push    1  
.text:0041E8B7                mov     ecx, 59DDC8h  
.text:0041E8BC                call    sub_4177D0  
.text:0041E8C1                push    0EA60h          ; dwMilliseconds  
.text:0041E8C6                mov     dword ptr [ebp+144h], 9  
.text:0041E8D0                mov     byte ptr [esp+4DC8h+var_4DB4], 0  
.text:0041E8D5                call    ebx ; Sleep  
.text:0041E8D7                jmp     loc_41F3BF  
.text:0041E8DC ; ----------------------------------------------------------------------------  
.text:0041E8DC  
.text:0041E8DC loc_41E8DC:                              ; CODE XREF: sub_41E480+42Ej  
.text:0041E8DC                cmp     al, 1  
.text:0041E8DE                jz      short loc_41E8EA  
.text:0041E8E0                push    offset a4Z      ; "4 账号登陆失败!"  
.text:0041E8E5                jmp     loc_41F3B3  
.text:0041E8EA ; ----------------------------------------------------------------------------  
.text:0041E8EA  
.text:0041E8EA loc_41E8EA:                              ; CODE XREF: sub_41E480+45Ej  
.text:0041E8EA                mov     al, [ebp+142h]  
.text:0041E8F0                test    al, al  
.text:0041E8F2                jnz     loc_41EB1A  
.text:0041E8F8                mov     ecx, 10h  
.text:0041E8FD                xor     eax, eax  
.text:0041E8FF                lea     edi, [esp+4DC4h+var_4D88]  
.text:0041E903                push    eax             ; time_t *  
.text:0041E904                rep stosd               ; 外挂验证_ 登陆  
.text:0041E906                call    _time           ; 生成加密用的种子  
.text:0041E90B                push    eax             ; unsigned int  
.text:0041E90C                call    _srand  
.text:0041E911                call    _rand           ; 随机函数  
.text:0041E916                mov     esi, eax  
.text:0041E918                shl     esi, 10h  
.text:0041E91B                sub     esi, eax  
.text:0041E91D                call    _rand           ; 随机函数  
.text:0041E922                mov     edx, [ebp+0]  
.text:0041E925                add     esi, eax  
.text:0041E927                dec     edx  
.text:0041E928                mov     [esp+4DCCh+var_4D88], esi  
.text:0041E92C                push    edx  
.text:0041E92D                mov     [esp+4DD0h+var_4D80], 53h  
.text:0041E934                mov     [esp+4DD0h+var_4D7C], 1  
.text:0041E93B                call    sub_434D10  
.text:0041E940                mov     [esp+4DD0h+var_4D7A], ax  
.text:0041E945                lea     edi, [ebp+2Ch]  
.text:0041E948                or      ecx, 0FFFFFFFFh  
.text:0041E94B                xor     eax, eax  
.text:0041E94D                add     esp, 0Ch  
.text:0041E950                lea     edx, [esp+4DC4h+var_4D78]  
.text:0041E954                repne scasb  
.text:0041E956                not     ecx  
.text:0041E958                sub     edi, ecx  
.text:0041E95A                mov     [esp+4DC4h+var_4D7E], 64h  
.text:0041E961                mov     eax, ecx  
.text:0041E963                mov     esi, edi  
.text:0041E965                mov     edi, edx  
.text:0041E967                lea     edx, [esp+4DC4h+var_4D6C]  
.text:0041E96B                shr     ecx, 2  
.text:0041E96E                rep movsd  
.text:0041E970                mov     ecx, eax  
.text:0041E972                xor     eax, eax  
.text:0041E974                and     ecx, 3  
.text:0041E977                rep movsb  
.text:0041E979                mov     edi, offset pszPath  
.text:0041E97E                or      ecx, 0FFFFFFFFh  
.text:0041E981                repne scasb  
.text:0041E983                not     ecx  
.text:0041E985                sub     edi, ecx  
.text:0041E987                mov     eax, ecx  
.text:0041E989                mov     esi, edi  
.text:0041E98B                mov     edi, edx  
.text:0041E98D                lea     edx, [esp+4DC4h+var_4D60]  
.text:0041E991                shr     ecx, 2  
.text:0041E994                rep movsd  
.text:0041E996                mov     ecx, eax  
.text:0041E998                xor     eax, eax  
.text:0041E99A                and     ecx, 3  
.text:0041E99D                rep movsb  
.text:0041E99F                mov     edi, offset pszPath  
.text:0041E9A4                or      ecx, 0FFFFFFFFh  
.text:0041E9A7                repne scasb  
.text:0041E9A9                not     ecx  
.text:0041E9AB                sub     edi, ecx  
.text:0041E9AD                mov     eax, ecx  
.text:0041E9AF                mov     esi, edi  
.text:0041E9B1                mov     edi, edx  
.text:0041E9B3                shr     ecx, 2  
.text:0041E9B6                rep movsd  
.text:0041E9B8                mov     ecx, eax  
.text:0041E9BA                and     ecx, 3  
.text:0041E9BD                rep movsb  
.text:0041E9BF                call    _rand           ; 随机函数  
.text:0041E9C4                mov     esi, eax  
.text:0041E9C6                shl     esi, 10h  
.text:0041E9C9                sub     esi, eax  
.text:0041E9CB                call    _rand           ; 随机函数  
.text:0041E9D0                add     esi, eax  
.text:0041E9D2                mov     eax, dword ptr pszPath+8E0h  
.text:0041E9D7                test    eax, eax  
.text:0041E9D9                mov     [esp+4DC4h+var_4D84], esi  
.text:0041E9DD                jnz     short loc_41E9E9  
.text:0041E9DF                call    sub_435BA0  
.text:0041E9E4                mov     dword ptr pszPath+8E0h, eax  
.text:0041E9E9  
.text:0041E9E9 loc_41E9E9:                              ; CODE XREF: sub_41E480+55Dj  
.text:0041E9E9                mov     ecx, dword ptr pszPath+8A0h  
.text:0041E9EF                mov     [esp+4DC4h+var_4D54], eax  
.text:0041E9F3                add     ecx, 6864h  
.text:0041E9F9                call    sub_411570  
.text:0041E9FE                mov     edx, dword ptr pszPath+8A0h  
.text:0041EA04                lea     ecx, [edx+6864h]  
.text:0041EA0A                call    sub_411590  
.text:0041EA0F                mov     eax, dword ptr pszPath+8A0h  
.text:0041EA14                lea     ecx, [eax+6864h]  
.text:0041EA1A                call    sub_4115B0  
.text:0041EA1F                mov     ecx, dword ptr pszPath+8A0h  
.text:0041EA25                mov     esi, eax  
.text:0041EA27                add     ecx, 6864h  
.text:0041EA2D                call    sub_4115C0  
.text:0041EA32                mov     edi, eax  
.text:0041EA34                lea     edx, [esp+4DC4h+var_4A1C]  
.text:0041EA3B                push    22A1h  
.text:0041EA40                lea     eax, [esp+4DC8h+var_4A9C]  
.text:0041EA47                push    edx                     ;0484B4F0  
.text:0041EA48                push    eax                     ;0484B470  
.text:0041EA49                lea     ecx, [esp+4DD0h+var_4D88]  
```
