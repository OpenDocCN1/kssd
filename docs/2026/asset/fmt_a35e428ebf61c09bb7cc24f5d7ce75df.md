
```text
004F774D   |.  FF35 1C8C5400      |push dword ptr ds:[548C1C]                   ; |Arg2 = 0054858A
004F7753   |.  53                 |push ebx                                     ; |Arg1
004F7754   |.  E8 7DF2FFFF        |call <IP_TOOLS.sub_4F69D6>                   ; \IP_TOOLS.004F69D6
004F7759   |.  83C4 0C            |add esp,0C
004F775C <>|>  8B45 FC            |mov eax,dword ptr ss:[ebp-4]                 ;  loc_4F775C
004F775F   |.  66:8B00            |mov ax,word ptr ds:[eax]
004F7762   |.  66:2B05 568C5400   |sub ax,word ptr ds:[548C56]
004F7769   |.  66:85C0            |test ax,ax
004F776C   |.  7F 31              |jg short <IP_TOOLS.loc_4F779F>
004F776E   |.  66:85C0            |test ax,ax
004F7771   |.  75 3D              |jnz short <IP_TOOLS.loc_4F77B0>
004F7773   |.  66:8B16            |mov dx,word ptr ds:[esi]
004F7776   |.  66:3B15 788C5400   |cmp dx,word ptr ds:[548C78]
004F777D   |.  77 20              |ja short <IP_TOOLS.loc_4F779F>
004F777F   |.  66:8B0E            |mov cx,word ptr ds:[esi]
004F7782   |.  66:3B0D 788C5400   |cmp cx,word ptr ds:[548C78]
004F7789   |.  75 25              |jnz short <IP_TOOLS.loc_4F77B0>
004F778B   |.  FF35 188C5400      |push dword ptr ds:[548C18]                   ; /Arg2 = 0054850A
004F7791   |.  53                 |push ebx                                     ; |Arg1
004F7792   |.  E8 10F3FFFF        |call <IP_TOOLS.sub_4F6AA7>                   ; \IP_TOOLS.004F6AA7
004F7797   |.  83C4 08            |add esp,8
004F779A   |.  66:85C0            |test ax,ax
004F779D   |.  7C 11              |jl short <IP_TOOLS.loc_4F77B0>
004F779F <>|>  6A 00              |push 0                                       ; /loc_4F779F
004F77A1   |.  FF35 188C5400      |push dword ptr ds:[548C18]                   ; |Arg2 = 0054850A
004F77A7   |.  53                 |push ebx                                     ; |Arg1
004F77A8   |.  E8 29F2FFFF        |call <IP_TOOLS.sub_4F69D6>                   ; \IP_TOOLS.004F69D6
004F77AD   |.  83C4 0C            |add esp,0C
004F77B0 <>|>  8B45 FC            |mov eax,dword ptr ss:[ebp-4]                 ;  loc_4F77B0
004F77B3   |.  66:8B00            |mov ax,word ptr ds:[eax]
004F77B6   |.  66:2B05 548C5400   |sub ax,word ptr ds:[548C54]
004F77BD   |.  66:85C0            |test ax,ax
004F77C0   |.  7F 31              |jg short <IP_TOOLS.loc_4F77F3>
004F77C2   |.  66:85C0            |test ax,ax
004F77C5   |.  75 3D              |jnz short <IP_TOOLS.loc_4F7804>
004F77C7   |.  66:8B16            |mov dx,word ptr ds:[esi]
004F77CA   |.  66:3B15 768C5400   |cmp dx,word ptr ds:[548C76]
004F77D1   |.  77 20              |ja short <IP_TOOLS.loc_4F77F3>
004F77D3   |.  66:8B0E            |mov cx,word ptr ds:[esi]
004F77D6   |.  66:3B0D 768C5400   |cmp cx,word ptr ds:[548C76]
004F77DD   |.  75 25              |jnz short <IP_TOOLS.loc_4F7804>
004F77DF   |.  FF35 148C5400      |push dword ptr ds:[548C14]                   ; /Arg2 = 0054848A
004F77E5   |.  53                 |push ebx                                     ; |Arg1
004F77E6   |.  E8 BCF2FFFF        |call <IP_TOOLS.sub_4F6AA7>                   ; \IP_TOOLS.004F6AA7
004F77EB   |.  83C4 08            |add esp,8
004F77EE   |.  66:85C0            |test ax,ax
004F77F1   |.  7C 11              |jl short <IP_TOOLS.loc_4F7804>
004F77F3 <>|>  6A 00              |push 0                                       ; /loc_4F77F3
004F77F5   |.  FF35 148C5400      |push dword ptr ds:[548C14]                   ; |Arg2 = 0054848A
004F77FB   |.  53                 |push ebx                                     ; |Arg1
004F77FC   |.  E8 D5F1FFFF        |call <IP_TOOLS.sub_4F69D6>                   ; \IP_TOOLS.004F69D6
004F7801   |.  83C4 0C            |add esp,0C
004F7804 <>|>  8B45 FC            |mov eax,dword ptr ss:[ebp-4]                 ;  loc_4F7804
004F7807   |.  66:8B00            |mov ax,word ptr ds:[eax]
004F780A   |.  66:2B05 528C5400   |sub ax,word ptr ds:[548C52]
004F7811   |.  66:85C0            |test ax,ax
004F7814   |.  7F 31              |jg short <IP_TOOLS.loc_4F7847>
004F7816   |.  66:85C0            |test ax,ax
004F7819   |.  75 3D              |jnz short <IP_TOOLS.loc_4F7858>
004F781B   |.  66:8B16            |mov dx,word ptr ds:[esi]
004F781E   |.  66:3B15 748C5400   |cmp dx,word ptr ds:[548C74]
004F7825   |.  77 20              |ja short <IP_TOOLS.loc_4F7847>
004F7827   |.  66:8B0E            |mov cx,word ptr ds:[esi]
004F782A   |.  66:3B0D 748C5400   |cmp cx,word ptr ds:[548C74]
004F7831   |.  75 25              |jnz short <IP_TOOLS.loc_4F7858>
004F7833   |.  FF35 108C5400      |push dword ptr ds:[548C10]                   ; /Arg2 = 0054840A
004F7839   |.  53                 |push ebx                                     ; |Arg1
004F783A   |.  E8 68F2FFFF        |call <IP_TOOLS.sub_4F6AA7>                   ; \IP_TOOLS.004F6AA7
004F783F   |.  83C4 08            |add esp,8
004F7842   |.  66:85C0            |test ax,ax
004F7845   |.  7C 11              |jl short <IP_TOOLS.loc_4F7858>
004F7847 <>|>  6A 00              |push 0                                       ; /loc_4F7847
004F7849   |.  FF35 108C5400      |push dword ptr ds:[548C10]                   ; |Arg2 = 0054840A
004F784F   |.  53                 |push ebx                                     ; |Arg1
004F7850   |.  E8 81F1FFFF        |call <IP_TOOLS.sub_4F69D6>                   ; \IP_TOOLS.004F69D6
004F7855   |.  83C4 0C            |add esp,0C
004F7858 <>|>  8B45 FC            |mov eax,dword ptr ss:[ebp-4]                 ;  loc_4F7858
004F785B   |.  66:8B00            |mov ax,word ptr ds:[eax]
004F785E   |.  66:2B05 508C5400   |sub ax,word ptr ds:[548C50]
004F7865   |.  66:85C0            |test ax,ax
004F7868   |.  7F 31              |jg short <IP_TOOLS.loc_4F789B>
004F786A   |.  66:85C0            |test ax,ax
004F786D   |.  75 3D              |jnz short <IP_TOOLS.loc_4F78AC>
004F786F   |.  66:8B16            |mov dx,word ptr ds:[esi]
004F7872   |.  66:3B15 728C5400   |cmp dx,word ptr ds:[548C72]
004F7879   |.  77 20              |ja short <IP_TOOLS.loc_4F789B>
004F787B   |.  66:8B0E            |mov cx,word ptr ds:[esi]
004F787E   |.  66:3B0D 728C5400   |cmp cx,word ptr ds:[548C72]
004F7885   |.  75 25              |jnz short <IP_TOOLS.loc_4F78AC>
004F7887   |.  FF35 0C8C5400      |push dword ptr ds:[548C0C]                   ; /Arg2 = 00548102
004F788D   |.  53                 |push ebx                                     ; |Arg1
004F788E   |.  E8 14F2FFFF        |call <IP_TOOLS.sub_4F6AA7>                   ; \IP_TOOLS.004F6AA7
004F7893   |.  83C4 08            |add esp,8
004F7896   |.  66:85C0            |test ax,ax
004F7899   |.  7C 11              |jl short <IP_TOOLS.loc_4F78AC>
004F789B <>|>  6A 00              |push 0                                       ; /loc_4F789B
004F789D   |.  FF35 0C8C5400      |push dword ptr ds:[548C0C]                   ; |Arg2 = 00548102
004F78A3   |.  53                 |push ebx                                     ; |Arg1
004F78A4   |.  E8 2DF1FFFF        |call <IP_TOOLS.sub_4F69D6>                   ; \IP_TOOLS.004F69D6
004F78A9   |.  83C4 0C            |add esp,0C
004F78AC <>|>  83EF 02            |sub edi,2                                    ;  loc_4F78AC
004F78AF <>|>  66:8B45 FA          mov ax,word ptr ss:[ebp-6]                   ;  这里是用户名的字长
004F78B3   |.  66:8345 FA FF      |add word ptr ss:[ebp-6],0FFFF
004F78B8   |.  66:85C0            |test ax,ax
004F78BB   |.^ 0F85 E8F8FFFF      \jnz <IP_TOOLS.loc_4F71A9>
004F78C1   |.  33C0               xor eax,eax
}
```

-------------------------------------------------------------------------------------------------------

下面是我用C写的核心函数,称之为θ函数`Y=θ(message)`

```text
void theta1(WORD message[Len])
{
  WORD arg[8]={0x80,0x40,0x20,0x10,0x8,0x4,0x2,0x1};
  WORD w;
  WORD wltemp;
  WORD whtemp;
```
