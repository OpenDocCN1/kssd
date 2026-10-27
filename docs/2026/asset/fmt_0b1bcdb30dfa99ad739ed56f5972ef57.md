
```
00412A74: 0000                      add byte ptr [eax],al

00412A76: 0000                      add byte ptr [eax],al

00412A78: 0000                      add byte ptr [eax],al

00412A7A: 0000                      add byte ptr [eax],al

00412A7C: 0000                      add byte ptr [eax],al

00412A7E: 0000                      add byte ptr [eax],al

00412A80: 0000                      add byte ptr [eax],al

00412A82: 0000                      add byte ptr [eax],al

00412A84: 0000                      add byte ptr [eax],al

00412A86: 0000                      add byte ptr [eax],al

00412A88: 0000                      add byte ptr [eax],al

00412A8A: 0000                      add byte ptr [eax],al

00412A8C: 0000                      add byte ptr [eax],al

00412A8E: 0000                      add byte ptr [eax],al

00412A90: 0000                      add byte ptr [eax],al

00412A92: 0000                      add byte ptr [eax],al

00412A94: 0000                      add byte ptr [eax],al

00412A96: 00
```

------------------------------------------------------------------

--------------------------------------------------------

```
00412A97: 60                        pushad
00412A98: 8B9D69B44000              mov ebx,dword ptr [ebp+0040B469]  <--41311C
00412A9E: 83FB00                    cmp ebx,00000000
00412AA1: 0F84A6000000              jz 00412B4D
00412AA7: 039DD3B24000              add ebx,dword ptr [ebp+0040B2D3]
00412AAD: 8B430C                    mov eax,dword ptr [ebx+0C]
00412AB0: 83F800                    cmp eax,00000000
00412AB3: 0F8494000000              jz 00412B4D
00412AB9: 0385D3B24000              add eax,dword ptr [ebp+0040B2D3]
00412ABF: 8BF0                      mov esi,eax
00412AC1: 50                        push eax
00412AC2: FF9564BF4000              call dword ptr [ebp+0040BF64]
00412AC8: 89857DBA4000              mov dword ptr [ebp+0040BA7D],eax
00412ACE: 8B33                      mov esi,dword ptr [ebx]
00412AD0: 8B7B10                    mov edi,dword ptr [ebx+10]
00412AD3: 83FE00                    cmp esi,00000000
00412AD6: 7502                      jnz 00412ADA
00412AD8: 8BF7                      mov esi,edi
00412ADA: 03B5D3B24000              add esi,dword ptr [ebp+0040B2D3]
00412AE0: 03BDD3B24000              add edi,dword ptr [ebp+0040B2D3]
00412AE6: 8B06                      mov eax,dword ptr [esi]
00412AE8: 83F800                    cmp eax,00000000
00412AEB: 7505                      jnz 00412AF2
00412AED: 83C314                    add ebx,00000014
00412AF0: EBBB                      jmp 00412AAD
00412AF2: 807E0380                  cmp byte ptr [esi+03],80
00412AF6: 7514                      jnz 00412B0C
00412AF8: 33C0                      xor eax,eax
00412AFA: 668706                    xchg word ptr [esi],ax
00412AFD: 50                        push eax
00412AFE: FFB57DBA4000              push dword ptr [ebp+0040BA7D]
00412B04: FF9568BF4000              call dword ptr [ebp+0040BF68]
00412B0A: EB37                      jmp 00412B43
00412B0C: 33C0                      xor eax,eax
00412B0E: 8706                      xchg dword ptr [esi],eax
00412B10: 0385D3B24000              add eax,dword ptr [ebp+0040B2D3]
00412B16: 83C002                    add eax,00000002
00412B19: 8BC8                      mov ecx,eax
00412B1B: E83B000000                call 00412B5B            <---判断是否为ExitProcess字符串
00412B20: 663D0100                  cmp ax,0001
00412B24: 7510                      jnz 00412B36            <--不同，则跳转
00412B26: C6855DBD400001            mov byte ptr [ebp+0040BD5D],01
00412B2D: B89EBE4000                mov eax,0040BE9E
00412B32: 03C5                      add eax,ebp
00412B34: EB0D                      jmp 00412B43
00412B36: 50                        push eax
00412B37: FFB57DBA4000              push dword ptr [ebp+0040BA7D]
00412B3D: FF9568BF4000              call dword ptr [ebp+0040BF68]
00412B43: 8907                      mov dword ptr [edi],eax
00412B45: 83C604                    add esi,00000004
00412B48: 83C704                    add edi,00000004
00412B4B: EB99                      jmp 00412AE6
00412B4D: 61                        popad
00412B4E: C3                        ret
```

-----------------------------------------------------

--------------------非代码区------------

```
00412B4F: 45                        inc ebp
00412B50: 7869                      js 00412BBB
00412B52: 7450                      jz 00412BA4
00412B54: 726F                      jb 00412BC5
00412B56: 636573                    arpl word ptr [ebp+73],sp
00412B59: 7300                      jnb 00412B5B
```

-----------------------------------

----------------判断是否为ExitProcess字符串--------------------

```
00412B5B: 53                        push ebx
00412B5C: 56                        push esi
00412B5D: 57                        push edi
00412B5E: 8BF0                      mov esi,eax
00412B60: BF9CAE4000                mov edi,0040AE9C
00412B65: 03FD                      add edi,ebp        <---412B4F

\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*

ExitProcess

\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*

00412B67: 8A1E                      mov bl,byte ptr [esi]
00412B69: 3A1F                      cmp bl,byte ptr [edi]
00412B6B: 750C                      jnz 00412B79
00412B6D: 46                        inc esi
00412B6E: 47                        inc edi
00412B6F: 80FB00                    cmp bl,00
00412B72: 75F3                      jnz 00412B67
00412B74: B801000000                mov eax,00000001
00412B79: 5F                        pop edi
00412B7A: 5E                        pop esi
00412B7B: 5B                        pop ebx
00412B7C: C3                        ret
```

 ---------------------------------------------------

--------------------------------------------------

```
00412B7D: 60                        pushad
00412B7E: 33C0                      xor eax,eax
00412B80: 8BB56DB44000              mov esi,dword ptr [ebp+0040B46D]
00412B86: 83FE00                    cmp esi,00000000
00412B89: 7459                      jz 00412BE4
00412B8B: 03B5D3B24000              add esi,dword ptr [ebp+0040B2D3]
00412B91: 668B3E                    mov di,word ptr [esi]
00412B94: 663B7E02                  cmp di,word ptr [esi+02]
00412B98: 744A                      jz 00412BE4
00412B9A: 8B3E                      mov edi,dword ptr [esi]
00412B9C: 83FF00                    cmp edi,00000000
00412B9F: 7443                      jz 00412BE4
00412BA1: 03BDD3B24000              add edi,dword ptr [ebp+0040B2D3]
00412BA7: 8B4E04                    mov ecx,dword ptr [esi+04]
00412BAA: 83E908                    sub ecx,00000008
00412BAD: 83F900                    cmp ecx,00000000
00412BB0: 7432                      jz 00412BE4
00412BB2: D1E9                      shr ecx,1
00412BB4: 83C608                    add esi,00000008
00412BB7: 668B06                    mov ax,word ptr [esi]
00412BBA: 662500F0                  and ax,F000
00412BBE: 663D0030                  cmp ax,3000
00412BC2: 7519                      jnz 00412BDD
00412BC4: 668B06                    mov ax,word ptr [esi]
00412BC7: 6625FF0F                  and ax,0FFF
00412BCB: 8B9D79B44000              mov ebx,dword ptr [ebp+0040B479]
00412BD1: 291C07                    sub dword ptr [edi+eax],ebx
00412BD4: 8B9DD3B24000              mov ebx,dword ptr [ebp+0040B2D3]
00412BDA: 011C07                    add dword ptr [edi+eax],ebx
00412BDD: 83C602                    add esi,00000002
00412BE0: E2D5                      loop 00412BB7
00412BE2: EBAD                      jmp 00412B91
00412BE4: 61                        popad
00412BE5: C3                        ret
```

----------------------------------------------------

```
00412BE6: 60                        pushad
```
