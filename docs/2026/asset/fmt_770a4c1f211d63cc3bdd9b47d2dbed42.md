
```
.text:10001C16                 cmp     dword ptr [esp+20h], 0AE1982AEh
.text:10001C1E                 jnz     short loc_10001C35
.text:10001C20                 call    GetFakeData
.text:10001C25                 test    edx, edx
.text:10001C27                 jz      short loc_10001C35
.text:10001C29                 mov     edx, [edx+8]
.text:10001C2C                 mov     eax, [esp+8]
.text:10001C30                 mov     [eax], edx
.text:10001C32                 xor     eax, eax
.text:10001C34                 retn
.text:10001C35 ; ---------------------------------------------------------------------------
.text:10001C35
.text:10001C35 loc_10001C35:                           ; CODE XREF: .text:10001C1Ej
.text:10001C35                                         ; .text:10001C27j
.text:10001C35                 push    edx
.text:10001C36                 call    sub_10001DF1
.text:10001C3B                 cmp     dword ptr [edx+4], 0
.text:10001C3F                 jnz     short loc_10001C4A
.text:10001C41                 pop     edx
.text:10001C42                 lea     edx, [esp+8]
.text:10001C46                 int     2Eh             ; DOS 2+ internal - EXECUTE COMMAND
.text:10001C46                                         ; DS:SI -> counted CR-terminated command string
.text:10001C48                 jmp     short locret_10001C56
.text:10001C4A ; ---------------------------------------------------------------------------
.text:10001C4A
.text:10001C4A loc_10001C4A:                           ; CODE XREF: .text:10001C3Fj
.text:10001C4A                 pop     edx
.text:10001C4B                 lea     edx, [esp+8]
.text:10001C4F                 call    large dword ptr fs:0C0h
.text:10001C56
.text:10001C56 locret_10001C56:                        ; CODE XREF: .text:10001C48j
.text:10001C56                 retn
```

首先，判断文件句柄是不是假文件句柄，如果是的话，返回之前没有Close的Section句柄

`ZwQuerySection`

```
:10001D3D                 call    GetFakeData
.text:10001D42                 test    edx, edx
.text:10001D44                 push    edx
.text:10001D45                 jz      short loc_10001D8C
.text:10001D47                 mov     edx, [edx+8]
.text:10001D4A                 cmp     edx, [esp+0Ch]
.text:10001D4E                 jnz     short loc_10001D8C
.text:10001D50                 cmp     dword ptr [esp+10h], 1
.text:10001D55                 jnz     short loc_10001D8C
.text:10001D57                 cmp     dword ptr [esp+18h], 30h
.text:10001D5C                 jl      short loc_10001D85
.text:10001D5E                 pop     edx
.text:10001D5F                 push    ecx
.text:10001D60                 push    esi
.text:10001D61                 push    edi
.text:10001D62                 lea     esi, [edx+50h]
.text:10001D65                 mov     edi, [esp+1Ch]
.text:10001D69                 mov     ecx, 30h
.text:10001D6E                 rep movsb
.text:10001D70                 pop     edi
.text:10001D71                 pop     esi
.text:10001D72                 pop     ecx
.text:10001D73                 mov     eax, [esp+18h]
.text:10001D77                 cmp     eax, 0
.text:10001D7A                 jz      short loc_10001D82
.text:10001D7C                 mov     dword ptr [eax], 30h
.text:10001D82
.text:10001D82 loc_10001D82:                           ; CODE XREF: .text:10001D7Aj
.text:10001D82                 xor     eax, eax
.text:10001D84                 retn
.text:10001D85 ; ---------------------------------------------------------------------------
.text:10001D85
.text:10001D85 loc_10001D85:                           ; CODE XREF: .text:10001D5Cj
.text:10001D85                 pop     edx
.text:10001D86                 mov     eax, 0C000000Dh
.text:10001D8B                 retn
.text:10001D8C loc_10001D8C:                           ; CODE XREF: .text:10001D45j
.text:10001D8C                                         ; .text:10001D4Ej ...
.text:10001D8C                 pop     edx
.text:10001D8D                 push    edx
.text:10001D8E                 call    sub_10001DF1
.text:10001D93                 cmp     dword ptr [edx+4], 0
.text:10001D97                 jnz     short loc_10001DA2
.text:10001D99                 pop     edx
.text:10001D9A                 lea     edx, [esp+8]
.text:10001D9E                 int     2Eh             ; DOS 2+ internal - EXECUTE COMMAND
.text:10001D9E                                         ; DS:SI -> counted CR-terminated command string
.text:10001DA0                 jmp     short locret_10001DAE
.text:10001DA2 ; ---------------------------------------------------------------------------
.text:10001DA2
.text:10001DA2 loc_10001DA2:                           ; CODE XREF: .text:10001D97j
.text:10001DA2                 pop     edx
.text:10001DA3                 lea     edx, [esp+8]
.text:10001DA7                 call    large dword ptr fs:0C0h
.text:10001DAE
.text:10001DAE locret_10001DAE:                        ; CODE XREF: .text:10001DA0j
.text:10001DAE                 retn
```

判断Size是不是0x30，不是的话，返回0C000000Dh，如果section句柄是指定的句柄，是的话，将内存中的PE头相关信息复制到对应的参数

`ZwMapViewOfSection`
