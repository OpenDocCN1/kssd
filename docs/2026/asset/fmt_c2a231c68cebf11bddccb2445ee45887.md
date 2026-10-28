

---

```
004016EF  |.  83C4 14       add     esp, 14
004016F2  |.  46            inc     esi
004016F3  |.  EB 1A         jmp     short 0040170F
```

刚刚错怪`call    00403720`了，现在调用`CreateDirectoryA`创建文件夹

马上去`temp`下去看了看，还真给新建了个“`E_N4`”文件夹。。

---

```
0040170F  |> \8D85 6CFEFFFF lea     eax, dword ptr [ebp-194]
00401715  |.  53            push    ebx                              ; /pSecurity
00401716  |.  50            push    eax                              ; |Path
00401717  |.  FF15 10704000 call    dword ptr [<&KERNEL32.CreateDire>; \CreateDirectoryA
0040171D  |.  8D85 6CFEFFFF lea     eax, dword ptr [ebp-194]
00401723  |.  68 8C814000   push    0040818C
00401728  |.  50            push    eax
00401729  |.  E8 F21F0000   call    00403720
0040172E  |.  836D 08 08    sub     dword ptr [ebp+8], 8
00401732  |.  8B46 04       mov     eax, dword ptr [esi+4]
00401735  |.  395D 08       cmp     dword ptr [ebp+8], ebx
00401738  |.  59            pop     ecx
00401739  |.  59            pop     ecx
0040173A  |.  8945 EC       mov     dword ptr [ebp-14], eax
0040173D  |.  0F8E A9010000 jle     004018EC
0040171D  |.  8D85 6CFEFFFF lea     eax, dword ptr [ebp-194]
00401723  |.  68 8C814000   push    0040818C
00401728  |.  50            push    eax
00401729  |.  E8 F21F0000   call    00403720
0040172E  |.  836D 08 08    sub     dword ptr [ebp+8], 8
00401732  |.  8B46 04       mov     eax, dword ptr [esi+4]
00401735  |.  395D 08       cmp     dword ptr [ebp+8], ebx
00401738  |.  59            pop     ecx
00401739  |.  59            pop     ecx
0040173A  |.  8945 EC       mov     dword ptr [ebp-14], eax
0040173D  |.  0F8E A9010000 jle     004018EC
00401743  |.  813E 0D0F3E03 cmp     dword ptr [esi], 33E0F0D
00401749  |.  0F85 9D010000 jnz     004018EC
0040174F  |.  3BC3          cmp     eax, ebx
00401751  |.  0F8E 95010000 jle     004018EC
00401757  |.  50            push    eax
00401758  |.  E8 2E210000   call    0040388B
0040175D  |.  8BF8          mov     edi, eax
0040175F  |.  59            pop     ecx
00401760  |.  3BFB          cmp     edi, ebx
00401762  |.  75 0C         jnz     short 00401770
00401770  |> \FF75 08       push    dword ptr [ebp+8]
00401773  |.  83C6 08       add     esi, 8
00401776  |.  8D45 EC       lea     eax, dword ptr [ebp-14]
00401779  |.  56            push    esi
0040177A  |.  50            push    eax
0040177B  |.  57            push    edi
0040177C  |.  E8 EE1E0000   call    0040366F
00401781  |.  83C4 10       add     esp, 10
00401784  |.  85C0          test    eax, eax
00401786  |.  74 13         je      short 0040179B
0040179B  |> \FF75 F8       push    dword ptr [ebp-8]
0040179E  |.  E8 5D200000   call    00403800
004017A3  |.  8B45 EC       mov     eax, dword ptr [ebp-14]
004017A6  |.  59            pop     ecx
004017A7  |.  03C7          add     eax, edi
004017A9  |.  897D F8       mov     dword ptr [ebp-8], edi
004017AC  |.  3BF8          cmp     edi, eax
004017AE  |.  8BF7          mov     esi, edi
004017B0  |.  8945 F4       mov     dword ptr [ebp-C], eax
004017B3  |.  885D A4       mov     byte ptr [ebp-5C], bl
004017B6  |.  0F83 B4000000 jnb     00401870
004017BC  |>  8BFE          /mov     edi, esi
004017BE  |.  56            |push    esi
004017BF  |.  897D 08       |mov     dword ptr [ebp+8], edi
004017C2  |.  E8 49200000   |call    00403810
004017C7  |.  C70424 4C8140>|mov     dword ptr [esp], 0040814C       ;  ASCII "krnln.fnr"
004017CE  |.  57            |push    edi
004017CF  |.  8D7406 01     |lea     esi, dword ptr [esi+eax+1]
004017D3  |.  E8 48480000   |call    00406020
004017D8  |.  59            |pop     ecx
004017D9  |.  85C0          |test    eax, eax
004017DB  |.  59            |pop     ecx
004017DC  |.  74 11         |je      short 004017EF
004017EF  |> \8D45 A4       |lea     eax, dword ptr [ebp-5C]
004017F2  |.  57            |push    edi
004017F3  |.  50            |push    eax
004017F4  |.  E8 171F0000   |call    00403710
004017F9  |.  59            |pop     ecx
004017FA  |.  59            |pop     ecx
004017FB  |>  8B3E          |mov     edi, dword ptr [esi]
004017FD  |.  8D85 6CFEFFFF |lea     eax, dword ptr [ebp-194]
00401803  |.  50            |push    eax
00401804  |.  8D85 68FDFFFF |lea     eax, dword ptr [ebp-298]
0040180A  |.  50            |push    eax
0040180B  |.  83C6 04       |add     esi, 4
0040180E  |.  E8 FD1E0000   |call    00403710
00401813  |.  FF75 08       |push    dword ptr [ebp+8]
00401816  |.  8D85 68FDFFFF |lea     eax, dword ptr [ebp-298]
0040181C  |.  50            |push    eax
0040181D  |.  E8 FE1E0000   |call    00403720
00401822  |.  83C4 10       |add     esp, 10
00401825  |.  8D85 68FDFFFF |lea     eax, dword ptr [ebp-298]        ;eax==0012FC9C  C:\DOCUME~1\ADMINI~1\LOCALS~1\Temp\E_N4\krnln.fnr.
```

在`temp`下新建了个文件（`C:\DOCUME~1\ADMINI~1\LOCALS~1\Temp\E_N4\krnln.fnr`）

---

```
0040182B  |.  53            |push    ebx                             ; /hTemplateFile
0040182C  |.  68 80000000   |push    80                              ; |Attributes = NORMAL
00401831  |.  6A 02         |push    2                               ; |Mode = CREATE_ALWAYS
00401833  |.  53            |push    ebx                             ; |pSecurity
00401834  |.  53            |push    ebx                             ; |ShareMode
00401835  |.  68 00000040   |push    40000000                        ; |Access = GENERIC_WRITE
0040183A  |.  50            |push    eax                             ; |FileName
0040183B  |.  FF15 20704000 |call    dword ptr [<&KERNEL32.CreateFil>; \CreateFileA
```

向文件里面写数据（`C:\DOCUME~1\ADMINI~1\LOCALS~1\Temp\E_N4\krnln.fnr`）

---

```
00401841  |.  83F8 FF       |cmp     eax, -1
00401844  |.  8945 08       |mov     dword ptr [ebp+8], eax
00401847  |.  74 17         |je      short 00401860
00401849  |.  8D4D D8       |lea     ecx, dword ptr [ebp-28]
0040184C  |.  53            |push    ebx                             ; /pOverlapped
0040184D  |.  51            |push    ecx                             ; |pBytesWritten
0040184E  |.  57            |push    edi                             ; |nBytesToWrite
0040184F  |.  56            |push    esi                             ; |Buffer
00401850  |.  50            |push    eax                             ; |hFile
00401851  |.  FF15 0C704000 |call    dword ptr [<&KERNEL32.WriteFile>; \WriteFile
00401857  |.  FF75 08       |push    dword ptr [ebp+8]               ; /hObject
0040185A  |.  FF15 08704000 |call    dword ptr [<&KERNEL32.CloseHand>; \CloseHandle
00401860  |>  03F7          |add     esi, edi
00401862  |.  3B75 F4       |cmp     esi, dword ptr [ebp-C]
00401865  |.^ 0F82 51FFFFFF \jb      004017BC
```

循环，估计要再建个`0012FC8C   00C4D02E  ASCII "HtmlView.fne"`的文件

