
```
00412B67: 8A1E                  mov bl,byte ptr [esi]  
00412B69: 3A1F                  cmp bl,byte ptr [edi]  
00412B6B: 750C                  jnz 00412B79  
00412B6D: 46                    inc esi  
00412B6E: 47                    inc edi  
00412B6F: 80FB00                cmp bl,00  
00412B72: 75F3                  jnz 00412B67  
00412B74: B801000000            mov eax,00000001  
00412B79: 5F                    pop edi  
00412B7A: 5E                    pop esi  
00412B7B: 5B                    pop ebx  
00412B7C: C3                    ret  
---------------------------------------------------
```

```
00412B7D: 60                    pushad   
00412B7E: 33C0                  xor eax,eax  
00412B80: 8BB56DB44000          mov esi,dword ptr [ebp+0040B46D]  
00412B86: 83FE00                cmp esi,00000000  
00412B89: 7459                  jz 00412BE4  
00412B8B: 03B5D3B24000          add esi,dword ptr [ebp+0040B2D3]  
00412B91: 668B3E                mov di,word ptr [esi]  
00412B94: 663B7E02              cmp di,word ptr [esi+02]  
00412B98: 744A                  jz 00412BE4  
00412B9A: 8B3E                  mov edi,dword ptr [esi]  
00412B9C: 83FF00                cmp edi,00000000  
00412B9F: 7443                  jz 00412BE4  
00412BA1: 03BDD3B24000          add edi,dword ptr [ebp+0040B2D3]  
00412BA7: 8B4E04                mov ecx,dword ptr [esi+04]  
00412BAA: 83E908                sub ecx,00000008  
00412BAD: 83F900                cmp ecx,00000000  
00412BB0: 7432                  jz 00412BE4  
00412BB2: D1E9                  shr ecx,1  
00412BB4: 83C608                add esi,00000008  
00412BB7: 668B06                mov ax,word ptr [esi]  
00412BBA: 662500F0              and ax,F000  
00412BBE: 663D0030              cmp ax,3000  
00412BC2: 7519                  jnz 00412BDD  
00412BC4: 668B06                mov ax,word ptr [esi]  
00412BC7: 6625FF0F              and ax,0FFF  
00412BCB: 8B9D79B44000          mov ebx,dword ptr [ebp+0040B479]  
00412BD1: 291C07                sub dword ptr [edi+eax],ebx  
00412BD4: 8B9DD3B24000          mov ebx,dword ptr [ebp+0040B2D3]  
00412BDA: 011C07                add dword ptr [edi+eax],ebx  
00412BDD: 83C602                add esi,00000002  
00412BE0: E2D5                  loop 00412BB7  
00412BE2: EBAD                  jmp 00412B91  
00412BE4: 61                    popad   
00412BE5: C3                    ret  
----------------------------------------------------
```

```
00412BE6: 60                    pushad   
00412BE7: 8BB571B44000          mov esi,dword ptr [ebp+0040B471]  
00412BED: 83FE00                cmp esi,00000000  
00412BF0: 7432                  jz 00412C24  
00412BF2: 03B5D3B24000          add esi,dword ptr [ebp+0040B2D3]  
00412BF8: 8B8D75B44000          mov ecx,dword ptr [ebp+0040B475]  
00412BFE: 83F900                cmp ecx,00000000  
00412C01: 7421                  jz 00412C24  
00412C03: 8B06                  mov eax,dword ptr [esi]  
00412C05: 83F800                cmp eax,00000000  
00412C08: 7412                  jz 00412C1C  
00412C0A: 2B8579B44000          sub eax,dword ptr [ebp+0040B479]  
00412C10: 2020                  and byte ptr [eax],ah  
00412C12: 2020                  and byte ptr [eax],ah  
00412C14: 2032                  and byte ptr [edx],dh  
00412C16: 37                    aaa  
00412C17: 3133                  xor dword ptr [ebx],esi  
00412C19: 3800                  cmp byte ptr [eax],al  
00412C1B: 0000                  add byte ptr [eax],al  
00412C1D: 0000                  add byte ptr [eax],al  
00412C1F: 0000                  add byte ptr [eax],al  
00412C00: 0074218B              add byte ptr [ecx-75],dh  
00412C04: 06                    push es  
00412C05: 83F800                cmp eax,00000000  
00412C08: 7412                  jz 00412C1C  
00412C0A: 2B8579B44000          sub eax,dword ptr [ebp+0040B479]  
00412C10: 0385D3B24000          add eax,dword ptr [ebp+0040B2D3]  
00412C16: C70000000000          mov dword ptr [eax],00000000  
00412C1C: 83C604                add esi,00000004  
00412C1F: 83E904                sub ecx,00000004  
00412C22: 75DF                  jnz 00412C03  
00412C24: 61                    popad   
00412C25: C3                    ret  
---------------------显示 NAG 窗口------------------------
```

```
00412C26: B865B74000            mov eax,0040B765  
00412C2B: 03C5                  add eax,ebp          <--413418
```

```
*************

欢迎你使用幻影 1.5 beta2，
幻影 v1.5 是共享软件，如果你喜欢它，希望你注册。 

E-mail   : ding-boy@netease.com 
Homepage : http://fsdb.yeah.net 

*************
```

```
00412C2D: 50                    push eax  
00412C2E: FF95C0BC4000          call dword ptr [ebp+0040BCC0]  <--ShowTryWindow  
00412C34: C3                    ret  
---------------------------
```

```
----------------查寻注册键，找到则进行还原并且返回 1，否则返回 0-----------

00412C35: 60                    pushad   
00412C36: 66C7855EB140000000    mov word ptr [ebp+0040B15E],0000  <--412E11  
00412C3F: B848B14000            mov eax,0040B148  
00412C44: 03C5                  add eax,ebp          <--412DFB  
00412C46: BB40B14000            mov ebx,0040B140  
00412C4B: 03DD                  add ebx,ebp          <--412DF3  
00412C4D: B91EB14000            mov ecx,0040B11E  
00412C52: 03CD                  add ecx,ebp          <--412DD1  
```

`wrifile\shell\open\command\config`

```
00412C54: 50                    push eax  
00412C55: 53                    push ebx  
00412C56: 683F001F00            push 001F003F  
00412C5B: 6A00                  push 00000000  
00412C5D: 6A00                  push 00000000  
00412C5F: 6A00                  push 00000000  
00412C61: 6A00                  push 00000000  
00412C63: 51                    push ecx  
00412C64: 6800000080            push 80000000  
00412C69: FF9576BC4000          call dword ptr [ebp+0040BC76]   <--RegCreateKeyA  
00412C6F: 663D0000              cmp ax,0000  
00412C73: 7566                  jnz 00412CDB  
00412C75: BE1AB14000            mov esi,0040B11A           
00412C7A: 03F5                  add esi,ebp          <--412DCD  
00412C7C: BF16B14000            mov edi,0040B116  
00412C81: 03FD                  add edi,ebp          <--  
00412C83: B860B14000            mov eax,0040B160  
00412C88: 03C5                  add eax,ebp          <--412E13  
00412C8A: BB50B14000            mov ebx,0040B150  
00412C8F: 03DD                  add ebx,ebp          <--412E03  
```

`91 80 88 80 B8 80 99 80 EE 82`

```
00412C91: 8B8D40B14000          mov ecx,dword ptr [ebp+0040B140]  
00412C97: 56                    push esi           <--412DCD "="  
00412C98: 50                    push eax           <--412E13  
00412C99: 57                    push edi  
```
