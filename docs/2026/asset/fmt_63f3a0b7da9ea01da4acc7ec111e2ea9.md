
```
0053506C   .  55            push    ebp

0053506D   .  8BEC          mov     ebp, esp

0053506F   .  B9 05000000   mov     ecx, 5

00535074   >  6A 00         push    0

00535076   .  6A 00         push    0

00535078   .  49            dec     ecx

00535079   .^ 75 F9         jnz     short 00535074

0053507B   .  53            push    ebx

0053507C   .  8BD8          mov     ebx, eax

0053507E   .  33C0          xor     eax, eax

00535080   .  55            push    ebp

00535081   .  68 4F525300   push    0053524F

00535086   .  64:FF30       push    dword ptr fs:[eax]

00535089   .  64:8920       mov     dword ptr fs:[eax], esp

0053508C   .  8D55 F4       lea     edx, dword ptr [ebp-C]

0053508F   .  8B83 0C030000 mov     eax, dword ptr [ebx+30C]

00535095   .  E8 960EF2FF   call    00455F30

0053509A   .  8B45 F4       mov     eax, dword ptr [ebp-C]           ;  //试炼码

0053509D   .  8D55 F8       lea     edx, dword ptr [ebp-8]

005350A0   .  E8 933DEDFF   call    00408E38

005350A5   .  A0 5C525300   mov     al, byte ptr [53525C]

005350AA   .  50            push    eax

005350AB   .  8D45 F0       lea     eax, dword ptr [ebp-10]

005350AE   .  50            push    eax

005350AF   .  33C9          xor     ecx, ecx

005350B1   .  BA 68525300   mov     edx, 00535268                    ;  -

005350B6   .  8B45 F8       mov     eax, dword ptr [ebp-8]

005350B9   .  E8 A293EDFF   call    0040E460                         ;  //去掉试炼码中的"-"

005350BE   .  8B55 F0       mov     edx, dword ptr [ebp-10]          ;  //试炼码

005350C1   .  8D45 F8       lea     eax, dword ptr [ebp-8]

005350C4   .  E8 EFF3ECFF   call    004044B8

005350C9   .  8D55 EC       lea     edx, dword ptr [ebp-14]

005350CC   .  8B83 FC020000 mov     eax, dword ptr [ebx+2FC]

005350D2   .  E8 590EF2FF   call    00455F30

005350D7   .  8B45 EC       mov     eax, dword ptr [ebp-14]          ;  //定单号

005350DA   .  E8 01F6ECFF   call    004046E0

005350DF   .  83F8 08       cmp     eax, 8

005350E2   .  0F85 FC000000 jnz     005351E4                         ;  //定单号长度不是8则跳

005350E8   .  8D55 E4       lea     edx, dword ptr [ebp-1C]

005350EB   .  8B83 FC020000 mov     eax, dword ptr [ebx+2FC]

005350F1   .  E8 3A0EF2FF   call    00455F30

005350F6   .  8B55 E4       mov     edx, dword ptr [ebp-1C]          ;  //定单号

005350F9   .  8D4D E8       lea     ecx, dword ptr [ebp-18]

005350FC   .  8BC3          mov     eax, ebx

005350FE   .  E8 A9020000   call    005353AC                         ;  //算法CALL

00535103   .  8B45 E8       mov     eax, dword ptr [ebp-18]          ;  //注册码

00535106   .  8B55 F8       mov     edx, dword ptr [ebp-8]           ;  //试炼码

00535109   .  E8 1EF7ECFF   call    0040482C                         ;  //比较CALL

0053510E   .  0F85 D0000000 jnz     005351E4                         ;  //关键跳转

00535114   .  B2 01         mov     dl, 1

00535116   .  A1 80C54300   mov     eax, dword ptr [43C580]

0053511B   .  E8 6075F0FF   call    0043C680

00535120   .  8945 FC       mov     dword ptr [ebp-4], eax

00535123   .  33C0          xor     eax, eax

00535125   .  55            push    ebp

00535126   .  68 DD515300   push    005351DD

0053512B   .  64:FF30       push    dword ptr fs:[eax]

0053512E   .  64:8920       mov     dword ptr fs:[eax], esp

00535131   .  33C9          xor     ecx, ecx

00535133   .  BA 74525300   mov     edx, 00535274                    ;  \software\microsoft\icss2

00535138   .  8B45 FC       mov     eax, dword ptr [ebp-4]

0053513B   .  E8 4476F0FF   call    0043C784

00535140   .  8D55 E0       lea     edx, dword ptr [ebp-20]

00535143   .  8B83 FC020000 mov     eax, dword ptr [ebx+2FC]

00535149   .  E8 E20DF2FF   call    00455F30

0053514E   .  8B4D E0       mov     ecx, dword ptr [ebp-20]

00535151   .  BA 98525300   mov     edx, 00535298                    ;  o

00535156   .  8B45 FC       mov     eax, dword ptr [ebp-4]

00535159   .  E8 C277F0FF   call    0043C920

0053515E   .  8B4D F8       mov     ecx, dword ptr [ebp-8]

00535161   .  BA A4525300   mov     edx, 005352A4                    ;  k

00535166   .  8B45 FC       mov     eax, dword ptr [ebp-4]

00535169   .  E8 B277F0FF   call    0043C920

0053516E   .  6A 40         push    40

00535170   .  B9 A8525300   mov     ecx, 005352A8                    ;  registration complete

00535175   .  BA C0525300   mov     edx, 005352C0                    ;  thank you for registering colorschemer studio!  your software is now fully functional.

0053517A   .  A1 90C35300   mov     eax, dword ptr [53C390]

0053517F   .  8B00          mov     eax, dword ptr [eax]

00535181   .  E8 0625F4FF   call    0047768C

00535186   .  8D55 D8       lea     edx, dword ptr [ebp-28]

00535189   .  8B83 FC020000 mov     eax, dword ptr [ebx+2FC]

0053518F   .  E8 9C0DF2FF   call    00455F30

00535194   .  8B4D D8       mov     ecx, dword ptr [ebp-28]

00535197   .  8D45 DC       lea     eax, dword ptr [ebp-24]

0053519A   .  BA 20535300   mov     edx, 00535320                    ;  order number:

0053519F   .  E8 88F5ECFF   call    0040472C

005351A4   .  8B55 DC       mov     edx, dword ptr [ebp-24]

005351A7   .  A1 94C05300   mov     eax, dword ptr [53C094]

005351AC   .  8B00          mov     eax, dword ptr [eax]

005351AE   .  8B80 04030000 mov     eax, dword ptr [eax+304]

005351B4   .  E8 A70DF2FF   call    00455F60

005351B9   .  C683 10030000>mov     byte ptr [ebx+310], 1

005351C0   .  8BC3          mov     eax, ebx

005351C2   .  E8 A1EAF3FF   call    00473C68

005351C7   .  33C0          xor     eax, eax

005351C9   .  5A            pop     edx

005351CA   .  59            pop     ecx

005351CB   .  59            pop     ecx

005351CC   .  64:8910       mov     dword ptr fs:[eax], edx

005351CF   .  68 FC515300   push    005351FC

005351D4   >  8D45 FC       lea     eax, dword ptr [ebp-4]

005351D7   .  E8 1C99EDFF   call    0040EAF8

005351DC   .  C3            retn

005351DD   .^ E9 BEEAECFF   jmp     00403CA0

005351E2   .^ EB F0         jmp     short 005351D4

005351E4   >  6A 10         push    10

005351E6   .  B9 30535300   mov     ecx, 00535330                    ;  invalid license key

005351EB   .  BA 44535300   mov     edx, 00535344                    ;  the license key you have provided is invalid.  please recheck your order number and registration key.

005351F0   .  A1 90C35300   mov     eax, dword ptr [53C390]

005351F5   .  8B00          mov     eax, dword ptr [eax]

005351F7   .  E8 9024F4FF   call    0047768C

005351FC   .  33C0          xor     eax, eax

005351FE   .  5A            pop     edx

005351FF   .  59            pop     ecx

00535200   .  59            pop     ecx

00535201   .  64:8910       mov     dword ptr fs:[eax], edx

00535204   .  68 56525300   push    00535256

00535209   >  8D45 D8       lea     eax, dword ptr [ebp-28]

0053520C   .  E8 0FF2ECFF   call    00404420

00535211   .  8D45 DC       lea     eax, dword ptr [ebp-24]

00535214   .  E8 07F2ECFF   call    00404420

00535219   .  8D45 E0       lea     eax, dword ptr [ebp-20]

0053521C   .  BA 02000000   mov     edx, 2

00535221   .  E8 1EF2ECFF   call    00404444

00535226   .  8D45 E8       lea     eax, dword ptr [ebp-18]

00535229   .  E8 F2F1ECFF   call    00404420

0053522E   .  8D45 EC       lea     eax, dword ptr [ebp-14]

00535231   .  E8 EAF1ECFF   call    00404420

00535236   .  8D45 F0       lea     eax, dword ptr [ebp-10]

00535239   .  E8 E2F1ECFF   call    00404420

0053523E   .  8D45 F4       lea     eax, dword ptr [ebp-C]

00535241   .  E8 DAF1ECFF   call    00404420

00535246   .  8D45 F8       lea     eax, dword ptr [ebp-8]

00535249   .  E8 D2F1ECFF   call    00404420

0053524E   .  C3            retn

0053524F   .^ E9 4CEAECFF   jmp     00403CA0

00535254   .^ EB B3         jmp     short 00535209

00535256   .  5B            pop     ebx

00535257   .  8BE5          mov     esp, ebp

00535259   .  5D            pop     ebp

0053525A   .  C3            retn
```
