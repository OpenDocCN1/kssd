
```asm
:0040363A 85C9                   test ecx, ecx
:0040363C 7E28                   jle 00403666

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00403660(C)
|

:0040363E 8B842488000000         mov eax, dword ptr [esp+00000088]
:00403645 8A1C06                 mov bl, byte ptr [esi+eax]
:00403648 B8F73E1600             mov eax, 00163EF7
:0040364D 0FBEFB                 movsx edi, bl
:00403650 99                     cdq
:00403651 F7FF                   idiv edi
:00403653 0FBED3                 movsx edx, bl
:00403656 03C6                   add eax, esi
:00403658 0FAFC2                 imul eax, edx
:0040365B 03E8                   add ebp, eax
:0040365D 46                     inc esi
:0040365E 3BF1                   cmp esi, ecx
:00403660 7CDC                   jl 0040363E
:00403662 8B5C2410               mov ebx, dword ptr [esp+10]

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:0040363C(C)
|

:00403666 81C5EDD98500           add ebp, 0085D9ED
:0040366C 8D442414               lea eax, dword ptr [esp+14]
:00403670 55                     push ebp

* Possible StringData Ref from Data Obj -> "%i"
|

:00403671 6854354100             push 00413554
:00403676 50                     push eax

* Reference To: MSVCRT.sprintf, Ord:02B2h
|

:00403677 FF156C034100           Call dword ptr [0041036C]
:0040367D 83C40C                 add esp, 0000000C
:00403680 8BF3                   mov esi, ebx
:00403682 8D442414               lea eax, dword ptr [esp+14]
:00403686 5D                     pop ebp

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:004036A9(C)
|

:00403687 8A10                   mov dl, byte ptr [eax] 〈-真注册码
:00403689 8A1E                   mov bl, byte ptr [esi] 〈-你输入的注册码
:0040368B 8ACA                   mov cl, dl
:0040368D 3AD3                   cmp dl, bl
:0040368F 751E                   jne 004036AF
:00403691 84C9                   test cl, cl
:00403693 7416                   je 004036AB
:00403695 8A5001                 mov dl, byte ptr [eax+01]
:00403698 8A5E01                 mov bl, byte ptr [esi+01]
:0040369B 8ACA                   mov cl, dl
:0040369D 3AD3                   cmp dl, bl
:0040369F 750E                   jne 004036AF
:004036A1 83C002                 add eax, 00000002
:004036A4 83C602                 add esi, 00000002
:004036A7 84C9                   test cl, cl
:004036A9 75DC                   jne 00403687
4dd466d0e1e0cc123afbe504da2462ca
```
