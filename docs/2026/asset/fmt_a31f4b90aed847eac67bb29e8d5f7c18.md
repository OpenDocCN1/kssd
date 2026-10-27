
```
004F7345   |.  75 25              |jnz short <IP_TOOLS.loc_4F736C>
004F7347   |.  FF35 4C8C5400      |push dword ptr ds:[548C4C]                   ; /Arg2 = 00548B8A
004F734D   |.  53                 |push ebx                                     ; |Arg1
004F734E   |.  E8 54F7FFFF        |call <IP_TOOLS.sub_4F6AA7>                   ; \IP_TOOLS.004F6AA7
004F7353   |.  83C4 08            |add esp,8
004F7356   |.  66:85C0            |test ax,ax
004F7359   |.  7C 11              |jl short <IP_TOOLS.loc_4F736C>
004F735B <>|>  6A 00              |push 0                                       ; /loc_4F735B
004F735D   |.  FF35 4C8C5400      |push dword ptr ds:[548C4C]                   ; |Arg2 = 00548B8A
004F7363   |.  53                 |push ebx                                     ; |Arg1
004F7364   |.  E8 6DF6FFFF        |call <IP_TOOLS.sub_4F69D6>                   ; \IP_TOOLS.004F69D6
```

---

```
004F69D6 <>/$  55                 push ebp                                      ;  SubHash
004F69D7   |.  8BEC               mov ebp,esp
004F69D9   |.  53                 push ebx
004F69DA   |.  56                 push esi
004F69DB   |.  57                 push edi
004F69DC   |.  8B45 10            mov eax,dword ptr ss:[ebp+10]
004F69DF   |.  8B4D 0C            mov ecx,dword ptr ss:[ebp+C]
004F69E2   |.  8B55 08            mov edx,dword ptr ss:[ebp+8]
004F69E5   |.  66:8B35 08845400   mov si,word ptr ds:[548408]
004F69EC   |.  EB 25              jmp short <IP_TOOLS.loc_4F6A13>
004F69EE <>|>  0FB71A             /movzx ebx,word ptr ds:[edx]
004F69F1   |.  0FB739             |movzx edi,word ptr ds:[ecx]
004F69F4   |.  2BDF               |sub ebx,edi                     ;ebx-=edi;
004F69F6   |.  25 FF000000        |and eax,0FF                     ;eax&=0xff
004F69FB   |.  2BD8               |sub ebx,eax                     ;ebx-=eax;
004F69FD   |.  8BC3               |mov eax,ebx                     ;eax=ebx
004F69FF   |.  83C1 02            |add ecx,2
004F6A02   |.  66:8902            |mov word ptr ds:[edx],ax        ;ax=WORD(eax)
004F6A05   |.  83C2 02            |add edx,2
004F6A08   |.  A9 00000100        |test eax,10000                  ;if(eax&0x10000)
004F6A0D   |.  0F95C0             |setne al                        ;如果不等于0就置al=1
004F6A10   |.  83E0 01            |and eax,1                       ;和1相与
004F6A13 <>|>  8BDE                mov ebx,esi
004F6A15   |.  66:83C6 FF         |add si,0FFFF
004F6A19   |.  66:85DB            |test bx,bx
004F6A1C   |.^ 75 D0              \jnz short <IP_TOOLS.loc_4F69EE>
004F6A1E   |.  5F                 pop edi
004F6A1F   |.  5E                 pop esi
004F6A20   |.  5B                 pop ebx
004F6A21   |.  5D                 pop ebp
004F6A22   \.  C3                 retn
```

---

我写的C代码,672位的大数减法

```
void SubHash(WORD result[],WORD subkey[],int carry)

{

  DWORD ebx,edi;

  for(int i=0;i<Len;i++)

  {

    ebx=DWORD(result[i]);

    edi=DWORD(subkey[i]);

    ebx-=edi;

    carry=(carry&0xFF);

    ebx-=carry;

    result[i]=WORD(ebx);

    if((ebx&0x10000)!=0)

      carry=1;

    else

      carry=0;

    carry=(carry&1);

  }

}
```

---

```
004F7369   |.  83C4 0C            |add esp,0C
004F736C <>|>  8B45 FC            |mov eax,dword ptr ss:[ebp-4]                 ;  loc_4F736C
004F736F   |.  66:8B00            |mov ax,word ptr ds:[eax]
004F7372   |.  66:2B05 6E8C5400   |sub ax,word ptr ds:[548C6E]
004F7379   |.  66:85C0            |test ax,ax
004F737C   |.  7F 31              |jg short <IP_TOOLS.loc_4F73AF>
004F737E   |.  66:85C0            |test ax,ax
004F7381   |.  75 3D              |jnz short <IP_TOOLS.loc_4F73C0>
004F7383   |.  66:8B16            |mov dx,word ptr ds:[esi]
004F7386   |.  66:3B15 908C5400   |cmp dx,word ptr ds:[548C90]
004F738D   |.  77 20              |ja short <IP_TOOLS.loc_4F73AF>
004F738F   |.  66:8B0E            |mov cx,word ptr ds:[esi]
004F7392   |.  66:3B0D 908C5400   |cmp cx,word ptr ds:[548C90]
004F7399   |.  75 25              |jnz short <IP_TOOLS.loc_4F73C0>
004F739B   |.  FF35 488C5400      |push dword ptr ds:[548C48]                   ; /Arg2 = 00548B0A
004F73A1   |.  53                 |push ebx                                     ; |Arg1
004F73A2   |.  E8 00F7FFFF        |call <IP_TOOLS.sub_4F6AA7>                   ; \IP_TOOLS.004F6AA7
004F73A7   |.  83C4 08            |add esp,8
004F73AA   |.  66:85C0            |test ax,ax
004F73AD   |.  7C 11              |jl short <IP_TOOLS.loc_4F73C0>
004F73AF <>|>  6A 00              |push 0                                       ; /loc_4F73AF
004F73B1   |.  FF35 488C5400      |push dword ptr ds:[548C48]                   ; |Arg2 = 00548B0A
004F73B7   |.  53                 |push ebx                                     ; |Arg1
004F73B8   |.  E8 19F6FFFF        |call <IP_TOOLS.sub_4F69D6>                   ; \IP_TOOLS.004F69D6
004F73BD   |.  83C4 0C            |add esp,0C
004F73C0 <>|>  8B45 FC            |mov eax,dword ptr ss:[ebp-4]                 ;  loc_4F73C0
004F73C3   |.  66:8B00            |mov ax,word ptr ds:[eax]
004F73C6   |.  66:2B05 6C8C5400   |sub ax,word ptr ds:[548C6C]
004F73CD   |.  66:85C0            |test ax,ax
004F73D0   |.  7F 31              |jg short <IP_TOOLS.loc_4F7403>
004F73D2   |.  66:85C0            |test ax,ax
004F73D5   |.  75 3D              |jnz short <IP_TOOLS.loc_4F7414>
004F73D7   |.  66:8B16            |mov dx,word ptr ds:[esi]
004F73DA   |.  66:3B15 8E8C5400   |cmp dx,word ptr ds:[548C8E]
004F73E1   |.  77 20              |ja short <IP_TOOLS.loc_4F7403>
004F73E3   |.  66:8B0E            |mov cx,word ptr ds:[esi]
004F73E6   |.  66:3B0D 8E8C5400   |cmp cx,word ptr ds:[548C8E]
004F73ED   |.  75 25              |jnz short <IP_TOOLS.loc_4F7414>
004F73EF   |.  FF35 448C5400      |push dword ptr ds:[548C44]                   ; /Arg2 = 00548A8A
004F73F5   |.  53                 |push ebx                                     ; |Arg1
004F73F6   |.  E8 ACF6FFFF        |call <IP_TOOLS.sub_4F6AA7>                   ; \IP_TOOLS.004F6AA7
004F73FB   |.  83C4 08            |add esp,8
004F73FE   |.  66:85C0            |test ax,ax
004F7401   |.  7C 11              |jl short <IP_TOOLS.loc_4F7414>
004F7403 <>|>  6A 00              |push 0                                       ; /loc_4F7403
004F7405   |.  FF35 448C5400      |push dword ptr ds:[548C44]                   ; |Arg2 = 00548A8A
004F740B   |.  53                 |push ebx                                     ; |Arg1
004F740C   |.  E8 C5F5FFFF        |call <IP_TOOLS.sub_4F69D6>                   ; \IP_TOOLS.004F69D6
004F7411   |.  83C4 0C            |add esp,0C
004F7414 <>|>  8B45 FC            |mov eax,dword ptr ss:[ebp-4]                 ;  loc_4F7414
004F7417   |.  66:8B00            |mov ax,word ptr ds:[eax]
004F741A   |.  66:2B05 6A8C5400   |sub ax,word ptr ds:[548C6A]
004F7421   |.  66:85C0            |test ax,ax
004F7424   |.  7F 31              |jg short <IP_TOOLS.loc_4F7457>
004F7426   |.  66:85C0            |test ax,ax
004F7429   |.  75 3D              |jnz short <IP_TOOLS.loc_4F7468>
004F742B   |.  66:8B16            |mov dx,word ptr ds:[esi]
004F742E   |.  66:3B15 8C8C5400   |cmp dx,word ptr ds:[548C8C]
004F7435   |.  77 20              |ja short <IP_TOOLS.loc_4F7457>
004F7437   |.  66:8B0E            |mov cx,word ptr ds:[esi]
004F743A   |.  66:3B0D 8C8C5400   |cmp cx,word ptr ds:[548C8C]
004F7441   |.  75 25              |jnz short <IP_TOOLS.loc_4F7468>
004F7443   |.  FF35 408C5400      |push dword ptr ds:[548C40]                   ; /Arg2 = 00548A0A
004F7449   |.  53                 |push ebx                                     ; |Arg1
```
