
:10001DA4 4F                        `dec edi`

:10001DA5 83CFF0                    `or edi, FFFFFFF0`

:10001DA8 47                        `inc edi`

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:

`:10001DA2(C)`

:10001DA9 99                        `cdq`

:10001DAA 83E20F                   `and edx, 0000000F`

:10001DAD 03C2                     `add eax, edx`

:10001DAF C1F804                   `sar eax, 04`

:10001DB2 8BD8                     `mov ebx, eax`

:10001DB4 81E30F000080             `and ebx, 8000000F`      //`ebx` 中存放该数的 15～12 bit

:10001DBA 7905                     `jns 10001DC1`

:10001DBC 4B                       `dec ebx`

:10001DBD 83CBF0                   `or ebx, FFFFFFF0`

:10001DC0 43                       `inc ebx`

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:

`:10001DBA(C)`

:10001DC1 8BC3                     `mov eax, ebx`

:10001DC3 99                       `cdq`

:10001DC4 83E207                   `and edx, 00000007`

:10001DC7 03C2                     `add eax, edx`

:10001DC9 C1F803                   `sar eax, 03`

:10001DCC 8BE8                     `mov ebp, eax`

:10001DCE 81E501000080             `and ebp, 80000001`      //用掩码取出该数的第 15 bit

:10001DD4 7905                     `jns 10001DDB`

:10001DD6 4D                       `dec ebp`

:10001DD7 83CDFE                   `or ebp, FFFFFFFE`

:10001DDA 45                       `inc ebp`

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:

`:10001DD4(C)`

:10001DDB 8BC7                     `mov eax, edi`

:10001DDD 99                       `cdq`

:10001DDE 83E207                   `and edx, 00000007`

:10001DE1 03C2                     `add eax, edx`

:10001DE3 C1F803                   `sar eax, 03`

:10001DE6 2501000080               `and eax, 80000001`    //用掩码取出该数的第 11 bit

:10001DEB 7905                     `jns 10001DF2`

:10001DED 48                       `dec eax`

:10001DEE 83C8FE                   `or eax, FFFFFFFE`

:10001DF1 40                       `inc eax`

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:

`:10001DEB(C)`

:10001DF2 8D2C68                   `lea ebp, dword ptr [eax+2*ebp]` //乘以 2（二进制）再加上新的 bit

:10001DF5 8BC6                     `mov eax, esi`

:10001DF7 99                       `cdq`

:10001DF8 83E207                   `and edx, 00000007`

:10001DFB 03C2                     `add eax, edx`

:10001DFD C1F803                   `sar eax, 03`

:10001E00 2501000080               `and eax, 80000001`   //用掩码取出该数的第 7 bit

:10001E05 7905                     `jns 10001E0C`

:10001E07 48                       `dec eax`

:10001E08 83C8FE                   `or eax, FFFFFFFE`

:10001E0B 40                       `inc eax`

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:

`:10001E05(C)`

:10001E0C 8D2C68                   `lea ebp, dword ptr [eax+2*ebp]` //乘以 2（二进制）再加上新的 bit

:10001E0F 8BC1                     `mov eax, ecx`

:10001E11 99                       `cdq`

:10001E12 83E207                   `and edx, 00000007`

:10001E15 03C2                     `add eax, edx`

:10001E17 C1F803                   `sar eax, 03`

:10001E1A 2501000080               `and eax, 80000001`      //用掩码取出该数的第 3 bit

:10001E1F 7905                     `jns 10001E26`

:10001E21 48                       `dec eax`

:10001E22 83C8FE                   `or eax, FFFFFFFE`

:10001E25 40                       `inc eax`

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:

`:10001E1F(C)`

:10001E26 8D2C68                   `lea ebp, dword ptr [eax+2*ebp]` //乘以 2（二进制）再加上新的 bit

:10001E29 8BC3                     `mov eax, ebx`

:10001E2B 99                       `cdq`

:10001E2C 83E203                   `and edx, 00000003`

:10001E2F 03C2                     `add eax, edx`

:10001E31 C1F802                   `sar eax, 02`

:10001E34 2501000080               `and eax, 80000001`    //用掩码取出该数的第 14 bit

:10001E39 7905                     `jns 10001E40`

:10001E3B 48                       `dec eax`

:10001E3C 83C8FE                   `or eax, FFFFFFFE`

:10001E3F 40                       `inc eax`

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:

`:10001E39(C)`

:10001E40 8D2C68                   `lea ebp, dword ptr [eax+2*ebp]` //乘以 2（二进制）再加上新的 bit

:10001E43 8BC7                     `mov eax, edi`

:10001E45 99                       `cdq`

:10001E46 83E203                   `and edx, 00000003`

:10001E49 03C2                     `add eax, edx`

:10001E4B C1F802                   `sar eax, 02`

:10001E4E 2501000080               `and eax, 80000001`   //用掩码取出该数的第 10 bit

:10001E53 7905                     `jns 10001E5A`

:10001E55 48                       `dec eax`

:10001E56 83C8FE                   `or eax, FFFFFFFE`

:10001E59 40                       `inc eax`

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:

`:10001E53(C)`

:10001E5A 8D2C68                   `lea ebp, dword ptr [eax+2*ebp]` //乘以 2（二进制）再加上新的 bit

:10001E5D 8BC6                     `mov eax, esi`

:10001E5F 99                       `cdq`

:10001E60 83E203                   `and edx, 00000003`

:10001E63 03C2                     `add eax, edx`

:10001E65 C1F802                   `sar eax, 02`

:10001E68 2501000080               `and eax, 80000001` //用掩码取出该数的第 6 bit

:10001E6D 7905                     `jns 10001E74`

:10001E6F 48                       `dec eax`

:10001E70 83C8FE                   `or eax, FFFFFFFE`

:10001E73 40                       `inc eax`

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:

`:10001E6D(C)`

:10001E74 8D2C68                   `lea ebp, dword ptr [eax+2*ebp]` //乘以 2（二进制）再加上新的 bit

:10001E77 8BC1                     `mov eax, ecx`

:10001E79 99                       `cdq`

:10001E7A 83E203                   `and edx, 00000003`

:10001E7D 03C2                     `add eax, edx`

:10001E7F C1F802                   `sar eax, 02`

:10001E82 2501000080               `and eax, 80000001`   //用掩码取出该数的第 2 bit

:10001E87 7905                     `jns 10001E8E`

:10001E89 48                       `dec eax`

:10001E8A 83C8FE                   `or eax, FFFFFFFE`

:10001E8D 40                       `inc eax`

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:

`:10001E87(C)`

:10001E8E 8D2C68                   `lea ebp, dword ptr [eax+2*ebp]` //乘以 2（二进制）再加上新的 bit

:10001E91 8BC3                     `mov eax, ebx`

:10001E93 99                       `cdq`

:10001E94 2BC2                     `sub eax, edx`

:10001E96 D1F8                     `sar eax, 1`

:10001E98 2501000080               `and eax, 80000001`   //用掩码取出该数的第 13 bit

:10001E9D 7905                     `jns 10001EA4`

:10001E9F 48                       `dec eax`

:10001EA0 83C8FE                   `or eax, FFFFFFFE`

:10001EA3 40                       `inc eax`

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:

`:10001E9D(C)`

:10001EA4 8D2C68                   `lea ebp, dword ptr [eax+2*ebp]` //乘以 2（二进制）再加上新的 bit

:10001EA7 8BC7                     `mov eax, edi`

:10001EA9 99                       `cdq`

:10001EAA 2BC2                     `sub eax, edx`

:10001EAC D1F8                     `sar eax, 1`

:10001EAE 2501000080               `and eax, 80000001`   //用掩码取出该数的第 9 bit

:10001EB3 7905                     `jns 10001EBA`

:10001EB5 48                       `dec eax`

:10001EB6 83C8FE                   `or eax, FFFFFFFE`

:10001EB9 40                       `inc eax`

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:

`:10001EB3(C)`

```
:10001EBA 8D2C68                   lea ebp, dword ptr [eax+2*ebp] //乘以 2（二进制）再加上新的 bit
:10001EBD 8BC6                     mov eax, esi
:10001EBF 99                       cdq
```
