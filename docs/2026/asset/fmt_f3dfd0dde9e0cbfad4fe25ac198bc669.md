
## 判断是否是缺省NAG提示信息

```
0001362E: 0F8444010000              jz 00013778
00013634: 6681BDF1B440007C7C        cmp word ptr [ebp+0040B4F1],7C7C
0001363D: 741A                      jz 00013659    <--相等，就跳走
0001363F: B8BEB94000                mov eax,0040B9BE
00013644: 03C5                      add eax,ebp
00013646: BBF1B44000                mov ebx,0040B4F1
0001364B: 03DD                      add ebx,ebp
0001364D: 6A30                      push 00000030
0001364F: 50                        push eax
00013650: 2020                      and byte ptr [eax],ah
00013652: 2020                      and byte ptr [eax],ah
00013654: 2038                      and byte ptr [eax],bh
```

## 显示用户加的NAG提示信息窗口

```
00013656: 3530343400                xor eax,00343430
0001365B: 0000                      add byte ptr [eax],al
0001365D: 0000                      add byte ptr [eax],al
0001365F: 0000                      add byte ptr [eax],al
00013640: BEB9400003                mov esi,030040B9
00013645: C5BBF1B44000              lds edi,pword ptr [ebx+0040B4F1]
0001364B: 03DD                      add ebx,ebp
0001364D: 6A30                      push 00000030
0001364F: 50                        push eax
00013650: 53                        push ebx
00013651: 6A00                      push 00000000           <--欢迎 ,^-^
00013653: FF9500BC4000              call dword ptr [ebp+0040BC00] <--MessageBoxA
00013659: E8A0020000                call 000138FE   <--跳到此处
0001365E: BB91B84000                mov ebx,0040B891
00013663: 03DD                      add ebx,ebp
00013665: 663D0000                  cmp ax,0000
00013669: 0F8409010000              jz 00013778   <--等于0，就跳走
0001366F: E882010000                call 000137F6   <--获取系统时间
00013674: BB1DB64000                mov ebx,0040B61D
00013679: 03DD                      add ebx,ebp
0001367B: 663D0000                  cmp ax,0000
0001367F: 0F84F3000000              jz 00013778
00013685: E8C3010000                call 0001384D   <--注册表和磁盘
0001368A: BB1DB64000                mov ebx,0040B61D
0001368F: 03DD                      add ebx,ebp
00013691: 663D0000                  cmp ax,0000
00013695: 0F84DD000000              jz 00013778
0001369B: 6683BD7DB4400000          cmp word ptr [ebp+0040B47D],0000
000136A3: 751F                      jnz 000136C4
000136A5: 83BD5DB7400000            cmp dword ptr [ebp+0040B75D],00000000
000136AC: 7416                      jz 000136C4
000136AE: 6683BD7FB4400001          cmp word ptr [ebp+0040B47F],0001
000136B6: 740C                      jz 000136C4
000136B8: BBC9B24000                mov ebx,0040B2C9
000136BD: 03DD                      add ebx,ebp
000136BF: E9B4000000                jmp 00013778
000136C4: BED7B24000                mov esi,0040B2D7
000136C9: 03F5                      add esi,ebp
000136CB: 833E00                    cmp dword ptr [esi],00000000   <---此处循环将还原各Section
000136CE: 746F                      jz 0001373F               <--是否循环结束
000136D0: 8B9DD3B24000              mov ebx,dword ptr [ebp+0040B2D3]
000136D6: 031E                      add ebx,dword ptr [esi]
000136D8: 8B4E04                    mov ecx,dword ptr [esi+04]
000136DB: 83F900                    cmp ecx,00000000
000136DE: 7505                      jnz 000136E5
000136E0: 83C608                    add esi,00000008
000136E3: EBE6                      jmp 000136CB
000136E5: D1E9                      shr ecx,1
000136E7: 668B8567B44000            mov ax,word ptr [ebp+0040B467]
000136EE: 66F7D0                    not ax
000136F1: 663103                    xor word ptr [ebx],ax
000136F4: 6648                      dec ax
000136F6: 66813B3F3F                cmp word ptr [ebx],3F3F
000136FB: 7518                      jnz 00013715
000136FD: 817BFC44425045            cmp dword ptr [ebx-04],45504244
00013704: 750F                      jnz 00013715
00013706: 6683BD7DB4400001          cmp word ptr [ebp+0040B47D],0001
0001370E: 7505                      jnz 00013715
00013710: 66C7032B2B                mov word ptr [ebx],2B2B
00013715: 66817BFF3F3F              cmp word ptr [ebx-01],3F3F
0001371B: 7519                      jnz 00013736
0001371D: 817BFB44425045            cmp dword ptr [ebx-05],45504244
00013724: 7510                      jnz 00013736
00013726: 6683BD7DB4400001          cmp word ptr [ebp+0040B47D],0001
0001372E: 7506                      jnz 00013736
00013730: 66C743FF2B2B              mov word ptr [ebx-01],2B2B
00013736: 43                        inc ebx
00013737: 43                        inc ebx
00013738: E2B7                      loop 000136F1
0001373A: 83C608                    add esi,00000008
0001373D: EB8C                      jmp 000136CB              <---大循环
0001373F: 90                        nop
00013740: 89AD9FBE4000              mov dword ptr [ebp+0040BE9F],ebp
00013746: E84C030000                call 00013A97
0001374B: E82D040000                call 00013B7D
00013750: 8B85CFB24000              mov eax,dword ptr [ebp+0040B2CF]
00013756: 8B9DD3B24000              mov ebx,dword ptr [ebp+0040B2D3]
0001375C: 03C3                      add eax,ebx
0001375E: 8985F6B94000              mov dword ptr [ebp+0040B9F6],eax
00013764: 8BC5                      mov eax,ebp
00013766: 5B                        pop ebx
00013767: 59                        pop ecx
00013768: 5A                        pop edx
00013769: 5E                        pop esi
0001376A: 5F                        pop edi
0001376B: 5D                        pop ebp
0001376C: 9D                        popfd
0001376D: FFB0F6B94000              push dword ptr [eax+0040B9F6]
00013773: E90F120000                jmp 00014987         <---跳到后面处理入口处的代码
00013778: 66813B7C7C                cmp word ptr [ebx],7C7C
0001377D: 7413                      jz 00013792
0001377F: B8C0B94000                mov eax,0040B9C0
00013784: 03C5                      add eax,ebp
00013786: 6A30                      push 00000030
00013788: 50                        push eax
00013789: 53                        push ebx
0001378A: 6A00                      push 00000000             <---显示dial.dll没有找到
0001378C: FF9500BC4000              call dword ptr [ebp+0040BC00]   <--MessageBoxA
00013792: 83BD5DB7400000            cmp dword ptr [ebp+0040B75D],00000000
00013799: 7446                      jz 000137E1
0001379B: E810010000                call 000138B0
000137A0: 6689857DB44000            mov word ptr [ebp+0040B47D],ax
000137A7: 50                        push eax
000137A8: 663D0100                  cmp ax,0001
000137AC: 7507                      jnz 000137B5
000137AE: BBF5B84000                mov ebx,0040B8F5
000137B3: EB05                      jmp 000137BA
000137B5: BB59B94000                mov ebx,0040B959
000137BA: 03DD                      add ebx,ebp
000137BC: 66813B7C7C                cmp word ptr [ebx],7C7C
000137C1: 7413                      jz 000137D6
000137C3: B8C0B94000                mov eax,0040B9C0
000137C8: 03C5                      add eax,ebp
000137CA: 6A30                      push 00000030
000137CC: 50                        push eax
000137CD: 53                        push ebx
000137CE: 6A00                      push 00000000
000137D0: FF9500BC4000              call dword ptr [ebp+0040BC00]   <--MessageBoxA
000137D6: 58                        pop eax
000137D7: 663D0100                  cmp ax,0001
000137DB: 0F84E3FEFFFF              jz 000136C4
000137E1: C6855DBD400000            mov byte ptr [ebp+0040BD5D],00
000137E8: 8BC5                      mov eax,ebp
000137EA: 5B                        pop ebx
000137EB: 59                        pop ecx
000137EC: 5A                        pop edx
000137ED: 5E                        pop esi
```
