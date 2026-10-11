
```
0041F876 72 03          jb short 试炼 ACP.0041F87B
0041F878 73 01          jnb short 试炼 ACP.0041F87B
0041F87B 66:81C1 FEEE   add cx,0EEFE
0041F880 EB 01          jmp short 试炼 ACP.0041F883
0041F883 4D             dec ebp
0041F884 50             push eax
0041F885 E8 01000000    call 试炼 ACP.0041F88B
0041F88B 58             pop eax
0041F88C 58             pop eax
0041F88D 66:C1D5 54     rcl bp,54
0041F891 E8 01000000    call 试炼 ACP.0041F897
0041F897 830424 06      add dword ptr ss:[esp],6
0041F89B C3             retn
0041F89C FC             cld
0041F89D EB 01          jmp short 试炼 ACP.0041F8A0
0041F8A0 87CB           xchg ebx,ecx
0041F8A2 72 03          jb short 试炼 ACP.0041F8A7
0041F8A4 73 01          jnb short 试炼 ACP.0041F8A7
0041F8A7 77 03          ja short 试炼 ACP.0041F8AC
0041F8AC E8 01000000    call 试炼 ACP.0041F8B2
0041F8B2 830424 06      add dword ptr ss:[esp],6
0041F8B6 C3             retn
0041F8B7 66:81E9 352E   sub cx,2E35
0041F8BC EB 01          jmp short 试炼 ACP.0041F8BF
0041F8BF 0F88 04000000  js 试炼 ACP.0041F8C9
0041F8C5 |66:C1FB C9    sar bx,0C9
0041F8C9 E8 01000000    call 试炼 ACP.0041F8CF
0041F8CF 830424 06      add dword ptr ss:[esp],6
0041F8D3 C3             retn
0041F8D4 8BEB           mov ebp,ebx
0041F8D6 7A 03          jpe short 试炼 ACP.0041F8DB
0041F8DB 0F82 06000000  jb 试炼 ACP.0041F8E7
0041F8E1 |E8 00000000   call 试炼 ACP.0041F8E6
0041F8E6 |59            pop ecx
0041F8E7 EB 01          jmp short 试炼 ACP.0041F8EA
0041F8EA 41             inc ecx
0041F8EB E8 01000000    call 试炼 ACP.0041F8F1
0041F8F1 830424 06      add dword ptr ss:[esp],6
0041F8F5 C3             retn
0041F8F6 F9             stc
0041F8F7 50             push eax
0041F8F8 E8 01000000    call 试炼 ACP.0041F8FE
0041F8FE 58             pop eax
0041F8FF 58             pop eax
0041F900 F8             clc
0041F901 E8 01000000    call 试炼 ACP.0041F907
0041F907 83C4 04        add esp,4
0041F90A 0F8D 04000000  jge 试炼 ACP.0041F914
0041F914 E8 01000000    call 试炼 ACP.0041F91A
0041F91A 830424 06      add dword ptr ss:[esp],6
0041F91E C3             retn
0041F91F 4D             dec ebp
0041F920 EB 01          jmp short 试炼 ACP.0041F923
0041F923 8BDE           mov ebx,esi
0041F925 50             push eax
0041F926 E8 01000000    call 试炼 ACP.0041F92C
0041F92C 58             pop eax
0041F92D 58             pop eax
0041F92E 66:13E9        adc bp,cx
0041F931 E8 01000000    call 试炼 ACP.0041F937
0041F937 830424 06      add dword ptr ss:[esp],6
0041F93B C3             retn
0041F93C FC             cld
0041F93D E8 01000000    call 试炼 ACP.0041F943
0041F943 830424 06      add dword ptr ss:[esp],6
0041F947 C3             retn
0041F948 85CB           test ebx,ecx
0041F94A 76 03          jbe short 试炼 ACP.0041F94F
0041F94C 77 01          ja short 试炼 ACP.0041F94F
0041F94F 0F8B 05000000  jpo 试炼 ACP.0041F95A
0041F95A EB 01          jmp short 试炼 ACP.0041F95D
0041F95D 66:03CE        add cx,si
0041F960 E8 01000000    call 试炼 ACP.0041F966
0041F966 830424 06      add dword ptr ss:[esp],6
0041F96A C3             retn
0041F96B 4B             dec ebx
0041F96C E8 01000000    call 试炼 ACP.0041F972
0041F972 830424 06      add dword ptr ss:[esp],6
0041F976 C3             retn
0041F977 1BCF           sbb ecx,edi
0041F979 E8 00000000    call 试炼 ACP.0041F97E
0041F97E 5D             pop ebp
0041F97F 8BC5           mov eax,ebp
0041F981 3B45 1C        cmp eax,dword ptr ss:[ebp+1C]
0041F984 7C 06          jl short 试炼 ACP.0041F98C
0041F986 0345 1C        add eax,dword ptr ss:[ebp+1C]
0041F989 8945 1C        mov dword ptr ss:[ebp+1C],eax
0041F98C EB 01          jmp short 试炼 ACP.0041F98F
0041F98F 66:C1D5 58     rcl bp,58
0041F993 87EB           xchg ebx,ebp
0041F995 D3ED           shr ebp,cl
0041F997 85CD           test ebp,ecx
0041F999 68 C5FA4100    push 试炼 ACP.0041FAC5
0041F99E 66:D3EB        shr bx,cl
0041F9A1 5A             pop edx
0041F9A2 E8 01000000    call 试炼 ACP.0041F9A8
0041F9A8 830424 06      add dword ptr ss:[esp],6
0041F9AC C3             retn
0041F9AD 4B             dec ebx
0041F9AE C1C3 E9        rol ebx,0E9
0041F9B1 E9 05000000    jmp 试炼 ACP.0041F9BB
0041F9BB BE 38CEF63E    mov esi,3EF6CE38
0041F9C0 75 04          jnz short 试炼 ACP.0041F9C6
0041F9C6 81F6 F74B5C74  xor esi,745C4BF7
0041F9CC E8 01000000    call 试炼 ACP.0041F9D2
0041F9D2 83C4 04        add esp,4
0041F9D5 85EB           test ebx,ebp
0041F9D7 C1E3 79        shl ebx,79
0041F9DA B8 0C000000    mov eax,0C
0041F9DF EB 01          jmp short 试炼 ACP.0041F9E2
0041F9E2 E9 03000000    jmp 试炼 ACP.0041F9EA
0041F9EA 66:8BEF        mov bp,di
0041F9ED 8B3A           mov edi,dword ptr ds:[edx]
0041F9EF E8 01000000    call 试炼 ACP.0041F9F5
0041F9F5 830424 06      add dword ptr ss:[esp],6
0041F9F9 C3             retn
0041F9FA F8             clc
0041F9FB D3D5           rcl ebp,cl
0041F9FD E9 08000000    jmp 试炼 ACP.0041FA0A
0041FA0A 33FE           xor edi,esi
0041FA0C E8 01000000    call 试炼 ACP.0041FA12
0041FA12 83C4 04        add esp,4
0041FA15 E9 05000000    jmp 试炼 ACP.0041FA1F
0041FA1F 8BCF           mov ecx,edi
0041FA21 C1CF 19        ror edi,19
0041FA24 E8 01000000    call 试炼 ACP.0041FA2A
0041FA2A 83C4 04        add esp,4
0041FA2D 7A 05          jpe short 试炼 ACP.0041FA34
0041FA2F B9 021E239D    mov ecx,9D231E02
0041FA34 4D             dec ebp
0041FA35 E9 02000000    jmp 试炼 ACP.0041FA3C
0041FA3C 337A 04        xor edi,dword ptr ds:[edx+4]
0041FA3F 50             push eax
0041FA40 E8 01000000    call 试炼 ACP.0041FA46
0041FA46 58             pop eax
0041FA47 58             pop eax
0041FA48 E9 05000000    jmp 试炼 ACP.0041FA52
0041FA52 85CD           test ebp,ecx
0041FA54 85E9           test ecx,ebp
0041FA56 893A           mov dword ptr ds:[edx],edi
0041FA58 50             push eax
0041FA59 E8 01000000    call 试炼 ACP.0041FA5F
0041FA5F 58             pop eax
0041FA60 58             pop eax
0041FA61 F8             clc
0041FA62 E9 06000000    jmp 试炼 ACP.0041FA6D
0041FA6D 81C6 097B16A2  add esi,A2167B09
```
