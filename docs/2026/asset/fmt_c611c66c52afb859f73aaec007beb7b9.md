
```
0127AAC6     0F85 A6020000        jnz 0127AD72                     ; 如果是第2种jmp xxx，从这里跳走
0127AACC     836D FC 04           sub dword ptr ss:[ebp-4],4
0127AAD0     8B45 F8              mov eax,dword ptr ss:[ebp-8]
0127AAD3     8B80 E0000000        mov eax,dword ptr ds:[eax+E0]
0127AAD9     0345 F0              add eax,dword ptr ss:[ebp-10]
0127AADC     0345 F4              add eax,dword ptr ss:[ebp-C]
0127AADF     8B55 FC              mov edx,dword ptr ss:[ebp-4]
0127AAE2     8902                 mov dword ptr ds:[edx],eax
0127AAE4     E9 89020000          jmp 0127AD72
```

## 第3种：`edx+4F`的情况，模拟`add x,n`

```
0127AAE9     8B55 F8              mov edx,dword ptr ss:[ebp-8]
0127AAEC     3A42 4F              cmp al,byte ptr ds:[edx+4F]          ; 第3种，add x,n
0127AAEF     75 6F                jnz short 0127AB60
0127AAF1     33C0                 xor eax,eax
0127AAF3     8A46 05              mov al,byte ptr ds:[esi+5]
0127AAF6     8D0440               lea eax,dword ptr ds:[eax+eax*2]
0127AAF9     8B55 F8              mov edx,dword ptr ss:[ebp-8]
0127AAFC     8B5482 68            mov edx,dword ptr ds:[edx+eax*4+68]
0127AB00     8BC3                 mov eax,ebx
0127AB02     FFD2                 call edx               //(x) 0=eax,1=ecx,2=edx,3=ebx,4=esp,5=ebp,6=esi,7=edi
0127AB04     8845 EB              mov byte ptr ss:[ebp-15],al
0127AB07     EB 01                jmp short 0127AB0A
0127AB0A     33C0                 xor eax,eax
0127AB0C     8A46 07              mov al,byte ptr ds:[esi+7]
0127AB0F     8D0440               lea eax,dword ptr ds:[eax+eax*2]
0127AB12     8B55 F8              mov edx,dword ptr ss:[ebp-8]
0127AB15     8B5482 68            mov edx,dword ptr ds:[edx+eax*4+68]
0127AB19     8BC3                 mov eax,ebx
0127AB1B     FFD2                 call edx                //返回值是n
0127AB1D     8BD8                 mov ebx,eax
0127AB1F     EB 01                jmp short 0127AB22
```

## 第4种：`edx+50`的情况，模拟`MOV xxx1,xxx2`

```
0127AB60     8B55 F8              mov edx,dword ptr ss:[ebp-8]
0127AB63     3A42 50              cmp al,byte ptr ds:[edx+50]    ; 第4种,mov xxx1,xxx2
0127AB66     75 63                jnz short 0127ABCB
0127AB68     33C0                 xor eax,eax
0127AB6A     8A46 05              mov al,byte ptr ds:[esi+5]
0127AB6D     8D0440               lea eax,dword ptr ds:[eax+eax*2]
0127AB70     8B55 F8              mov edx,dword ptr ss:[ebp-8]
0127AB73     8B5482 68            mov edx,dword ptr ds:[edx+eax*4+68]
0127AB77     8BC3                 mov eax,ebx
0127AB79     FFD2                 call edx           //0=eax,1=ecx,2=edx,3=ebx,4=esp,5=ebp,6=esi,7=edi(xxx1)
0127AB7B     8845 EB              mov byte ptr ss:[ebp-15],al
0127AB7E     EB 01                jmp short 0127AB81
0127AB81     33C0                 xor eax,eax
0127AB83     8A46 06              mov al,byte ptr ds:[esi+6]
0127AB86     8D0440               lea eax,dword ptr ds:[eax+eax*2]
0127AB89     8B55 F8              mov edx,dword ptr ss:[ebp-8]
0127AB8C     8B5482 68            mov edx,dword ptr ds:[edx+eax*4+68]
0127AB90     8BC3                 mov eax,ebx
0127AB92     FFD2                 call edx           //0=eax,1=ecx,2=edx,3=ebx,4=esp,5=ebp,6=esi,7=edi(xxx2)
0127AB94     8845 EA              mov byte ptr ss:[ebp-16],al
0127AB97     8A4D EA              mov cl,byte ptr ss:[ebp-16]
0127AB9A     8B55 10              mov edx,dword ptr ss:[ebp+10]
0127AB9D     8B45 F8              mov eax,dword ptr ss:[ebp-8]
0127ABA0     E8 3B020000          call 0127ADE0
0127ABA5     EB 01                jmp short 0127ABA8
```

## 第5种情况：`edx+51`的情况，模拟`MOV [n],xxx1`

```
0127ABCB     8B55 F8              mov edx,dword ptr ss:[ebp-8]
0127ABCE     3A42 51              cmp al,byte ptr ds:[edx+51]   ; 第5种情况，mov [n],xxx1
0127ABD1     75 5E                jnz short 0127AC31
0127ABD3     EB 01                jmp short 0127ABD6
0127ABD6     33C0                 xor eax,eax
0127ABD8     8A46 05              mov al,byte ptr ds:[esi+5]
0127ABDB     8D0440               lea eax,dword ptr ds:[eax+eax*2]
0127ABDE     8B55 F8              mov edx,dword ptr ss:[ebp-8]
0127ABE1     8B5482 68            mov edx,dword ptr ds:[edx+eax*4+68]
0127ABE5     8BC3                 mov eax,ebx
0127ABE7     FFD2                 call edx      //(xxx1)0=eax,1=ecx,2=edx,3=ebx,4=esp,5=ebp,6=esi,7=edi
0127ABE9     8845 EB              mov byte ptr ss:[ebp-15],al
0127ABEC     EB 01                jmp short 0127ABEF
0127ABEF     33C0                 xor eax,eax
0127ABF1     8A46 07              mov al,byte ptr ds:[esi+7]
0127ABF4     8D0440               lea eax,dword ptr ds:[eax+eax*2]
0127ABF7     8B55 F8              mov edx,dword ptr ss:[ebp-8]
0127ABFA     8B5482 68            mov edx,dword ptr ds:[edx+eax*4+68]
0127ABFE     8BC3                 mov eax,ebx
0127AC00     FFD2                 call edx
0127AC02     8BD8                 mov ebx,eax
0127AC04     8A4D EB              mov cl,byte ptr ss:[ebp-15]
0127AC07     8B55 10              mov edx,dword ptr ss:[ebp+10]
0127AC0A     8B45 F8              mov eax,dword ptr ss:[ebp-8]
0127AC0D     E8 CE010000          call 0127ADE0
0127AC12     8B55 F8              mov edx,dword ptr ss:[ebp-8]
0127AC15     035A 14              add ebx,dword ptr ds:[edx+14]     // <-ebx==n
0127AC18     8903                 mov dword ptr ds:[ebx],eax
0127AC1A     EB 01                jmp short 0127AC1D
```

## 第6种情况：`edx+52`的情况，模拟`mov [x+n],y`

```
0127AC31     8B55 F8              mov edx,dword ptr ss:[ebp-8]
0127AC34     3A42 52              cmp al,byte ptr ds:[edx+52]    ; 第6种情况，mov     [x+n],y
0127AC37     0F85 80000000        jnz 0127ACBD
0127AC3D     EB 01                jmp short 0127AC40
0127AC40     33C0                 xor eax,eax
0127AC42     8A46 05              mov al,byte ptr ds:[esi+5]
0127AC45     8D0440               lea eax,dword ptr ds:[eax+eax*2]
0127AC48     8B55 F8              mov edx,dword ptr ss:[ebp-8]
0127AC4B     8B5482 68            mov edx,dword ptr ds:[edx+eax*4+68]
0127AC4F     8BC3                 mov eax,ebx
0127AC51     FFD2                 call edx              // (X) 0=eax,1=ecx,2=edx,3=ebx,4=esp,5=ebp,6=esi,7=edi
0127AC53     8845 EB              mov byte ptr ss:[ebp-15],al
0127AC56     33C0                 xor eax,eax
0127AC58     8A46 06              mov al,byte ptr ds:[esi+6]
0127AC5B     8D0440               lea eax,dword ptr ds:[eax+eax*2]
0127AC5E     8B55 F8              mov edx,dword ptr ss:[ebp-8]
0127AC61     8B5482 68            mov edx,dword ptr ds:[edx+eax*4+68]
0127AC65     8BC3                 mov eax,ebx
0127AC67     FFD2                 call edx              // (Y) 0=eax,1=ecx,2=edx,3=ebx,4=esp,5=ebp,6=esi,7=edi
0127AC69     8845 EA              mov byte ptr ss:[ebp-16],al
0127AC6C     EB 01                jmp short 0127AC6F
0127AC6F     33C0                 xor eax,eax
0127AC71     8A46 07              mov al,byte ptr ds:[esi+7]
0127AC74     8D0440               lea eax,dword ptr ds:[eax+eax*2]
0127AC77     8B55 F8              mov edx,dword ptr ss:[ebp-8]
0127AC7A     8B5482 68            mov edx,dword ptr ds:[edx+eax*4+68]
0127AC7E     8BC3                 mov eax,ebx
0127AC80     FFD2                 call edx             //常数n
0127AC82     8BD8                 mov ebx,eax
0127AC84     8A4D EB              mov cl,byte ptr ss:[ebp-15]
0127AC87     8B55 10              mov edx,dword ptr ss:[ebp+10]
0127AC8A     8B45 F8              mov eax,dword ptr ss:[ebp-8]
0127AC8D     E8 4E010000          call 0127ADE0
0127AC92     8BF0                 mov esi,eax
0127AC94     8A4D EA              mov cl,byte ptr ss:[ebp-16]
0127AC97     8B55 10              mov edx,dword ptr ss:[ebp+10]
0127AC9A     8B45 F8              mov eax,dword ptr ss:[ebp-8]
0127AC9D     E8 3E010000          call 0127ADE0
0127ACA2     03F3                 add esi,ebx
0127ACA4     8906                 mov dword ptr ds:[esi],eax
0127ACA6     EB 01                jmp short 0127ACA9
```
