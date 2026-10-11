
```
004093A0  /$  81EC 58020000  sub     esp, 258
004093A6  |.  A1 04804100   mov     eax, dword ptr [418004]
004093AB  |.  33C4          xor     eax, esp
004093AD  |.  898424 540200>mov     dword ptr [esp+254], eax
004093B4  |.  8B8424 5C0200>mov     eax, dword ptr [esp+25C]
004093BB  |.  53            push    ebx
004093BC  |.  55            push    ebp
004093BD  |.  56            push    esi
004093BE  |.  57            push    edi
004093BF  |.  8D4C24 18     lea     ecx, dword ptr [esp+18]
004093C3  |.  51            push    ecx                       ; /pBufferSize
004093C4  |.  8D5424 60     lea     edx, dword ptr [esp+60]     ; |
004093C8  |.  52            push    edx                       ; |Buffer
004093C9  |.  894424 1C     mov     dword ptr [esp+1C], eax     ; |
004093CD  |.  C74424 20 040>mov     dword ptr [esp+20], 104     ; |
004093D5  |.  FF15 98514100 call    dword ptr [<&KERNEL32.GetCompute>; \GetComputerNameW
004093DB  |.  8D4424 5C     lea     eax, dword ptr [esp+5C]     ;  取得计算机名 unicode
004093DF  |.  50            push    eax                       ; /String
004093E0  |.  66:C74424 5C >mov     word ptr [esp+5C], 0        ; |
004093E7  |.  FF15 94514100 call    dword ptr [<&KERNEL32.lstrlenW>] ; \计算机名长度 2
004093ED  |.  83F8 0E       cmp     eax, 0E                   ;  计算机名长度小于 14 位
004093F0  |.  7C 14         jl      short 00409406
004093F2  |.  6A 0E         push    0E                        ; /n = E (14.)
004093F4  |.  8D4C24 60     lea     ecx, dword ptr [esp+60]     ; |
004093F8  |.  51            push    ecx                       ; |String2
004093F9  |.  8D5424 44     lea     edx, dword ptr [esp+44]     ; |
004093FD  |.  52            push    edx                       ; |String1
004093FE  |.  FF15 90514100 call    dword ptr [<&KERNEL32.lstrcpynW>>; \lstrcpynW
00409404  |.  EB 3B         jmp     short 00409441
00409406  |>  33D2          xor     edx, edx
00409408  |.  8D1C00        lea     ebx, dword ptr [eax+eax]    ;  ebx=len(name)*2
0040940B  |.  8D7C24 3C     lea     edi, dword ptr [esp+3C]
0040940F  |.  90            nop
00409410  |>  33C9          /xor    ecx, ecx
00409412  |.  85C0          |test   eax, eax
00409414  |.  7E 22         |jle     short 00409438
00409416  |.  8BF7          |mov     esi, edi
00409418  |>  8D2C11        |/lea    ebp, dword ptr [ecx+edx]
0040941B  |.  83FD 0E       ||cmp    ebp, 0E
0040941E  |.  7D 0E         ||jge    short 0040942E
00409420  |.  66:8B6C4C 5C  ||mov    bp, word ptr [esp+ecx*2+5C]   ;  取得 name 字符 1
00409425  |.  66:03EA       ||add    bp, dx
00409428  |.  66:03E9       ||add    bp, cx                    ;  +计数器
0040942B  |.  66:892E       ||mov    word ptr [esi], bp
0040942E  |>  83C1 01       ||add    ecx, 1                    ;  计数器+1
00409431  |.  83C6 02       ||add    esi, 2
00409434  |.  3BC8          ||cmp    ecx, eax
00409436  |.^ 7C E0         |\jl     short 00409418
00409438  |>  03D0          |add     edx, eax                  ;  eax=len(name)
0040943A  |.  03FB          |add     edi, ebx
0040943C  |.  83FA 0E       |cmp     edx, 0E                   ;  变为 14 位字符串 M
0040943F  |.^ 7C CF         \jl     short 00409410
00409441  |>  C64424 38 00  mov     byte ptr [esp+38], 0
00409446  |.  33D2          xor     edx, edx
00409448  |.  EB 06         jmp     short 00409450
0040944A  |   8D9B 00000000 lea     ebx, dword ptr [ebx]
00409450  |>  8A4414 3D     /mov    al, byte ptr [esp+edx+3D]     ;  取字符串 M 字符
00409454  |.  024414 3C     |add    al, byte ptr [esp+edx+3C]
00409458  |.  8AC8          |mov    cl, al
0040945A  |.  C0E8 04       |shr    al, 4                   ;  右移 4 位
0040945D  |.  0FBEC0        |movsx  eax, al
00409460  |.  83C0 FF       |add    eax, -1                 ;  -1
00409463  |.  80E1 0F       |and    cl, 0F                  ;  字符串 M 字符保留个位
00409466  |.  83F8 0E       |cmp    eax, 0E                 ;  Switch (cases 0..E)
00409469  |.  77 43         |ja     short 004094AE          ;  大于 E 则=0
0040946B  |.  FF2485 B89540>|jmp    dword ptr [eax*4+4095B8]
00409472  |>  B0 66         |mov    al, 66                  ;  Case E of switch 00409466
00409474  |.  EB 3A         |jmp    short 004094B0
00409476  |>  B0 65         |mov    al, 65                  ;  Case D of switch 00409466
00409478  |.  EB 36         |jmp    short 004094B0
0040947A  |>  B0 64         |mov    al, 64                  ;  Case C of switch 00409466
0040947C  |.  EB 32         |jmp    short 004094B0
0040947E  |>  B0 63         |mov    al, 63                  ;  Case B of switch 00409466
00409480  |.  EB 2E         |jmp    short 004094B0
00409482  |>  B0 62         |mov    al, 62                  ;  Case A of switch 00409466
00409484  |.  EB 2A         |jmp    short 004094B0
00409486  |>  B0 61         |mov    al, 61                  ;  Case 9 of switch 00409466
00409488  |.  EB 26         |jmp    short 004094B0
0040948A  |>  B0 39         |mov    al, 39                  ;  Case 8 of switch 00409466
0040948C  |.  EB 22         |jmp    short 004094B0
0040948E  |>  B0 38         |mov    al, 38                  ;  Case 7 of switch 00409466
00409490  |.  EB 1E         |jmp    short 004094B0
00409492  |>  B0 37         |mov    al, 37                  ;  Case 6 of switch 00409466
00409494  |.  EB 1A         |jmp    short 004094B0
00409496  |>  B0 36         |mov    al, 36                  ;  Case 5 of switch 00409466
00409498  |.  EB 16         |jmp    short 004094B0
0040949A  |>  B0 35         |mov    al, 35                  ;  Case 4 of switch 00409466
0040949C  |.  EB 12         |jmp    short 004094B0
0040949E  |>  B0 34         |mov    al, 34                  ;  Case 3 of switch 00409466
004094A0  |.  EB 0E         |jmp    short 004094B0
004094A2  |>  B0 33         |mov    al, 33                  ;  Case 2 of switch 00409466
004094A4  |.  EB 0A         |jmp    short 004094B0
004094A6  |>  B0 32         |mov    al, 32                  ;  Case 1 of switch 00409466
004094A8  |.  EB 06         |jmp    short 004094B0
004094AA  |>  B0 31         |mov    al, 31                  ;  Case 0 of switch 00409466
004094AC  |.  EB 02         |jmp    short 004094B0
004094AE  |>  B0 30         |mov    al, 30                  ;  Default case of switch 00409466
004094B0  |>  884414 1C     |mov    byte ptr [esp+edx+1C], al
004094B4  |.  0FBEC1        |movsx  eax, cl
004094B7  |.  83C0 FF       |add    eax, -1                 ;  Switch (cases 1..F)
004094BA  |.  83F8 0E       |cmp    eax, 0E
004094BD  |.  77 43         |ja     short 00409502
004094BF  |.  FF2485 F49540>|jmp    dword ptr [eax*4+4095F4]
004094C6  |>  B0 66         |mov    al, 66                  ;  Case F of switch 004094B7
004094C8  |.  EB 3A         |jmp    short 00409504
004094CA  |>  B0 65         |mov    al, 65                  ;  Case E of switch 004094B7
004094CC  |.  EB 36         |jmp    short 00409504
004094CE  |>  B0 64         |mov    al, 64                  ;  Case D of switch 004094B7
004094D0  |.  EB 32         |jmp    short 00409504
004094D2  |>  B0 63         |mov    al, 63                  ;  Case C of switch 004094B7
004094D4  |.  EB 2E         |jmp    short 00409504
004094D6  |>  B0 62         |mov    al, 62                  ;  Case B of switch 004094B7
004094D8  |.  EB 2A         |jmp    short 00409504
004094DA  |>  B0 61         |mov    al, 61                  ;  Case A of switch 004094B7
004094DC  |.  EB 26         |jmp    short 00409504
004094DE  |>  B0 39         |mov    al, 39                  ;  Case 9 of switch 004094B7
004094E0  |.  EB 22         |jmp    short 00409504
004094E2  |>  B0 38         |mov    al, 38                  ;  Case 8 of switch 004094B7
004094E4  |.  EB 1E         |jmp    short 00409504
004094E6  |>  B0 37         |mov    al, 37                  ;  Case 7 of switch 004094B7
004094E8  |.  EB 1A         |jmp    short 00409504
004094EA  |>  B0 36         |mov    al, 36                  ;  Case 6 of switch 004094B7
004094EC  |.  EB 16         |jmp    short 00409504
004094EE  |>  B0 35         |mov    al, 35                  ;  Case 5 of switch 004094B7
004094F0  |.  EB 12         |jmp    short 00409504
004094F2  |>  B0 34         |mov    al, 34                  ;  Case 4 of switch 004094B7
004094F4  |.  EB 0E         |jmp    short 00409504
004094F6  |>  B0 33         |mov    al, 33                  ;  Case 3 of switch 004094B7
004094F8  |.  EB 0A         |jmp    short 00409504
004094FA  |>  B0 32         |mov    al, 32                  ;  Case 2 of switch 004094B7
004094FC  |.  EB 06         |jmp    short 00409504
004094FE  |>  B0 31         |mov    al, 31                  ;  Case 1 of switch 004094B7
00409500  |.  EB 02         |jmp    short 00409504
00409502  |>  B0 30         |mov    al, 30                  ;  Default case of switch 004094B7
00409504  |>  884414 1D     |mov    byte ptr [esp+edx+1D], al
00409508  |.  83C2 02       |add    edx, 2
0040950B  |.  83FA 1C       |cmp    edx, 1C                 ;  28 位
0040950E  |.^ 0F8C 3CFFFFFF \jl     00409450
00409514  |.  8B4424 14     mov     eax, dword ptr [esp+14]
00409518  |.  8D4C24 1C     lea     ecx, dword ptr [esp+1C]
0040951C  |.  51            push    ecx                       ; /String
0040951D  |.  C600 00       mov     byte ptr [eax], 0         ; |
00409520  |.  33F6          xor     esi, esi                  ; |
00409522  |.  FF15 C0514100 call    dword ptr [<&KERNEL32.lstrlenA>] ; \lstrlenA
00409528  |.  85C0          test    eax, eax                  ;  取得长度 1c
0040952A  |.  7E 73         jle     short 0040959F
0040952C  |.  8B2D DC514100 mov     ebp, dword ptr [<&KERNEL32.lstrc>;  kernel32.lstrcatA
00409532  |.  8D5C24 1C     lea     ebx, dword ptr [esp+1C]
00409536  |.  83EB 01       sub     ebx, 1
00409539  |.  8DA424 000000>lea     esp, dword ptr [esp]
00409540  |>  8B4C24 14     /mov    ecx, dword ptr [esp+14]
00409544  |.  8A5433 01     |mov    dl, byte ptr [ebx+esi+1]
00409548  |.  8D4424 10     |lea    eax, dword ptr [esp+10]
0040954C  |.  8D7E 01       |lea    edi, dword ptr [esi+1]
0040954F  |.  50            |push   eax
00409550  |.  51            |push   ecx
00409551  |.  885424 18     |mov    byte ptr [esp+18], dl
00409555  |.  C64424 19 00  |mov    byte ptr [esp+19], 0
0040955A  |.  FFD5          |call   ebp
0040955C  |.  85F6          |test   esi, esi
0040955E  |.  74 2E         |je     short 0040958E
00409560  |.  8D5424 1C     |lea    edx, dword ptr [esp+1C]
00409564  |.  52            |push   edx                       ; /String
00409565  |.  FF15 C0514100 |call   dword ptr [<&KERNEL32.lstrlenA>>; \lstrlenA
0040956B  |.  83E8 01       |sub    eax, 1
0040956E  |.  3BF0          |cmp    esi, eax
00409570  |.  74 1C         |je     short 0040958E
00409572  |.  8BC7          |mov    eax, edi
00409574  |.  25 03000080   |and    eax, 80000003
00409579  |.  79 05         |jns    short 00409580
0040957B  |.  48            |dec    eax
0040957C  |.  83C8 FC       |or     eax, FFFFFFFC
0040957F  |.  40            |inc    eax
00409580  |>  75 0C         |jnz    short 0040958E
00409582  |.  8B4C24 14     |mov    ecx, dword ptr [esp+14]
00409586  |.  68 7C694100   |push   0041697C
0040958B  |.  51            |push   ecx
0040958C  |.  FFD5          |call   ebp
0040958E  |>  8D5424 1C     |lea    edx, dword ptr [esp+1C]
00409592  |.  52            |push   edx                       ; /String
00409593  |.  8BF7          |mov    esi, edi                  ; |
00409595  |.  FF15 C0514100 |call   dword ptr [<&KERNEL32.lstrlenA>>; \lstrlenA
0040959B  |.  3BF0          |cmp    esi, eax
0040959D  |.^ 7C A1         \jl    short 00409540
0040959F  |>  8B8C24 640200>mov    ecx, dword ptr [esp+264]
004095A6  |.  5F            pop     edi
004095A7  |.  5E            pop     esi
004095A8  |.  5D            pop     ebp
004095A9  |.  5B            pop     ebx
004095AA  |.  33CC          xor     ecx, esp
004095AC  |.  E8 021E0000   call    0040B3B3                  ;  四个一组加上“-”
004095B1  |.  81C4 58020000 add    esp, 258
004095B7  \.  C3            retn
```
