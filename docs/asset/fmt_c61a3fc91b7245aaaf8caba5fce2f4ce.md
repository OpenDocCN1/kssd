
```assembly
.text:00441369 33 55 F8                 xor   edx, [ebp+var_8]
.text:0044136C 89 55 E8                 mov   [ebp+var_18], edx           ; var_18=tmp1=tmp1^tmp
.text:0044136F 8B 45 FC                 mov   eax, [ebp+var_4]
.text:00441372 33 45 F8                 xor   eax, [ebp+var_8]
.text:00441375 89 45 FC                 mov   [ebp+var_4], eax           ; s4=s4^tmp
.text:00441378 8B 4D F8                 mov   ecx, [ebp+var_8]
.text:0044137B 8B 55 E8                 mov   edx, [ebp+var_18]
.text:0044137E D3 C2                     rol   edx, cl
.text:00441380 8B 45 F0                 mov   eax, [ebp+var_10]
.text:00441383 2B C2                     sub   eax, edx
.text:00441385 89 45 F0                 mov   [ebp+var_10], eax         ; s2=s2-(rol tmp1,(tmp&0Fh))
==========================================================================
上面的的逆算法为:
             var_1C=tmp2=s3+table[84h]
             s3=(rol s3,0Dh)
             var_8=tmp=s3*table[88h]
             var_18=tmp1=table_1[(tmp2&1FFh)*4]
             var_8=tmp=(rol tmp,5)
             var_18=tmp1=tmp1^tmp
             s1=s1+(rol tmp2,(tmp&0Fh))
             var_8=tmp=(rol tmp,5)
             var_18=tmp1=tmp1^tmp
             s4=s4^tmp
             s2=s2+(rol tmp1,(tmp&0Fh))
==========================================================================
.text:00441385                                                ; ____________
.text:00441388 8B 4D E0                 mov   ecx, [ebp+var_20]
.text:0044138B 8B 55 F0                 mov   edx, [ebp+var_10]          ; s2
.text:0044138E 0F AF 91 80 00 00 00     imul  edx, [ecx+80h]
.text:00441395 89 55 F8                 mov   [ebp+var_8], edx          ; var_8=tmp=s2*table[80h]
.text:00441398 8B 45 F0                 mov   eax, [ebp+var_10]
.text:0044139B C1 C8 0D                 ror   eax, 0Dh
.text:0044139E 89 45 F0                 mov   [ebp+var_10], eax         ; s2=(ror s2,0Dh)
.text:004413A1 8B 4D E0                 mov   ecx, [ebp+var_20]
.text:004413A4 8B 55 F0                 mov   edx, [ebp+var_10]
.text:004413A7 03 51 7C                 add   edx, [ecx+7Ch]
.text:004413AA 89 55 E4                 mov   [ebp+var_1C], edx         ; var_1C=tmp2=s2+table[7Ch]
.text:004413AD 8B 45 E4                 mov   eax, [ebp+var_1C]
.text:004413B0 25 FF 01 00 00           and   eax, 1FFh
.text:004413B5 8B 0C 85 BC ED 4D 00     mov   ecx, table_1[eax*4]
.text:004413BC 89 4D E8                 mov   [ebp+var_18], ecx         ; var_18=tmp1=table_1[(tmp2&1FFh)*4]
.text:004413BF 8B 55 F8                 mov   edx, [ebp+var_8]
.text:004413C2 C1 C2 05                 rol   edx, 5
.text:004413C5 89 55 F8                 mov   [ebp+var_8], edx          ; var_8=tmp=(rol tmp,5)
.text:004413C8 8B 45 E8                 mov   eax, [ebp+var_18]
.text:004413CB 33 45 F8                 xor   eax, [ebp+var_8]
.text:004413CE 89 45 E8                 mov   [ebp+var_18], eax         ; var_18=tmp1=tmp1^tmp
.text:004413D1 8B 4D F8                 mov   ecx, [ebp+var_8]
.text:004413D4 8B 55 E4                 mov   edx, [ebp+var_1C]
.text:004413D7 D3 C2                     rol   edx, cl
.text:004413D9 8B 45 FC                 mov   eax, [ebp+var_4]
.text:004413DC 2B C2                     sub   eax, edx
.text:004413DE 89 45 FC                 mov   [ebp+var_4], eax          ; s4=s4-(rol tmp2,(tmp&0Fh))
.text:004413E1 8B 4D F8                 mov   ecx, [ebp+var_8]
.text:004413E4 C1 C1 05                 rol   ecx, 5
.text:004413E7 89 4D F8                 mov   [ebp+var_8], ecx          ; var_8=tmp=(rol tmp,5)
.text:004413EA 8B 55 E8                 mov   edx, [ebp+var_18]
.text:004413ED 33 55 F8                 xor   edx, [ebp+var_8]
.text:004413F0 89 55 E8                 mov   [ebp+var_18], edx         ; var_18=tmp1=tmp1^tmp
.text:004413F3 8B 45 F4                 mov   eax, [ebp+var_C]
.text:004413F6 33 45 F8                 xor   eax, [ebp+var_8]
.text:004413F9 89 45 F4                 mov   [ebp+var_C], eax          ; s3=s3^tmp
.text:004413FC 8B 4D F8                 mov   ecx, [ebp+var_8]
.text:004413FF 8B 55 E8                 mov   edx, [ebp+var_18]
.text:00441402 D3 C2                     rol   edx, cl
.text:00441404 8B 45 EC                 mov   eax, [ebp+var_14]
.text:00441407 2B C2                     sub   eax, edx
.text:00441409 89 45 EC                 mov   [ebp+var_14], eax         ; s1=s1-(rol tmp1,(tmp&0Fh))
==========================================================================
上面的的逆算法为:
             var_1C=tmp2=s2+table[7Ch]
             s2=(rol s2,0Dh)
             var_8=tmp=s2*table[80h]
             var_18=tmp1=table_1[(tmp2&1FFh)*4]
             var_8=tmp=(rol tmp,5)
             var_18=tmp1=tmp1^tmp
             s4=s4+(rol tmp2,(tmp&0Fh))
             var_8=tmp=(rol tmp,5)
             var_18=tmp1=tmp1^tmp
             s3=s3^tmp
             s1=s1+(rol tmp1,(tmp&0Fh))
==========================================================================
.text:00441409                                                ; ____________
.text:0044140C 8B 4D E0                 mov   ecx, [ebp+var_20]
.text:0044140F 8B 55 EC                 mov   edx, [ebp+var_14]          ; s1
.text:00441412 0F AF 51 78              imul  edx, [ecx+78h]
.text:00441416 89 55 F8                 mov   [ebp+var_8], edx          ; var_8=tmp=s1*table[78h]
.text:00441419 8B 45 EC                 mov   eax, [ebp+var_14]
.text:0044141C C1 C8 0D                 ror   eax, 0Dh
.text:0044141F 89 45 EC                 mov   [ebp+var_14], eax         ; s1=(ror s1,0Dh)
.text:00441422 8B 4D E0                 mov   ecx, [ebp+var_20]
.text:00441425 8B 55 EC                 mov   edx, [ebp+var_14]
.text:00441428 03 51 74                 add   edx, [ecx+74h]
.text:0044142B 89 55 E4                 mov   [ebp+var_1C], edx         ; var_1C=tmp2=s1+table[74h]
.text:0044142E 8B 45 E4                 mov   eax, [ebp+var_1C]
.text:00441431 25 FF 01 00 00           and   eax, 1FFh
.text:00441436 8B 0C 85 BC ED 4D 00     mov   ecx, table_1[eax*4]
.text:0044143D 89 4D E8                 mov   [ebp+var_18], ecx         ; var_18=tmp1=table_1[(tmp2&1FFh)*4]
.text:00441440 8B 55 F8                 mov   edx, [ebp+var_8]
.text:00441443 C1 C2 05                 rol   edx, 5
.text:00441446 89 55 F8                 mov   [ebp+var_8], edx          ; var_8=tmp=(rol tmp,5)
.text:00441449 8B 45 E8                 mov   eax, [ebp+var_18]
.text:0044144C 33 45 F8                 xor   eax, [ebp+var_8]
.text:0044144F 89 45 E8                 mov   [ebp+var_18], eax         ; var_18=tmp1=tmp1^tmp
.text:00441452 8B 4D F8                 mov   ecx, [ebp+var_8]
.text:00441455 8B 55 E4                 mov   edx, [ebp+var_1C]
.text:00441458 D3 C2                     rol   edx, cl
```
