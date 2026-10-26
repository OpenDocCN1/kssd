
```
004011EF   |.  0F8C BA020000     jl 3\_.004014AF

004011F5   |.  3B45 F4           cmp eax,dword ptr ss:[ebp-C]

004011F8   |.  0F8D B1020000     jge 3\_.004014AF

004011FE   |.  50                push eax

004011FF   |.  E8 32220000       call 3\_.00403436

00401204   |.  3BC3              cmp eax,ebx

00401206   |.  59                pop ecx

00401207   |.  8945 F8           mov dword ptr ss:[ebp-8],eax

0040120A   |.  0F84 07010000     je 3\_.00401317

00401210   |.  6A 02             push 2

00401212   |.  53                push ebx

00401213   |.  6A F8             push -8

00401215   |.  895D E8           mov dword ptr ss:[ebp-18],ebx

00401218   |.  58                pop eax

00401219   |.  2B45 08           sub eax,dword ptr ss:[ebp+8]

0040121C   |.  50                push eax

0040121D   |.  57                push edi

0040121E   |.  FFD6              call esi

00401220   |.  83F8 FF           cmp eax,-1

00401223   |.  0F84 7D020000     je 3\_.004014A6

00401229   |.  8B75 F8           mov esi,dword ptr ss:[ebp-8]

0040122C   |.  8D45 E8           lea eax,dword ptr ss:[ebp-18]

0040122F   |.  53                push ebx                                         ; /pOverlapped

00401230   |.  50                push eax                                         ; |pBytesRead

00401231   |.  FF75 08           push dword ptr ss:[ebp+8]                        ; |BytesToRead

00401234   |.  56                push esi                                         ; |Buffer

00401235   |.  57                push edi                                         ; |hFile

00401236   |.  FF15 18604000     call dword ptr ds:[<&kernel32.ReadFile>]         ; \ReadFile

0040123C   |.  85C0              test eax,eax

0040123E   |.  0F84 62020000     je 3\_.004014A6

00401244   |.  8B45 08           mov eax,dword ptr ss:[ebp+8]

00401247   |.  3945 E8           cmp dword ptr ss:[ebp-18],eax

0040124A   |.  0F85 56020000     jnz 3\_.004014A6

00401250   |.  813E A5B79A82     cmp dword ptr ds:[esi],829AB7A5                  ;  再次比较A5B79A82

00401256   |.  0F85 4A020000     jnz 3\_.004014A6                                  ;  这时聪明的你会想到在原文件查找A5B79A8216进制数据

0040125C   |.  8D85 6CFEFFFF     lea eax,dword ptr ss:[ebp-194]                   ;  找到2个地方有一个是末端一个是离末端前B690F0处

00401262   |.  83C6 04           add esi,4                                        ;  用HEXWORKSHOP在空地方插入B690F0大小字节的块，然后可以从原文件粘贴过来

00401265   |.  50                push eax                                         ; /Buffer

00401266   |.  68 04010000       push 104                                         ; |BufSize = 104 (260.)

0040126B   |.  FF15 14604000     call dword ptr ds:[<&kernel32.GetTempPathA>]     ; \GetTempPathA

00401271   |.  85C0              test eax,eax                                     ;  保存运行 成功了！

00401273   |.  75 0C             jnz short 3\_.00401281

00401275   |.  C745 FC 98714000  mov dword ptr ss:[ebp-4],3\_.00407198             ;  ASCII "Can't retrieve the temporary directory!"

0040127C   |.  E9 3E020000       jmp 3\_.004014BF

00401281   |>  8B06              mov eax,dword ptr ds:[esi]

00401283   |.  83C6 04           add esi,4

00401286   |.  50                push eax                                         ; /<%X>

00401287   |.  8D85 70FFFFFF     lea eax,dword ptr ss:[ebp-90]                    ; |

0040128D   |.  68 90714000       push 3\_.00407190                                 ; |Format = "E\_%X"

00401292   |.  50                push eax                                         ; |s

00401293   |.  FF15 B0604000     call dword ptr ds:[<&user32.wsprintfA>]          ; \wsprintfA

00401299   |.  8D85 70FFFFFF     lea eax,dword ptr ss:[ebp-90]

0040129F   |.  50                push eax

004012A0   |.  8D85 6CFEFFFF     lea eax,dword ptr ss:[ebp-194]

004012A6   |.  50                push eax

004012A7   |.  E8 24200000       call 3\_.004032D0

004012AC   |.  83C4 14           add esp,14

004012AF   |.  8D85 6CFEFFFF     lea eax,dword ptr ss:[ebp-194]

004012B5   |.  53                push ebx                                         ; /pSecurity

004012B6   |.  50                push eax                                         ; |Path

004012B7   |.  FF15 10604000     call dword ptr ds:[<&kernel32.CreateDirectoryA>] ; \CreateDirectoryA

004012BD   |.  8D85 6CFEFFFF     lea eax,dword ptr ss:[ebp-194]

004012C3   |.  68 8C714000       push 3\_.0040718C

004012C8   |.  50                push eax

004012C9   |.  E8 02200000       call 3\_.004032D0

004012CE   |.  FF36              push dword ptr ds:[esi]

004012D0   |.  836D 08 0C        sub dword ptr ss:[ebp+8],0C

004012D4   |.  8D7E 04           lea edi,dword ptr ds:[esi+4]

004012D7   |.  FF75 08           push dword ptr ss:[ebp+8]

004012DA   |.  57                push edi

004012DB   |.  E8 39FEFFFF       call 3\_.00401119

004012E0   |.  836D 08 08        sub dword ptr ss:[ebp+8],8

004012E4   |.  8B47 04           mov eax,dword ptr ds:[edi+4]

004012E7   |.  83C4 14           add esp,14

004012EA   |.  395D 08           cmp dword ptr ss:[ebp+8],ebx

004012ED   |.  8945 EC           mov dword ptr ss:[ebp-14],eax

004012F0   |.  0F8E A7010000     jle 3\_.0040149D

004012F6   |.  813F 0D0F3E03     cmp dword ptr ds:[edi],33E0F0D

004012FC   |.  0F85 9B010000     jnz 3\_.0040149D

00401302   |.  3BC3              cmp eax,ebx

00401304   |.  0F8E 93010000     jle 3\_.0040149D

0040130A   |.  50                push eax

0040130B   |.  E8 26210000       call 3\_.00403436

00401310   |.  8BF0              mov esi,eax

00401312   |.  59                pop ecx

00401313   |.  3BF3              cmp esi,ebx

00401315   |.  75 0C             jnz short 3\_.00401323

00401317   |>  C745 FC 74714000  mov dword ptr ss:[ebp-4],3\_.00407174             ;  ASCII "Insufficient memory!"

0040131E   |.  E9 9C010000       jmp 3\_.004014BF

00401323   |>  FF75 08           push dword ptr ss:[ebp+8]

00401326   |.  83C7 08           add edi,8

00401329   |.  8D45 EC           lea eax,dword ptr ss:[ebp-14]

0040132C   |.  57                push edi

0040132D   |.  50                push eax

0040132E   |.  56                push esi

0040132F   |.  E8 E71E0000       call 3\_.0040321B

00401334   |.  83C4 10           add esp,10

00401337   |.  85C0              test eax,eax

00401339   |.  74 13             je short 3\_.0040134E

0040133B   |.  56                push esi

0040133C   |.  E8 EA200000       call 3\_.0040342B

00401341   |.  59                pop ecx

00401342   |.  C745 FC 58714000  mov dword ptr ss:[ebp-4],3\_.00407158             ;  ASCII "Failed to decompress data!"

00401349   |.  E9 71010000       jmp 3\_.004014BF

0040134E   |>  FF75 F8           push dword ptr ss:[ebp-8]

00401351   |.  E8 D5200000       call 3\_.0040342B

00401356   |.  8B45 EC           mov eax,dword ptr ss:[ebp-14]

00401359   |.  59                pop ecx

0040135A   |.  03C6              add eax,esi

0040135C   |.  8975 F8           mov dword ptr ss:[ebp-8],esi

0040135F   |.  3BF0              cmp esi,eax

00401361   |.  8945 F4           mov dword ptr ss:[ebp-C],eax

00401364   |.  885D A4           mov byte ptr ss:[ebp-5C],bl

00401367   |.  0F83 B4000000     jnb 3\_.00401421

0040136D   |>  8BFE              /mov edi,esi

0040136F   |.  56                |push esi

00401370   |.  897D 08           |mov dword ptr ss:[ebp+8],edi

00401373   |.  E8 38200000       |call 3\_.004033B0

00401378   |.  C70424 4C714000   |mov dword ptr ss:[esp],3\_.0040714C              ;  ASCII "krnln.fnr"

0040137F   |.  57                |push edi

00401380   |.  8D7406 01         |lea esi,dword ptr ds:[esi+eax+1]

00401384   |.  E8 47480000       |call 3\_.00405BD0

00401389   |.  59                |pop ecx

0040138A   |.  85C0              |test eax,eax

0040138C   |.  59                |pop ecx

0040138D   |.  74 11             |je short 3\_.004013A0

0040138F   |.  68 40714000       |push 3\_.00407140                                ;  ASCII "krnln.fne"
```
