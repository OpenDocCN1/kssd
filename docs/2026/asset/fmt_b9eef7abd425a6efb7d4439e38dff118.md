
呵呵，下面是关键运算`CALL`里面的东西。变！其实作者如果也对最后4位字符的值进行如此这般的运算，那么我们求逆就会复杂的多了。`^Q^^Q^^Q^^Q^`

* 
Referenced by a CALL at Addresses:

|:004159FE , :00415A2F , :00415AAB , :00415ADF |

```
:00415B10 83EC44
         sub esp, 00000044
:00415B13 8B02
               mov eax, dword ptr [edx]
:00415B15
53                      push
ebx
:00415B16 8B5904
 mov ebx, dword ptr [ecx+04]
:00415B19 55
             push ebp
:00415B1A 894C2448
               mov dword ptr [esp+48],
ecx
:00415B1E 56
   push esi
:00415B1F 8B710C
     mov esi, dword ptr [ecx+0C]
:00415B22 57
                 push edi
:00415B23
8B7908                  mov edi,
dword ptr [ecx+08]
:00415B26 8B09
       mov ecx, dword ptr [ecx]
:00415B28 8BEE
               mov ebp, esi
:00415B2A
33EF                    xor
ebp, edi
:00415B2C 23EB
   and ebp, ebx
:00415B2E 33EE
         xor ebp, esi
:00415B30 03E8
               add ebp, eax
:00415B32
8D8C0D78A46AD7          lea ecx, dword ptr [ebp+ecx-28955B88]
:00415B39
89442410                mov dword ptr
[esp+10], eax
:00415B3D 8BC1
     mov eax, ecx
:00415B3F C1E107
         shl ecx, 07
:00415B42 C1E819
             shr eax, 19
:00415B45 0BC1
                   or eax, ecx
:00415B47
8B4A04                  mov ecx,
dword ptr [edx+04]
:00415B4A 03C3
       add eax, ebx
:00415B4C 8BEF
             mov ebp, edi
:00415B4E 33EB
                   xor ebp,
ebx
:00415B50 23E8
   and ebp, eax
:00415B52 33EF
         xor ebp, edi
:00415B54 03E9
               add ebp, ecx
:00415B56
894C2424                mov dword ptr
[esp+24], ecx
:00415B5A 8D8C2E56B7C7E8          lea
ecx, dword ptr [esi+ebp-173848AA]
:00415B61 8BF1
           mov esi, ecx
:00415B63 C1E10C
               shl ecx, 0C
:00415B66
C1EE14                  shr esi,
14
:00415B69 0BF1
 or esi, ecx
:00415B6B 8B4A08
     mov ecx, dword ptr [edx+08]
:00415B6E 03F0
               add esi, eax
:00415B70
8BEB                    mov
ebp, ebx
:00415B72 33E8
   xor ebp, eax
:00415B74 23EE
         and ebp, esi
:00415B76 33EB
               xor ebp, ebx
:00415B78
03E9                    add
ebp, ecx
:00415B7A 894C2444
 mov dword ptr [esp+44], ecx
:00415B7E 8D8C2FDB702024
   lea ecx, dword ptr [edi+ebp+242070DB]
:00415B85 8BF9
               mov edi, ecx
:00415B87
C1E90F                  shr ecx,
0F
:00415B8A C1E711
 shl edi, 11
:00415B8D 0BF9
       or edi, ecx
:00415B8F 8B4A0C
           mov ecx, dword ptr [edx+0C]
:00415B92
03FE                    add
edi, esi
:00415B94 8BEE
   mov ebp, esi
:00415B96 33E8
         xor ebp, eax
:00415B98 23EF
               and ebp, edi
:00415B9A
33E8                    xor
ebp, eax
:00415B9C 03E9
   add ebp, ecx
:00415B9E 894C241C
     mov dword ptr [esp+1C], ecx
:00415BA2 8D8C2BEECEBDC1
       lea ecx, dword ptr [ebx+ebp-3E423112]
:00415BA9
8BD9                    mov
ebx, ecx
:00415BAB C1E90A
   shr ecx, 0A
:00415BAE C1E316
       shl ebx, 16
:00415BB1 0BD9
             or ebx, ecx
:00415BB3 8B4A10
                 mov ecx, dword ptr
[edx+10]
:00415BB6 03DF
   add ebx, edi
:00415BB8 8BEE
         mov ebp, esi
:00415BBA 33EF
               xor ebp, edi
:00415BBC
23EB                    and
ebp, ebx
:00415BBE 33EE
   xor ebp, esi
:00415BC0 03E9
         add ebp, ecx
:00415BC2 8D8428AF0F7CF5
       lea eax, dword ptr [eax+ebp-0A83F051]
:00415BC9
894C2434                mov dword ptr
[esp+34], ecx
:00415BCD 8BC8
     mov ecx, eax
:00415BCF C1E919
         shr ecx, 19
:00415BD2 C1E007
             shl eax, 07
:00415BD5 0BC8
                   or ecx, eax
:00415BD7
8B4214                  mov eax,
dword ptr [edx+14]
:00415BDA 8BEF
       mov ebp, edi
:00415BDC 33EB
             xor ebp, ebx
:00415BDE 03CB
                   add ecx,
ebx
:00415BE0 23E9
   and ebp, ecx
:00415BE2 33EF
         xor ebp, edi
:00415BE4 89442418
           mov dword ptr [esp+18], eax
:00415BE8
03E8                    add
ebp, eax
:00415BEA 8DB42E2AC68747          lea esi,
dword ptr [esi+ebp+4787C62A]
:00415BF1 8BC6
         mov eax, esi
:00415BF3 C1E60C
             shl esi, 0C
:00415BF6 C1E814
                 shr eax, 14
:00415BF9
0BC6                    or eax,
esi
:00415BFB 8B7218
 mov esi, dword ptr [edx+18]
:00415BFE 03C1
           add eax, ecx
:00415C00 8BEB
                 mov ebp, ebx
:00415C02
33E9                    xor
ebp, ecx
:00415C04 23E8
   and ebp, eax
:00415C06 33EB
         xor ebp, ebx
:00415C08 03EE
               add ebp, esi
:00415C0A
8DBC2F134630A8          lea edi, dword ptr [edi+ebp-57CFB9ED]
:00415C11
8974242C                mov dword ptr
[esp+2C], esi
:00415C15 8BF7
     mov esi, edi
:00415C17 C1EF0F
         shr edi, 0F
:00415C1A C1E611
             shl esi, 11
:00415C1D 0BF7
                   or esi, edi
:00415C1F
8B7A1C                  mov edi,
dword ptr [edx+1C]
:00415C22 03F0
       add esi, eax
:00415C24 8BE8
             mov ebp, eax
:00415C26 33E9
                   xor ebp,
ecx
:00415C28 23EE
   and ebp, esi
:00415C2A 33E9
         xor ebp, ecx
:00415C2C 03EF
               add ebp, edi
:00415C2E
8D9C2B019546FD          lea ebx, dword ptr [ebx+ebp-02B96AFF]
:00415C35
897C2414                mov dword ptr
[esp+14], edi
:00415C39 8BFB
     mov edi, ebx
:00415C3B C1EB0A
         shr ebx, 0A
:00415C3E 8BE8
               mov ebp, eax
:00415C40
33EE                    xor
ebp, esi
:00415C42 C1E716
   shl edi, 16
:00415C45 0BFB
         or edi, ebx
:00415C47 8B5A20
             mov ebx, dword ptr [edx+20]
:00415C4A
03FE                    add
edi, esi
:00415C4C 23EF
   and ebp, edi
:00415C4E 33E8
         xor ebp, eax
:00415C50 03EB
               add ebp, ebx
:00415C52
8D8C29D8988069          lea ecx, dword ptr [ecx+ebp+698098D8]
:00415C59
895C2428                mov dword ptr
[esp+28], ebx
:00415C5D 8BD9
     mov ebx, ecx
:00415C5F C1E107
         shl ecx, 07
:00415C62 8BEE
               mov ebp, esi
:00415C64
33EF                    xor
ebp, edi
:00415C66 C1EB19
   shr ebx, 19
:00415C69 0BD9
         or ebx, ecx
:00415C6B 8B4A24
             mov ecx, dword ptr [edx+24]
:00415C6E
03DF                    add
ebx, edi
:00415C70 23EB
   and ebp, ebx
:00415C72 33EE
         xor ebp, esi
:00415C74 03E9
               add ebp, ecx
:00415C76
8D8428AFF7448B          lea eax, dword ptr [eax+ebp-74BB0851]
:00415C7D
894C243C                mov dword ptr [esp+3C], ecx
:00415C81 8BC8
     mov ecx, eax
:00415C83 C1E00C
         shl eax, 0C
:00415C86 C1E914
             shr ecx, 14
:00415C89 0BC8
                   or ecx, eax
:00415C8B
8B4228                  mov eax,
dword ptr [edx+28]
:00415C8E 03CB
       add ecx, ebx
:00415C90 8BEF
             mov ebp, edi
:00415C92 33EB
                   xor ebp,
ebx
:00415C94 23E9
   and ebp, ecx
:00415C96 33EF
         xor ebp, edi
:00415C98 03E8
               add ebp, eax
:00415C9A
8DB42EB15BFFFF          lea esi, dword ptr [esi+ebp-0000A44F]
:00415CA1
89442420                mov dword ptr
[esp+20], eax
:00415CA5 8BC6
     mov eax, esi
:00415CA7 C1EE0F
         shr esi, 0F
:00415CAA C1E011
             shl eax, 11
:00415CAD 0BC6
                   or eax, esi
:00415CAF
8B722C                  mov esi,
dword ptr [edx+2C]
:00415CB2 8BE9
       mov ebp, ecx
:00415CB4 33EB
             xor ebp, ebx
:00415CB6 03C1
                   add eax,
ecx
:00415CB8 23E8
   and ebp, eax
:00415CBA 33EB
         xor ebp, ebx
:00415CBC 03EE
               add ebp, esi
:00415CBE
89742438                mov dword ptr
[esp+38], esi
:00415CC2 8DBC2FBED75C89          lea
edi, dword ptr [edi+ebp-76A32842]
:00415CC9 8BF7
           mov esi, edi
:00415CCB C1E616
               shl esi, 16
:00415CCE
C1EF0A                  shr edi,
0A
:00415CD1 0BF7
 or esi, edi
```
