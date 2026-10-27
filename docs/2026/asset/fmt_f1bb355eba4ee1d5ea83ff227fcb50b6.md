
```
000137EE: 5F                        pop edi
000137EF: 5D                        pop ebp
000137F0: 9D                        popfd
000137F1: E991110000                jmp 00014987
```

### 子程序开始

***获取系统时间，不知道要干什么，不会是比较时间差来判断是否被SOFTICE跟踪吧***

```
000137F6: BEE1B94000                mov esi,0040B9E1
000137FB: 03F5                      add esi,ebp
000137FD: 56                        push esi
000137FE: FF959FBB4000              call dword ptr [ebp+0040BB9F]   <---GetSystemTime
00013804: 668B8585B44000            mov ax,word ptr [ebp+0040B485]
0001380B: 668B9D81B44000            mov bx,word ptr [ebp+0040B481]
00013812: 663D0000                  cmp ax,0000
00013816: 7430                      jz 00013848
00013818: 663985E3B94000            cmp word ptr [ebp+0040B9E3],ax
0001381F: 7209                      jb 0001382A
00013821: 66399DE1B94000            cmp word ptr [ebp+0040B9E1],bx
00013828: 7309                      jnb 00013833
0001382A: 66399DE1B94000            cmp word ptr [ebp+0040B9E1],bx
00013831: 7606                      jbe 00013839
00013833: 66B80000                  mov ax,0000
00013837: EB0D                      jmp 00013846
00013839: 66C7857DB440000100        mov word ptr [ebp+0040B47D],0001
00013842: 66B80100                  mov ax,0001
00013846: EB04                      jmp 0001384C
00013848: 66B80100                  mov ax,0001
0001384C: C3                        ret
```

### 子程序结束

### 子程序开始

```
0001384D: 60                        pushad
0001384E: 6683BD89B4400000          cmp word ptr [ebp+0040B489],0000
00013856: 750B                      jnz 00013863
00013858: 66C785FBAB40000100        mov word ptr [ebp+0040ABFB],0001
00013861: EB42                      jmp 000138A5
00013863: E8CD030000                call 00013C35   <--从注册表中获取信息
00013868: BE60B14000                mov esi,0040B160
0001386D: 03F5                      add esi,ebp
0001386F: 663D0100                  cmp ax,0001
00013873: 7405                      jz 0001387A
00013875: 66C7060000                mov word ptr [esi],0000
0001387A: 66833E00                  cmp word ptr [esi],0000
0001387E: 7417                      jz 00013897
00013880: 66FF0E                    dec word ptr [esi]
00013883: 66C785FBAB40000100        mov word ptr [ebp+0040ABFB],0001
0001388C: 66C7857DB440000100        mov word ptr [ebp+0040B47D],0001
00013895: EB09                      jmp 000138A0
00013897: 66C785FBAB40000000        mov word ptr [ebp+0040ABFB],0000
000138A0: E84C040000                call 00013CF1  <--获取磁盘和卷的信息
000138A5: 61                        popad
000138A6: 668B85FBAB4000            mov ax,word ptr [ebp+0040ABFB]
000138AD: C3                        ret
000138AE: 0000                      add byte ptr [eax],al
000138B0: E8BB000000                call 00013970
000138B5: 8B85A8AD4000              mov eax,dword ptr [ebp+0040ADA8]
000138BB: BBB0AD4000                mov ebx,0040ADB0
000138C0: 03DD                      add ebx,ebp
000138C2: B983B14000                mov ecx,0040B183
000138C7: 03CD                      add ecx,ebp
000138C9: 50                        push eax
000138CA: 53                        push ebx
000138CB: 51                        push ecx
000138CC: FF9504BC4000              call dword ptr [ebp+0040BC04]
000138D2: 83C40C                    add esp,0000000C
000138D5: B865B14000                mov eax,0040B165
000138DA: 03C5                      add eax,ebp
000138DC: BB79B14000                mov ebx,0040B179
000138E1: 03DD                      add ebx,ebp
000138E3: B983B14000                mov ecx,0040B183
000138E8: 03CD                      add ecx,ebp
000138EA: 51                        push ecx
000138EB: 53                        push ebx
000138EC: 50                        push eax
000138ED: FF95BCBC4000              call dword ptr [ebp+0040BCBC]
000138F3: E8F9030000                call 00013CF1
000138F8: E873000000                call 00013970
000138FD: C3                        ret
```

### 子程序

```
000138FE: 6683BD49B740007C          cmp word ptr [ebp+0040B749],007C
00013906: 7505                      jnz 0001390D
00013908: 66B80100                  mov ax,0001
0001390C: C3                        ret
0001390D: B89FAC4000                mov eax,0040AC9F
00013912: 03C5                      add eax,ebp
00013914: 50                        push eax
00013915: FF95B8BC4000              call dword ptr [ebp+0040BCB8]
0001391B: BE49B74000                mov esi,0040B749
00013920: 03F5                      add esi,ebp
00013922: BF9FAC4000                mov edi,0040AC9F
00013927: 03FD                      add edi,ebp
00013929: 668B8567B44000            mov ax,word ptr [ebp+0040B467]
00013930: 2020                      and byte ptr [eax],ah
00013932: 2020                      and byte ptr [eax],ah
00013934: 2038                      and byte ptr [eax],bh
00013936: 3530343400                xor eax,00343430
0001393B: 0000                      add byte ptr [eax],al
0001393D: 0000                      add byte ptr [eax],al
0001393F: 0000                      add byte ptr [eax],al
00013920: 03F5                      add esi,ebp
00013922: BF9FAC4000                mov edi,0040AC9F
00013927: 03FD                      add edi,ebp
00013929: 668B8567B44000            mov ax,word ptr [ebp+0040B467]
00013930: 8A26                      mov ah,byte ptr [esi]
00013932: 3A27                      cmp ah,byte ptr [edi]
00013934: 7509                      jnz 0001393F
00013936: 80FC00                    cmp ah,00
00013939: 7409                      jz 00013944
0001393B: 46                        inc esi
0001393C: 47                        inc edi
0001393D: EBF1                      jmp 00013930
0001393F: 66B80000                  mov ax,0000
00013943: C3                        ret
00013944: 66C7857DB440000100        mov word ptr [ebp+0040B47D],0001
0001394D: 66B80100                  mov ax,0001
00013951: C3                        ret
```

### 子程序

***非代码区***

```
00013952: 0000                      add byte ptr [eax],al
00013954: 0000                      add byte ptr [eax],al
00013956: 0000                      add byte ptr [eax],al
00013958: 0000                      add byte ptr [eax],al
0001395A: 0000                      add byte ptr [eax],al
0001395C: 0000                      add byte ptr [eax],al
0001395E: 0000                      add byte ptr [eax],al
00013960: 0000                      add byte ptr [eax],al
00013962: 0000                      add byte ptr [eax],al
00013964: 0000                      add byte ptr [eax],al
00013966: 0000                      add byte ptr [eax],al
00013968: 0000                      add byte ptr [eax],al
0001396A: 0000                      add byte ptr [eax],al
0001396C: 0000                      add byte ptr [eax],al
0001396E: 0000                      add byte ptr [eax],al
```

***非代码区结束***

### 子程序开始

***获取C盘的卷信息***

```
00013970: B810000000                mov eax,00000010
00013975: 50                        push eax
00013976: B8B8AD4000                mov eax,0040ADB8
0001397B: 03C5                      add eax,ebp
0001397D: 50                        push eax
0001397E: B8C8AD4000                mov eax,0040ADC8
00013983: 03C5                      add eax,ebp
00013985: 50                        push eax
00013986: B8CCAD4000                mov eax,0040ADCC
0001398B: 03C5                      add eax,ebp
0001398D: 50                        push eax
0001398E: B8D0AD4000                mov eax,0040ADD0
00013993: 03C5                      add eax,ebp
00013995: 50                        push eax
```
