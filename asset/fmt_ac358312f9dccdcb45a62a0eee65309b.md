
### 第 6 种情况：`edx+52` 的情况，模拟 `mov [x+n],y`

```
0127AC15     035A 14              add ebx,dword ptr ds:[edx+14]     // <-ebx==n
0127AC18     8903                mov dword ptr ds:[ebx],eax
0127AC1A     EB 01               jmp short 0127AC1D

0127AC31     8B55 F8             mov edx,dword ptr ss:[ebp-8]
0127AC34     3A42 52             cmp al,byte ptr ds:[edx+52]    ; 第 6 种情况，mov [x+n],y
0127AC37     0F85 80000000       jnz 0127ACBD
0127AC3D     EB 01               jmp short 0127AC40

0127AC40     33C0                xor eax,eax
0127AC42     8A46 05             mov al,byte ptr ds:[esi+5]
0127AC45     8D0440              lea eax,dword ptr ds:[eax+eax*2]
0127AC48     8B55 F8             mov edx,dword ptr ss:[ebp-8]
0127AC4B     8B5482 68           mov edx,dword ptr ds:[edx+eax*4+68]
0127AC4F     8BC3                mov eax,ebx
0127AC51     FFD2                call edx                // (X) 0=eax,1=ecx,2=edx,3=ebx,4=esp,5=ebp,6=esi,7=edi
0127AC53     8845 EB             mov byte ptr ss:[ebp-15],al
0127AC56     33C0                xor eax,eax
0127AC58     8A46 06             mov al,byte ptr ds:[esi+6]
0127AC5B     8D0440              lea eax,dword ptr ds:[eax+eax*2]
0127AC5E     8B55 F8             mov edx,dword ptr ss:[ebp-8]
0127AC61     8B5482 68           mov edx,dword ptr ds:[edx+eax*4+68]
0127AC65     8BC3                mov eax,ebx
0127AC67     FFD2                call edx                // (Y) 0=eax,1=ecx,2=edx,3=ebx,4=esp,5=ebp,6=esi,7=edi
0127AC69     8845 EA             mov byte ptr ss:[ebp-16],al
0127AC6C     EB 01               jmp short 0127AC6F
0127AC6F     33C0                xor eax,eax
0127AC71     8A46 07             mov al,byte ptr ds:[esi+7]
0127AC74     8D0440              lea eax,dword ptr ds:[eax+eax*2]
0127AC77     8B55 F8             mov edx,dword ptr ss:[ebp-8]
0127AC7A     8B5482 68           mov edx,dword ptr ds:[edx+eax*4+68]
0127AC7E     8BC3                mov eax,ebx
0127AC80     FFD2                call edx               //常数 n
0127AC82     8BD8                mov ebx,eax
0127AC84     8A4D EB             mov cl,byte ptr ss:[ebp-15]
0127AC87     8B55 10             mov edx,dword ptr ss:[ebp+10]
0127AC8A     8B45 F8             mov eax,dword ptr ss:[ebp-8]
0127AC8D     E8 4E010000         call 0127ADE0
0127AC92     8BF0                mov esi,eax
0127AC94     8A4D EA             mov cl,byte ptr ss:[ebp-16]
0127AC97     8B55 10             mov edx,dword ptr ss:[ebp+10]
0127AC9A     8B45 F8             mov eax,dword ptr ss:[ebp-8]
0127AC9D     E8 3E010000         call 0127ADE0
0127ACA2     03F3                add esi,ebx
0127ACA4     8906                mov dword ptr ds:[esi],eax
0127ACA6     EB 01               jmp short 0127ACA9
```

### 第 7 种情况：`edx+4C` 的情况，模拟：`CMP x,y`；`jxx n`

```
0127ACBD     8B55 F8             mov edx,dword ptr ss:[ebp-8]
0127ACC0     3A42 4C             cmp al,byte ptr ds:[edx+4C]   ; 第 7 种情况，CMP x,y；jxx n
0127ACC3     75 67               jnz short 0127AD2C
0127ACC5     EB 01               jmp short 0127ACC8

0127ACC8     8BCB                mov ecx,ebx
0127ACCA     8B55 10             mov edx,dword ptr ss:[ebp+10]
0127ACCD     8B45 F8             mov eax,dword ptr ss:[ebp-8]
0127ACD0     E8 E7F6FFFF         call 0127A3BC                 ; 这里进入
----------------------------------------------------
0127A3EF     33C0                xor eax,eax
0127A3F1     8A46 07             mov al,byte ptr ds:[esi+7]
0127A3F4     8D0440              lea eax,dword ptr ds:[eax+eax*2]
0127A3F7     8B5483 68           mov edx,dword ptr ds:[ebx+eax*4+68]
0127A3FB     8BC7                mov eax,edi
0127A3FD     FFD2                call edx
0127A3FF     8945 F8             mov dword ptr ss:[ebp-8],eax
0127A402     33C0                xor eax,eax
0127A404     8A46 05             mov al,byte ptr ds:[esi+5]
0127A407     8D0440              lea eax,dword ptr ds:[eax+eax*2]
0127A40A     8B5483 68           mov edx,dword ptr ds:[ebx+eax*4+68]
0127A40E     8BC7                mov eax,edi
0127A410     FFD2                call edx           ; (x)0=eax,1=ecx,2=edx,3=ebx,4=esp,5=ebp,6=esi,7=edi
0127A412     8BD0                mov edx,eax
0127A414     80EA 08             sub dl,8
0127A417     0F92C2              setb dl
0127A41A     80FA 01             cmp dl,1
0127A41D     75 0F               jnz short 0127A42E
0127A41F     8BC8                mov ecx,eax
0127A421     8B55 FC             mov edx,dword ptr ss:[ebp-4]
0127A424     8BC3                mov eax,ebx
0127A426     E8 B5090000         call 0127ADE0
0127A42B     8945 EC             mov dword ptr ss:[ebp-14],eax
0127A42E     33C0                xor eax,eax
0127A430     8A46 09             mov al,byte ptr ds:[esi+9]
0127A433     8D0440              lea eax,dword ptr ds:[eax+eax*2]
0127A436     8B5483 68           mov edx,dword ptr ds:[ebx+eax*4+68]
0127A43A     8BC7                mov eax,edi
0127A43C     FFD2                call edx
0127A43E     8845 F3             mov byte ptr ss:[ebp-D],al
0127A441     EB 01               jmp short 0127A444
0127A444     33C0                xor eax,eax
0127A446     8A46 08             mov al,byte ptr ds:[esi+8]
0127A449     8D0440              lea eax,dword ptr ds:[eax+eax*2]
0127A44C     8B5483 68           mov edx,dword ptr ds:[ebx+eax*4+68]
0127A450     8BC7                mov eax,edi
0127A452     FFD2                call edx             ;  如果是 cmp x,n 情况，此处返回常数 n,否则为 0
0127A454     8945 F4             mov dword ptr ss:[ebp-C],eax
0127A457     33C0                xor eax,eax
0127A459     8A46 06             mov al,byte ptr ds:[esi+6]
0127A45C     8D0440              lea eax,dword ptr ds:[eax+eax*2]
0127A45F     8B5483 68           mov edx,dword ptr ds:[ebx+eax*4+68]
0127A463     8BC7                mov eax,edi
0127A465     FFD2                call edx           ; (y)0=eax,1=ecx,2=edx,3=ebx,4=esp,5=ebp,6=esi,7=edi
0127A467     8BD0                mov edx,eax
0127A469     80EA 08             sub dl,8
0127A46C     0F92C2              setb dl
0127A46F     80FA 01             cmp dl,1
0127A472     75 12               jnz short 0127A486
----------------------------------------------------
0127ACD5     8945 14             mov dword ptr ss:[ebp+14],eax
0127ACD8     EB 01               jmp short 0127ACDB

0127ACDB     33C0                xor eax,eax
0127ACDD     8A46 04             mov al,byte ptr ds:[esi+4]
0127ACE0     8D0440              lea eax,dword ptr ds:[eax+eax*2]
0127ACE3     8B55 F8             mov edx,dword ptr ss:[ebp-8]
0127ACE6     8B5482 68           mov edx,dword ptr ds:[edx+eax*4+68]
0127ACEA     8BC3                mov eax,ebx
0127ACEC     FFD2                call edx         ; 跳转类型，如是机器码是 74 则返回 4,如是 75 则返回 5，依次类推
0127ACEE     8BD8                mov ebx,eax
0127ACF0     8B4D 14             mov ecx,dword ptr ss:[ebp+14]
0127ACF3     8BD3                mov edx,ebx
0127ACF5     8B45 F8             mov eax,dword ptr ss:[ebp-8]
0127ACF8     E8 57F4FFFF         call 0127A154
0127ACFD     84C0                test al,al
0127ACFF     74 1A               je short 0127AD1B         ; al=0，跳转未实现；1，跳转实现
0127AD01     EB 01               jmp short 0127AD04
0127AD04     8B45 F8             mov eax,dword ptr ss:[ebp-8]
0127AD07     8B98 E0000000       mov ebx,dword ptr ds:[eax+E0]
0127AD0D     035D EC             add ebx,dword ptr ss:[ebp-14]
0127AD10     8B45 F8             mov eax,dword ptr ss:[ebp-8]
```
