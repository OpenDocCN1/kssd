
# 汇编代码分析与修改

## 主代码段

```
00B85D79     8B43 08             mov eax,dword ptr ds:[ebx+8]  
00B85D7C     8B30                mov esi,dword ptr ds:[eax]  
00B85D7E     8343 08 04          add dword ptr ds:[ebx+8],4  
00B85D82     8B43 08             mov eax,dword ptr ds:[ebx+8]  
00B85D85     8B28                mov ebp,dword ptr ds:[eax]  
00B85D87     8343 08 04          add dword ptr ds:[ebx+8],4  
00B85D8B     8B43 08             mov eax,dword ptr ds:[ebx+8]  
00B85D8E     8B00                mov eax,dword ptr ds:[eax]  
00B85D90     894424 2C           mov dword ptr ss:[esp+2C],eax  
00B85D94     8343 08 04          add dword ptr ds:[ebx+8],4  
00B85D98     837B 30 00          cmp dword ptr ds:[ebx+30],0  
00B85D9C     75 0A               jnz short 00B85DA8  
00B85D9E     68 505FB800         push 0B85F50      ; ASCII "81"  
00B85DA3     E8 ECE0FFFF         call 00B83E94  
00B85DA8     8D5424 30           lea edx,dword ptr ss:[esp+30]  
00B85DAC     8BC3                mov eax,ebx  
00B85DAE     E8 51F8FFFF         call 00B85604  
00B85DB3     8B4424 2C           mov eax,dword ptr ss:[esp+2C]  
00B85DB7     50                  push eax  
00B85DB8     8D4424 34           lea eax,dword ptr ss:[esp+34]  
00B85DBC     50                  push eax  
00B85DBD     55                  push ebp  
00B85DBE     A1 7C25B900         mov eax,dword ptr ds:[B9257C]  
00B85DC3     8B00                mov eax,dword ptr ds:[eax]  
00B85DC5     50                  push eax  
00B85DC6     56                  push esi  
00B85DC7     8B4424 28           mov eax,dword ptr ss:[esp+28]  
00B85DCB     50                  push eax  
00B85DCC     FF53 30             call dword ptr ds:[ebx+30]  
00B85DCF     E9 3BFDFFFF         jmp 00B85B0F  
```

`//修改④：jmp 00B99A36 ★ 跳到 Patch 部分`

```
00B85DD4     8B43 08             mov eax,dword ptr ds:[ebx+8]  
00B85DD7     8B00                mov eax,dword ptr ds:[eax]  
00B85DD9     890424              mov dword ptr ss:[esp],eax  
00B85DDC     8343 08 04          add dword ptr ds:[ebx+8],4  
00B85DE0     8B43 08             mov eax,dword ptr ds:[ebx+8]  
00B85DE3     66:8B00             mov ax,word ptr ds:[eax]  
00B85DE6     66:894424 04        mov word ptr ss:[esp+4],ax  
00B85DEB     8343 08 02          add dword ptr ds:[ebx+8],2  
00B85DEF     8B0C24              mov ecx,dword ptr ss:[esp]  
00B85DF2     66:8B5424 04        mov dx,word ptr ss:[esp+4]  
00B85DF7     8B43 08             mov eax,dword ptr ds:[ebx+8]  
00B85DFA     E8 A1E3FFFF         call 00B841A0  
00B85DFF     8B4424 10           mov eax,dword ptr ss:[esp+10]  
00B85E03     E8 3CC7FEFF         call 00B72544  
00B85E08     894424 0C           mov dword ptr ss:[esp+C],eax  
00B85E0C     8B43 08             mov eax,dword ptr ds:[ebx+8]  
00B85E0F     50                  push eax  
00B85E10     8B4424 18           mov eax,dword ptr ss:[esp+18]  
00B85E14     50                  push eax  
00B85E15     A1 C026B900         mov eax,dword ptr ds:[B926C0]  
00B85E1A     8B00                mov eax,dword ptr ds:[eax]  
00B85E1C     FFD0                call eax  
00B85E1E     8BE8                mov ebp,eax  
00B85E20     85ED                test ebp,ebp  
00B85E22     75 0C               jnz short 00B85E30  
00B85E24     68 605FB800         push 0B85F60     ; ASCII "250"  
00B85E29     E8 66E0FFFF         call 00B83E94  
00B85E2E     EB 15               jmp short 00B85E45  
00B85E30     A1 F425B900         mov eax,dword ptr ds:[B925F4]  
00B85E35     3B28                cmp ebp,dword ptr ds:[eax]  
00B85E37     75 0C               jnz short 00B85E45  
00B85E39     837B 34 00          cmp dword ptr ds:[ebx+34],0  
00B85E3D     74 06               je short 00B85E45  
00B85E3F     8B6B 34             mov ebp,dword ptr ds:[ebx+34]  
00B85E42     036B 40             add ebp,dword ptr ds:[ebx+40]  
00B85E45     8B4424 0C           mov eax,dword ptr ss:[esp+C]  
00B85E49     8928                mov dword ptr ds:[eax],ebp  
00B85E4B     8B4424 0C           mov eax,dword ptr ss:[esp+C]  
```

`//修改⑤：jmp 00B999FE ★ 跳到 Patch 部分`

```
00B85E4F     8906                mov dword ptr ds:[esi],eax  
00B85E51     0FB74424 04         movzx eax,word ptr ss:[esp+4]  
00B85E56     0143 08             add dword ptr ds:[ebx+8],eax  
00B85E59     E9 B1FCFFFF         jmp 00B85B0F  
```

☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆

## Patch 处理④

```
00B99A36     51                  push ecx  
00B99A37     52                  push edx  
00B99A38     3E:8B5424 1C        mov edx,dword ptr ds:[esp+1C]  
00B99A3D     3B15 C0ACB900       cmp edx,dword ptr ds:[B9ACC0]  
00B99A43     74 0D               je short 00B99A52  
00B99A45     8915 C0ACB900       mov dword ptr ds:[B9ACC0],edx  
00B99A4B     8305 C8ACB900 04    add dword ptr ds:[B9ACC8],4  
00B99A52     8B0D C8ACB900       mov ecx,dword ptr ds:[B9ACC8]  
00B99A58     3E:8B5424 E4        mov edx,dword ptr ds:[esp-1C]  
00B99A5D     8911                mov dword ptr ds:[ecx],edx  
00B99A5F     3E:8B5424 C0        mov edx,dword ptr ds:[esp-40]  
```

`//用 VolX 兄的方法，呵呵`

`//[esp-40]和[esp-3C]要看你本机的代码来确定 ★`

```
00B99A64     3E:035424 C4        add edx,dword ptr ds:[esp-3C]  
00B99A69     81C2 00004000       add edx,400000  
00B99A6F     890A                mov dword ptr ds:[edx],ecx  
00B99A71     8305 C8ACB900 04    add dword ptr ds:[B9ACC8],4  
00B99A78     5A                  pop edx  
00B99A79     59                  pop ecx  
00B99A7A     E9 90C0FEFF         jmp 00B85B0F  
```

☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆

## Patch 处理⑤

```
00B999FE     51                  push ecx  
00B999FF     52                  push edx  
00B99A00     3E:8B5424 1C        mov edx,dword ptr ds:[esp+1C]  
00B99A05     3B15 C0ACB900       cmp edx,dword ptr ds:[B9ACC0]  
00B99A0B     74 0D               je short 00B99A1A  
00B99A0D     8915 C0ACB900       mov dword ptr ds:[B9ACC0],edx  
00B99A13     8305 C8ACB900 04    add dword ptr ds:[B9ACC8],4  
00B99A1A     8B0D C8ACB900       mov ecx,dword ptr ds:[B9ACC8]  
00B99A20     8929                mov dword ptr ds:[ecx],ebp  
00B99A22     890E                mov dword ptr ds:[esi],ecx  
00B99A24     8305 C8ACB900 04    add dword ptr ds:[B9ACC8],4  
00B99A2B     5A                  pop edx  
00B99A2C     59                  pop ecx  
00B99A2D     E9 1FC4FEFF         jmp 00B85E51  
```

————————————————————————

## 断点设置

修改完以上部分后在`00B85F27`处下断，`Shift+F9`中断

```
00B85F20     83C4 38             add esp,38  
00B85F23     5D                  pop ebp  
00B85F24     5F                  pop edi  
00B85F25     5E                  pop esi  
00B85F26     5B                  pop ebx  
00B85F27     C3                  retn  
```

`//中断在这里后 IAT 处理完毕！`

—————————————————————————————————

## 三、附注：看看 AL=3 的 00B85E7E 处三个分支处理

### 1、0B846B8 是 GetProcAddress

```
00B84FB4     53                  push ebx  
00B84FB5     56                  push esi  
00B84FB6     8BF2                mov esi,edx  
00B84FB8     8BD8                mov ebx,eax  
00B84FBA     FEC9                dec cl  
00B84FBC     74 06               je short 00B84FC4  
00B84FBE     FEC9                dec cl  
00B84FC0     74 0C               je short 00B84FCE  
00B84FC2     EB 2B               jmp short 00B84FEF  
00B84FC4     B8 B846B800         mov eax,0B846B8  
00B84FC9     8903                mov dword ptr ds:[ebx],eax  
00B84FCB     5E                  pop esi  
00B84FCC     5B                  pop ebx  
00B84FCD     C3                  retn  

00B846B8     55                  push ebp  
00B846B9     8BEC                mov ebp,esp  
00B846BB     8B55 0C             mov edx,dword ptr ss:[ebp+C]  
00B846BE     8B45 08             mov eax,dword ptr ss:[ebp+8]  
00B846C1     3B05 F823B900       cmp eax,dword ptr ds:[B923F8]  
00B846C7     75 09               jnz short 00B846D2  
```
