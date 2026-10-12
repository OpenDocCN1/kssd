
```asm
.text:10042625

.text:10042625  loc_10042625:               ; CODE XREF: sub_10042520+FC  j

.text:10042625                 mov     [esp+18h+hKey], 0

.text:1004262D                 mov     edx, [edi+0Ch]

.text:10042630                 lea     ecx, [edi+10h]

.text:10042633                 xor     eax, eax

.text:10042635

.text:10042635  loc_10042635:               ; CODE XREF: sub_10042520+128  j

.text:10042635                 test    eax, eax

.text:10042637                 jnz     short loc_1004263E

.text:10042639                 mov     byte ptr [ecx], 1

.text:1004263C                 jmp     short loc_10042642

.text:1004263E  ; 哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪?

.text:1004263E

.text:1004263E  loc_1004263E:               ; CODE XREF: sub_10042520+117  j

.text:1004263E                 mov     byte ptr [eax+ecx], 0

.text:10042642

.text:10042642  loc_10042642:               ; CODE XREF: sub_10042520+11C  j

.text:10042642                 add     eax, 1

.text:10042645                 cmp     eax, 4

.text:10042648                 jb      short loc_10042635

.text:1004264A                 mov     eax, edx

.text:1004264C                 mov     esi, edx

.text:1004264E                 shr     eax, 4

.text:10042651                 shr     esi, 3

.text:10042654                 lea     edx, [esi+eax*2+4]

.text:10042658                 add     ecx, edx

.text:1004265A                 xor     edx, edx

.text:1004265C                 test    eax, eax

.text:1004265E                 jbe     short loc_10042674

.text:10042660

.text:10042660  loc_10042660:               ; CODE XREF: sub_10042520+152  j

.text:10042660                 test    edx, edx

.text:10042662                 jnz     short loc_10042669

.text:10042664                 mov     byte ptr [ecx], 1

.text:10042667                 jmp     short loc_1004266D

.text:10042669  ; 哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪?

.text:10042669

.text:10042669  loc_10042669:               ; CODE XREF: sub_10042520+142  j

.text:10042669                 mov     byte ptr [edx+ecx], 0

.text:1004266D

.text:1004266D  loc_1004266D:               ; CODE XREF: sub_10042520+147  j

.text:1004266D                 add     edx, 1

.text:10042670                 cmp     edx, eax

.text:10042672                 jb      short loc_10042660

.text:10042674

.text:10042674  loc_10042674:               ; CODE XREF: sub_10042520+13E  j

.text:10042674                 add     ecx, eax

.text:10042676                 xor     edx, edx

.text:10042678                 test    eax, eax

.text:1004267A                 jbe     short loc_10042694

.text:1004267C                 lea     esp, [esp+0]

.text:10042680

.text:10042680  loc_10042680:               ; CODE XREF: sub_10042520+172  j

.text:10042680                 test    edx, edx

.text:10042682                 jnz     short loc_10042689

.text:10042684                 mov     byte ptr [ecx], 1

.text:10042687                 jmp     short loc_1004268D

.text:10042689  ; 哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪?

.text:10042689

.text:10042689  loc_10042689:               ; CODE XREF: sub_10042520+162  j

.text:10042689                 mov     byte ptr [edx+ecx], 0

.text:1004268D

.text:1004268D  loc_1004268D:               ; CODE XREF: sub_10042520+167  j

.text:1004268D                 add     edx, 1

.text:10042690                 cmp     edx, eax

.text:10042692                 jb      short loc_10042680

.text:10042694

.text:10042694  loc_10042694:               ; CODE XREF: sub_10042520+15A  j

.text:10042694                 lea     ecx, [ecx+eax*2]

.text:10042697                 xor     eax, eax

.text:10042699                 test    esi, esi

.text:1004269B                 jbe     short loc_100426B4

.text:1004269D                 lea     ecx, [ecx+0]

.text:100426A0

.text:100426A0  loc_100426A0:               ; CODE XREF: sub_10042520+192  j

.text:100426A0                 test    eax, eax

.text:100426A2                 jnz     short loc_100426A9

.text:100426A4                 mov     byte ptr [ecx], 1

.text:100426A7                 jmp     short loc_100426AD

.text:100426A9  ; 哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪?

.text:100426A9

.text:100426A9  loc_100426A9:               ; CODE XREF: sub_10042520+182  j

.text:100426A9                 mov     byte ptr [eax+ecx], 0

.text:100426AD

.text:100426AD  loc_100426AD:               ; CODE XREF: sub_10042520+187  j

.text:100426AD                 add     eax, 1

.text:100426B0                 cmp     eax, esi

.text:100426B2                 jb      short loc_100426A0

.text:100426B4

.text:100426B4  loc_100426B4:               ; CODE XREF: sub_10042520+17B  j

.text:100426B4                 mov     ecx, [esp+18h+dwDataLen]

.text:100426B8                 lea     eax, [esp+18h+hPubKey]

.text:100426BC                 push    eax             ; phKey

.text:100426BD                 push    0               ; dwFlags

.text:100426BF                 push    0               ; hPubKey

.text:100426C1                 push    ecx             ; dwDataLen

.text:100426C2                 push    edi             ; pbData

.text:100426C3                 push    ebp             ; hProv

.text:100426C4                 mov     ebp, ds:CryptImportKey  ; Transfer a cryptographic key

.text:100426C4                                                 ; from a key blob to the CSP

.text:100426CA                 call    ebp ; CryptImportKey ; Transfer a cryptographic key

.text:100426CA                                                 ; from a key blob to the CSP

.text:100426CC                 push    edi             ; hMem

.text:100426CD                 mov     esi, eax

.text:100426CF                 call    ds:LocalFree

.text:100426D5                 mov     eax, [esp+18h+hKey]

.text:100426D9                 test    eax, eax

.text:100426DB                 jz      short loc_100426E4

.text:100426DD                 push    eax             ; hKey

.text:100426DE                 call    ds:CryptDestroyKey

.text:100426E4

.text:100426E4  loc_100426E4:               ; CODE XREF: sub_10042520+1BB  j

.text:100426E4                 test    esi, esi

.text:100426E6                 jnz     short loc_10042717

.text:100426E8

.text:100426E8  loc_100426E8:               ; CODE XREF: sub_10042520+E4  j

.text:100426E8                                                 ; sub_10042520+F1  j ...

.text:100426E8                 mov     eax, BeOS

.text:100426ED                 mov     edx, [eax]

.text:100426EF                 push    offset byte_101A8F68

.text:100426F4                 push    0

.text:100426F6                 push    0FFFFFFFEh

.text:100426F8                 push    eax

.text:100426F9                 call    dword ptr [edx+34h]

.text:100426FC                 mov     eax, [esp+28h+hPubKey]

.text:10042700                 add     esp, 10h

.text:10042703                 test    eax, eax

.text:10042705                 jz      short loc_1004270E

.text:10042707                 push    eax             ; hKey

.text:10042708                 call    ds:CryptDestroyKey

.text:1004270E

.text:1004270E  loc_1004270E:               ; CODE XREF: sub_10042520+1E5  j

.text:1004270E                 pop     edi

.text:1004270F                 pop     esi

.text:10042710                 xor     al, al

.text:10042712                 pop     ebp

.text:10042713                 add     esp, 0Ch
```
