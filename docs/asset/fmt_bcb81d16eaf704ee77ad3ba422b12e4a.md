
```
.text:00489489 -60C              call    ds:\_\_vbaFreeVar ; Indirect Call Near Procedure  

.text:00489489   

.text:0048948F -60C              jmp     loc\_48955F      ; Jump  

.text:0048948F   

.text:00489494     ; ---------------------------------------------------------------------------  

.text:00489494   

.text:00489494     loc\_489494:                       ; CODE XREF: sub\_488E70+5A3j  

.text:00489494 -618              cmp     esi, 9          ; Compare Two Operands  

.text:00489497 -618              mov     \[ebp+var\_108\], offset dword\_436AF4  

.text:004894A1 -618              mov     \[ebp+var\_110\], 8  

.text:004894AB -618              jb      short loc\_4894AF ; Jump if Below (CF=1)  

.text:004894AB   

.text:004894AD -618              call    edi ; \_\_vbaStrVarMove ; Indirect Call Near Procedure  

.text:004894AD   

.text:004894AF   

.text:004894AF     loc\_4894AF:                       ; CODE XREF: sub\_488E70+63Bj  

.text:004894AF -618              mov     ecx, \[ebp+var\_30\]  

.text:004894B2 -618              lea     eax, \[ebp+var\_100\] ; Load Effective Address  

.text:004894B8 -618              push    eax  

.text:004894B9 -614              mov     \[ebp+var\_100\], 4003h  

.text:004894C3 -614              lea     edx, \[ecx+esi\*4\] ; Load Effective Address  

.text:004894C6 -614              lea     ecx, \[ebp+var\_D0\] ; Load Effective Address  

.text:004894CC -614              push    ecx  

.text:004894CD -610              mov     \[ebp+var\_F8\], edx  

.text:004894D3 -610              call    ds:rtcVarStrFromVar ; Indirect Call Near Procedure  

.text:004894D3   

.text:004894D9 -610              lea     edx, \[ebp+var\_D0\] ; Load Effective Address  

.text:004894DF -610              lea     eax, \[ebp+var\_E0\] ; Load Effective Address  

.text:004894E5 -610              push    edx  

.text:004894E6 -60C              push    eax  

.text:004894E7 -608              call    ds:rtcTrimVar   ; Indirect Call Near Procedure  

.text:004894E7   

.text:004894ED -608              cmp     esi, 9          ; Compare Two Operands  

.text:004894F0 -608              jb      short loc\_4894F4 ; Jump if Below (CF=1)  

.text:004894F0   

.text:004894F2 -608              call    edi ; \_\_vbaStrVarMove ; Indirect Call Near Procedure  

.text:004894F2   

.text:004894F4   

.text:004894F4     loc\_4894F4:                       ; CODE XREF: sub\_488E70+680j  

.text:004894F4 -608              lea     ecx, \[ebp+var\_110\] ; Load Effective Address  

.text:004894FA -608              lea     edx, \[ebp+var\_E0\] ; Load Effective Address  

.text:00489500 -608              push    ecx  

.text:00489501 -604              lea     eax, \[ebp+var\_F0\] ; Load Effective Address  

.text:00489507 -604              push    edx  

.text:00489508 -600              push    eax  

.text:00489509 -5FC              call    ds:\_\_vbaVarCat  ; Indirect Call Near Procedure  

.text:00489509   

.text:0048950F -5FC              mov     edi, ds:\_\_vbaStrVarMove  

.text:00489515 -5FC              push    eax  

.text:00489516 -5F8              call    edi ; \_\_vbaStrVarMove ; Indirect Call Near Procedure  

.text:00489516   

.text:00489518 -5F8              mov     edx, eax  

.text:0048951A -5F8              lea     ecx, \[ebp+var\_C0\] ; Load Effective Address  

.text:00489520 -5F8              call    ebx ; \_\_vbaStrMove ; Indirect Call Near Procedure  

.text:00489520   

.text:00489522 -5F8              mov     ecx, \[ebp+var\_A0\]  

.text:00489528 -5F8              mov     edx, eax  

.text:0048952A -5F8              lea     ecx, \[ecx+esi\*4\] ; Load Effective Address  

.text:0048952D -5F8              call    ds:\_\_vbaStrCopy ; Indirect Call Near Procedure  

.text:0048952D   

.text:00489533 -5F8              lea     ecx, \[ebp+var\_C0\] ; Load Effective Address  

.text:00489539 -5F8              call    ds:\_\_vbaFreeStr ; Indirect Call Near Procedure  

.text:00489539   

.text:0048953F -5F8              lea     edx, \[ebp+var\_F0\] ; Load Effective Address  

.text:00489545 -5F8              lea     eax, \[ebp+var\_E0\] ; Load Effective Address  

.text:0048954B -5F8              push    edx  

.text:0048954C -5F4              lea     ecx, \[ebp+var\_D0\] ; Load Effective Address  

.text:00489552 -5F4              push    eax  

.text:00489553 -5F0              push    ecx  

.text:00489554 -5EC              push    3  

.text:00489556 -5E8              call    ds:\_\_vbaFreeVarList ; Indirect Call Near Procedure  

.text:00489556   

.text:0048955C -5F4              add     esp, 10h        ; Add  

.text:0048955F   

.text:0048955F     loc\_48955F:                       ; CODE XREF: sub\_488E70+61Fj  

.text:0048955F -604              mov     edx, \[ebp+var\_1C\]  

.text:00489562 -604              cmp     esi, 9          ; Compare Two Operands  

.text:00489565 -604              mov     \[ebp+var\_108\], edx  

.text:0048956B -604              mov     \[ebp+var\_110\], 8  

.text:00489575 -604              jb      short loc\_48957D ; Jump if Below (CF=1)  

.text:00489575   

.text:00489577 -604              call    ds:\_\_vbaGenerateBoundsError ; Indirect Call Near Procedure  

.text:00489577   

.text:0048957D   

.text:0048957D     loc\_48957D:                       ; CODE XREF: sub\_488E70+705j  

.text:0048957D -604              mov     eax, \[ebp+var\_A0\]  

.text:00489583 -604              lea     edx, \[ebp+var\_100\] ; Load Effective Address  

.text:00489589 -604              push    edx  

.text:0048958A -600              mov     \[ebp+var\_100\], 4008h  

.text:00489594 -600              lea     ecx, \[eax+esi\*4\] ; Load Effective Address  

.text:00489597 -600              lea     eax, \[ebp+var\_D0\] ; Load Effective Address  

.text:0048959D -600              push    eax  

.text:0048959E -5FC              mov     \[ebp+var\_F8\], ecx  

.text:004895A4 -5FC              call    ds:rtcTrimVar   ; Indirect Call Near Procedure  

.text:004895A4   

.text:004895AA -5FC              lea     ecx, \[ebp+var\_110\] ; Load Effective Address  

.text:004895B0 -5FC              lea     edx, \[ebp+var\_D0\] ; Load Effective Address  

.text:004895B6 -5FC              push    ecx  

.text:004895B7 -5F8              lea     eax, \[ebp+var\_E0\] ; Load Effective Address  

.text:004895BD -5F8              push    edx  

.text:004895BE -5F4              push    eax  

.text:004895BF -5F0              call    ds:\_\_vbaVarCat  ; Indirect Call Near Procedure  

.text:004895BF   

.text:004895C5 -5F0              push    eax  

.text:004895C6 -5EC              call    edi ; \_\_vbaStrVarMove ; Indirect Call Near Procedure  

.text:004895C6   

.text:004895C8 -5EC              mov     edx, eax  

.text:004895CA -5EC              lea     ecx, \[ebp+var\_1C\] ; Load Effective Address  

.text:004895CD -5EC              call    ebx ; \_\_vbaStrMove ; Indirect Call Near Procedure  

.text:004895CD   

.text:004895CF -5EC              lea     ecx, \[ebp+var\_E0\] ; Load Effective Address  

.text:004895D5 -5EC              lea     edx, \[ebp+var\_D0\] ; Load Effective Address  

.text:004895DB -5EC              push    ecx  

.text:004895DC -5E8              push    edx  

.text:004895DD -5E4              push    2  

.text:004895DF -5E0              call    ds:\_\_vbaFreeVarList ; Indirect Call Near Procedure  

.text:004895DF   

.text:004895E5 -5EC              mov     ecx, \[ebp+var\_18\]  

.text:004895E8 -5EC              mov     eax, 1  

.text:004895ED -5EC              add     esp, 0Ch        ; Add  

.text:004895F0 -5F8              add     eax, ecx        ; Add  
```
