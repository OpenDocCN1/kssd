

# 3D 文本出现次数统计脱壳（自校验）破解

> 原文：[`www.pediy.com/kssd/pediy07/pediy7-568.htm`](https://www.pediy.com/kssd/pediy07/pediy7-568.htm)

3D 文本出现次数统计脱壳（自校验）破解  
软件下载:http://www.5icrack.com/bbs/viewthread.php?tid=105656&pid=1301337&page=1#pid1301337  
peid 区断显示 nsp0，nsp1  
打开信箱有人求我破这个软件,我出于好奇破了一下.PEID 查看加了个未知压缩壳.手脱后有自校验.  
【调试环境】：Win2003、OllyDBG、PEiD、ImportREC，HEXWORKSHOP。  
第一步脱衣：  
00442F61 3>  9C                  pushfd              ；OD 载入  
00442F62     60                  pushad  
00442F63     E8 00000000         call 3D 文本出.00442F68  
00442F68     5D                  pop ebp  
00442F69     B8 B1854000         mov eax,3D 文本出.004085B1  
00442F6E     2D AA854000         sub eax,3D 文本出.004085AA  
00442F73     2BE8                sub ebp,eax  
00442F75     8DB5 CBFEFFFF       lea esi,dword ptr ss:\[ebp-135\]  
00442F7B     8B06                mov eax,dword ptr ds:\[esi\]  
00442F7D     83F8 00             cmp eax,0  
00442F80     74 11               je short 3D 文本出.00442F93  
00442F82     8DB5 D7FEFFFF       lea esi,dword ptr ss:\[ebp-129\]  
00442F88     8B06                mov eax,dword ptr ds:\[esi\]  
00442F8A     83F8 01             cmp eax,1  
00442F8D     0F84 F1010000       je 3D 文本出.00443184  
00442F93     C706 01000000       mov dword ptr ds:\[esi\],1  
00442F99     8BD5                mov edx,ebp  
00442F9B     8B85 A7FEFFFF       mov eax,dword ptr ss:\[ebp-159\]  
00442FA1     2BD0                sub edx,eax  
00442FA3     8995 A7FEFFFF       mov dword ptr ss:\[ebp-159\],edx  
00442FA9     0195 BFFEFFFF       add dword ptr ss:\[ebp-141\],edx  
00442FAF     8DB5 DBFEFFFF       lea esi,dword ptr ss:\[ebp-125\]  
00442FB5     0116                add dword ptr ds:\[esi\],edx  
00442FB7     8B36                mov esi,dword ptr ds:\[esi\]  
00442FB9     8BFD                mov edi,ebp  
00442FBB     60                  pushad  
00442FBC     6A 40               push 40  
00442FBE     68 00100000         push 1000  
00442FC3     68 00100000         push 1000  
00442FC8     6A 00               push 0  
00442FCA     FF95 FBFEFFFF       call dword ptr ss:\[ebp-105\]  
00442FD0     85C0                test eax,eax  
00442FD2     0F84 06030000       je 3D 文本出.004432DE  
00442FD8     8985 BBFEFFFF       mov dword ptr ss:\[ebp-145\],eax  
00442FDE     E8 00000000         call 3D 文本出.00442FE3  
00442FE3     5B                  pop ebx  
00442FE4     B9 2F894000         mov ecx,3D 文本出.0040892F  
00442FE9     81E9 2C864000       sub ecx,3D 文本出.0040862C  
00442FEF     03D9                add ebx,ecx  
00442FF1     50                  push eax  
00442FF2     53                  push ebx  
00442FF3     E8 3D020000         call 3D 文本出.00443235  
00442FF8     61                  popad  
00442FF9     03BD 9FFEFFFF       add edi,dword ptr ss:\[ebp-161\]  
00442FFF     8BDF                mov ebx,edi  
00443001     833F 00             cmp dword ptr ds:\[edi\],0  
00443004     75 0A               jnz short 3D 文本出.00443010  
00443006     83C7 04             add edi,4  
00443009     B9 00000000         mov ecx,0  
0044300E     EB 16               jmp short 3D 文本出.00443026  
00443010     B9 01000000         mov ecx,1  
00443015     033B                add edi,dword ptr ds:\[ebx\]  
00443017     83C3 04             add ebx,4  
0044301A     833B 00             cmp dword ptr ds:\[ebx\],0  
0044301D     74 2D               je short 3D 文本出.0044304C  
0044301F     0113                add dword ptr ds:\[ebx\],edx  
00443021     8B33                mov esi,dword ptr ds:\[ebx\]  
00443023     037B 04             add edi,dword ptr ds:\[ebx+4\]  
00443026     57                  push edi  
00443027     51                  push ecx  
00443028     52                  push edx  
00443029     53                  push ebx  
0044302A     FFB5 FFFEFFFF       push dword ptr ss:\[ebp-101\]  
00443030     FFB5 FBFEFFFF       push dword ptr ss:\[ebp-105\]  
00443036     56                  push esi  
00443037     57                  push edi  
00443038     FF95 BBFEFFFF       call dword ptr ss:\[ebp-145\]  
0044303E     5B                  pop ebx  
0044303F     5A                  pop edx  
00443040     59                  pop ecx  
00443041     5F                  pop edi  
00443042     83F9 00             cmp ecx,0  
00443045     74 05               je short 3D 文本出.0044304C  
00443047     83C3 08             add ebx,8  
0044304A   ^ EB CE               jmp short 3D 文本出.0044301A  
0044304C     68 00800000         push 8000  
00443051     6A 00               push 0  
00443053     FFB5 BBFEFFFF       push dword ptr ss:\[ebp-145\]  
00443059     FF95 FFFEFFFF       call dword ptr ss:\[ebp-101\]  
0044305F     8DB5 BFFEFFFF       lea esi,dword ptr ss:\[ebp-141\]  
00443065     8B4E 04             mov ecx,dword ptr ds:\[esi+4\]  
00443068     8D56 08             lea edx,dword ptr ds:\[esi+8\]  
0044306B     8B36                mov esi,dword ptr ds:\[esi\]  
0044306D     8BFE                mov edi,esi  
0044306F     83F9 00             cmp ecx,0  
00443072     74 3F               je short 3D 文本出.004430B3  
00443074     8A07                mov al,byte ptr ds:\[edi\]  
00443076     47                  inc edi  
00443077     2C E8               sub al,0E8  
00443079     3C 01               cmp al,1  
0044307B   ^ 77 F7               ja short 3D 文本出.00443074  
0044307D     8B07                mov eax,dword ptr ds:\[edi\]  
0044307F     807A 01 00          cmp byte ptr ds:\[edx+1\],0  
00443083     74 14               je short 3D 文本出.00443099  
00443085     8A1A                mov bl,byte ptr ds:\[edx\]  
00443087     381F                cmp byte ptr ds:\[edi\],bl  
00443089   ^ 75 E9               jnz short 3D 文本出.00443074  
0044308B     8A5F 04             mov bl,byte ptr ds:\[edi+4\]  
0044308E     66:C1E8 08          shr ax,8  
00443092     C1C0 10             rol eax,10  
00443095     86C4                xchg ah,al  
00443097     EB 0A               jmp short 3D 文本出.004430A3  
00443099     8A5F 04             mov bl,byte ptr ds:\[edi+4\]  
0044309C     86C4                xchg ah,al  
0044309E     C1C0 10             rol eax,10  
004430A1     86C4                xchg ah,al  
004430A3     2BC7                sub eax,edi  
004430A5     03C6                add eax,esi  
004430A7     8907                mov dword ptr ds:\[edi\],eax  
004430A9     83C7 05             add edi,5  
004430AC     80EB E8             sub bl,0E8  
004430AF     8BC3                mov eax,ebx  
004430B1   ^ E2 C6               loopd short 3D 文本出.00443079  
004430B3     E8 D3000000         call 3D 文本出.0044318B  
004430B8     8D8D CBFEFFFF       lea ecx,dword ptr ss:\[ebp-135\]  
004430BE     8B41 04             mov eax,dword ptr ds:\[ecx+4\]  
004430C1     83F8 00             cmp eax,0  
004430C4     0F84 81000000       je 3D 文本出.0044314B  
004430CA     8BF2                mov esi,edx  
004430CC     2B71 08             sub esi,dword ptr ds:\[ecx+8\]  
004430CF     74 7A               je short 3D 文本出.0044314B  
004430D1     8971 08             mov dword ptr ds:\[ecx+8\],esi  
004430D4     8B01                mov eax,dword ptr ds:\[ecx\]  
004430D6     8DB5 DBFEFFFF       lea esi,dword ptr ss:\[ebp-125\]  
004430DC     8B36                mov esi,dword ptr ds:\[esi\]  
004430DE     8D5E FC             lea ebx,dword ptr ds:\[esi-4\]  
004430E1     83F8 01             cmp eax,1  
004430E4     74 0A               je short 3D 文本出.004430F0  
004430E6     8BFA                mov edi,edx  
004430E8     0379 04             add edi,dword ptr ds:\[ecx+4\]  
004430EB     8B49 08             mov ecx,dword ptr ds:\[ecx+8\]  
004430EE     EB 08               jmp short 3D 文本出.004430F8  
004430F0     8BFE                mov edi,esi  
004430F2     0379 04             add edi,dword ptr ds:\[ecx+4\]  
004430F5     8B49 08             mov ecx,dword ptr ds:\[ecx+8\]  
004430F8     33C0                xor eax,eax  
004430FA     8A07                mov al,byte ptr ds:\[edi\]  
004430FC     47                  inc edi  
004430FD     0BC0                or eax,eax  
004430FF     74 20               je short 3D 文本出.00443121  
00443101     3C EF               cmp al,0EF  
00443103     77 06               ja short 3D 文本出.0044310B  
00443105     03D8                add ebx,eax  
00443107     010B                add dword ptr ds:\[ebx\],ecx  
00443109   ^ EB ED               jmp short 3D 文本出.004430F8  
0044310B     24 0F               and al,0F  
0044310D     C1E0 10             shl eax,10  
00443110     66:8B07             mov ax,word ptr ds:\[edi\]  
00443113     83C7 02             add edi,2  
00443116     0BC0                or eax,eax  
00443118   ^ 75 EB               jnz short 3D 文本出.00443105  
0044311A     8B07                mov eax,dword ptr ds:\[edi\]  
0044311C     83C7 04             add edi,4  
0044311F   ^ EB E4               jmp short 3D 文本出.00443105  
00443121     33DB                xor ebx,ebx  
00443123     87FE                xchg esi,edi  
00443125     8B06                mov eax,dword ptr ds:\[esi\]  
00443127     83F8 00             cmp eax,0  
0044312A     74 1F               je short 3D 文本出.0044314B  
0044312C     AD                  lods dword ptr ds:\[esi\]  
0044312D     0BC0                or eax,eax  
0044312F     74 08               je short 3D 文本出.00443139  
00443131     03D8                add ebx,eax  
00443133     66:010C3B           add word ptr ds:\[ebx+edi\],cx  
00443137   ^ EB F3               jmp short 3D 文本出.0044312C  
00443139     33DB                xor ebx,ebx  
0044313B     C1E9 10             shr ecx,10  
0044313E     AD                  lods dword ptr ds:\[esi\]  
0044313F     0BC0                or eax,eax  
00443141     74 08               je short 3D 文本出.0044314B  
00443143     03D8                add ebx,eax  
00443145     66:010C3B           add word ptr ds:\[ebx+edi\],cx  
00443149   ^ EB F3               jmp short 3D 文本出.0044313E  
0044314B     8BDD                mov ebx,ebp  
0044314D     81EB 2D000000       sub ebx,2D  
00443153     33C9                xor ecx,ecx  
00443155     8A0B                mov cl,byte ptr ds:\[ebx\]  
00443157     83F9 00             cmp ecx,0  
0044315A     74 28               je short 3D 文本出.00443184  
0044315C     43                  inc ebx  
0044315D     8DB5 A7FEFFFF       lea esi,dword ptr ss:\[ebp-159\]  
00443163     8B16                mov edx,dword ptr ds:\[esi\]  
00443165     56                  push esi  
00443166     51                  push ecx  
00443167     53                  push ebx  
00443168     52                  push edx  
00443169     56                  push esi  
0044316A     FF33                push dword ptr ds:\[ebx\]  
0044316C     FF73 04             push dword ptr ds:\[ebx+4\]  
0044316F     8B43 08             mov eax,dword ptr ds:\[ebx+8\]  
00443172     03C2                add eax,edx  
00443174     50                  push eax  
00443175     FF95 F7FEFFFF       call dword ptr ss:\[ebp-109\]  
0044317B     5A                  pop edx  
0044317C     5B                  pop ebx  
0044317D     59                  pop ecx  
0044317E     5E                  pop esi  
0044317F     83C3 0C             add ebx,0C  
00443182   ^ E2 E1               loopd short 3D 文本出.00443165  
00443184     61                  popad                                ////////这里下断  
00443185     9D                  popfd  
00443186   - E9 A606FCFF         jmp 3D 文本出.00403831                  ；OEP  
  
第二步解除自校验  
\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
0040113A   /$  55                push ebp  
0040113B   |.  8BEC              mov ebp,esp  
0040113D   |.  81EC 98020000     sub esp,298                                      ;  脱壳后有自校验  
00401143   |.  53                push ebx                                         ;  根据提示下断  
00401144   |.  56                push esi  
00401145   |.  57                push edi  
00401146   |.  8D85 6CFEFFFF     lea eax,dword ptr ss:\[ebp-194\]  
0040114C   |.  68 04010000       push 104                                         ; /BufSize = 104 (260.)  
00401151   |.  50                push eax                                         ; |PathBuffer  
00401152   |.  FF75 08           push dword ptr ss:\[ebp+8\]                        ; |hModule  
00401155   |.  33DB              xor ebx,ebx                                      ; |  
00401157   |.  895D FC           mov dword ptr ss:\[ebp-4\],ebx                     ; |  
0040115A   |.  895D F8           mov dword ptr ss:\[ebp-8\],ebx                     ; |  
0040115D   |.  895D F0           mov dword ptr ss:\[ebp-10\],ebx                    ; |  
00401160   |.  FF15 24604000     call dword ptr ds:\[<&kernel32.GetModuleFileNameA>; \\GetModuleFileNameA  
00401166   |.  53                push ebx                                         ; /hTemplateFile => NULL  
00401167   |.  68 80000000       push 80                                          ; |Attributes = NORMAL  
0040116C   |.  6A 03             push 3                                           ; |Mode = OPEN\_EXISTING  
0040116E   |.  53                push ebx                                         ; |pSecurity => NULL  
0040116F   |.  6A 01             push 1                                           ; |ShareMode = FILE\_SHARE\_READ  
00401171   |.  8D85 6CFEFFFF     lea eax,dword ptr ss:\[ebp-194\]                   ; |  
00401177   |.  68 00000080       push 80000000                                    ; |Access = GENERIC\_READ  
0040117C   |.  50                push eax                                         ; |FileName  
0040117D   |.  FF15 20604000     call dword ptr ds:\[<&kernel32.CreateFileA>\]      ; \\CreateFileA  
00401183   |.  8BF8              mov edi,eax  
00401185   |.  83FF FF           cmp edi,-1  
00401188   |.  75 0C             jnz short 3\_.00401196  
0040118A   |.  C745 FC C0714000  mov dword ptr ss:\[ebp-4\],3\_.004071C0             ;  ASCII "Can't open file!"  
00401191   |.  E9 37030000       jmp 3\_.004014CD  
00401196   |>  8B35 1C604000     mov esi,dword ptr ds:\[<&kernel32.SetFilePointer>>;  kernel32.SetFilePointer  
0040119C   |.  6A 02             push 2                                           ; /Origin = FILE\_END  
0040119E   |.  53                push ebx                                         ; |pOffsetHi  
0040119F   |.  6A F8             push -8                                          ; |OffsetLo = FFFFFFF8 (-8.)  
004011A1   |.  57                push edi                                         ; |hFile  
004011A2   |.  FFD6              call esi                                         ; \\SetFilePointer  
004011A4   |.  3D E8030000       cmp eax,3E8                                      ;  定位到文件后 8 个字节  
004011A9   |.  8945 F4           mov dword ptr ss:\[ebp-C\],eax  
004011AC   |.  0F82 FD020000     jb 3\_.004014AF  
004011B2   |.  8D45 E4           lea eax,dword ptr ss:\[ebp-1C\]  
004011B5   |.  53                push ebx                                         ; /pOverlapped  
004011B6   |.  50                push eax                                         ; |pBytesRead  
004011B7   |.  8D45 DC           lea eax,dword ptr ss:\[ebp-24\]                    ; |  
004011BA   |.  6A 08             push 8                                           ; |BytesToRead = 8  
004011BC   |.  50                push eax                                         ; |Buffer  
004011BD   |.  57                push edi                                         ; |hFile  
004011BE   |.  895D E4           mov dword ptr ss:\[ebp-1C\],ebx                    ; |  
004011C1   |.  FF15 18604000     call dword ptr ds:\[<&kernel32.ReadFile>\]         ; \\ReadFile  
004011C7   |.  85C0              test eax,eax  
004011C9   |.  0F84 E9020000     je 3\_.004014B8  
004011CF   |.  837D E4 08        cmp dword ptr ss:\[ebp-1C\],8  
004011D3   |.  0F85 DF020000     jnz 3\_.004014B8  
004011D9   |.  8B45 DC           mov eax,dword ptr ss:\[ebp-24\]  
004011DC   |.  817D E0 A5B79A82  cmp dword ptr ss:\[ebp-20\],829AB7A5               ;  看后 8 个字节是否等于 A5B79A82 注意 16 进制存放格式  
004011E3   |.  8945 08           mov dword ptr ss:\[ebp+8\],eax  
004011E6   |.  0F85 C3020000     jnz 3\_.004014AF  
004011EC   |.  83F8 04           cmp eax,4  
004011EF   |.  0F8C BA020000     jl 3\_.004014AF  
004011F5   |.  3B45 F4           cmp eax,dword ptr ss:\[ebp-C\]  
004011F8   |.  0F8D B1020000     jge 3\_.004014AF  
004011FE   |.  50                push eax  
004011FF   |.  E8 32220000       call 3\_.00403436  
00401204   |.  3BC3              cmp eax,ebx  
00401206   |.  59                pop ecx  
00401207   |.  8945 F8           mov dword ptr ss:\[ebp-8\],eax  
0040120A   |.  0F84 07010000     je 3\_.00401317  
00401210   |.  6A 02             push 2  
00401212   |.  53                push ebx  
00401213   |.  6A F8             push -8  
00401215   |.  895D E8           mov dword ptr ss:\[ebp-18\],ebx  
00401218   |.  58                pop eax  
00401219   |.  2B45 08           sub eax,dword ptr ss:\[ebp+8\]  
0040121C   |.  50                push eax  
0040121D   |.  57                push edi  
0040121E   |.  FFD6              call esi  
00401220   |.  83F8 FF           cmp eax,-1  
00401223   |.  0F84 7D020000     je 3\_.004014A6  
00401229   |.  8B75 F8           mov esi,dword ptr ss:\[ebp-8\]  
0040122C   |.  8D45 E8           lea eax,dword ptr ss:\[ebp-18\]  
0040122F   |.  53                push ebx                                         ; /pOverlapped  
00401230   |.  50                push eax                                         ; |pBytesRead  
00401231   |.  FF75 08           push dword ptr ss:\[ebp+8\]                        ; |BytesToRead  
00401234   |.  56                push esi                                         ; |Buffer  
00401235   |.  57                push edi                                         ; |hFile  
00401236   |.  FF15 18604000     call dword ptr ds:\[<&kernel32.ReadFile>\]         ; \\ReadFile  
0040123C   |.  85C0              test eax,eax  
0040123E   |.  0F84 62020000     je 3\_.004014A6  
00401244   |.  8B45 08           mov eax,dword ptr ss:\[ebp+8\]  
00401247   |.  3945 E8           cmp dword ptr ss:\[ebp-18\],eax  
0040124A   |.  0F85 56020000     jnz 3\_.004014A6  
00401250   |.  813E A5B79A82     cmp dword ptr ds:\[esi\],829AB7A5                  ;  再次比较 A5B79A82  
00401256   |.  0F85 4A020000     jnz 3\_.004014A6                                  ;  这时聪明的你会想到在原文件查找 A5B79A8216 进制数据  
0040125C   |.  8D85 6CFEFFFF     lea eax,dword ptr ss:\[ebp-194\]                   ;  找到 2 个地方有一个是末端一个是离末端前 B690F0 处  
00401262   |.  83C6 04           add esi,4                                        ;  用 HEXWORKSHOP 在空地方插入 B690F0 大小字节的块，然后可以从原文件粘贴过来  
00401265   |.  50                push eax                                         ; /Buffer  
00401266   |.  68 04010000       push 104                                         ; |BufSize = 104 (260.)  
0040126B   |.  FF15 14604000     call dword ptr ds:\[<&kernel32.GetTempPathA>\]     ; \\GetTempPathA  
00401271   |.  85C0              test eax,eax                                     ;  保存运行 成功了！  
00401273   |.  75 0C             jnz short 3\_.00401281  
00401275   |.  C745 FC 98714000  mov dword ptr ss:\[ebp-4\],3\_.00407198             ;  ASCII "Can't retrieve the temporary directory!"  
0040127C   |.  E9 3E020000       jmp 3\_.004014BF  
00401281   |>  8B06              mov eax,dword ptr ds:\[esi\]  
00401283   |.  83C6 04           add esi,4  
00401286   |.  50                push eax                                         ; /<%X>  
00401287   |.  8D85 70FFFFFF     lea eax,dword ptr ss:\[ebp-90\]                    ; |  
0040128D   |.  68 90714000       push 3\_.00407190                                 ; |Format = "E\_%X"  
00401292   |.  50                push eax                                         ; |s  
00401293   |.  FF15 B0604000     call dword ptr ds:\[<&user32.wsprintfA>\]          ; \\wsprintfA  
00401299   |.  8D85 70FFFFFF     lea eax,dword ptr ss:\[ebp-90\]  
0040129F   |.  50                push eax  
004012A0   |.  8D85 6CFEFFFF     lea eax,dword ptr ss:\[ebp-194\]  
004012A6   |.  50                push eax  
004012A7   |.  E8 24200000       call 3\_.004032D0  
004012AC   |.  83C4 14           add esp,14  
004012AF   |.  8D85 6CFEFFFF     lea eax,dword ptr ss:\[ebp-194\]  
004012B5   |.  53                push ebx                                         ; /pSecurity  
004012B6   |.  50                push eax                                         ; |Path  
004012B7   |.  FF15 10604000     call dword ptr ds:\[<&kernel32.CreateDirectoryA>\] ; \\CreateDirectoryA  
004012BD   |.  8D85 6CFEFFFF     lea eax,dword ptr ss:\[ebp-194\]  
004012C3   |.  68 8C714000       push 3\_.0040718C  
004012C8   |.  50                push eax  
004012C9   |.  E8 02200000       call 3\_.004032D0  
004012CE   |.  FF36              push dword ptr ds:\[esi\]  
004012D0   |.  836D 08 0C        sub dword ptr ss:\[ebp+8\],0C  
004012D4   |.  8D7E 04           lea edi,dword ptr ds:\[esi+4\]  
004012D7   |.  FF75 08           push dword ptr ss:\[ebp+8\]  
004012DA   |.  57                push edi  
004012DB   |.  E8 39FEFFFF       call 3\_.00401119  
004012E0   |.  836D 08 08        sub dword ptr ss:\[ebp+8\],8  
004012E4   |.  8B47 04           mov eax,dword ptr ds:\[edi+4\]  
004012E7   |.  83C4 14           add esp,14  
004012EA   |.  395D 08           cmp dword ptr ss:\[ebp+8\],ebx  
004012ED   |.  8945 EC           mov dword ptr ss:\[ebp-14\],eax  
004012F0   |.  0F8E A7010000     jle 3\_.0040149D  
004012F6   |.  813F 0D0F3E03     cmp dword ptr ds:\[edi\],33E0F0D  
004012FC   |.  0F85 9B010000     jnz 3\_.0040149D  
00401302   |.  3BC3              cmp eax,ebx  
00401304   |.  0F8E 93010000     jle 3\_.0040149D  
0040130A   |.  50                push eax  
0040130B   |.  E8 26210000       call 3\_.00403436  
00401310   |.  8BF0              mov esi,eax  
00401312   |.  59                pop ecx  
00401313   |.  3BF3              cmp esi,ebx  
00401315   |.  75 0C             jnz short 3\_.00401323  
00401317   |>  C745 FC 74714000  mov dword ptr ss:\[ebp-4\],3\_.00407174             ;  ASCII "Insufficient memory!"  
0040131E   |.  E9 9C010000       jmp 3\_.004014BF  
00401323   |>  FF75 08           push dword ptr ss:\[ebp+8\]  
00401326   |.  83C7 08           add edi,8  
00401329   |.  8D45 EC           lea eax,dword ptr ss:\[ebp-14\]  
0040132C   |.  57                push edi  
0040132D   |.  50                push eax  
0040132E   |.  56                push esi  
0040132F   |.  E8 E71E0000       call 3\_.0040321B  
00401334   |.  83C4 10           add esp,10  
00401337   |.  85C0              test eax,eax  
00401339   |.  74 13             je short 3\_.0040134E  
0040133B   |.  56                push esi  
0040133C   |.  E8 EA200000       call 3\_.0040342B  
00401341   |.  59                pop ecx  
00401342   |.  C745 FC 58714000  mov dword ptr ss:\[ebp-4\],3\_.00407158             ;  ASCII "Failed to decompress data!"  
00401349   |.  E9 71010000       jmp 3\_.004014BF  
0040134E   |>  FF75 F8           push dword ptr ss:\[ebp-8\]  
00401351   |.  E8 D5200000       call 3\_.0040342B  
00401356   |.  8B45 EC           mov eax,dword ptr ss:\[ebp-14\]  
00401359   |.  59                pop ecx  
0040135A   |.  03C6              add eax,esi  
0040135C   |.  8975 F8           mov dword ptr ss:\[ebp-8\],esi  
0040135F   |.  3BF0              cmp esi,eax  
00401361   |.  8945 F4           mov dword ptr ss:\[ebp-C\],eax  
00401364   |.  885D A4           mov byte ptr ss:\[ebp-5C\],bl  
00401367   |.  0F83 B4000000     jnb 3\_.00401421  
0040136D   |>  8BFE              /mov edi,esi  
0040136F   |.  56                |push esi  
00401370   |.  897D 08           |mov dword ptr ss:\[ebp+8\],edi  
00401373   |.  E8 38200000       |call 3\_.004033B0  
00401378   |.  C70424 4C714000   |mov dword ptr ss:\[esp\],3\_.0040714C              ;  ASCII "krnln.fnr"  
0040137F   |.  57                |push edi  
00401380   |.  8D7406 01         |lea esi,dword ptr ds:\[esi+eax+1\]  
00401384   |.  E8 47480000       |call 3\_.00405BD0  
00401389   |.  59                |pop ecx  
0040138A   |.  85C0              |test eax,eax  
0040138C   |.  59                |pop ecx  
0040138D   |.  74 11             |je short 3\_.004013A0  
0040138F   |.  68 40714000       |push 3\_.00407140                                ;  ASCII "krnln.fne"  
00401394   |.  57                |push edi  
00401395   |.  E8 36480000       |call 3\_.00405BD0  
0040139A   |.  59                |pop ecx  
0040139B   |.  85C0              |test eax,eax  
0040139D   |.  59                |pop ecx  
0040139E   |.  75 0C             |jnz short 3\_.004013AC  
004013A0   |>  8D45 A4           |lea eax,dword ptr ss:\[ebp-5C\]  
004013A3   |.  57                |push edi  
004013A4   |.  50                |push eax  
004013A5   |.  E8 161F0000       |call 3\_.004032C0  
004013AA   |.  59                |pop ecx  
004013AB   |.  59                |pop ecx  
004013AC   |>  8B3E              |mov edi,dword ptr ds:\[esi\]  
004013AE   |.  8D85 6CFEFFFF     |lea eax,dword ptr ss:\[ebp-194\]  
004013B4   |.  50                |push eax  
004013B5   |.  8D85 68FDFFFF     |lea eax,dword ptr ss:\[ebp-298\]  
004013BB   |.  50                |push eax  
004013BC   |.  83C6 04           |add esi,4  
004013BF   |.  E8 FC1E0000       |call 3\_.004032C0  
004013C4   |.  FF75 08           |push dword ptr ss:\[ebp+8\]  
004013C7   |.  8D85 68FDFFFF     |lea eax,dword ptr ss:\[ebp-298\]  
004013CD   |.  50                |push eax  
004013CE   |.  E8 FD1E0000       |call 3\_.004032D0  
004013D3   |.  83C4 10           |add esp,10  
004013D6   |.  8D85 68FDFFFF     |lea eax,dword ptr ss:\[ebp-298\]  
004013DC   |.  53                |push ebx                                        ; /hTemplateFile  
004013DD   |.  68 80000000       |push 80                                         ; |Attributes = NORMAL  
004013E2   |.  6A 02             |push 2                                          ; |Mode = CREATE\_ALWAYS  
004013E4   |.  53                |push ebx                                        ; |pSecurity  
004013E5   |.  53                |push ebx                                        ; |ShareMode  
004013E6   |.  68 00000040       |push 40000000                                   ; |Access = GENERIC\_WRITE  
004013EB   |.  50                |push eax                                        ; |FileName  
004013EC   |.  FF15 20604000     |call dword ptr ds:\[<&kernel32.CreateFileA>\]     ; \\CreateFileA  
004013F2   |.  83F8 FF           |cmp eax,-1  
004013F5   |.  8945 08           |mov dword ptr ss:\[ebp+8\],eax  
004013F8   |.  74 17             |je short 3\_.00401411  
004013FA   |.  8D4D D8           |lea ecx,dword ptr ss:\[ebp-28\]  
004013FD   |.  53                |push ebx                                        ; /pOverlapped  
004013FE   |.  51                |push ecx                                        ; |pBytesWritten  
004013FF   |.  57                |push edi                                        ; |nBytesToWrite  
00401400   |.  56                |push esi                                        ; |Buffer  
00401401   |.  50                |push eax                                        ; |hFile  
00401402   |.  FF15 0C604000     |call dword ptr ds:\[<&kernel32.WriteFile>\]       ; \\WriteFile  
00401408   |.  FF75 08           |push dword ptr ss:\[ebp+8\]                       ; /hObject  
0040140B   |.  FF15 08604000     |call dword ptr ds:\[<&kernel32.CloseHandle>\]     ; \\CloseHandle  
00401411   |>  03F7              |add esi,edi  
00401413   |.  3B75 F4           |cmp esi,dword ptr ss:\[ebp-C\]  
00401416   |.^ 0F82 51FFFFFF     \\jb 3\_.0040136D  
0040141C   |.  385D A4           cmp byte ptr ss:\[ebp-5C\],bl  
0040141F   |.  75 0C             jnz short 3\_.0040142D  
00401421   |>  C745 FC 20714000  mov dword ptr ss:\[ebp-4\],3\_.00407120             ;  ASCII "Not found the kernel library!"  
00401428   |.  E9 92000000       jmp 3\_.004014BF  
0040142D   |>  8D85 6CFEFFFF     lea eax,dword ptr ss:\[ebp-194\]  
00401433   |.  50                push eax  
00401434   |.  8D85 68FDFFFF     lea eax,dword ptr ss:\[ebp-298\]  
0040143A   |.  50                push eax  
0040143B   |.  E8 801E0000       call 3\_.004032C0  
00401440   |.  8D45 A4           lea eax,dword ptr ss:\[ebp-5C\]  
00401443   |.  50                push eax  
00401444   |.  8D85 68FDFFFF     lea eax,dword ptr ss:\[ebp-298\]  
0040144A   |.  50                push eax  
0040144B   |.  E8 801E0000       call 3\_.004032D0  
00401450   |.  83C4 10           add esp,10  
00401453   |.  8D85 68FDFFFF     lea eax,dword ptr ss:\[ebp-298\]  
00401459   |.  50                push eax                                         ; /FileName  
0040145A   |.  FF15 04604000     call dword ptr ds:\[<&kernel32.LoadLibraryA>\]     ; \\LoadLibraryA  
00401460   |.  3BC3              cmp eax,ebx  
00401462   |.  75 09             jnz short 3\_.0040146D  
00401464   |.  C745 FC 00714000  mov dword ptr ss:\[ebp-4\],3\_.00407100             ;  ASCII "Failed to load kernel library!"  
0040146B   |.  EB 52             jmp short 3\_.004014BF  
0040146D   |>  68 F4704000       push 3\_.004070F4                                 ; /ProcNameOrOrdinal = "GetNewSock"  
00401472   |.  50                push eax                                         ; |hModule  
00401473   |.  FF15 00604000     call dword ptr ds:\[<&kernel32.GetProcAddress>\]   ; \\GetProcAddress  
00401479   |.  3BC3              cmp eax,ebx  
0040147B   |.  75 09             jnz short 3\_.00401486  
0040147D   |.  C745 FC D4704000  mov dword ptr ss:\[ebp-4\],3\_.004070D4             ;  ASCII "The kernel library is invalid!"  
00401484   |.  EB 39             jmp short 3\_.004014BF  
00401486   |>  68 E8030000       push 3E8  
0040148B   |.  FFD0              call eax  
0040148D   |.  3BC3              cmp eax,ebx  
0040148F   |.  8945 F0           mov dword ptr ss:\[ebp-10\],eax  
00401492   |.  75 2B             jnz short 3\_.004014BF  
00401494   |.  C745 FC A8704000  mov dword ptr ss:\[ebp-4\],3\_.004070A8             ;  ASCII "The interface of kernel library is invalid!"  
0040149B   |.  EB 22             jmp short 3\_.004014BF  
0040149D   |>  C745 FC 8C704000  mov dword ptr ss:\[ebp-4\],3\_.0040708C             ;  ASCII "Invalid data in the file!"  
004014A4   |.  EB 19             jmp short 3\_.004014BF  
004014A6   |>  C745 FC 5C704000  mov dword ptr ss:\[ebp-4\],3\_.0040705C             ;  ASCII "Failed to read file or invalid data in file!"  
004014AD   |.  EB 10             jmp short 3\_.004014BF  
004014AF   |>  C745 FC 8C704000  mov dword ptr ss:\[ebp-4\],3\_.0040708C             ;  ASCII "Invalid data in the file!"  
004014B6   |.  EB 15             jmp short 3\_.004014CD  
004014B8   |>  C745 FC 38704000  mov dword ptr ss:\[ebp-4\],3\_.00407038             ;  ASCII "Failed to read data from the file!"  
004014BF   |>  395D F8           cmp dword ptr ss:\[ebp-8\],ebx  
004014C2   |.  74 09             je short 3\_.004014CD  
004014C4   |.  FF75 F8           push dword ptr ss:\[ebp-8\]  
004014C7   |.  E8 5F1F0000       call 3\_.0040342B  
004014CC   |.  59                pop ecx  
004014CD   |>  395D FC           cmp dword ptr ss:\[ebp-4\],ebx                     ; |  
004014D0   |.  75 13             jnz short 3\_.004014E5                            ; |  
004014D2   |.  8B45 F0           mov eax,dword ptr ss:\[ebp-10\]                    ; |  
004014D5   |.  E8 00000000       call 3\_.004014DA                                 ; \\3\_.004014DA  
004014DA   |$  810424 267B0000   add dword ptr ss:\[esp\],7B26  
004014E1   |.  FFD0              call eax  
004014E3   |.  EB 11             jmp short 3\_.004014F6  
004014E5   |>  6A 10             push 10                                          ; /Style = MB\_OK|MB\_ICONHAND|MB\_APPLMODAL  
004014E7   |.  68 30704000       push 3\_.00407030                                 ; |Title = "Error"  
004014EC   |.  FF75 FC           push dword ptr ss:\[ebp-4\]                        ; |Text  
004014EF   |.  53                push ebx                                         ; |hOwner  
004014F0   |.  FF15 AC604000     call dword ptr ds:\[<&user32.MessageBoxA>\]        ; \\MessageBoxA  
004014F6   |>  5F                pop edi  
004014F7   |.  5E                pop esi  
  
  
\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
第三步破解:  
  
  
/////////////////////////////////////////////////////////////////////////////////////////////  
00439D91   |.  81EC 0C000000     sub esp,0C  
00439D97   |.  6A 00             push 0  
00439D99   |.  6A 00             push 0  
00439D9B   |.  6A 00             push 0  
00439D9D   |.  68 04000080       push 80000004  
00439DA2   |.  6A 00             push 0  
00439DA4   |.  68 250E4100       push 3\_.00410E25                                 ;  ASCII "SOFTWARE\\microsoft\\windows help\\test"  
00439DA9   |.  68 01030080       push 80000301  
00439DAE   |.  6A 00             push 0                                           ;  储存注册码的注册键  
00439DB0   |.  68 03000000       push 3  
00439DB5   |.  68 03000000       push 3  
00439DBA   |.  BB 98060000       mov ebx,698  
00439DBF   |.  E8 4F510000       call 3\_.0043EF13                                 ;  假码  
00439DC4   |.  83C4 28           add esp,28  
00439DC7   |.  8945 FC           mov dword ptr ss:\[ebp-4\],eax  
00439DCA   |.  8B1D FC03CA00     mov ebx,dword ptr ds:\[CA03FC\]  
00439DD0   |.  85DB              test ebx,ebx  
00439DD2   |.  74 09             je short 3\_.00439DDD  
00439DD4   |.  53                push ebx  
00439DD5   |.  E8 2D510000       call 3\_.0043EF07  
00439DDA   |.  83C4 04           add esp,4  
00439DDD   |>  8B45 FC           mov eax,dword ptr ss:\[ebp-4\]  
00439DE0   |.  A3 FC03CA00       mov dword ptr ds:\[CA03FC\],eax  
00439DE5   |.  E8 9F010000       call 3\_.00439F89                                 ;  关键算法 F7  
00439DEA   |.  8945 FC           mov dword ptr ss:\[ebp-4\],eax                     ;  这里可以看到真码  
00439DED   |.  8B1D 0004CA00     mov ebx,dword ptr ds:\[CA0400\]  
00439DF3   |.  85DB              test ebx,ebx  
00439DF5   |.  74 09             je short 3\_.00439E00  
00439DF7   |.  53                push ebx  
00439DF8   |.  E8 0A510000       call 3\_.0043EF07  
00439DFD   |.  83C4 04           add esp,4  
00439E00   |>  8B45 FC           mov eax,dword ptr ss:\[ebp-4\]  
00439E03   |.  A3 0004CA00       mov dword ptr ds:\[CA0400\],eax  
00439E08   |.  8B1D 0404CA00     mov ebx,dword ptr ds:\[CA0404\]  
00439E0E   |.  85DB              test ebx,ebx  
00439E10   |.  74 09             je short 3\_.00439E1B  
00439E12   |.  53                push ebx  
00439E13   |.  E8 EF500000       call 3\_.0043EF07  
00439E18   |.  83C4 04           add esp,4  
00439E1B   |>  B8 F6914000       mov eax,3\_.004091F6  
00439E20   |.  A3 0404CA00       mov dword ptr ds:\[CA0404\],eax  
00439E25   |.  A1 0004CA00       mov eax,dword ptr ds:\[CA0400\]  
00439E2A   |.  50                push eax  
00439E2B   |.  FF35 FC03CA00     push dword ptr ds:\[CA03FC\]  
00439E31   |.  E8 0337FEFF       call 3\_.0041D539  
00439E36   |.  83C4 08           add esp,8  
00439E39   |.  83F8 00           cmp eax,0  
00439E3C   |.  0F84 E6000000     je 3\_.00439F28  
00439E42   |.  68 02000080       push 80000002  
00439E47   |.  6A 00             push 0  
00439E49   |.  68 01000000       push 1  
00439E4E   |.  6A 00             push 0  
00439E50   |.  6A 00             push 0  
00439E52   |.  6A 00             push 0  
00439E54   |.  68 01000100       push 10001  
00439E59   |.  68 07050106       push 6010507  
00439E5E   |.  68 06050152       push 52010506  
00439E63   |.  68 03000000       push 3  
00439E68   |.  BB 20030000       mov ebx,320  
00439E6D   |.  E8 A1500000       call 3\_.0043EF13  
00439E72   |.  83C4 28           add esp,28  
00439E75   |.  68 40924000       push 3\_.00409240                                 ;  UNICODE "0123456789"  
00439E7A   |.  FF35 0404CA00     push dword ptr ds:\[CA0404\]  
00439E80   |.  E8 B436FEFF       call 3\_.0041D539  
00439E85   |.  83C4 08           add esp,8  
00439E88   |.  83F8 00           cmp eax,0  
00439E8B   |.  0F85 23000000     jnz 3\_.00439EB4  
00439E91   |.  68 42924000       push 3\_.00409242                                 ;  UNICODE "123456789"  
00439E96   |.  FF35 0404CA00     push dword ptr ds:\[CA0404\]  
00439E9C   |.  E8 9836FEFF       call 3\_.0041D539  
00439EA1   |.  83C4 08           add esp,8  
00439EA4   |.  83F8 00           cmp eax,0  
00439EA7   |.  0F84 07000000     je 3\_.00439EB4  
00439EAD   |.  B8 01000000       mov eax,1  
00439EB2   |.  EB 02             jmp short 3\_.00439EB6  
00439EB4   |>  33C0              xor eax,eax  
00439EB6   |>  85C0              test eax,eax  
00439EB8   |.  0F84 3A000000     je 3\_.00439EF8  
00439EBE   |.  6A 00             push 0  
00439EC0   |.  6A 00             push 0  
00439EC2   |.  6A 00             push 0  
00439EC4   |.  68 01030080       push 80000301  
00439EC9   |.  6A 00             push 0  
00439ECB   |.  68 10000000       push 10  
00439ED0   |.  68 04000080       push 80000004  
00439ED5   |.  6A 00             push 0  
00439ED7   |.  68 4A0E4100       push 3\_.00410E4A  
00439EDC   |.  68 03000000       push 3  
00439EE1   |.  BB 00030000       mov ebx,300  
00439EE6   |.  E8 28500000       call 3\_.0043EF13  
00439EEB   |.  83C4 28           add esp,28  
00439EEE   |.  6A 00             push 0  
00439EF0   |.  E8 FA4F0000       call 3\_.0043EEEF  
00439EF5   |.  83C4 04           add esp,4  
00439EF8   |>  6A 00             push 0  
00439EFA   |.  6A 00             push 0  
00439EFC   |.  6A 00             push 0  
00439EFE   |.  68 01030080       push 80000301  
00439F03   |.  6A 00             push 0  
00439F05   |.  68 00000000       push 0  
00439F0A   |.  68 04000080       push 80000004  
00439F0F   |.  6A 00             push 0  
00439F11   |.  68 580E4100       push 3\_.00410E58  
00439F16   |.  68 03000000       push 3  
00439F1B   |.  BB 00030000       mov ebx,300  
00439F20   |.  E8 EE4F0000       call 3\_.0043EF13  
00439F25   |.  83C4 28           add esp,28  
00439F28   |>  68 01030080       push 80000301  
00439F2D   |.  6A 00             push 0  
00439F2F   |.  68 03000000       push 3  
00439F34   |.  68 01000000       push 1  
00439F39   |.  BB 00000000       mov ebx,0  
00439F3E   |.  B8 01000000       mov eax,1  
00439F43   |.  E8 E34F0000       call 3\_.0043EF2B  
00439F48   |.  83C4 10           add esp,10  
00439F4B   |.  6A 00             push 0  
00439F4D   |.  68 00000000       push 0  
00439F52   |.  6A FF             push -1  
00439F54   |.  6A 13             push 13  
00439F56   |.  68 E6000116       push 160100E6  
00439F5B   |.  68 01000152       push 52010001  
00439F60   |.  E8 C04F0000       call 3\_.0043EF25  
00439F65   |.  83C4 18           add esp,18  
00439F68   |.  6A 00             push 0  
00439F6A   |.  68 00000000       push 0  
00439F6F   |.  6A FF             push -1  
00439F71   |.  6A 13             push 13  
00439F73   |.  68 E8000116       push 160100E8  
00439F78   |.  68 01000152       push 52010001  
00439F7D   |.  E8 A34F0000       call 3\_.0043EF25  
00439F82   |.  83C4 18           add esp,18  
00439F85   |.  8BE5              mov esp,ebp  
  
\==========================================================================================  
00439F89   /$  55                push ebp  
00439F8A   |.  8BEC              mov ebp,esp  
00439F8C   |.  81EC 34000000     sub esp,34  
00439F92   |.  68 04000000       push 4  
00439F97   |.  E8 714F0000       call 3\_.0043EF0D  
00439F9C   |.  83C4 04           add esp,4  
00439F9F   |.  8945 FC           mov dword ptr ss:\[ebp-4\],eax  
00439FA2   |.  8BD8              mov ebx,eax  
00439FA4   |.  C703 00000000     mov dword ptr ds:\[ebx\],0  
00439FAA   |.  68 0A000600       push 6000A  
00439FAF   |.  6A 00             push 0  
00439FB1   |.  53                push ebx  
00439FB2   |.  6A 01             push 1  
00439FB4   |.  BB F8000000       mov ebx,0F8  
00439FB9   |.  B8 05000000       mov eax,5  
00439FBE   |.  E8 684F0000       call 3\_.0043EF2B  
00439FC3   |.  83C4 10           add esp,10  
00439FC6   |.  68 04000000       push 4  
00439FCB   |.  E8 3D4F0000       call 3\_.0043EF0D  
00439FD0   |.  83C4 04           add esp,4  
00439FD3   |.  8945 F8           mov dword ptr ss:\[ebp-8\],eax  
00439FD6   |.  8BD8              mov ebx,eax  
00439FD8   |.  C703 00000000     mov dword ptr ds:\[ebx\],0  
00439FDE   |.  68 0A000600       push 6000A  
00439FE3   |.  6A 00             push 0  
00439FE5   |.  53                push ebx  
00439FE6   |.  6A 01             push 1  
00439FE8   |.  BB F8000000       mov ebx,0F8  
00439FED   |.  B8 05000000       mov eax,5  
00439FF2   |.  E8 344F0000       call 3\_.0043EF2B  
00439FF7   |.  83C4 10           add esp,10  
00439FFA   |.  68 04000000       push 4  
00439FFF   |.  E8 094F0000       call 3\_.0043EF0D  
0043A004   |.  83C4 04           add esp,4  
0043A007   |.  8945 F4           mov dword ptr ss:\[ebp-C\],eax  
0043A00A   |.  8BD8              mov ebx,eax  
0043A00C   |.  C703 00000000     mov dword ptr ds:\[ebx\],0  
0043A012   |.  68 0A000600       push 6000A  
0043A017   |.  6A 00             push 0  
0043A019   |.  53                push ebx  
0043A01A   |.  6A 01             push 1  
0043A01C   |.  BB F8000000       mov ebx,0F8  
0043A021   |.  B8 05000000       mov eax,5  
0043A026   |.  E8 004F0000       call 3\_.0043EF2B  
0043A02B   |.  83C4 10           add esp,10  
0043A02E   |.  C745 F0 00000000  mov dword ptr ss:\[ebp-10\],0  
0043A035   |.  68 00000000       push 0  
0043A03A   |.  BB C4060000       mov ebx,6C4  
0043A03F   |.  E8 CF4E0000       call 3\_.0043EF13                                 ;  机器码 16 进制格式  
0043A044   |.  83C4 04           add esp,4  
0043A047   |.  68 01030080       push 80000301  
0043A04C   |.  6A 00             push 0  
0043A04E   |.  50                push eax  
0043A04F   |.  68 01000000       push 1  
0043A054   |.  BB 68010000       mov ebx,168  
0043A059   |.  E8 B54E0000       call 3\_.0043EF13                                 ;  机器码字符串格式  
0043A05E   |.  83C4 10           add esp,10  
0043A061   |.  8945 E8           mov dword ptr ss:\[ebp-18\],eax  
0043A064   |.  68 04000080       push 80000004  
0043A069   |.  6A 00             push 0  
0043A06B   |.  8B45 E8           mov eax,dword ptr ss:\[ebp-18\]  
0043A06E   |.  85C0              test eax,eax  
0043A070   |.  75 05             jnz short 3\_.0043A077  
0043A072   |.  B8 F6914000       mov eax,3\_.004091F6  
0043A077   |>  50                push eax  
0043A078   |.  68 0A000600       push 6000A  
0043A07D   |.  6A 00             push 0  
0043A07F   |.  FF75 FC           push dword ptr ss:\[ebp-4\]  
0043A082   |.  68 02000000       push 2  
0043A087   |.  BB 0C010000       mov ebx,10C  
0043A08C   |.  B8 05000000       mov eax,5  
0043A091   |.  E8 954E0000       call 3\_.0043EF2B  
0043A096   |.  83C4 1C           add esp,1C  
0043A099   |.  8B5D E8           mov ebx,dword ptr ss:\[ebp-18\]  
0043A09C   |.  85DB              test ebx,ebx  
0043A09E   |.  74 09             je short 3\_.0043A0A9  
0043A0A0   |.  53                push ebx  
0043A0A1   |.  E8 614E0000       call 3\_.0043EF07  
0043A0A6   |.  83C4 04           add esp,4  
0043A0A9   |>  68 04000080       push 80000004  
0043A0AE   |.  6A 00             push 0  
0043A0B0   |.  68 610E4100       push 3\_.00410E61                                 ;  ASCII "85178709130498756416136855615456466498723998190278340987347"  
0043A0B5   |.  68 0A000600       push 6000A  
0043A0BA   |.  6A 00             push 0  
0043A0BC   |.  FF75 F8           push dword ptr ss:\[ebp-8\]  
0043A0BF   |.  68 02000000       push 2  
0043A0C4   |.  BB 0C010000       mov ebx,10C  
0043A0C9   |.  B8 05000000       mov eax,5  
0043A0CE   |.  E8 584E0000       call 3\_.0043EF2B  
0043A0D3   |.  83C4 1C           add esp,1C  
0043A0D6   |.  68 0A000600       push 6000A  
0043A0DB   |.  6A 00             push 0  
0043A0DD   |.  FF75 FC           push dword ptr ss:\[ebp-4\]  
0043A0E0   |.  68 0A000600       push 6000A  
0043A0E5   |.  6A 00             push 0  
0043A0E7   |.  FF75 F8           push dword ptr ss:\[ebp-8\]  
0043A0EA   |.  68 02000000       push 2  
0043A0EF   |.  BB 78010000       mov ebx,178  
0043A0F4   |.  B8 05000000       mov eax,5  
0043A0F9   |.  E8 2D4E0000       call 3\_.0043EF2B  
0043A0FE   |.  83C4 1C           add esp,1C  
0043A101   |.  8945 EC           mov dword ptr ss:\[ebp-14\],eax  
0043A104   |.  8B5D F4           mov ebx,dword ptr ss:\[ebp-C\]  
0043A107   |.  53                push ebx  
0043A108   |.  68 0A000600       push 6000A  
0043A10D   |.  6A 00             push 0  
0043A10F   |.  53                push ebx  
0043A110   |.  6A 01             push 1  
0043A112   |.  BB FC000000       mov ebx,0FC  
0043A117   |.  B8 05000000       mov eax,5  
0043A11C   |.  E8 0A4E0000       call 3\_.0043EF2B  
0043A121   |.  83C4 10           add esp,10  
0043A124   |.  E8 DE4D0000       call 3\_.0043EF07  
0043A129   |.  83C4 04           add esp,4  
0043A12C   |.  8B45 EC           mov eax,dword ptr ss:\[ebp-14\]  
0043A12F   |.  8945 F4           mov dword ptr ss:\[ebp-C\],eax  
0043A132   |.  8B5D FC           mov ebx,dword ptr ss:\[ebp-4\]  
0043A135   |.  53                push ebx  
0043A136   |.  68 0A000600       push 6000A  
0043A13B   |.  6A 00             push 0  
0043A13D   |.  53                push ebx  
0043A13E   |.  6A 01             push 1  
0043A140   |.  BB FC000000       mov ebx,0FC  
0043A145   |.  B8 05000000       mov eax,5  
0043A14A   |.  E8 DC4D0000       call 3\_.0043EF2B  
0043A14F   |.  83C4 10           add esp,10  
0043A152   |.  E8 B04D0000       call 3\_.0043EF07  
0043A157   |.  83C4 04           add esp,4  
0043A15A   |.  8B45 F4           mov eax,dword ptr ss:\[ebp-C\]  
0043A15D   |.  50                push eax  
0043A15E   |.  68 04000000       push 4  
0043A163   |.  E8 A54D0000       call 3\_.0043EF0D  
0043A168   |.  83C4 04           add esp,4  
0043A16B   |.  5B                pop ebx  
0043A16C   |.  50                push eax  
0043A16D   |.  68 0A000600       push 6000A  
0043A172   |.  6A 00             push 0  
0043A174   |.  53                push ebx  
0043A175   |.  68 0A000600       push 6000A  
0043A17A   |.  6A 00             push 0  
0043A17C   |.  50                push eax  
0043A17D   |.  6A 02             push 2  
0043A17F   |.  BB B8010000       mov ebx,1B8  
0043A184   |.  B8 05000000       mov eax,5  
0043A189   |.  E8 9D4D0000       call 3\_.0043EF2B  
0043A18E   |.  83C4 1C           add esp,1C  
0043A191   |.  58                pop eax  
0043A192   |.  8945 FC           mov dword ptr ss:\[ebp-4\],eax  
0043A195   |.  68 04000080       push 80000004  
0043A19A   |.  6A 00             push 0  
0043A19C   |.  68 9D0E4100       push 3\_.00410E9D                                 ;  ASCII "29389982575461846103175166825962346516893593247598327459815"  
0043A1A1   |.  68 0A000600       push 6000A  
0043A1A6   |.  6A 00             push 0  
0043A1A8   |.  FF75 F8           push dword ptr ss:\[ebp-8\]  
0043A1AB   |.  68 02000000       push 2  
0043A1B0   |.  BB 0C010000       mov ebx,10C  
0043A1B5   |.  B8 05000000       mov eax,5  
0043A1BA   |.  E8 6C4D0000       call 3\_.0043EF2B  
0043A1BF   |.  83C4 1C           add esp,1C  
0043A1C2   |.  68 0A000600       push 6000A  
0043A1C7   |.  6A 00             push 0  
0043A1C9   |.  FF75 FC           push dword ptr ss:\[ebp-4\]  
0043A1CC   |.  68 0A000600       push 6000A  
0043A1D1   |.  6A 00             push 0  
0043A1D3   |.  FF75 F8           push dword ptr ss:\[ebp-8\]  
0043A1D6   |.  68 02000000       push 2  
0043A1DB   |.  BB 78010000       mov ebx,178  
0043A1E0   |.  B8 05000000       mov eax,5  
0043A1E5   |.  E8 414D0000       call 3\_.0043EF2B  
0043A1EA   |.  83C4 1C           add esp,1C  
0043A1ED   |.  8945 EC           mov dword ptr ss:\[ebp-14\],eax  
0043A1F0   |.  8B5D F4           mov ebx,dword ptr ss:\[ebp-C\]  
0043A1F3   |.  53                push ebx  
0043A1F4   |.  68 0A000600       push 6000A  
0043A1F9   |.  6A 00             push 0  
0043A1FB   |.  53                push ebx  
0043A1FC   |.  6A 01             push 1  
0043A1FE   |.  BB FC000000       mov ebx,0FC  
0043A203   |.  B8 05000000       mov eax,5  
0043A208   |.  E8 1E4D0000       call 3\_.0043EF2B  
0043A20D   |.  83C4 10           add esp,10  
0043A210   |.  E8 F24C0000       call 3\_.0043EF07  
0043A215   |.  83C4 04           add esp,4  
0043A218   |.  8B45 EC           mov eax,dword ptr ss:\[ebp-14\]  
0043A21B   |.  8945 F4           mov dword ptr ss:\[ebp-C\],eax  
0043A21E   |.  68 0A000600       push 6000A  
0043A223   |.  6A 00             push 0  
0043A225   |.  FF75 F4           push dword ptr ss:\[ebp-C\]  
0043A228   |.  68 01000000       push 1  
0043A22D   |.  BB 10010000       mov ebx,110  
0043A232   |.  B8 05000000       mov eax,5  
0043A237   |.  E8 EF4C0000       call 3\_.0043EF2B  
0043A23C   |.  83C4 10           add esp,10  
0043A23F   |.  8945 EC           mov dword ptr ss:\[ebp-14\],eax  
0043A242   |.  68 01030080       push 80000301  
0043A247   |.  6A 00             push 0  
0043A249   |.  68 0F000000       push 0F  
0043A24E   |.  68 04000080       push 80000004  
0043A253   |.  6A 00             push 0  
0043A255   |.  8B45 EC           mov eax,dword ptr ss:\[ebp-14\]  
0043A258   |.  85C0              test eax,eax  
0043A25A   |.  75 05             jnz short 3\_.0043A261  
0043A25C   |.  B8 F6914000       mov eax,3\_.004091F6  
0043A261   |>  50                push eax  
0043A262   |.  68 02000000       push 2  
0043A267   |.  BB 34010000       mov ebx,134  
0043A26C   |.  E8 A24C0000       call 3\_.0043EF13  
0043A271   |.  83C4 1C           add esp,1C  
0043A274   |.  8945 E8           mov dword ptr ss:\[ebp-18\],eax  
0043A277   |.  8B5D EC           mov ebx,dword ptr ss:\[ebp-14\]  
0043A27A   |.  85DB              test ebx,ebx  
0043A27C   |.  74 09             je short 3\_.0043A287  
0043A27E   |.  53                push ebx  
0043A27F   |.  E8 834C0000       call 3\_.0043EF07  
0043A284   |.  83C4 04           add esp,4  
0043A287   |>  68 04000080       push 80000004  
0043A28C   |.  6A 00             push 0  
0043A28E   |.  8B45 E8           mov eax,dword ptr ss:\[ebp-18\]  
0043A291   |.  85C0              test eax,eax  
0043A293   |.  75 05             jnz short 3\_.0043A29A  
0043A295   |.  B8 F6914000       mov eax,3\_.004091F6  
0043A29A   |>  50                push eax  
0043A29B   |.  68 0A000600       push 6000A  
0043A2A0   |.  6A 00             push 0  
0043A2A2   |.  FF75 FC           push dword ptr ss:\[ebp-4\]  
0043A2A5   |.  68 02000000       push 2  
0043A2AA   |.  BB 0C010000       mov ebx,10C  
0043A2AF   |.  B8 05000000       mov eax,5  
0043A2B4   |.  E8 724C0000       call 3\_.0043EF2B  
0043A2B9   |.  83C4 1C           add esp,1C  
0043A2BC   |.  8B5D E8           mov ebx,dword ptr ss:\[ebp-18\]  
0043A2BF   |.  85DB              test ebx,ebx  
0043A2C1   |.  74 09             je short 3\_.0043A2CC  
0043A2C3   |.  53                push ebx  
0043A2C4   |.  E8 3E4C0000       call 3\_.0043EF07  
0043A2C9   |.  83C4 04           add esp,4  
0043A2CC   |>  68 0A000600       push 6000A  
0043A2D1   |.  6A 00             push 0  
0043A2D3   |.  FF75 F4           push dword ptr ss:\[ebp-C\]  
0043A2D6   |.  68 01000000       push 1  
0043A2DB   |.  BB 10010000       mov ebx,110  
0043A2E0   |.  B8 05000000       mov eax,5  
0043A2E5   |.  E8 414C0000       call 3\_.0043EF2B  
0043A2EA   |.  83C4 10           add esp,10  
0043A2ED   |.  8945 EC           mov dword ptr ss:\[ebp-14\],eax  
0043A2F0   |.  68 01030080       push 80000301  
0043A2F5   |.  6A 00             push 0  
0043A2F7   |.  68 0C000000       push 0C  
0043A2FC   |.  68 01030080       push 80000301  
0043A301   |.  6A 00             push 0  
0043A303   |.  68 16000000       push 16  
0043A308   |.  68 04000080       push 80000004  
0043A30D   |.  6A 00             push 0  
0043A30F   |.  8B45 EC           mov eax,dword ptr ss:\[ebp-14\]  
0043A312   |.  85C0              test eax,eax  
0043A314   |.  75 05             jnz short 3\_.0043A31B  
0043A316   |.  B8 F6914000       mov eax,3\_.004091F6  
0043A31B   |>  50                push eax  
0043A31C   |.  68 03000000       push 3  
0043A321   |.  BB 3C010000       mov ebx,13C  
0043A326   |.  E8 E84B0000       call 3\_.0043EF13  
0043A32B   |.  83C4 28           add esp,28  
0043A32E   |.  8945 E8           mov dword ptr ss:\[ebp-18\],eax  
0043A331   |.  8B5D EC           mov ebx,dword ptr ss:\[ebp-14\]  
0043A334   |.  85DB              test ebx,ebx  
0043A336   |.  74 09             je short 3\_.0043A341  
0043A338   |.  53                push ebx  
0043A339   |.  E8 C94B0000       call 3\_.0043EF07  
0043A33E   |.  83C4 04           add esp,4  
0043A341   |>  68 04000080       push 80000004  
0043A346   |.  6A 00             push 0  
0043A348   |.  8B45 E8           mov eax,dword ptr ss:\[ebp-18\]  
0043A34B   |.  85C0              test eax,eax  
0043A34D   |.  75 05             jnz short 3\_.0043A354  
0043A34F   |.  B8 F6914000       mov eax,3\_.004091F6  
0043A354   |>  50                push eax  
0043A355   |.  68 0A000600       push 6000A  
0043A35A   |.  6A 00             push 0  
0043A35C   |.  FF75 F8           push dword ptr ss:\[ebp-8\]  
0043A35F   |.  68 02000000       push 2  
0043A364   |.  BB 0C010000       mov ebx,10C  
0043A369   |.  B8 05000000       mov eax,5  
0043A36E   |.  E8 B84B0000       call 3\_.0043EF2B  
0043A373   |.  83C4 1C           add esp,1C  
0043A376   |.  8B5D E8           mov ebx,dword ptr ss:\[ebp-18\]  
0043A379   |.  85DB              test ebx,ebx  
0043A37B   |.  74 09             je short 3\_.0043A386  
0043A37D   |.  53                push ebx  
0043A37E   |.  E8 844B0000       call 3\_.0043EF07  
0043A383   |.  83C4 04           add esp,4  
0043A386   |>  68 0A000600       push 6000A  
0043A38B   |.  6A 00             push 0  
0043A38D   |.  FF75 FC           push dword ptr ss:\[ebp-4\]  
0043A390   |.  68 0A000600       push 6000A  
0043A395   |.  6A 00             push 0  
0043A397   |.  FF75 F8           push dword ptr ss:\[ebp-8\]  
0043A39A   |.  68 02000000       push 2  
0043A39F   |.  BB 78010000       mov ebx,178  
0043A3A4   |.  B8 05000000       mov eax,5  
0043A3A9   |.  E8 7D4B0000       call 3\_.0043EF2B  
0043A3AE   |.  83C4 1C           add esp,1C  
0043A3B1   |.  8945 EC           mov dword ptr ss:\[ebp-14\],eax  
0043A3B4   |.  8B5D F4           mov ebx,dword ptr ss:\[ebp-C\]  
0043A3B7   |.  53                push ebx  
0043A3B8   |.  68 0A000600       push 6000A  
0043A3BD   |.  6A 00             push 0  
0043A3BF   |.  53                push ebx  
0043A3C0   |.  6A 01             push 1  
0043A3C2   |.  BB FC000000       mov ebx,0FC  
0043A3C7   |.  B8 05000000       mov eax,5  
0043A3CC   |.  E8 5A4B0000       call 3\_.0043EF2B  
0043A3D1   |.  83C4 10           add esp,10  
0043A3D4   |.  E8 2E4B0000       call 3\_.0043EF07  
0043A3D9   |.  83C4 04           add esp,4  
0043A3DC   |.  8B45 EC           mov eax,dword ptr ss:\[ebp-14\]  
0043A3DF   |.  8945 F4           mov dword ptr ss:\[ebp-C\],eax  
0043A3E2   |.  68 0A000600       push 6000A  
0043A3E7   |.  6A 00             push 0  
0043A3E9   |.  FF75 F4           push dword ptr ss:\[ebp-C\]  
0043A3EC   |.  68 01000000       push 1  
0043A3F1   |.  BB 10010000       mov ebx,110  
0043A3F6   |.  B8 05000000       mov eax,5  
0043A3FB   |.  E8 2B4B0000       call 3\_.0043EF2B  
0043A400   |.  83C4 10           add esp,10  
0043A403   |.  8945 EC           mov dword ptr ss:\[ebp-14\],eax  
0043A406   |.  8B5D F0           mov ebx,dword ptr ss:\[ebp-10\]  
0043A409   |.  85DB              test ebx,ebx  
0043A40B   |.  74 09             je short 3\_.0043A416  
0043A40D   |.  53                push ebx  
0043A40E   |.  E8 F44A0000       call 3\_.0043EF07  
0043A413   |.  83C4 04           add esp,4  
0043A416   |>  8B45 EC           mov eax,dword ptr ss:\[ebp-14\]  
0043A419   |.  8945 F0           mov dword ptr ss:\[ebp-10\],eax  
0043A41C   |.  68 01030080       push 80000301  
0043A421   |.  6A 00             push 0  
0043A423   |.  68 02000000       push 2  
0043A428   |.  68 01030080       push 80000301  
0043A42D   |.  6A 00             push 0  
0043A42F   |.  68 03000000       push 3  
0043A434   |.  68 04000080       push 80000004  
0043A439   |.  6A 00             push 0  
0043A43B   |.  8B45 F0           mov eax,dword ptr ss:\[ebp-10\]  
0043A43E   |.  85C0              test eax,eax  
0043A440   |.  75 05             jnz short 3\_.0043A447  
0043A442   |.  B8 F6914000       mov eax,3\_.004091F6  
0043A447   |>  50                push eax  
0043A448   |.  68 03000000       push 3  
0043A44D   |.  BB 3C010000       mov ebx,13C  
0043A452   |.  E8 BC4A0000       call 3\_.0043EF13  
0043A457   |.  83C4 28           add esp,28  
0043A45A   |.  8945 EC           mov dword ptr ss:\[ebp-14\],eax  
0043A45D   |.  68 01030080       push 80000301  
0043A462   |.  6A 00             push 0  
0043A464   |.  68 02000000       push 2  
0043A469   |.  68 01030080       push 80000301  
0043A46E   |.  6A 00             push 0  
0043A470   |.  68 09000000       push 9  
0043A475   |.  68 04000080       push 80000004  
0043A47A   |.  6A 00             push 0  
0043A47C   |.  8B45 F0           mov eax,dword ptr ss:\[ebp-10\]  
0043A47F   |.  85C0              test eax,eax  
0043A481   |.  75 05             jnz short 3\_.0043A488  
0043A483   |.  B8 F6914000       mov eax,3\_.004091F6  
0043A488   |>  50                push eax  
0043A489   |.  68 03000000       push 3  
0043A48E   |.  BB 3C010000       mov ebx,13C  
0043A493   |.  E8 7B4A0000       call 3\_.0043EF13  
0043A498   |.  83C4 28           add esp,28  
0043A49B   |.  8945 E8           mov dword ptr ss:\[ebp-18\],eax  
0043A49E   |.  68 01030080       push 80000301  
0043A4A3   |.  6A 00             push 0  
0043A4A5   |.  68 02000000       push 2  
0043A4AA   |.  68 01030080       push 80000301  
0043A4AF   |.  6A 00             push 0  
0043A4B1   |.  68 0D000000       push 0D  
0043A4B6   |.  68 04000080       push 80000004  
0043A4BB   |.  6A 00             push 0  
0043A4BD   |.  8B45 F0           mov eax,dword ptr ss:\[ebp-10\]  
0043A4C0   |.  85C0              test eax,eax  
0043A4C2   |.  75 05             jnz short 3\_.0043A4C9  
0043A4C4   |.  B8 F6914000       mov eax,3\_.004091F6  
0043A4C9   |>  50                push eax  
0043A4CA   |.  68 03000000       push 3  
0043A4CF   |.  BB 3C010000       mov ebx,13C  
0043A4D4   |.  E8 3A4A0000       call 3\_.0043EF13  
0043A4D9   |.  83C4 28           add esp,28  
0043A4DC   |.  8945 E4           mov dword ptr ss:\[ebp-1C\],eax  
0043A4DF   |.  68 01030080       push 80000301  
0043A4E4   |.  6A 00             push 0  
0043A4E6   |.  68 02000000       push 2  
0043A4EB   |.  68 01030080       push 80000301  
0043A4F0   |.  6A 00             push 0  
0043A4F2   |.  68 11000000       push 11  
0043A4F7   |.  68 04000080       push 80000004  
0043A4FC   |.  6A 00             push 0  
0043A4FE   |.  8B45 F0           mov eax,dword ptr ss:\[ebp-10\]  
0043A501   |.  85C0              test eax,eax  
0043A503   |.  75 05             jnz short 3\_.0043A50A  
0043A505   |.  B8 F6914000       mov eax,3\_.004091F6  
0043A50A   |>  50                push eax  
0043A50B   |.  68 03000000       push 3  
0043A510   |.  BB 3C010000       mov ebx,13C  
0043A515   |.  E8 F9490000       call 3\_.0043EF13  
0043A51A   |.  83C4 28           add esp,28  
0043A51D   |.  8945 E0           mov dword ptr ss:\[ebp-20\],eax  
0043A520   |.  68 01030080       push 80000301  
0043A525   |.  6A 00             push 0  
0043A527   |.  68 02000000       push 2  
0043A52C   |.  68 01030080       push 80000301  
0043A531   |.  6A 00             push 0  
0043A533   |.  68 06000000       push 6  
0043A538   |.  68 04000080       push 80000004  
0043A53D   |.  6A 00             push 0  
0043A53F   |.  8B45 F0           mov eax,dword ptr ss:\[ebp-10\]  
0043A542   |.  85C0              test eax,eax  
0043A544   |.  75 05             jnz short 3\_.0043A54B  
0043A546   |.  B8 F6914000       mov eax,3\_.004091F6  
0043A54B   |>  50                push eax  
0043A54C   |.  68 03000000       push 3  
0043A551   |.  BB 3C010000       mov ebx,13C  
0043A556   |.  E8 B8490000       call 3\_.0043EF13  
0043A55B   |.  83C4 28           add esp,28  
0043A55E   |.  8945 DC           mov dword ptr ss:\[ebp-24\],eax  
0043A561   |.  68 01030080       push 80000301  
0043A566   |.  6A 00             push 0  
0043A568   |.  68 02000000       push 2  
0043A56D   |.  68 01030080       push 80000301  
0043A572   |.  6A 00             push 0  
0043A574   |.  68 0B000000       push 0B  
0043A579   |.  68 04000080       push 80000004  
0043A57E   |.  6A 00             push 0  
0043A580   |.  8B45 F0           mov eax,dword ptr ss:\[ebp-10\]  
0043A583   |.  85C0              test eax,eax  
0043A585   |.  75 05             jnz short 3\_.0043A58C  
0043A587   |.  B8 F6914000       mov eax,3\_.004091F6  
0043A58C   |>  50                push eax  
0043A58D   |.  68 03000000       push 3  
0043A592   |.  BB 3C010000       mov ebx,13C  
0043A597   |.  E8 77490000       call 3\_.0043EF13  
0043A59C   |.  83C4 28           add esp,28  
0043A59F   |.  8945 D8           mov dword ptr ss:\[ebp-28\],eax  
0043A5A2   |.  68 01030080       push 80000301  
0043A5A7   |.  6A 00             push 0  
0043A5A9   |.  68 02000000       push 2  
0043A5AE   |.  68 01030080       push 80000301  
0043A5B3   |.  6A 00             push 0  
0043A5B5   |.  68 14000000       push 14  
0043A5BA   |.  68 04000080       push 80000004  
0043A5BF   |.  6A 00             push 0  
0043A5C1   |.  8B45 F0           mov eax,dword ptr ss:\[ebp-10\]  
0043A5C4   |.  85C0              test eax,eax  
0043A5C6   |.  75 05             jnz short 3\_.0043A5CD  
0043A5C8   |.  B8 F6914000       mov eax,3\_.004091F6  
0043A5CD   |>  50                push eax  
0043A5CE   |.  68 03000000       push 3  
0043A5D3   |.  BB 3C010000       mov ebx,13C  
0043A5D8   |.  E8 36490000       call 3\_.0043EF13  
0043A5DD   |.  83C4 28           add esp,28  
0043A5E0   |.  8945 D4           mov dword ptr ss:\[ebp-2C\],eax  
0043A5E3   |.  68 01030080       push 80000301  
0043A5E8   |.  6A 00             push 0  
0043A5EA   |.  68 02000000       push 2  
0043A5EF   |.  68 01030080       push 80000301  
0043A5F4   |.  6A 00             push 0  
0043A5F6   |.  68 19000000       push 19  
0043A5FB   |.  68 04000080       push 80000004  
0043A600   |.  6A 00             push 0  
0043A602   |.  8B45 F0           mov eax,dword ptr ss:\[ebp-10\]  
0043A605   |.  85C0              test eax,eax  
0043A607   |.  75 05             jnz short 3\_.0043A60E  
0043A609   |.  B8 F6914000       mov eax,3\_.004091F6  
0043A60E   |>  50                push eax  
0043A60F   |.  68 03000000       push 3  
0043A614   |.  BB 3C010000       mov ebx,13C  
0043A619   |.  E8 F5480000       call 3\_.0043EF13  
0043A61E   |.  83C4 28           add esp,28  
0043A621   |.  8945 D0           mov dword ptr ss:\[ebp-30\],eax  
0043A624   |.  FF75 D0           push dword ptr ss:\[ebp-30\]                       ; /Arg8  
0043A627   |.  FF75 D4           push dword ptr ss:\[ebp-2C\]                       ; |Arg7  
0043A62A   |.  FF75 D8           push dword ptr ss:\[ebp-28\]                       ; |Arg6  
0043A62D   |.  FF75 DC           push dword ptr ss:\[ebp-24\]                       ; |Arg5  
0043A630   |.  FF75 E0           push dword ptr ss:\[ebp-20\]                       ; |Arg4  
0043A633   |.  FF75 E4           push dword ptr ss:\[ebp-1C\]                       ; |Arg3  
0043A636   |.  FF75 E8           push dword ptr ss:\[ebp-18\]                       ; |Arg2  
0043A639   |.  FF75 EC           push dword ptr ss:\[ebp-14\]                       ; |Arg1  
0043A63C   |.  B9 08000000       mov ecx,8                                        ; |  
0043A641   |.  E8 1233FEFF       call 3\_.0041D958                                 ; \\3\_.0041D958  
0043A646   |.  83C4 20           add esp,20  
0043A649   |.  8945 CC           mov dword ptr ss:\[ebp-34\],eax  
0043A64C   |.  8B5D EC           mov ebx,dword ptr ss:\[ebp-14\]  
0043A64F   |.  85DB              test ebx,ebx  
0043A651   |.  74 09             je short 3\_.0043A65C  
0043A653   |.  53                push ebx  
0043A654   |.  E8 AE480000       call 3\_.0043EF07  
0043A659   |.  83C4 04           add esp,4  
0043A65C   |>  8B5D E8           mov ebx,dword ptr ss:\[ebp-18\]  
0043A65F   |.  85DB              test ebx,ebx  
0043A661   |.  74 09             je short 3\_.0043A66C  
0043A663   |.  53                push ebx  
0043A664   |.  E8 9E480000       call 3\_.0043EF07  
0043A669   |.  83C4 04           add esp,4  
0043A66C   |>  8B5D E4           mov ebx,dword ptr ss:\[ebp-1C\]  
0043A66F   |.  85DB              test ebx,ebx  
0043A671   |.  74 09             je short 3\_.0043A67C  
0043A673   |.  53                push ebx  
0043A674   |.  E8 8E480000       call 3\_.0043EF07  
0043A679   |.  83C4 04           add esp,4  
0043A67C   |>  8B5D E0           mov ebx,dword ptr ss:\[ebp-20\]  
0043A67F   |.  85DB              test ebx,ebx  
0043A681   |.  74 09             je short 3\_.0043A68C  
0043A683   |.  53                push ebx  
0043A684   |.  E8 7E480000       call 3\_.0043EF07  
0043A689   |.  83C4 04           add esp,4  
0043A68C   |>  8B5D DC           mov ebx,dword ptr ss:\[ebp-24\]  
0043A68F   |.  85DB              test ebx,ebx  
0043A691   |.  74 09             je short 3\_.0043A69C  
0043A693   |.  53                push ebx  
0043A694   |.  E8 6E480000       call 3\_.0043EF07  
0043A699   |.  83C4 04           add esp,4  
0043A69C   |>  8B5D D8           mov ebx,dword ptr ss:\[ebp-28\]  
0043A69F   |.  85DB              test ebx,ebx  
0043A6A1   |.  74 09             je short 3\_.0043A6AC  
0043A6A3   |.  53                push ebx  
0043A6A4   |.  E8 5E480000       call 3\_.0043EF07  
0043A6A9   |.  83C4 04           add esp,4  
0043A6AC   |>  8B5D D4           mov ebx,dword ptr ss:\[ebp-2C\]  
0043A6AF   |.  85DB              test ebx,ebx  
0043A6B1   |.  74 09             je short 3\_.0043A6BC  
0043A6B3   |.  53                push ebx  
0043A6B4   |.  E8 4E480000       call 3\_.0043EF07  
0043A6B9   |.  83C4 04           add esp,4  
0043A6BC   |>  8B5D D0           mov ebx,dword ptr ss:\[ebp-30\]  
0043A6BF   |.  85DB              test ebx,ebx  
0043A6C1   |.  74 09             je short 3\_.0043A6CC  
0043A6C3   |.  53                push ebx  
0043A6C4   |.  E8 3E480000       call 3\_.0043EF07  
0043A6C9   |.  83C4 04           add esp,4  
0043A6CC   |>  8B5D F0           mov ebx,dword ptr ss:\[ebp-10\]  
0043A6CF   |.  85DB              test ebx,ebx  
0043A6D1   |.  74 09             je short 3\_.0043A6DC  
0043A6D3   |.  53                push ebx  
0043A6D4   |.  E8 2E480000       call 3\_.0043EF07  
0043A6D9   |.  83C4 04           add esp,4  
0043A6DC   |>  8B45 CC           mov eax,dword ptr ss:\[ebp-34\]  
0043A6DF   |.  8945 F0           mov dword ptr ss:\[ebp-10\],eax  
0043A6E2   |.  68 04000080       push 80000004  
0043A6E7   |.  6A 00             push 0  
0043A6E9   |.  8B45 F0           mov eax,dword ptr ss:\[ebp-10\]  
0043A6EC   |.  85C0              test eax,eax  
0043A6EE   |.  75 05             jnz short 3\_.0043A6F5  
0043A6F0   |.  B8 F6914000       mov eax,3\_.004091F6  
0043A6F5   |>  50                push eax  
0043A6F6   |.  68 01000000       push 1  
0043A6FB   |.  BB 64010000       mov ebx,164  
0043A700   |.  E8 0E480000       call 3\_.0043EF13  
0043A705   |.  83C4 10           add esp,10  
0043A708   |.  8945 E0           mov dword ptr ss:\[ebp-20\],eax  
0043A70B   |.  8955 E4           mov dword ptr ss:\[ebp-1C\],edx  
0043A70E   |.  DD45 E0           fld qword ptr ss:\[ebp-20\]  
0043A711   |.  E8 B031FEFF       call 3\_.0041D8C6  
0043A716   |.  68 01030080       push 80000301  
0043A71B   |.  6A 00             push 0  
0043A71D   |.  50                push eax  
0043A71E   |.  68 01000000       push 1  
0043A723   |.  BB D4010000       mov ebx,1D4  
0043A728   |.  E8 E6470000       call 3\_.0043EF13  
0043A72D   |.  83C4 10           add esp,10  
0043A730   |.  8945 DC           mov dword ptr ss:\[ebp-24\],eax  
0043A733   |.  68 04000080       push 80000004  
0043A738   |.  6A 00             push 0  
0043A73A   |.  8B45 DC           mov eax,dword ptr ss:\[ebp-24\]  
0043A73D   |.  85C0              test eax,eax  
0043A73F   |.  75 05             jnz short 3\_.0043A746  
0043A741   |.  B8 F6914000       mov eax,3\_.004091F6  
0043A746   |>  50                push eax  
0043A747   |.  68 01000000       push 1  
0043A74C   |.  BB 68010000       mov ebx,168  
0043A751   |.  E8 BD470000       call 3\_.0043EF13  
0043A756   |.  83C4 10           add esp,10  
0043A759   |.  8945 D8           mov dword ptr ss:\[ebp-28\],eax  
0043A75C   |.  8B5D DC           mov ebx,dword ptr ss:\[ebp-24\]  
0043A75F   |.  85DB              test ebx,ebx  
0043A761   |.  74 09             je short 3\_.0043A76C  
0043A763   |.  53                push ebx  
0043A764   |.  E8 9E470000       call 3\_.0043EF07  
0043A769   |.  83C4 04           add esp,4  
0043A76C   |>  8B5D F0           mov ebx,dword ptr ss:\[ebp-10\]  
0043A76F   |.  85DB              test ebx,ebx  
0043A771   |.  74 09             je short 3\_.0043A77C  
0043A773   |.  53                push ebx  
0043A774   |.  E8 8E470000       call 3\_.0043EF07  
0043A779   |.  83C4 04           add esp,4  
0043A77C   |>  8B45 D8           mov eax,dword ptr ss:\[ebp-28\]  
0043A77F   |.  8945 F0           mov dword ptr ss:\[ebp-10\],eax  
0043A782   |.  8B45 F0           mov eax,dword ptr ss:\[ebp-10\]  
0043A785   |.  85C0              test eax,eax  
0043A787   |.  74 15             je short 3\_.0043A79E  
0043A789   |.  50                push eax  
0043A78A   |.  8BD8              mov ebx,eax  
0043A78C   |.  E8 5C31FEFF       call 3\_.0041D8ED  
0043A791   |.  40                inc eax  
0043A792   |.  50                push eax  
0043A793   |.  E8 75470000       call 3\_.0043EF0D  
0043A798   |.  59                pop ecx  
0043A799   |.  5E                pop esi  
0043A79A   |.  8BF8              mov edi,eax  
0043A79C   |.  F3:A4             rep movs byte ptr es:\[edi\],byte ptr ds:\[esi\]  
0043A79E   |>  E9 00000000       jmp 3\_.0043A7A3  
0043A7A3   |>  50                push eax  
0043A7A4   |.  8B5D FC           mov ebx,dword ptr ss:\[ebp-4\]  
0043A7A7   |.  53                push ebx  
0043A7A8   |.  68 0A000600       push 6000A  
0043A7AD   |.  6A 00             push 0  
0043A7AF   |.  53                push ebx  
0043A7B0   |.  6A 01             push 1  
0043A7B2   |.  BB FC000000       mov ebx,0FC  
0043A7B7   |.  B8 05000000       mov eax,5  
0043A7BC   |.  E8 6A470000       call 3\_.0043EF2B  
0043A7C1   |.  83C4 10           add esp,10  
0043A7C4   |.  E8 3E470000       call 3\_.0043EF07  
0043A7C9   |.  83C4 04           add esp,4  
0043A7CC   |.  8B5D F8           mov ebx,dword ptr ss:\[ebp-8\]  
0043A7CF   |.  53                push ebx  
0043A7D0   |.  68 0A000600       push 6000A  
0043A7D5   |.  6A 00             push 0  
0043A7D7   |.  53                push ebx  
0043A7D8   |.  6A 01             push 1  
0043A7DA   |.  BB FC000000       mov ebx,0FC  
0043A7DF   |.  B8 05000000       mov eax,5  
0043A7E4   |.  E8 42470000       call 3\_.0043EF2B  
0043A7E9   |.  83C4 10           add esp,10  
0043A7EC   |.  E8 16470000       call 3\_.0043EF07  
0043A7F1   |.  83C4 04           add esp,4  
0043A7F4   |.  8B5D F4           mov ebx,dword ptr ss:\[ebp-C\]  
0043A7F7   |.  53                push ebx  
0043A7F8   |.  68 0A000600       push 6000A  
0043A7FD   |.  6A 00             push 0  
0043A7FF   |.  53                push ebx  
0043A800   |.  6A 01             push 1  
0043A802   |.  BB FC000000       mov ebx,0FC  
0043A807   |.  B8 05000000       mov eax,5  
0043A80C   |.  E8 1A470000       call 3\_.0043EF2B  
0043A811   |.  83C4 10           add esp,10  
0043A814   |.  E8 EE460000       call 3\_.0043EF07  
0043A819   |.  83C4 04           add esp,4  
0043A81C   |.  8B5D F0           mov ebx,dword ptr ss:\[ebp-10\]  
0043A81F   |.  85DB              test ebx,ebx  
0043A821   |.  74 09             je short 3\_.0043A82C  
0043A823   |.  53                push ebx  
0043A824   |.  E8 DE460000       call 3\_.0043EF07  
0043A829   |.  83C4 04           add esp,4  
0043A82C   |>  58                pop eax  
0043A82D   |.  8BE5              mov esp,ebp  
  
\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
总结：算法看了很讨厌,不过是明码的  
具体算法没看以后有空我会慢慢跟踪一下

 4dd466d0e1e0cc123afbe504da2462ca