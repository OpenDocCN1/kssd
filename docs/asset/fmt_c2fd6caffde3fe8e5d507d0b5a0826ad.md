
```text
 .text:0044169E 8B 45 FC               mov   eax, [ebp+var_4]

 .text:004416A1 C1 C8 0D               ror   eax, 0Dh

 .text:004416A4 89 45 FC               mov   [ebp+var_4], eax                ; s4=(ror s4,0Dh)

 .text:004416A7 8B 4D E0               mov   ecx, [ebp+var_20]

 .text:004416AA 8B 55 FC               mov   edx, [ebp+var_4]

 .text:004416AD 03 51 4C               add   edx, [ecx+4Ch]

 .text:004416B0 89 55 E4               mov   [ebp+var_1C], edx               ; var_1C=tmp2=s4+table[4Ch]

 .text:004416B3 8B 45 E4               mov   eax, [ebp+var_1C]

 .text:004416B6 25 FF 01 00 00          and   eax, 1FFh

 .text:004416BB 8B 0C 85 BC ED 4D 00     mov   ecx, table_1[eax*4]

 .text:004416C2 89 4D E8               mov   [ebp+var_18], ecx               ; var_18=tmp1=table_1[(tmp2&1FFh)*4]

 .text:004416C5 8B 55 F8               mov   edx, [ebp+var_8]

 .text:004416C8 C1 C2 05               rol   edx, 5

 .text:004416CB 89 55 F8               mov   [ebp+var_8], edx                ; var_8=tmp=(rol tmp,5)

 .text:004416CE 8B 45 E8               mov   eax, [ebp+var_18]

 .text:004416D1 33 45 F8               xor   eax, [ebp+var_8]

 .text:004416D4 89 45 E8               mov   [ebp+var_18], eax               ; var_18=tmp1=tmp1^tmp

 .text:004416D7 8B 4D F8               mov   ecx, [ebp+var_8]

 .text:004416DA 8B 55 E4               mov   edx, [ebp+var_1C]

 .text:004416DD D3 C2                 rol   edx, cl

 .text:004416DF 8B 45 F0               mov   eax, [ebp+var_10]

 .text:004416E2 2B C2                 sub   eax, edx

 .text:004416E4 89 45 F0               mov   [ebp+var_10], eax               ; s2=s2-(rol tmp2,(tmp&0Fh))

 .text:004416E7 8B 4D F8               mov   ecx, [ebp+var_8]

 .text:004416EA C1 C1 05               rol   ecx, 5

 .text:004416ED 89 4D F8               mov   [ebp+var_8], ecx                ; var_8=tmp=(rol tmp,5)

 .text:004416F0 8B 55 E8               mov   edx, [ebp+var_18]

 .text:004416F3 33 55 F8               xor   edx, [ebp+var_8]

 .text:004416F6 89 55 E8               mov   [ebp+var_18], edx               ; var_18=tmp1=tmp1^tmp

 .text:004416F9 8B 45 F4               mov   eax, [ebp+var_C]

 .text:004416FC 33 45 F8               xor   eax, [ebp+var_8]

 .text:004416FF 89 45 F4               mov   [ebp+var_C], eax                ; s3=s3^tmp

 .text:00441702 8B 4D F8               mov   ecx, [ebp+var_8]

 .text:00441705 8B 55 E8               mov   edx, [ebp+var_18]

 .text:00441708 D3 C2                 rol   edx, cl

 .text:0044170A 8B 45 EC               mov   eax, [ebp+var_14]

 .text:0044170D 2B C2                 sub   eax, edx

 .text:0044170F 89 45 EC               mov   [ebp+var_14], eax               ; s1=s1-(rol tmp1,(tmp&0Fh))

==========================================================================

上面的的逆算法为:

                 var_1C=tmp2=s4+table[4Ch]
                 s4=(rol s4,0Dh)
                 var_8=tmp=s4*table[50h]
                 var_18=tmp1=table_1[(tmp2&1FFh)*4]
                 var_8=tmp=(rol tmp,5)
                 var_18=tmp1=tmp1^tmp
                 s2=s2+(rol tmp2,(tmp&0Fh))
                 var_8=tmp=(rol tmp,5)
                 var_18=tmp1=tmp1^tmp
                 s3=s3^tmp
                 s1=s1+(rol tmp1,(tmp&0Fh))

==========================================================================


 .text:00441712 8B 4D E0               mov   ecx, [ebp+var_20]

 .text:00441715 8B 55 F4               mov   edx, [ebp+var_C]                ; s3

 .text:00441718 0F AF 51 48            imul  edx, [ecx+48h]

 .text:0044171C 89 55 F8               mov   [ebp+var_8], edx                ; var_8=tmp=s3*table[48h]

 .text:0044171F 8B 45 F4               mov   eax, [ebp+var_C]

 .text:00441722 C1 C8 0D               ror   eax, 0Dh

 .text:00441725 89 45 F4               mov   [ebp+var_C], eax                ; s3=(ror s3,0Dh)

 .text:00441728 8B 4D E0               mov   ecx, [ebp+var_20]

 .text:0044172B 8B 55 F4               mov   edx, [ebp+var_C]

 .text:0044172E 03 51 44               add   edx, [ecx+44h]

 .text:00441731 89 55 E4               mov   [ebp+var_1C], edx               ; var_1C=tmp2=s3+tabel[44h]

 .text:00441734 8B 45 E4               mov   eax, [ebp+var_1C]

 .text:00441737 25 FF 01 00 00          and   eax, 1FFh

 .text:0044173C 8B 0C 85 BC ED 4D 00     mov   ecx, table_1[eax*4]

 .text:00441743 89 4D E8               mov   [ebp+var_18], ecx               ; var_18=tmp1=table_1[(tmp2&1FFh)*4]

 .text:00441746 8B 55 F8               mov   edx, [ebp+var_8]

 .text:00441749 C1 C2 05               rol   edx, 5

 .text:0044174C 89 55 F8               mov   [ebp+var_8], edx                ; var_8=tmp=(rol tmp,5)

 .text:0044174F 8B 45 E8               mov   eax, [ebp+var_18]

 .text:00441752 33 45 F8               xor   eax, [ebp+var_8]

 .text:00441755 89 45 E8               mov   [ebp+var_18], eax               ; var_18=tmp1=tmp1^tmp

 .text:00441758 8B 4D F8               mov   ecx, [ebp+var_8]

 .text:0044175B 8B 55 E4               mov   edx, [ebp+var_1C]

 .text:0044175E D3 C2                 rol   edx, cl

 .text:00441760 8B 45 EC               mov   eax, [ebp+var_14]

 .text:00441763 2B C2                 sub   eax, edx

 .text:00441765 89 45 EC               mov   [ebp+var_14], eax               ; s1=s1-(rol tmp2,(tmp&0Fh))

 .text:00441768 8B 4D F8               mov   ecx, [ebp+var_8]

 .text:0044176B C1 C1 05               rol   ecx, 5

 .text:0044176E 89 4D F8               mov   [ebp+var_8], ecx                ; var_8=tmp=(rol tmp,5)

 .text:00441771 8B 55 E8               mov   edx, [ebp+var_18]

 .text:00441774 33 55 F8               xor   edx, [ebp+var_8]

 .text:00441777 89 55 E8               mov   [ebp+var_18], edx               ; var_18=tmp1=tmp1^tmp

 .text:0044177A 8B 45 F0               mov   eax, [ebp+var_10]

 .text:0044177D 33 45 F8               xor   eax, [ebp+var_8]

 .text:00441780 89 45 F0               mov   [ebp+var_10], eax               ; s2=s2^tmp

 .text:00441783 8B 4D F8               mov   ecx, [ebp+var_8]

 .text:00441786 8B 55 E8               mov   edx, [ebp+var_18]

 .text:00441789 D3 C2                 rol   edx, cl

 .text:0044178B 8B 45 FC               mov   eax, [ebp+var_4]

 .text:0044178E 2B C2                 sub   eax, edx

 .text:00441790 89 45 FC               mov   [ebp+var_4], eax                ; s4=s4-(rol tmp1,(tmp&0Fh))

==========================================================================

上面的的逆算法为:

                 var_1C=tmp2=s3+table[44h]
                 s3=(rol s3,0Dh)
                 var_8=tmp=s3*table[48h]
                 var_18=tmp1=table_1[(tmp2&1FFh)*4]
                 var_8=tmp=(rol tmp,5)
                 var_18=tmp1=tmp1^tmp
                 s1=s1+(rol tmp2,(tmp&0Fh))
                 var_8=tmp=(rol tmp,5)
                 var_18=tmp1=tmp1^tmp
                 s2=s2^tmp
                 s4=s4+(rol tmp1,(tmp&0Fh))

==========================================================================


 .text:00441793 8B 4D E0               mov   ecx, [ebp+var_20]
```
