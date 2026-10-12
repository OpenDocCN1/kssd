
```text
0042D8E3     90                     nop  

0042D8E4     8B5424 18            mov edx,dword ptr ss:[esp+18]  

0042D8E8     8B72 F4             mov esi,dword ptr ds:[edx-C]  

0042D8EB     90                     nop  

0042D8EC     90                     nop  

0042D8ED     90                     nop  

0042D8EE     8B5424 18            mov edx,dword ptr ss:[esp+18]  

0042D8F2     8B42 FC             mov eax,dword ptr ds:[edx-4]  

0042D8F5     B9 01000000          mov ecx,1  

0042D8FA     2BC8                 sub ecx,eax  

0042D8FC     8B42 F8             mov eax,dword ptr ds:[edx-8]  

0042D8FF     2BC6                 sub eax,esi  

0042D901     0BC1                 or eax,ecx  

0042D903     7D 0E                jge short x.0042D913  

0042D905     56                     push esi  

0042D906     8D4C24 1C            lea ecx,dword ptr ss:[esp+1C]  

0042D90A     E8 9157FDFF          call x.004030A0  

0042D90F     8B5424 18            mov edx,dword ptr ss:[esp+18]  

0042D913     56                     push esi  

0042D914     52                     push edx  

0042D915     E8 8624FFFF          call x.0041FDA0                   ; 若是，生成的新字串做进一步运算  

0042D91A     8B5424 20            mov edx,dword ptr ss:[esp+20]  

0042D91E     83C4 08              add esp,8  

0042D921     83FE FF              cmp esi,-1  

0042D924     8BC6                 mov eax,esi  

0042D926     75 16                jnz short x.0042D93E  

0042D928     85D2                 test edx,edx  

0042D92A     75 04                jnz short x.0042D930  

0042D92C     33C0                 xor eax,eax  

0042D92E     EB 16                jmp short x.0042D946  

0042D930     8BC2                 mov eax,edx  

0042D932     8D70 01              lea esi,dword ptr ds:[eax+1]  

0042D935     8A08                 mov cl,byte ptr ds:[eax]  

0042D937     40                     inc eax  

0042D938     84C9                 test cl,cl  

0042D93A     75 F9                jnz short x.0042D935  

0042D93C     2BC6                 sub eax,esi  

0042D93E     85C0                 test eax,eax  

0042D940     0F8C 27030000        jl x.0042DC6D  

0042D946     3B42 F8              cmp eax,dword ptr ds:[edx-8]  

0042D949     0F8F 1E030000        jg x.0042DC6D  

0042D94F     8942 F4              mov dword ptr ds:[edx-C],eax  

0042D952     8B4C24 18            mov ecx,dword ptr ss:[esp+18]  

0042D956     C60408 00            mov byte ptr ds:[eax+ecx],0  

0042D95A     90                     nop                                ; 不是，跳到此处  

0042D95B     90                     nop  

0042D95C     90                     nop  

0042D95D     8B7424 4C            mov esi,dword ptr ss:[esp+4C]  

0042D961     51                     push ecx  

0042D962     8B4E F0              mov ecx,dword ptr ds:[esi-10]  

0042D965     8B11                 mov edx,dword ptr ds:[ecx]  

0042D967     83C6 F0              add esi,-10  

0042D96A     896424 34            mov dword ptr ss:[esp+34],esp  

0042D96E     8BDC                 mov ebx,esp  

0042D970     FF52 10              call dword ptr ds:[edx+10]  

0042D973     8B56 0C              mov edx,dword ptr ds:[esi+C]  

0042D976     85D2                 test edx,edx  

0042D978     8D4E 0C              lea ecx,dword ptr ds:[esi+C]  

0042D97B     7C 11                jl short x.0042D98E  

0042D97D     3B06                 cmp eax,dword ptr ds:[esi]  

0042D97F     75 0D                jnz short x.0042D98E  

0042D981     8BC6                 mov eax,esi  

0042D983     BA 01000000          mov edx,1  

0042D988     F0:0FC111            lock xadd dword ptr ds:[ecx],edx  

0042D98C     EB 36                jmp short x.0042D9C4  

0042D98E     8B4E 04              mov ecx,dword ptr ds:[esi+4]  

0042D991     8B10                 mov edx,dword ptr ds:[eax]  

0042D993     6A 01                push 1  

0042D995     51                     push ecx  

0042D996     8BC8                 mov ecx,eax  

0042D998     FF12                 call dword ptr ds:[edx]  

0042D99A     85C0                 test eax,eax  

0042D99C     0F84 27020000        je x.0042DBC9  

0042D9A2     8B56 04              mov edx,dword ptr ds:[esi+4]  

0042D9A5     8950 04              mov dword ptr ds:[eax+4],edx  

0042D9A8     8B4E 04              mov ecx,dword ptr ds:[esi+4]  

0042D9AB     41                     inc ecx  

0042D9AC     8BD1                 mov edx,ecx  

0042D9AE     C1E9 02              shr ecx,2  

0042D9B1     83C6 10              add esi,10  

0042D9B4     8D78 10              lea edi,dword ptr ds:[eax+10]  

0042D9B7     F3:A5                rep movs dword ptr es:[edi],dword ptr ds:>  

0042D9B9     8BCA                 mov ecx,edx  

0042D9BB     83E1 03              and ecx,3  

0042D9BE     F3:A4                rep movs byte ptr es:[edi],byte ptr ds:[e>  

0042D9C0     8B7C24 20            mov edi,dword ptr ss:[esp+20]  

0042D9C4     83C0 10              add eax,10  

0042D9C7     8903                 mov dword ptr ds:[ebx],eax  

0042D9C9     8D4424 1C            lea eax,dword ptr ss:[esp+1C]  

0042D9CD     50                     push eax  

0042D9CE     E8 ED58FDFF          call x.004032C0                   ; 算法 call，跟进  

0042D9D3     83C4 08              add esp,8  

0042D9D6     90                     nop  

0042D9D7     90                     nop  

0042D9D8     90                     nop  

0042D9D9     8B4424 18            mov eax,dword ptr ss:[esp+18]  

0042D9DD     8B48 F4              mov ecx,dword ptr ds:[eax-C]  

0042D9E0     85C9                 test ecx,ecx  

0042D9E2     0F8C 85020000        jl x.0042DC6D  

0042D9E8     8038 00              cmp byte ptr ds:[eax],0            ; 判断新字串首字节是否小于 0  

0042D9EB     7E 4F                jle short x.0042DA3C  

0042D9ED     90                     nop  

0042D9EE     90                     nop  

0042D9EF     90                     nop  

0042D9F0     90                     nop  

0042D9F1     90                     nop  

0042D9F2     8B4424 18            mov eax,dword ptr ss:[esp+18]  

0042D9F6     8B48 F4              mov ecx,dword ptr ds:[eax-C]  

0042D9F9     85C9                 test ecx,ecx  

0042D9FB     0F8C 6C020000        jl x.0042DC6D  

0042DA01     0FBE00               movsx eax,byte ptr ds:[eax]        ; 是，取新字符串的首字符运算生成注册码  

0042DA04     99                     cdq  

0042DA05     B9 1A000000          mov ecx,1A  

0042DA0A     F7F9                 idiv ecx  

0042DA0C     8D4424 10            lea eax,dword ptr ss:[esp+10]  

0042DA10     C64424 11 00         mov byte ptr ss:[esp+11],0  

0042DA15     80C2 41              add dl,41  

0042DA18     885424 10            mov byte ptr ss:[esp+10],dl  

0042DA1C     8D50 01              lea edx,dword ptr ds:[eax+1]  

0042DA1F     90                     nop  

0042DA20     8A08                 mov cl,byte ptr ds:[eax]  

0042DA22     40                     inc eax  

0042DA23     84C9                 test cl,cl  

0042DA25     75 F9                jnz short x.0042DA20  

0042DA27     2BC2                 sub eax,edx  

0042DA29     50                     push eax  

0042DA2A     8D5424 14            lea edx,dword ptr ss:[esp+14]  

0042DA2E     52                     push edx  

0042DA2F     8D4C24 20            lea ecx,dword ptr ss:[esp+20]  

0042DA33     E8 6857FDFF          call x.004031A0  

0042DA38     90                     nop  

0042DA39     90                     nop  

0042DA3A     EB 54                jmp short x.0042DA90  

0042DA3C     90                     nop  

0042DA3D     90                     nop  

0042DA3E     90                     nop  

0042DA3F     90                     nop  

0042DA40     8B4424 18            mov eax,dword ptr ss:[esp+18]  

0042DA44     8B48 F4              mov ecx,dword ptr ds:[eax-C]  

0042DA47     85C9                 test ecx,ecx  

0042DA49     0F8C 1E020000        jl x.0042DC6D  

0042DA4F     0FBE00               movsx eax,byte ptr ds:[eax]        ; 不是，取新字符串的首字符运算生成注册码  

0042DA52     99                     cdq  

0042DA53     33C2                 xor eax,edx  

0042DA55     2BC2                 sub eax,edx  

0042DA57     99                     cdq  

0042DA58     B9 0A000000          mov ecx,0A  

0042DA5D     F7F9                 idiv ecx  

0042DA5F     8D4424 14            lea eax,dword ptr ss:[esp+14]  
```
