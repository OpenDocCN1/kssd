

# 税务师 200306 

> 原文：[`www.pediy.com/kssd/pediy08/pediy8-9.htm`](https://www.pediy.com/kssd/pediy08/pediy8-9.htm)

税务师(TaxExpert) 200306 通用版注册算法分析（KEYFILE，3DES）  
软件大小：  8924 KB  
软件语言：  简体中文  
软件类别：  国产软件 / 共享版 / 财务管理  
下载地址:http://www4.skycn.com/soft/8527.html  
应用平台：  Win9x/NT/2000/XP  
作者声明：初学 Crack，只是感兴趣，没有其他目的。失误之处敬请诸位大侠赐教！  
破解工具：   PEID ，OD ，VC.NET。  
\==============================================================================================================  
【破解过程】一开始破解这个软件因为逆推算法觉得无法做注册机。后来用 PEID 看该软用了 DES 就看了一些 DES 的一些加密文章  
和算法。主要是参考了王俊川的 3DES，C 代码一边用 OD 跟踪一边用 VC.NET 跟踪 3DES 代码，才恍然大悟。3DES 可以加密也可以  
解密。别罗嗦了，看下面的过程吧。  
  
  
0040708F         90                  nop  
00407090      $  6A FF               push -1  
00407092      .  68 D1234B00         push TaxExper.004B23D1                    ;  SE 句柄安装  
00407097      .  64:A1 00000000      mov eax,dword ptr fs:\[0\]  
0040709D      .  50                  push eax  
0040709E      .  64:8925 00000000    mov dword ptr fs:\[0\],esp  
004070A5      .  81EC 040C0000       sub esp,0C04                              ;  BP CreateFileA  
004070AB      .  53                  push ebx  
004070AC      .  33DB                xor ebx,ebx  
004070AE      .  56                  push esi  
004070AF      .  8BF1                mov esi,ecx  
004070B1      .  8D4C24 48           lea ecx,dword ptr ss:\[esp+48\]  
004070B5      .  895C24 58           mov dword ptr ss:\[esp+58\],ebx  
004070B9      .  895C24 5C           mov dword ptr ss:\[esp+5C\],ebx  
004070BD      .  895C24 60           mov dword ptr ss:\[esp+60\],ebx  
004070C1      .  895C24 64           mov dword ptr ss:\[esp+64\],ebx  
004070C5      .  895C24 68           mov dword ptr ss:\[esp+68\],ebx  
004070C9      .  895C24 6C           mov dword ptr ss:\[esp+6C\],ebx  
004070CD      .  E8 CCD90800         call TaxExper.00494A9E  
004070D2      .  8D4424 58           lea eax,dword ptr ss:\[esp+58\]  
004070D6      .  50                  push eax                                  ; /Arg2  
004070D7      .  68 F0C04B00         push TaxExper.004BC0F0                    ; |Arg1 = 004BC0F0 ASCII "TaxExpert.nfo"  
004070DC      .  899C24 1C0C0000     mov dword ptr ss:\[esp+C1C\],ebx            ; |  
004070E3      .  E8 B1E10800         call TaxExper.00495299                    ; \\打开  "TaxExpert.nfo"文件  
004070E8      .  85C0                test eax,eax  
004070EA      .  0F84 D5010000       je TaxExper.004072C5                      ;  打开是否成功  
004070F0      .  53                  push ebx  
004070F1      .  68 F0C04B00         push TaxExper.004BC0F0                    ;  ASCII "TaxExpert.nfo"  
004070F6      .  8D4C24 30           lea ecx,dword ptr ss:\[esp+30\]  
004070FA      .  E8 75DE0800         call TaxExper.00494F74  
004070FF      .  68 78030000         push 378                                  ; /Arg2 = 00000378  
00407104      .  8D8C24 84010000     lea ecx,dword ptr ss:\[esp+184\]            ; |  
0040710B      .  51                  push ecx                                  ; |Arg1  
0040710C      .  8D4C24 30           lea ecx,dword ptr ss:\[esp+30\]             ; |  
00407110      .  C68424 1C0C0000 01  mov byte ptr ss:\[esp+C1C\],1               ; |  
00407118      .  E8 8ED60800         call TaxExper.004947AB                    ; \\TaxExper.004947AB  
0040711D      .  3D 78030000         cmp eax,378                               ;  "TaxExpert.nfo" 字节长度是否为:0X378=888  
00407122      .  8D4C24 28           lea ecx,dword ptr ss:\[esp+28\]  
00407126      .  74 0A               je short TaxExper.00407132                ;  不等就 88  
00407128      .  E8 0FD90800         call TaxExper.00494A3C  
0040712D      .  E9 83010000         jmp TaxExper.004072B5  
00407132      >  57                  push edi  
00407133      .  E8 04D90800         call TaxExper.00494A3C                    ;  读"TaxExpert.nfo" 12F160 DD ESP+184 保存起始地址  
00407138      .  8B9424 59020000     mov edx,dword ptr ss:\[esp+259\]            ;  259-184=0xd5=213d 取第 213 位开始的 4 个字节  
0040713F      .  8B8C24 61020000     mov ecx,dword ptr ss:\[esp+261\]            ;  261-184=0xDD=213d 取第 221 位开始的 4 个字节  
00407146      .  8B8424 5D020000     mov eax,dword ptr ss:\[esp+25D\]            ;  25D-184=0xd9=213d 取第 217 位开始的 4 个字节  
0040714D      .  895424 3C           mov dword ptr ss:\[esp+3C\],edx             ;  取完保存在 edx \[esp+3C\] 取名 REGSTR\[\]  
00407151      .  8B9424 65020000     mov edx,dword ptr ss:\[esp+265\]            ;  265-184=0xE1=225d 取第 213 位开始的 4 个字节  
00407158      .  894C24 44           mov dword ptr ss:\[esp+44\],ecx  
0040715C      .  8D8C24 FC040000     lea ecx,dword ptr ss:\[esp+4FC\]            ;  4FC-184=0X378 尾部地址传给 ECX  
00407163      .  894424 40           mov dword ptr ss:\[esp+40\],eax  
00407167      .  895424 48           mov dword ptr ss:\[esp+48\],edx  
0040716B      .  E8 00240700         call TaxExper.00479570                    ;  影响 ESP 指针  
00407170      .  8B86 480E0000       mov eax,dword ptr ds:\[esi+E48\]            ;  机器码  
00407176      .  8B78 F4             mov edi,dword ptr ds:\[eax-C\]  
00407179      .  8B48 FC             mov ecx,dword ptr ds:\[eax-4\]  
0040717C      .  81C6 480E0000       add esi,0E48  
00407182      .  83E8 10             sub eax,10  
00407185      .  83F9 01             cmp ecx,1  
00407188      .  C68424 180C0000 02  mov byte ptr ss:\[esp+C18\],2  
00407190      .  7E 0B               jle short TaxExper.0040719D  
00407192      .  8B48 04             mov ecx,dword ptr ds:\[eax+4\]  
00407195      .  51                  push ecx  
00407196      .  8BCE                mov ecx,esi  
00407198      .  E8 73A6FFFF         call TaxExper.00401810                    ;  机器码  
0040719D      >  8B36                mov esi,dword ptr ds:\[esi\]  
0040719F      .  6A 01               push 1  
004071A1      .  57                  push edi  
004071A2      .  56                  push esi  
004071A3      .  6A 10               push 10  
004071A5      .  8D5424 4C           lea edx,dword ptr ss:\[esp+4C\]             ;  刚取的注册码 REGSTR\[\]  
004071A9      .  52                  push edx  
004071AA      .  8D4424 30           lea eax,dword ptr ss:\[esp+30\]  
004071AE      .  50                  push eax  
004071AF      .  8D8C24 14050000     lea ecx,dword ptr ss:\[esp+514\]  
004071B6      .  E8 85280000         call TaxExper.00409A40                    ;  关键算法  
004071BB      .  8B0D E0C04B00       mov ecx,dword ptr ds:\[4BC0E0\]             ;  TaxE  
004071C1      .  A1 E8C04B00         mov eax,dword ptr ds:\[4BC0E8\]             ;  t-V2  
004071C6      .  8B15 E4C04B00       mov edx,dword ptr ds:\[4BC0E4\]             ;  xper  
004071CC      .  894C24 0C           mov dword ptr ss:\[esp+C\],ecx  
004071D0      .  8B0D ECC04B00       mov ecx,dword ptr ds:\[4BC0EC\]             ;  003  
004071D6      .  894424 14           mov dword ptr ss:\[esp+14\],eax  
004071DA      .  894C24 18           mov dword ptr ss:\[esp+18\],ecx  
004071DE      .  895424 10           mov dword ptr ss:\[esp+10\],edx  
004071E2      .  B9 01000000         mov ecx,1  
004071E7      .  33C0                xor eax,eax  
004071E9      .  5F                  pop edi  
004071EA      .  8D9B 00000000       lea ebx,dword ptr ds:\[ebx\]  
004071F0      >  8A5404 08           mov dl,byte ptr ss:\[esp+eax+8\]            ;  ESP+8="TaxExpert-V2003"=12EFE8  
004071F4      .  3A5404 18           cmp dl,byte ptr ss:\[esp+eax+18\]           ;  ESP+18=12EFF8  
004071F8      .  74 02               je short TaxExper.004071FC  
004071FA      .  33C9                xor ecx,ecx  
004071FC      >  8A5404 09           mov dl,byte ptr ss:\[esp+eax+9\]  
00407200      .  3A5404 19           cmp dl,byte ptr ss:\[esp+eax+19\]  
00407204      .  74 02               je short TaxExper.00407208  
00407206      .  33C9                xor ecx,ecx  
00407208      >  8A5404 0A           mov dl,byte ptr ss:\[esp+eax+A\]  
0040720C      .  3A5404 1A           cmp dl,byte ptr ss:\[esp+eax+1A\]  
00407210      .  74 02               je short TaxExper.00407214  
00407212      .  33C9                xor ecx,ecx  
00407214      >  8A5404 0B           mov dl,byte ptr ss:\[esp+eax+B\]  
00407218      .  3A5404 1B           cmp dl,byte ptr ss:\[esp+eax+1B\]  
0040721C      .  74 02               je short TaxExper.00407220  
0040721E      .  33C9                xor ecx,ecx  
00407220      >  8A5404 0C           mov dl,byte ptr ss:\[esp+eax+C\]  
00407224      .  3A5404 1C           cmp dl,byte ptr ss:\[esp+eax+1C\]  
00407228      .  74 02               je short TaxExper.0040722C  
0040722A      .  33C9                xor ecx,ecx  
0040722C      >  8A5404 0D           mov dl,byte ptr ss:\[esp+eax+D\]  
00407230      .  3A5404 1D           cmp dl,byte ptr ss:\[esp+eax+1D\]  
00407234      .  74 02               je short TaxExper.00407238  
00407236      .  33C9                xor ecx,ecx  
00407238      >  8A5404 0E           mov dl,byte ptr ss:\[esp+eax+E\]  
0040723C      .  3A5404 1E           cmp dl,byte ptr ss:\[esp+eax+1E\]  
00407240      .  74 02               je short TaxExper.00407244  
00407242      .  33C9                xor ecx,ecx  
00407244      >  8A5404 0F           mov dl,byte ptr ss:\[esp+eax+F\]  
00407248      .  3A5404 1F           cmp dl,byte ptr ss:\[esp+eax+1F\]  
0040724C      .  74 02               je short TaxExper.00407250  
0040724E      .  33C9                xor ecx,ecx  
00407250      >  83C0 08             add eax,8  
00407253      .  83F8 10             cmp eax,10  
00407256      .^ 7C 98               jl short TaxExper.004071F0  
00407258      .  3BCB                cmp ecx,ebx  
0040725A      .  C68424 140C0000 01  mov byte ptr ss:\[esp+C14\],1  
00407262      .  8D8C24 F8040000     lea ecx,dword ptr ss:\[esp+4F8\]  
00407269      .  74 45               je short TaxExper.004072B0                ;  关键跳转 可以爆  
0040726B      .  E8 205B0600         call TaxExper.0046CD90  
00407270      .  8D4C24 28           lea ecx,dword ptr ss:\[esp+28\]  
  
\======================================================================================================  
call TaxExper.00409A40                    ;  关键算法  
\=========================================================================================================  
00409A40     /$  53                  push ebx  
00409A41     |.  55                  push ebp  
00409A42     |.  56                  push esi  
00409A43     |.  57                  push edi  
00409A44     |.  8B7C24 14           mov edi,dword ptr ss:\[esp+14\]  
00409A48     |.  85FF                test edi,edi  
00409A4A     |.  8BF1                mov esi,ecx  
00409A4C     |.  0F84 C3000000       je TaxExper.00409B15                      ;  机器码是否等于 0  
00409A52     |.  8B6C24 18           mov ebp,dword ptr ss:\[esp+18\]  
00409A56     |.  85ED                test ebp,ebp                              ;  REGSTR\[\]是否为空  
00409A58     |.  0F84 B7000000       je TaxExper.00409B15  
00409A5E     |.  8B4424 20           mov eax,dword ptr ss:\[esp+20\]  
00409A62     |.  85C0                test eax,eax  
00409A64     |.  0F84 AB000000       je TaxExper.00409B15  
00409A6A     |.  8B5C24 1C           mov ebx,dword ptr ss:\[esp+1C\]  
00409A6E     |.  83C3 07             add ebx,7  
00409A71     |.  83E3 F8             and ebx,FFFFFFF8                          ;  EBX=0X10  
00409A74     |.  0F84 9B000000       je TaxExper.00409B15  
00409A7A     |.  8B4C24 24           mov ecx,dword ptr ss:\[esp+24\]  
00409A7E     |.  51                  push ecx                                  ;  机器码长度  
00409A7F     |.  50                  push eax                                  ;  机器码入栈  
00409A80     |.  8BCE                mov ecx,esi  
00409A82     |.  E8 A9FDFFFF         call TaxExper.00409830                    ;  算法进入  
00409A87     |.  8A86 00060000       mov al,byte ptr ds:\[esi+600\]  
00409A8D     |.  C1FB 03             sar ebx,3                                 ;  EBX SHR 3  
00409A90     |.  84C0                test al,al  
00409A92     |.  75 25               jnz short TaxExper.00409AB9               ;   !Is3DES  
00409A94     |.  85DB                test ebx,ebx  
00409A96     |.  7E 74               jle short TaxExper.00409B0C  
00409A98     |>  8B5424 28           /mov edx,dword ptr ss:\[esp+28\]            ;  // 1 次 DES  
00409A9C     |.  52                  |push edx  
00409A9D     |.  56                  |push esi  
00409A9E     |.  55                  |push ebp  
00409A9F     |.  57                  |push edi  
00409AA0     |.  8BCE                |mov ecx,esi  
00409AA2     |.  E8 59FEFFFF         |call TaxExper.00409900                   ;  DES(Out, In, &SubKey\[0\], Type);  
00409AA7     |.  83C7 08             |add edi,8  
00409AAA     |.  83C5 08             |add ebp,8  
00409AAD     |.  4B                  |dec ebx  
00409AAE     |.^ 75 E8               \\jnz short TaxExper.00409A98  
00409AB0     |.  5F                  pop edi  
00409AB1     |.  5E                  pop esi  
00409AB2     |.  5D                  pop ebp  
00409AB3     |.  B0 01               mov al,1  
00409AB5     |.  5B                  pop ebx  
00409AB6     |.  C2 1800             retn 18  
\================================================================================================  
call TaxExper.00409830                    ;  算法进入  
\=================================================================================================  
00409830     /$  53                  push ebx  
00409831     |.  8BD9                mov ebx,ecx  
00409833     |.  33C9                xor ecx,ecx                               ;  ECX=0  
00409835     |.  8D83 01070000       lea eax,dword ptr ds:\[ebx+701\]  
0040983B     |.  8BD0                mov edx,eax  
0040983D     |.  890A                mov dword ptr ds:\[edx\],ecx  
0040983F     |.  894A 04             mov dword ptr ds:\[edx+4\],ecx  
00409842     |.  55                  push ebp  
00409843     |.  8B6C24 10           mov ebp,dword ptr ss:\[esp+10\]             ;  机器码长度  
00409847     |.  83FD 10             cmp ebp,10                                ;  机器码长度是否大于 16  
0040984A     |.  894A 08             mov dword ptr ds:\[edx+8\],ecx  
0040984D     |.  894A 0C             mov dword ptr ds:\[edx+C\],ecx  
00409850     |.  B9 10000000         mov ecx,10  
00409855     |.  7F 02               jg short TaxExper.00409859  
00409857     |.  8BCD                mov ecx,ebp  
00409859     |>  56                  push esi  
0040985A     |.  8B7424 10           mov esi,dword ptr ss:\[esp+10\]             ;  机器码  
0040985E     |.  8BD1                mov edx,ecx  
00409860     |.  57                  push edi  
00409861     |.  C1E9 02             shr ecx,2  
00409864     |.  8BF8                mov edi,eax  
00409866     |.  F3:A5               rep movs dword ptr es:\[edi\],dword ptr ds:>  
00409868     |.  8BCA                mov ecx,edx  
0040986A     |.  83E1 03             and ecx,3  
0040986D     |.  50                  push eax                                  ;  机器码入栈=KEY  
0040986E     |.  F3:A4               rep movs byte ptr es:\[edi\],byte ptr ds:\[e>  
00409870     |.  53                  push ebx  
00409871     |.  8BCB                mov ecx,ebx  
00409873     |.  E8 B8FEFFFF         call TaxExper.00409730                    ;  Des\_SetKey(const char Key\[8\])  
00409878     |.  83FD 08             cmp ebp,8                                 ;  机器码长度是否小于等于 8  
0040987B     |.  5F                  pop edi  
0040987C     |.  5E                  pop esi  
0040987D     |.  7E 22               jle short TaxExper.004098A1               ;  Is3DES = len>8 ? (SetSubKey(&SubKey\[1\], &deskey\[8\]), true) : false;  
0040987F     |.  8D83 09070000       lea eax,dword ptr ds:\[ebx+709\]  
00409885     |.  50                  push eax                                  ;  EAX=G3RZ3C  
00409886     |.  8D8B 00030000       lea ecx,dword ptr ds:\[ebx+300\]  
0040988C     |.  51                  push ecx  
0040988D     |.  8BCB                mov ecx,ebx  
0040988F     |.  E8 9CFEFFFF         call TaxExper.00409730  
00409894     |.  B0 01               mov al,1  
00409896     |.  5D                  pop ebp  
00409897     |.  8883 00060000       mov byte ptr ds:\[ebx+600\],al  
0040989D     |.  5B                  pop ebx  
0040989E     |.  C2 0800             retn 8  
004098A1     |>  32C0                xor al,al  
004098A3     |.  5D                  pop ebp  
004098A4     |.  8883 00060000       mov byte ptr ds:\[ebx+600\],al  
004098AA     |.  5B                  pop ebx  
004098AB     \\.  C2 0800             retn 8  
\===========================================================================================  
 call TaxExper.00409730                    ;  Des\_SetKey(const char Key\[8\])  
++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++  
00409831     |.  8BD9                mov ebx,ecx  
00409833     |.  33C9                xor ecx,ecx                               ;  ECX=0  
00409835     |.  8D83 01070000       lea eax,dword ptr ds:\[ebx+701\]  
0040983B     |.  8BD0                mov edx,eax  
0040983D     |.  890A                mov dword ptr ds:\[edx\],ecx  
0040983F     |.  894A 04             mov dword ptr ds:\[edx+4\],ecx  
00409842     |.  55                  push ebp  
00409843     |.  8B6C24 10           mov ebp,dword ptr ss:\[esp+10\]             ;  机器码长度  
00409847     |.  83FD 10             cmp ebp,10                                ;  机器码长度是否大于 16  
0040984A     |.  894A 08             mov dword ptr ds:\[edx+8\],ecx  
0040984D     |.  894A 0C             mov dword ptr ds:\[edx+C\],ecx  
00409850     |.  B9 10000000         mov ecx,10  
00409855     |.  7F 02               jg short TaxExper.00409859  
00409857     |.  8BCD                mov ecx,ebp  
00409859     |>  56                  push esi  
0040985A     |.  8B7424 10           mov esi,dword ptr ss:\[esp+10\]             ;  机器码  
0040985E     |.  8BD1                mov edx,ecx  
00409860     |.  57                  push edi  
00409861     |.  C1E9 02             shr ecx,2  
00409864     |.  8BF8                mov edi,eax  
00409866     |.  F3:A5               rep movs dword ptr es:\[edi\],dword ptr ds:>  
00409868     |.  8BCA                mov ecx,edx  
0040986A     |.  83E1 03             and ecx,3  
0040986D     |.  50                  push eax                                  ;  机器码入栈=KEY  
0040986E     |.  F3:A4               rep movs byte ptr es:\[edi\],byte ptr ds:\[e>  
00409870     |.  53                  push ebx  
00409871     |.  8BCB                mov ecx,ebx  
00409873     |.  E8 B8FEFFFF         call TaxExper.00409730                    ;  Des\_SetKey(const char Key\[8\])  
00409878     |.  83FD 08             cmp ebp,8                                 ;  机器码长度是否小于等于 8  
0040987B     |.  5F                  pop edi  
0040987C     |.  5E                  pop esi  
0040987D     |.  7E 22               jle short TaxExper.004098A1               ;  Is3DES = len>8 ? (SetSubKey(&SubKey\[1\], &deskey\[8\]), true) : false;  
0040987F     |.  8D83 09070000       lea eax,dword ptr ds:\[ebx+709\]  
00409885     |.  50                  push eax                                  ;  EAX=G3RZ3C  
00409886     |.  8D8B 00030000       lea ecx,dword ptr ds:\[ebx+300\]  
0040988C     |.  51                  push ecx  
0040988D     |.  8BCB                mov ecx,ebx  
0040988F     |.  E8 9CFEFFFF         call TaxExper.00409730  
00409894     |.  B0 01               mov al,1  
00409896     |.  5D                  pop ebp  
00409897     |.  8883 00060000       mov byte ptr ds:\[ebx+600\],al  
0040989D     |.  5B                  pop ebx  
  
\====================================================================================================  
  
00409730     /$  8B4424 08           mov eax,dword ptr ss:\[esp+8\]  
00409734     |.  53                  push ebx  
00409735     |.  55                  push ebp  
00409736     |.  56                  push esi  
00409737     |.  57                  push edi  
00409738     |.  6A 40               push 40                                   ;  64  
0040973A     |.  50                  push eax                                  ;  KEY=机器码  
0040973B     |.  68 A0A64E00         push TaxExper.004EA6A0                    ;  K  
00409740     |.  8BF9                mov edi,ecx  
00409742     |.  E8 49FFFFFF         call TaxExper.00409690                    ;  ByteToBit(K, Key, 64);  
00409747     |.  6A 38               push 38                                   ;  38=56  
00409749     |.  68 68C54B00         push TaxExper.004BC568                    ;  PC1\_Table\[56\]  
0040974E     |.  68 A0A64E00         push TaxExper.004EA6A0                    ;  K  
00409753     |.  68 A0A64E00         push TaxExper.004EA6A0                    ;  K  
00409758     |.  8BCF                mov ecx,edi  
0040975A     |.  E8 51FEFFFF         call TaxExper.004095B0                    ;  Transform(K, K, PC1\_Table, 56);  
0040975F     |.  8B5C24 14           mov ebx,dword ptr ss:\[esp+14\]  
00409763     |.  8B2D 9C744E00       mov ebp,dword ptr ds:\[4E749C\]             ;  TaxExper.004EA6A0  
00409769     |.  33F6                xor esi,esi  
0040976B     |.  EB 03               jmp short TaxExper.00409770  
0040976D     |   8D49 00             lea ecx,dword ptr ds:\[ecx\]  
00409770     |>  0FBE8E D0C54B00     /movsx ecx,byte ptr ds:\[esi+4BC5D0\]       ;  \[4BC5D0\]=LOOP\_Table\[16\]  
00409777     |.  51                  |push ecx                                 ;  LOOP\_Table\[i\]  
00409778     |.  6A 1C               |push 1C                                  ;  1C=28  
0040977A     |.  55                  |push ebp                                 ;  KL  
0040977B     |.  8BCF                |mov ecx,edi  
0040977D     |.  E8 AEFEFFFF         |call TaxExper.00409630                   ;  RotateL(KL, 28, LOOP\_Table\[i\]);  
00409782     |.  0FBE96 D0C54B00     |movsx edx,byte ptr ds:\[esi+4BC5D0\]  
00409789     |.  A1 98744E00         |mov eax,dword ptr ds:\[4E7498\]  
0040978E     |.  52                  |push edx  
0040978F     |.  6A 1C               |push 1C  
00409791     |.  50                  |push eax  
00409792     |.  8BCF                |mov ecx,edi  
00409794     |.  E8 97FEFFFF         |call TaxExper.00409630                   ;  RotateL(KL, 28, LOOP\_Table\[i\]);  
00409799     |.  6A 30               |push 30                                  ;  48  
0040979B     |.  68 A0C54B00         |push TaxExper.004BC5A0  
004097A0     |.  68 A0A64E00         |push TaxExper.004EA6A0  
004097A5     |.  53                  |push ebx  
004097A6     |.  8BCF                |mov ecx,edi  
004097A8     |.  E8 03FEFFFF         |call TaxExper.004095B0                   ;  Transform(SubKey\[i\], K, PC2\_Table, 48);  
004097AD     |.  46                  |inc esi  
004097AE     |.  83C3 30             |add ebx,30  
004097B1     |.  83FE 10             |cmp esi,10  
004097B4     |.^ 7C BA               \\jl short TaxExper.00409770               ;  for(int i=0; i<16; i++)  
004097B6     |.  5F                  pop edi  
004097B7     |.  5E                  pop esi  
004097B8     |.  5D                  pop ebp  
004097B9     |.  5B                  pop ebx  
004097BA     \\.  C2 0800             retn 8  
  
\=================================================================================================  
00409730     /$  8B4424 08           mov eax,dword ptr ss:\[esp+8\]  
00409734     |.  53                  push ebx  
00409735     |.  55                  push ebp  
00409736     |.  56                  push esi  
00409737     |.  57                  push edi  
00409738     |.  6A 40               push 40                                   ;  64  
0040973A     |.  50                  push eax                                  ;  KEY=机器码  
0040973B     |.  68 A0A64E00         push TaxExper.004EA6A0                    ;  K  
00409740     |.  8BF9                mov edi,ecx  
00409742     |.  E8 49FFFFFF         call TaxExper.00409690                    ;  ByteToBit(K, Key, 64);  
00409747     |.  6A 38               push 38                                   ;  38=56  
00409749     |.  68 68C54B00         push TaxExper.004BC568                    ;  PC1\_Table\[56\]  
0040974E     |.  68 A0A64E00         push TaxExper.004EA6A0                    ;  K  
00409753     |.  68 A0A64E00         push TaxExper.004EA6A0                    ;  K  
00409758     |.  8BCF                mov ecx,edi  
0040975A     |.  E8 51FEFFFF         call TaxExper.004095B0                    ;  Transform(K, K, PC1\_Table, 56);  
0040975F     |.  8B5C24 14           mov ebx,dword ptr ss:\[esp+14\]  
00409763     |.  8B2D 9C744E00       mov ebp,dword ptr ds:\[4E749C\]             ;  TaxExper.004EA6A0  
00409769     |.  33F6                xor esi,esi  
0040976B     |.  EB 03               jmp short TaxExper.00409770  
0040976D     |   8D49 00             lea ecx,dword ptr ds:\[ecx\]  
00409770     |>  0FBE8E D0C54B00     /movsx ecx,byte ptr ds:\[esi+4BC5D0\]       ;  \[4BC5D0\]=LOOP\_Table\[16\]  
00409777     |.  51                  |push ecx                                 ;  LOOP\_Table\[i\]  
00409778     |.  6A 1C               |push 1C                                  ;  1C=28  
0040977A     |.  55                  |push ebp                                 ;  KL  
0040977B     |.  8BCF                |mov ecx,edi  
0040977D     |.  E8 AEFEFFFF         |call TaxExper.00409630                   ;  RotateL(KL, 28, LOOP\_Table\[i\]);  
00409782     |.  0FBE96 D0C54B00     |movsx edx,byte ptr ds:\[esi+4BC5D0\]  
00409789     |.  A1 98744E00         |mov eax,dword ptr ds:\[4E7498\]  
0040978E     |.  52                  |push edx  
0040978F     |.  6A 1C               |push 1C  
00409791     |.  50                  |push eax  
00409792     |.  8BCF                |mov ecx,edi  
00409794     |.  E8 97FEFFFF         |call TaxExper.00409630                   ;  RotateL(KL, 28, LOOP\_Table\[i\]);  
00409799     |.  6A 30               |push 30                                  ;  48  
0040979B     |.  68 A0C54B00         |push TaxExper.004BC5A0  
004097A0     |.  68 A0A64E00         |push TaxExper.004EA6A0  
004097A5     |.  53                  |push ebx  
004097A6     |.  8BCF                |mov ecx,edi  
004097A8     |.  E8 03FEFFFF         |call TaxExper.004095B0                   ;  Transform(SubKey\[i\], K, PC2\_Table, 48);  
004097AD     |.  46                  |inc esi  
004097AE     |.  83C3 30             |add ebx,30  
004097B1     |.  83FE 10             |cmp esi,10  
004097B4     |.^ 7C BA               \\jl short TaxExper.00409770               ;  for(int i=0; i<16; i++)  
004097B6     |.  5F                  pop edi  
004097B7     |.  5E                  pop esi  
004097B8     |.  5D                  pop ebp  
004097B9     |.  5B                  pop ebx  
\======================================================================================================  
00409830     /$  53                  push ebx  
00409831     |.  8BD9                mov ebx,ecx  
00409833     |.  33C9                xor ecx,ecx                               ;  ECX=0  
00409835     |.  8D83 01070000       lea eax,dword ptr ds:\[ebx+701\]  
0040983B     |.  8BD0                mov edx,eax  
0040983D     |.  890A                mov dword ptr ds:\[edx\],ecx  
0040983F     |.  894A 04             mov dword ptr ds:\[edx+4\],ecx  
00409842     |.  55                  push ebp  
00409843     |.  8B6C24 10           mov ebp,dword ptr ss:\[esp+10\]             ;  机器码长度  
00409847     |.  83FD 10             cmp ebp,10                                ;  机器码长度是否大于 16  
0040984A     |.  894A 08             mov dword ptr ds:\[edx+8\],ecx  
0040984D     |.  894A 0C             mov dword ptr ds:\[edx+C\],ecx  
00409850     |.  B9 10000000         mov ecx,10  
00409855     |.  7F 02               jg short TaxExper.00409859  
00409857     |.  8BCD                mov ecx,ebp  
00409859     |>  56                  push esi  
0040985A     |.  8B7424 10           mov esi,dword ptr ss:\[esp+10\]             ;  机器码  
0040985E     |.  8BD1                mov edx,ecx  
00409860     |.  57                  push edi  
00409861     |.  C1E9 02             shr ecx,2  
00409864     |.  8BF8                mov edi,eax  
00409866     |.  F3:A5               rep movs dword ptr es:\[edi\],dword ptr ds:>  
00409868     |.  8BCA                mov ecx,edx  
0040986A     |.  83E1 03             and ecx,3  
0040986D     |.  50                  push eax                                  ;  机器码入栈=KEY  
0040986E     |.  F3:A4               rep movs byte ptr es:\[edi\],byte ptr ds:\[e>  
00409870     |.  53                  push ebx  
00409871     |.  8BCB                mov ecx,ebx  
00409873     |.  E8 B8FEFFFF         call TaxExper.00409730                    ;  Des\_SetKey(const char Key\[8\])  
00409878     |.  83FD 08             cmp ebp,8                                 ;  机器码长度是否小于等于 8  
0040987B     |.  5F                  pop edi  
0040987C     |.  5E                  pop esi  
0040987D     |.  7E 22               jle short TaxExper.004098A1               ;  Is3DES = len>8 ? (SetSubKey(&SubKey\[1\], &deskey\[8\]), true) : false;  
0040987F     |.  8D83 09070000       lea eax,dword ptr ds:\[ebx+709\]  
00409885     |.  50                  push eax                                  ;  EAX=G3RZ3C  
00409886     |.  8D8B 00030000       lea ecx,dword ptr ds:\[ebx+300\]  
0040988C     |.  51                  push ecx  
0040988D     |.  8BCB                mov ecx,ebx  
0040988F     |.  E8 9CFEFFFF         call TaxExper.00409730  
00409894     |.  B0 01               mov al,1  
00409896     |.  5D                  pop ebp  
00409897     |.  8883 00060000       mov byte ptr ds:\[ebx+600\],al  
0040989D     |.  5B                  pop ebx  
  
\==========================================================================================================  
00409A40     /$  53                  push ebx  
00409A41     |.  55                  push ebp  
00409A42     |.  56                  push esi  
00409A43     |.  57                  push edi  
00409A44     |.  8B7C24 14           mov edi,dword ptr ss:\[esp+14\]  
00409A48     |.  85FF                test edi,edi  
00409A4A     |.  8BF1                mov esi,ecx  
00409A4C     |.  0F84 C3000000       je TaxExper.00409B15                      ;  机器码是否等于 0  
00409A52     |.  8B6C24 18           mov ebp,dword ptr ss:\[esp+18\]  
00409A56     |.  85ED                test ebp,ebp                              ;  REGSTR\[\]是否为空  
00409A58     |.  0F84 B7000000       je TaxExper.00409B15  
00409A5E     |.  8B4424 20           mov eax,dword ptr ss:\[esp+20\]  
00409A62     |.  85C0                test eax,eax  
00409A64     |.  0F84 AB000000       je TaxExper.00409B15  
00409A6A     |.  8B5C24 1C           mov ebx,dword ptr ss:\[esp+1C\]  
00409A6E     |.  83C3 07             add ebx,7  
00409A71     |.  83E3 F8             and ebx,FFFFFFF8                          ;  EBX=0X10  
00409A74     |.  0F84 9B000000       je TaxExper.00409B15  
00409A7A     |.  8B4C24 24           mov ecx,dword ptr ss:\[esp+24\]  
00409A7E     |.  51                  push ecx                                  ;  机器码长度  
00409A7F     |.  50                  push eax                                  ;  机器码入栈  
00409A80     |.  8BCE                mov ecx,esi  
00409A82     |.  E8 A9FDFFFF         call TaxExper.00409830                    ;  算法进入  
00409A87     |.  8A86 00060000       mov al,byte ptr ds:\[esi+600\]  
00409A8D     |.  C1FB 03             sar ebx,3                                 ;  EBX SHR 3  
00409A90     |.  84C0                test al,al  
00409A92     |.  75 25               jnz short TaxExper.00409AB9               ;   !Is3DES  
00409A94     |.  85DB                test ebx,ebx  
00409A96     |.  7E 74               jle short TaxExper.00409B0C  
00409A98     |>  8B5424 28           /mov edx,dword ptr ss:\[esp+28\]            ;  // 1 次 DES  
00409A9C     |.  52                  |push edx  
00409A9D     |.  56                  |push esi  
00409A9E     |.  55                  |push ebp  
00409A9F     |.  57                  |push edi  
00409AA0     |.  8BCE                |mov ecx,esi  
00409AA2     |.  E8 59FEFFFF         |call TaxExper.00409900                   ;  DES(Out, In, &SubKey\[0\], Type);  
00409AA7     |.  83C7 08             |add edi,8  
00409AAA     |.  83C5 08             |add ebp,8  
00409AAD     |.  4B                  |dec ebx  
00409AAE     |.^ 75 E8               \\jnz short TaxExper.00409A98  
00409AB0     |.  5F                  pop edi  
00409AB1     |.  5E                  pop esi  
00409AB2     |.  5D                  pop ebp  
00409AB3     |.  B0 01               mov al,1  
00409AB5     |.  5B                  pop ebx  
00409AB6     |.  C2 1800             retn 18  
  
\==================================================================================================  
注册成功写入注册表。  
00409020     /$  6A FF               push -1  
00409022     |.  68 5C274B00         push TaxExper.004B275C                    ;  SE 句柄安装  
00409027     |.  64:A1 00000000      mov eax,dword ptr fs:\[0\]  
0040902D     |.  50                  push eax  
0040902E     |.  64:8925 00000000    mov dword ptr fs:\[0\],esp  
00409035     |.  81EC 80000000       sub esp,80  
0040903B     |.  53                  push ebx  
0040903C     |.  55                  push ebp  
0040903D     |.  56                  push esi  
0040903E     |.  8BF1                mov esi,ecx  
00409040     |.  57                  push edi  
00409041     |.  897424 1C           mov dword ptr ss:\[esp+1C\],esi  
00409045     |.  E8 F6A50200         call TaxExper.00433640  
0040904A     |.  33DB                xor ebx,ebx  
0040904C     |.  8D8E B8030000       lea ecx,dword ptr ds:\[esi+3B8\]  
00409052     |.  899C24 98000000     mov dword ptr ss:\[esp+98\],ebx  
00409059     |.  C706 40BF4B00       mov dword ptr ds:\[esi\],TaxExper.004BBF40  
0040905F     |.  C786 D0000000 3CBF4>mov dword ptr ds:\[esi+D0\],TaxExper.004BBF>  
00409069     |.  E8 72780300         call TaxExper.004408E0  
0040906E     |.  8D8E 60040000       lea ecx,dword ptr ds:\[esi+460\]  
00409074     |.  C68424 98000000 01  mov byte ptr ss:\[esp+98\],1  
0040907C     |.  E8 7F5F0300         call TaxExper.0043F000  
00409081     |.  8D8E 38060000       lea ecx,dword ptr ds:\[esi+638\]  
00409087     |.  C68424 98000000 02  mov byte ptr ss:\[esp+98\],2  
0040908F     |.  E8 5F370800         call TaxExper.0048C7F3  
00409094     |.  8D8E 40060000       lea ecx,dword ptr ds:\[esi+640\]  
0040909A     |.  C68424 98000000 03  mov byte ptr ss:\[esp+98\],3  
004090A2     |.  E8 4C370800         call TaxExper.0048C7F3  
004090A7     |.  8D8E 48060000       lea ecx,dword ptr ds:\[esi+648\]  
004090AD     |.  C68424 98000000 04  mov byte ptr ss:\[esp+98\],4  
004090B5     |.  E8 B6840300         call TaxExper.00441570  
004090BA     |.  8D8E 64070000       lea ecx,dword ptr ds:\[esi+764\]  
004090C0     |.  C68424 98000000 05  mov byte ptr ss:\[esp+98\],5  
004090C8     |.  E8 83FD0300         call TaxExper.00448E50  
004090CD     |.  8D8E 60080000       lea ecx,dword ptr ds:\[esi+860\]  
004090D3     |.  C68424 98000000 06  mov byte ptr ss:\[esp+98\],6  
004090DB     |.  E8 F0F30300         call TaxExper.004484D0  
004090E0     |.  8D8E 54090000       lea ecx,dword ptr ds:\[esi+954\]  
004090E6     |.  C68424 98000000 07  mov byte ptr ss:\[esp+98\],7  
004090EE     |.  E8 DDF30300         call TaxExper.004484D0  
004090F3     |.  8D8E 480A0000       lea ecx,dword ptr ds:\[esi+A48\]  
004090F9     |.  C68424 98000000 08  mov byte ptr ss:\[esp+98\],8  
00409101     |.  E8 CAF30300         call TaxExper.004484D0  
00409106     |.  8D8E 3C0B0000       lea ecx,dword ptr ds:\[esi+B3C\]  
0040910C     |.  C68424 98000000 09  mov byte ptr ss:\[esp+98\],9  
00409114     |.  E8 B7F30300         call TaxExper.004484D0  
00409119     |.  8D8E 380C0000       lea ecx,dword ptr ds:\[esi+C38\]  
0040911F     |.  C68424 98000000 0A  mov byte ptr ss:\[esp+98\],0A  
00409127     |.  E8 247A0200         call TaxExper.00430B50  
0040912C     |.  899E C40D0000       mov dword ptr ds:\[esi+DC4\],ebx  
00409132     |.  C786 C00D0000 AC264>mov dword ptr ds:\[esi+DC0\],TaxExper.004C2>  
0040913C     |.  8D8E C80D0000       lea ecx,dword ptr ds:\[esi+DC8\]  
00409142     |.  C68424 98000000 0C  mov byte ptr ss:\[esp+98\],0C  
0040914A     |.  E8 813C0200         call TaxExper.0042CDD0  
0040914F     |.  C68424 98000000 0D  mov byte ptr ss:\[esp+98\],0D  
00409157     |.  E8 D3B20800         call TaxExper.0049442F  
0040915C     |.  8B10                mov edx,dword ptr ds:\[eax\]  
0040915E     |.  8BC8                mov ecx,eax  
00409160     |.  FF52 0C             call dword ptr ds:\[edx+C\]  
00409163     |.  83C0 10             add eax,10  
00409166     |.  8986 300E0000       mov dword ptr ds:\[esi+E30\],eax  
0040916C     |.  C68424 98000000 0E  mov byte ptr ss:\[esp+98\],0E  
00409174     |.  E8 B6B20800         call TaxExper.0049442F  
00409179     |.  8B10                mov edx,dword ptr ds:\[eax\]  
0040917B     |.  8BC8                mov ecx,eax  
0040917D     |.  FF52 0C             call dword ptr ds:\[edx+C\]  
00409180     |.  83C0 10             add eax,10  
00409183     |.  8986 340E0000       mov dword ptr ds:\[esi+E34\],eax  
00409189     |.  C68424 98000000 0F  mov byte ptr ss:\[esp+98\],0F  
00409191     |.  8DBE 3C0E0000       lea edi,dword ptr ds:\[esi+E3C\]  
00409197     |.  68 BEB04B00         push TaxExper.004BB0BE  
0040919C     |.  8BCF                mov ecx,edi  
0040919E     |.  E8 7D91FFFF         call TaxExper.00402320  
004091A3     |.  8DAE 400E0000       lea ebp,dword ptr ds:\[esi+E40\]  
004091A9     |.  68 BEB04B00         push TaxExper.004BB0BE  
004091AE     |.  8BCD                mov ecx,ebp  
004091B0     |.  C68424 9C000000 10  mov byte ptr ss:\[esp+9C\],10  
004091B8     |.  E8 6391FFFF         call TaxExper.00402320  
004091BD     |.  8D8E 480E0000       lea ecx,dword ptr ds:\[esi+E48\]  
004091C3     |.  68 BEB04B00         push TaxExper.004BB0BE  
004091C8     |.  C68424 9C000000 11  mov byte ptr ss:\[esp+9C\],11  
004091D0     |.  899E 440E0000       mov dword ptr ds:\[esi+E44\],ebx  
004091D6     |.  E8 4591FFFF         call TaxExper.00402320  
004091DB     |.  68 BEB04B00         push TaxExper.004BB0BE  
004091E0     |.  8D8E 500E0000       lea ecx,dword ptr ds:\[esi+E50\]  
004091E6     |.  C68424 9C000000 12  mov byte ptr ss:\[esp+9C\],12  
004091EE     |.  899E 4C0E0000       mov dword ptr ds:\[esi+E4C\],ebx  
004091F4     |.  E8 2791FFFF         call TaxExper.00402320  
004091F9     |.  68 01000080         push 80000001  
004091FE     |.  8D4C24 24           lea ecx,dword ptr ss:\[esp+24\]  
00409202     |.  C68424 9C000000 13  mov byte ptr ss:\[esp+9C\],13  
0040920A     |.  E8 E1640200         call TaxExper.0042F6F0  
0040920F     |.  53                  push ebx  
00409210     |.  68 8CC44B00         push TaxExper.004BC48C                    ;  ASCII "RegID"  
00409215     |.  68 70B34B00         push TaxExper.004BB370                    ;  ASCII "Settings"  
0040921A     |.  8D4424 1C           lea eax,dword ptr ss:\[esp+1C\]  
0040921E     |.  50                  push eax  
0040921F     |.  8D4C24 30           lea ecx,dword ptr ss:\[esp+30\]  
00409223     |.  C68424 A8000000 14  mov byte ptr ss:\[esp+A8\],14  
0040922B     |.  E8 40650200         call TaxExper.0042F770  
00409230     |.  50                  push eax  
00409231     |.  8BCF                mov ecx,edi  
00409233     |.  C68424 9C000000 15  mov byte ptr ss:\[esp+9C\],15  
0040923B     |.  E8 008CFFFF         call TaxExper.00401E40  
00409240     |.  8B4424 10           mov eax,dword ptr ss:\[esp+10\]  
00409244     |.  83C0 F0             add eax,-10  
00409247     |.  C68424 98000000 14  mov byte ptr ss:\[esp+98\],14  
0040924F     |.  8D48 0C             lea ecx,dword ptr ds:\[eax+C\]  
00409252     |.  83CA FF             or edx,FFFFFFFF  
00409255     |.  F0:0FC111           lock xadd dword ptr ds:\[ecx\],edx  
00409259     |.  4A                  dec edx  
0040925A     |.  85D2                test edx,edx  
0040925C     |.  7F 08               jg short TaxExper.00409266  
0040925E     |.  8B08                mov ecx,dword ptr ds:\[eax\]  
00409260     |.  8B11                mov edx,dword ptr ds:\[ecx\]  
00409262     |.  50                  push eax  
00409263     |.  FF52 04             call dword ptr ds:\[edx+4\]  
00409266     |>  8B07                mov eax,dword ptr ds:\[edi\]  
00409268     |.  3958 F4             cmp dword ptr ds:\[eax-C\],ebx  
0040926B     |.  0F85 C0000000       jnz TaxExper.00409331  
00409271     |.  8D4C24 30           lea ecx,dword ptr ss:\[esp+30\]  
00409275     |.  51                  push ecx  
00409276     |.  FF15 D8A74B00       call dword ptr ds:\[<&ole32.CoCreateGuid>\] ;  ole32.CoCreateGuid  
0040927C     |.  6A 27               push 27  
0040927E     |.  8D5424 44           lea edx,dword ptr ss:\[esp+44\]  
00409282     |.  52                  push edx  
00409283     |.  8D4424 38           lea eax,dword ptr ss:\[esp+38\]  
00409287     |.  50                  push eax  
00409288     |.  FF15 DCA74B00       call dword ptr ds:\[<&ole32.StringFromGUID>;  ole32.StringFromGUID2  
0040928E     |.  8D4C24 40           lea ecx,dword ptr ss:\[esp+40\]  
00409292     |.  51                  push ecx  
00409293     |.  8D4C24 14           lea ecx,dword ptr ss:\[esp+14\]  
00409297     |.  E8 24E6FFFF         call TaxExper.004078C0  
0040929C     |.  6A 7B               push 7B  
0040929E     |.  8D4C24 14           lea ecx,dword ptr ss:\[esp+14\]  
004092A2     |.  C68424 9C000000 16  mov byte ptr ss:\[esp+9C\],16  
004092AA     |.  E8 71E1FFFF         call TaxExper.00407420  
004092AF     |.  6A 10               push 10  
004092B1     |.  8D5424 1C           lea edx,dword ptr ss:\[esp+1C\]  
004092B5     |.  52                  push edx  
004092B6     |.  8D4C24 18           lea ecx,dword ptr ss:\[esp+18\]  
004092BA     |.  E8 41E0FFFF         call TaxExper.00407300  
004092BF     |.  50                  push eax  
004092C0     |.  8BCF                mov ecx,edi  
004092C2     |.  C68424 9C000000 17  mov byte ptr ss:\[esp+9C\],17  
004092CA     |.  E8 718BFFFF         call TaxExper.00401E40  
004092CF     |.  8B4424 18           mov eax,dword ptr ss:\[esp+18\]  
004092D3     |.  83C0 F0             add eax,-10  
004092D6     |.  C68424 98000000 16  mov byte ptr ss:\[esp+98\],16  
004092DE     |.  8D48 0C             lea ecx,dword ptr ds:\[eax+C\]  
004092E1     |.  83CA FF             or edx,FFFFFFFF  
004092E4     |.  F0:0FC111           lock xadd dword ptr ds:\[ecx\],edx  
004092E8     |.  4A                  dec edx  
004092E9     |.  85D2                test edx,edx  
004092EB     |.  7F 08               jg short TaxExper.004092F5  
004092ED     |.  8B08                mov ecx,dword ptr ds:\[eax\]  
004092EF     |.  8B11                mov edx,dword ptr ds:\[ecx\]  
004092F1     |.  50                  push eax  
004092F2     |.  FF52 04             call dword ptr ds:\[edx+4\]  
004092F5     |>  8B07                mov eax,dword ptr ds:\[edi\]  
004092F7     |.  50                  push eax  
004092F8     |.  68 8CC44B00         push TaxExper.004BC48C                    ;  ASCII "RegID"  
004092FD     |.  68 70B34B00         push TaxExper.004BB370                    ;  ASCII "Settings"  
00409302     |.  8D4C24 2C           lea ecx,dword ptr ss:\[esp+2C\]  
00409306     |.  E8 B5610200         call TaxExper.0042F4C0  
0040930B     |.  8B4424 10           mov eax,dword ptr ss:\[esp+10\]  
0040930F     |.  83C0 F0             add eax,-10  
00409312     |.  C68424 98000000 14  mov byte ptr ss:\[esp+98\],14  
0040931A     |.  8D48 0C             lea ecx,dword ptr ds:\[eax+C\]  
0040931D     |.  83CA FF             or edx,FFFFFFFF  
00409320     |.  F0:0FC111           lock xadd dword ptr ds:\[ecx\],edx  
00409324     |.  4A                  dec edx  
00409325     |.  85D2                test edx,edx  
00409327     |.  7F 08               jg short TaxExper.00409331  
00409329     |.  8B08                mov ecx,dword ptr ds:\[eax\]  
0040932B     |.  8B11                mov edx,dword ptr ds:\[ecx\]  
0040932D     |.  50                  push eax  
0040932E     |.  FF52 04             call dword ptr ds:\[edx+4\]  
00409331     |>  8D4424 18           lea eax,dword ptr ss:\[esp+18\]  
00409335     |.  50                  push eax                                  ; /Arg1  
00409336     |.  8BCE                mov ecx,esi                               ; |  
00409338     |.  E8 63F8FFFF         call TaxExper.00408BA0                    ; \\TaxExper.00408BA0  
0040933D     |.  50                  push eax  
0040933E     |.  8BCD                mov ecx,ebp  
00409340     |.  C68424 9C000000 18  mov byte ptr ss:\[esp+9C\],18  
00409348     |.  E8 F38AFFFF         call TaxExper.00401E40  
0040934D     |.  8B4424 18           mov eax,dword ptr ss:\[esp+18\]  
00409351     |.  83C0 F0             add eax,-10  
00409354     |.  C68424 98000000 14  mov byte ptr ss:\[esp+98\],14  
0040935C     |.  8D48 0C             lea ecx,dword ptr ds:\[eax+C\]  
0040935F     |.  83CA FF             or edx,FFFFFFFF  
00409362     |.  F0:0FC111           lock xadd dword ptr ds:\[ecx\],edx  
00409366     |.  4A                  dec edx  
00409367     |.  85D2                test edx,edx  
00409369     |.  7F 08               jg short TaxExper.00409373  
0040936B     |.  8B08                mov ecx,dword ptr ds:\[eax\]  
0040936D     |.  8B11                mov edx,dword ptr ds:\[ecx\]  
0040936F     |.  50                  push eax  
00409370     |.  FF52 04             call dword ptr ds:\[edx+4\]  
00409373     |>  8B45 00             mov eax,dword ptr ss:\[ebp\]  
00409376     |.  3958 F4             cmp dword ptr ds:\[eax-C\],ebx  
00409379     |.  74 57               je short TaxExper.004093D2  
0040937B     |.  C786 440E0000 72000>mov dword ptr ds:\[esi+E44\],72  
00409385     |.  8378 F4 10          cmp dword ptr ds:\[eax-C\],10  
00409389     |.  7E 44               jle short TaxExper.004093CF  
0040938B     |.  6A 10               push 10  
0040938D     |.  8D4424 1C           lea eax,dword ptr ss:\[esp+1C\]  
00409391     |.  50                  push eax  
00409392     |.  8BCD                mov ecx,ebp  
00409394     |.  E8 67DFFFFF         call TaxExper.00407300  
00409399     |.  50                  push eax  
0040939A     |.  8BCD                mov ecx,ebp  
0040939C     |.  C68424 9C000000 19  mov byte ptr ss:\[esp+9C\],19  
004093A4     |.  E8 978AFFFF         call TaxExper.00401E40  
004093A9     |.  8B4424 18           mov eax,dword ptr ss:\[esp+18\]  
004093AD     |.  83C0 F0             add eax,-10  
004093B0     |.  C68424 98000000 14  mov byte ptr ss:\[esp+98\],14  
004093B8     |.  8D48 0C             lea ecx,dword ptr ds:\[eax+C\]  
004093BB     |.  83CA FF             or edx,FFFFFFFF  
004093BE     |.  F0:0FC111           lock xadd dword ptr ds:\[ecx\],edx  
004093C2     |.  4A                  dec edx  
004093C3     |.  85D2                test edx,edx  
004093C5     |.  7F 08               jg short TaxExper.004093CF  
004093C7     |.  8B08                mov ecx,dword ptr ds:\[eax\]  
004093C9     |.  8B11                mov edx,dword ptr ds:\[ecx\]  
004093CB     |.  50                  push eax  
004093CC     |.  FF52 04             call dword ptr ds:\[edx+4\]  
004093CF     |>  55                  push ebp  
004093D0     |.  EB 0B               jmp short TaxExper.004093DD  
004093D2     |>  C786 440E0000 73000>mov dword ptr ds:\[esi+E44\],73  
004093DC     |.  57                  push edi  
004093DD     |>  8D8E 480E0000       lea ecx,dword ptr ds:\[esi+E48\]  
004093E3     |.  E8 588AFFFF         call TaxExper.00401E40  
004093E8     |.  8BCE                mov ecx,esi  
004093EA     |.  E8 A1DCFFFF         call TaxExper.00407090  
004093EF     |.  53                  push ebx                                  ; /Arg3 = 00000000  
004093F0     |.  68 74B64B00         push TaxExper.004BB674                    ; |Arg2 = 004BB674 ASCII "UserInputFileIndex"  
004093F5     |.  68 70B34B00         push TaxExper.004BB370                    ; |Arg1 = 004BB370 ASCII "Settings"  
004093FA     |.  8D4C24 2C           lea ecx,dword ptr ss:\[esp+2C\]             ; |  
004093FE     |.  8986 4C0E0000       mov dword ptr ds:\[esi+E4C\],eax            ; |  
00409404     |.  E8 67610200         call TaxExper.0042F570                    ; \\TaxExper.0042F570  
00409409     |.  3BC3                cmp eax,ebx  
0040940B     |.  75 18               jnz short TaxExper.00409425  
0040940D     |.  68 30750000         push 7530                                 ; /Arg3 = 00007530  
00409412     |.  68 74B64B00         push TaxExper.004BB674                    ; |Arg2 = 004BB674 ASCII "UserInputFileIndex"  
00409417     |.  68 70B34B00         push TaxExper.004BB370                    ; |Arg1 = 004BB370 ASCII "Settings"  
0040941C     |.  8D4C24 2C           lea ecx,dword ptr ss:\[esp+2C\]             ; |  
00409420     |.  E8 FB5F0200         call TaxExper.0042F420                    ; \\TaxExper.0042F420  
00409425     |>  B8 BEB04B00         mov eax,TaxExper.004BB0BE  
0040942A     |.  8D50 01             lea edx,dword ptr ds:\[eax+1\]  
0040942D     |.  8D49 00             lea ecx,dword ptr ds:\[ecx\]  
00409430     |>  8A08                /mov cl,byte ptr ds:\[eax\]  
00409432     |.  40                  |inc eax  
00409433     |.  3ACB                |cmp cl,bl  
00409435     |.^ 75 F9               \\jnz short TaxExper.00409430  
00409437     |.  2BC2                sub eax,edx  
00409439     |.  50                  push eax  
0040943A     |.  68 BEB04B00         push TaxExper.004BB0BE  
0040943F     |.  8D8E 300E0000       lea ecx,dword ptr ds:\[esi+E30\]  
00409445     |.  E8 F685FFFF         call TaxExper.00401A40  
0040944A     |.  B8 BEB04B00         mov eax,TaxExper.004BB0BE  
0040944F     |.  8D78 01             lea edi,dword ptr ds:\[eax+1\]  
00409452     |>  8A08                /mov cl,byte ptr ds:\[eax\]  
00409454     |.  40                  |inc eax  
00409455     |.  3ACB                |cmp cl,bl  
00409457     |.^ 75 F9               \\jnz short TaxExper.00409452  
00409459     |.  2BC7                sub eax,edi  
0040945B     |.  50                  push eax  
0040945C     |.  68 BEB04B00         push TaxExper.004BB0BE  
00409461     |.  8D8E 340E0000       lea ecx,dword ptr ds:\[esi+E34\]  
00409467     |.  E8 D485FFFF         call TaxExper.00401A40  
0040946C     |.  6A 01               push 1                                    ; /Arg3 = 00000001  
0040946E     |.  68 34BF4B00         push TaxExper.004BBF34                    ; |Arg2 = 004BBF34 ASCII "bXPMode"  
00409473     |.  68 70B34B00         push TaxExper.004BB370                    ; |Arg1 = 004BB370 ASCII "Settings"  
00409478     |.  8D4C24 2C           lea ecx,dword ptr ss:\[esp+2C\]             ; |  
0040947C     |.  899E 380E0000       mov dword ptr ds:\[esi+E38\],ebx            ; |  
00409482     |.  899E 340C0000       mov dword ptr ds:\[esi+C34\],ebx            ; |  
00409488     |.  889E B4030000       mov byte ptr ds:\[esi+3B4\],bl              ; |  
0040948E     |.  899E 300C0000       mov dword ptr ds:\[esi+C30\],ebx            ; |  
00409494     |.  E8 D7600200         call TaxExper.0042F570                    ; \\TaxExper.0042F570  
00409499     |.  8BF8                mov edi,eax  
0040949B     |.  E8 D0B80200         call TaxExper.00434D70  
004094A0     |.  53                  push ebx  
004094A1     |.  68 7CB34B00         push TaxExper.004BB37C                    ;  ASCII "pwd"  
004094A6     |.  89B8 44010000       mov dword ptr ds:\[eax+144\],edi  
004094AC     |.  68 70B34B00         push TaxExper.004BB370                    ;  ASCII "Settings"  
004094B1     |.  8D4424 20           lea eax,dword ptr ss:\[esp+20\]  
004094B5     |.  50                  push eax  
004094B6     |.  8D4C24 30           lea ecx,dword ptr ss:\[esp+30\]  
004094BA     |.  E8 B1620200         call TaxExper.0042F770  
004094BF     |.  8B4C24 14           mov ecx,dword ptr ss:\[esp+14\]  
004094C3     |.  3959 F4             cmp dword ptr ds:\[ecx-C\],ebx  
004094C6     |.  C68424 98000000 1A  mov byte ptr ss:\[esp+98\],1A  
004094CE     |.  75 18               jnz short TaxExper.004094E8  
004094D0     |.  68 88C44B00         push TaxExper.004BC488                    ;  ASCII "123"  
004094D5     |.  68 7CB34B00         push TaxExper.004BB37C                    ;  ASCII "pwd"  
004094DA     |.  68 70B34B00         push TaxExper.004BB370                    ;  ASCII "Settings"  
004094DF     |.  8D4C24 2C           lea ecx,dword ptr ss:\[esp+2C\]  
004094E3     |.  E8 D85F0200         call TaxExper.0042F4C0  
004094E8     |>  E8 83B80200         call TaxExper.00434D70  
004094ED     |.  C780 48010000 01000>mov dword ptr ds:\[eax+148\],1  
004094F7     |.  8B4424 14           mov eax,dword ptr ss:\[esp+14\]  
004094FB     |.  83C0 F0             add eax,-10  
004094FE     |.  C68424 98000000 14  mov byte ptr ss:\[esp+98\],14  
00409506     |.  8D50 0C             lea edx,dword ptr ds:\[eax+C\]  
00409509     |.  83C9 FF             or ecx,FFFFFFFF  
0040950C     |.  F0:0FC10A           lock xadd dword ptr ds:\[edx\],ecx  
00409510     |.  49                  dec ecx  
00409511     |.  85C9                test ecx,ecx  
00409513     |.  7F 08               jg short TaxExper.0040951D  
00409515     |.  8B08                mov ecx,dword ptr ds:\[eax\]  
00409517     |.  8B11                mov edx,dword ptr ds:\[ecx\]  
00409519     |.  50                  push eax  
0040951A     |.  FF52 04             call dword ptr ds:\[edx+4\]  
0040951D     |>  8D4C24 20           lea ecx,dword ptr ss:\[esp+20\]  
00409521     |.  C68424 98000000 13  mov byte ptr ss:\[esp+98\],13  
00409529     |.  E8 C25E0200         call TaxExper.0042F3F0  
0040952E     |.  8B8C24 90000000     mov ecx,dword ptr ss:\[esp+90\]  
00409535     |.  5F                  pop edi  
00409536     |.  8BC6                mov eax,esi  
00409538     |.  5E                  pop esi  
00409539     |.  5D                  pop ebp  
0040953A     |.  5B                  pop ebx  
0040953B     |.  64:890D 00000000    mov dword ptr fs:\[0\],ecx  
00409542     |.  81C4 8C000000       add esp,8C  
00409548     \\.  C3                  retn  
  
\=================================================================================================  
算法总结：利用注册文件的字符进行 3DES 加密，你如果不懂 3DES 就无法进行逆推了。最好把 3DES 加密算法搞明白  
再做注册机。看一下注册机 C 代码就一目了然了。  
LRESULT CALLBACK WndProc(HWND hWnd, UINT message, WPARAM wParam, LPARAM lParam)  
{  
  int wmId, wmEvent,i,j;  
  PAINTSTRUCT ps;  
  HDC hdc;  
  HANDLE hFile;  
  DWORD dwLen;  
  char Buffer\[889\]="0";  
    
  char key\[\]="VNC220A2G3RZ3C",buf\[255\];  
  char str\[\]="TaxExpert-V2003";  
  
    
    
  HANDLE hFileMapping;  
  
  switch (message)   
  {  
  case WM\_COMMAND:  
    wmId    = LOWORD(wParam);   
    wmEvent = HIWORD(wParam);   
    // 分析菜单选择:  
    switch (wmId)  
    {  
    case IDM\_about :  
      DialogBox(hInst, (LPCTSTR)IDD\_ABOUTBOX, hWnd, (DLGPROC)About);  
      break;  
    case IDM\_EXIT:  
      DestroyWindow(hWnd);  
      break;  
    case ID\_NEW:  
      memset(buf, 0, sizeof(buf));  
    
      strcpy(buf, str);  
  
      for(i=0;i<888;i++)  
      {  
        Buffer\[i\]=0x41+i%58;  
      }  
      for(i=213,j=0;i<229;i++,j++)  
      {  
        Buffer\[i\]=0x30+j%0xf;  
      }  
  
      Des\_Go(buf, buf, sizeof(str), key, sizeof(key), ENCRYPT);//加密  
      for(i=213,j=0;i<229;i++,j++)  
      {  
        Buffer\[i\]=buf\[j\];  
      }  
  
  
  
  
      hFile = CreateFile(TEXT("TaxExpert.nfo"),  
                       GENERIC\_WRITE | GENERIC\_READ,  
                       FILE\_SHARE\_READ,   
                       NULL,  
                       CREATE\_ALWAYS,   
                       FILE\_FLAG\_SEQUENTIAL\_SCAN,   
                       NULL);  
  
      if(hFile  != INVALID\_HANDLE\_VALUE) //打开 hFile  成功  
      {  
      //ReadFile(hFile, Buffer, BufSize, &nBytesRead, NULL); 从 hFile 里读取数据到 Buffer 里  
             
      WriteFile(hFile, Buffer, 888, &dwLen, NULL);  
      //把 Buffer 里面的 BytesToWrite 字节写入 hPipe  
  
      }  
        
      CloseHandle(hFile);  
          
             break;  
  
3DES 加密代码利用了王俊川的 C 代码。再次感谢！  
/////////////////////////WjcDes.C/////////////////////////////////////////////////  
/\*  
    Provided by 王俊川, Northeastern University (www.neu.edu.cn)  
    Email: blackdrn@sohu.com  
  This product is free for use.  
\*/  
//////////////////////////////////////////////////////////////////////////  
  
#include <memory.h>  
#include "WjcDes.h"  
#include "stdafx.h"  
//////////////////////////////////////////////////////////////////////////  
  
// initial permutation IP  
const static char IP\_Table\[64\] = {  
  58, 50, 42, 34, 26, 18, 10, 2, 60, 52, 44, 36, 28, 20, 12, 4,  
  62, 54, 46, 38, 30, 22, 14, 6, 64, 56, 48, 40, 32, 24, 16, 8,  
  57, 49, 41, 33, 25, 17,  9, 1, 59, 51, 43, 35, 27, 19, 11, 3,  
    61, 53, 45, 37, 29, 21, 13, 5, 63, 55, 47, 39, 31, 23, 15, 7  
};  
// final permutation IP^-1   
const static char IPR\_Table\[64\] = {  
  40, 8, 48, 16, 56, 24, 64, 32, 39, 7, 47, 15, 55, 23, 63, 31,  
  38, 6, 46, 14, 54, 22, 62, 30, 37, 5, 45, 13, 53, 21, 61, 29,  
    36, 4, 44, 12, 52, 20, 60, 28, 35, 3, 43, 11, 51, 19, 59, 27,  
  34, 2, 42, 10, 50, 18, 58, 26, 33, 1, 41,  9, 49, 17, 57, 25  
};  
// expansion operation matrix  
static const char E\_Table\[48\] = {  
  32,  1,  2,  3,  4,  5,  4,  5,  6,  7,  8,  9,  
   8,  9, 10, 11, 12, 13, 12, 13, 14, 15, 16, 17,  
  16, 17, 18, 19, 20, 21, 20, 21, 22, 23, 24, 25,  
  24, 25, 26, 27, 28, 29, 28, 29, 30, 31, 32,  1  
};  
// 32-bit permutation function P used on the output of the S-boxes   
const static char P\_Table\[32\] = {  
  16, 7, 20, 21, 29, 12, 28, 17, 1,  15, 23, 26, 5,  18, 31, 10,  
  2,  8, 24, 14, 32, 27, 3,  9,  19, 13, 30, 6,  22, 11, 4,  25  
};  
// permuted choice table (key)   
const static char PC1\_Table\[56\] = {  
  57, 49, 41, 33, 25, 17,  9,  1, 58, 50, 42, 34, 26, 18,  
  10,  2, 59, 51, 43, 35, 27, 19, 11,  3, 60, 52, 44, 36,  
  63, 55, 47, 39, 31, 23, 15,  7, 62, 54, 46, 38, 30, 22,  
  14,  6, 61, 53, 45, 37, 29, 21, 13,  5, 28, 20, 12,  4  
};  
// permuted choice key (table)   
const static char PC2\_Table\[48\] = {  
  14, 17, 11, 24,  1,  5,  3, 28, 15,  6, 21, 10,  
  23, 19, 12,  4, 26,  8, 16,  7, 27, 20, 13,  2,  
  41, 52, 31, 37, 47, 55, 30, 40, 51, 45, 33, 48,  
  44, 49, 39, 56, 34, 53, 46, 42, 50, 36, 29, 32  
};  
// number left rotations of pc1   
const static char LOOP\_Table\[16\] = {  
  1,1,2,2,2,2,2,2,1,2,2,2,2,2,2,1  
};  
// The (in)famous S-boxes   
const static char S\_Box\[8\]\[4\]\[16\] = {  
  // S1   
  14,   4,  13,   1,  2, 15, 11,  8,  3, 10,  6, 12,  5,  9,  0,  7,  
   0, 15,  7,  4, 14,  2, 13,  1, 10,  6, 12, 11,  9,  5,  3,  8,  
   4,  1, 14,  8, 13,  6,  2, 11, 15, 12,  9,  7,  3, 10,  5,  0,  
    15, 12,  8,  2,  4,  9,  1,  7,  5, 11,  3, 14, 10,  0,  6, 13,  
  // S2   
    15,  1,  8, 14,  6, 11,  3,  4,  9,  7,  2, 13, 12,  0,  5, 10,  
   3, 13,  4,  7, 15,  2,  8, 14, 12,  0,  1, 10,  6,  9, 11,  5,  
   0, 14,  7, 11, 10,  4, 13,  1,  5,  8, 12,  6,  9,  3,  2, 15,  
    13,  8, 10,  1,  3, 15,  4,  2, 11,  6,  7, 12,  0,  5, 14,  9,  
  // S3   
    10,  0,  9, 14,  6,  3, 15,  5,  1, 13, 12,  7, 11,  4,  2,  8,  
  13,  7,  0,  9,  3,  4,  6, 10,  2,  8,  5, 14, 12, 11, 15,  1,  
  13,  6,  4,  9,  8, 15,  3,  0, 11,  1,  2, 12,  5, 10, 14,  7,  
     1, 10, 13,  0,  6,  9,  8,  7,  4, 15, 14,  3, 11,  5,  2, 12,  
  // S4   
     7, 13, 14,  3,  0,  6,  9, 10,  1,  2,  8,  5, 11, 12,  4, 15,  
  13,  8, 11,  5,  6, 15,  0,  3,  4,  7,  2, 12,  1, 10, 14,  9,  
  10,  6,  9,  0, 12, 11,  7, 13, 15,  1,  3, 14,  5,  2,  8,  4,  
     3, 15,  0,  6, 10,  1, 13,  8,  9,  4,  5, 11, 12,  7,  2, 14,  
  // S5   
     2, 12,  4,  1,  7, 10, 11,  6,  8,  5,  3, 15, 13,  0, 14,  9,  
  14, 11,  2, 12,  4,  7, 13,  1,  5,  0, 15, 10,  3,  9,  8,  6,  
   4,  2,  1, 11, 10, 13,  7,  8, 15,  9, 12,  5,  6,  3,  0, 14,  
    11,  8, 12,  7,  1, 14,  2, 13,  6, 15,  0,  9, 10,  4,  5,  3,  
  // S6   
    12,  1, 10, 15,  9,  2,  6,  8,  0, 13,  3,  4, 14,  7,  5, 11,  
  10, 15,  4,  2,  7, 12,  9,  5,  6,  1, 13, 14,  0, 11,  3,  8,  
   9, 14, 15,  5,  2,  8, 12,  3,  7,  0,  4, 10,  1, 13, 11,  6,  
     4,  3,  2, 12,  9,  5, 15, 10, 11, 14,  1,  7,  6,  0,  8, 13,  
  // S7   
     4, 11,  2, 14, 15,  0,  8, 13,  3, 12,  9,  7,  5, 10,  6,  1,  
  13,  0, 11,  7,  4,  9,  1, 10, 14,  3,  5, 12,  2, 15,  8,  6,  
   1,  4, 11, 13, 12,  3,  7, 14, 10, 15,  6,  8,  0,  5,  9,  2,  
     6, 11, 13,  8,  1,  4, 10,  7,  9,  5,  0, 15, 14,  2,  3, 12,  
  // S8   
    13,  2,  8,  4,  6, 15, 11,  1, 10,  9,  3, 14,  5,  0, 12,  7,  
   1, 15, 13,  8, 10,  3,  7,  4, 12,  5,  6, 11,  0, 14,  9,  2,  
   7, 11,  4,  1,  9, 12, 14,  2,  0,  6, 10, 13, 15,  3,  5,  8,  
     2,  1, 14,  7,  4, 10,  8, 13, 15, 12,  9,  0,  3,  5,  6, 11  
};  
  
//////////////////////////////////////////////////////////////////////////  
  
typedef bool    (\*PSubKey)\[16\]\[48\];  
  
//////////////////////////////////////////////////////////////////////////  
  
static void DES(char Out\[8\], char In\[8\], const PSubKey pSubKey, bool Type);//标准 DES 加/解密  
static void SetKey(const char\* Key, int len);// 设置密钥  
static void SetSubKey(PSubKey pSubKey, const char Key\[8\]);// 设置子密钥  
static void F\_func(bool In\[32\], const bool Ki\[48\]);// f 函数  
static void S\_func(bool Out\[32\], const bool In\[48\]);// S 盒代替  
static void Transform(bool \*Out, bool \*In, const char \*Table, int len);// 变换  
static void Xor(bool \*InA, const bool \*InB, int len);// 异或  
static void RotateL(bool \*In, int len, int loop);// 循环左移  
static void ByteToBit(bool \*Out, const char \*In, int bits);// 字节组转换成位组  
static void BitToByte(char \*Out, const bool \*In, int bits);// 位组转换成字节组  
  
//////////////////////////////////////////////////////////////////////////  
  
static bool SubKey\[2\]\[16\]\[48\];// 16 圈子密钥  
static bool Is3DES;// 3 次 DES 标志  
static char Tmp\[256\], deskey\[16\];  
  
//////////////////////////////////////////////////////////////////////////  
  
//////////////////////////////////////////////////////////////////////////  
// Code starts from Line 130  
//////////////////////////////////////////////////////////////////////////  
bool Des\_Go(char \*Out, char \*In, long datalen, const char \*Key, int keylen, bool Type)  
{  
    if( !( Out && In && Key && (datalen=(datalen+7)&0xfffffff8) ) )   
    return false;  
  SetKey(Key, keylen);  
  if( !Is3DES ) {   // 1 次 DES  
    for(long i=0,j=datalen>>3; i<j; ++i,Out+=8,In+=8)  
      DES(Out, In, &SubKey\[0\], Type);  
  } else{   // 3 次 DES 加密:加(key0)-解(key1)-加(key0) 解密::解(key0)-加(key1)-解(key0)  
    for(long i=0,j=datalen>>3; i<j; ++i,Out+=8,In+=8) {  
      DES(Out, In,  &SubKey\[0\], Type);  
      DES(Out, Out, &SubKey\[1\], !Type);  
      DES(Out, Out, &SubKey\[0\], Type);  
    }  
  }  
  return true;  
}  
void SetKey(const char\* Key, int len)  
{  
  memset(deskey, 0, 16);  
  memcpy(deskey, Key, len>16?16:len);  
  SetSubKey(&SubKey\[0\], &deskey\[0\]);  
  Is3DES = len>8 ? (SetSubKey(&SubKey\[1\], &deskey\[8\]), true) : false;  
}  
void DES(char Out\[8\], char In\[8\], const PSubKey pSubKey, bool Type)  
{  
    static bool M\[64\], tmp\[32\], \*Li=&M\[0\], \*Ri=&M\[32\];  
    ByteToBit(M, In, 64);  
    Transform(M, M, IP\_Table, 64);  
    if( Type == ENCRYPT ){  
        for(int i=0; i<16; ++i) {  
            memcpy(tmp, Ri, 32);  
            F\_func(Ri, (\*pSubKey)\[i\]);  
            Xor(Ri, Li, 32);  
            memcpy(Li, tmp, 32);  
        }  
    }else{  
        for(int i=15; i>=0; --i) {  
            memcpy(tmp, Li, 32);  
            F\_func(Li, (\*pSubKey)\[i\]);  
            Xor(Li, Ri, 32);  
            memcpy(Ri, tmp, 32);  
        }  
  }  
    Transform(M, M, IPR\_Table, 64);  
    BitToByte(Out, M, 64);  
}  
void SetSubKey(PSubKey pSubKey, const char Key\[8\])  
{  
    static bool K\[64\], \*KL=&K\[0\], \*KR=&K\[28\];  
    ByteToBit(K, Key, 64);  
    Transform(K, K, PC1\_Table, 56);  
    for(int i=0; i<16; ++i) {  
        RotateL(KL, 28, LOOP\_Table\[i\]);  
        RotateL(KR, 28, LOOP\_Table\[i\]);  
        Transform((\*pSubKey)\[i\], K, PC2\_Table, 48);  
    }  
}  
void F\_func(bool In\[32\], const bool Ki\[48\])  
{  
    static bool MR\[48\];  
    Transform(MR, In, E\_Table, 48);  
    Xor(MR, Ki, 48);  
    S\_func(In, MR);  
    Transform(In, In, P\_Table, 32);  
}  
void S\_func(bool Out\[32\], const bool In\[48\])  
{  
    for(char i=0,j,k; i<8; ++i,In+=6,Out+=4) {  
        j = (In\[0\]<<1) + In\[5\];  
        k = (In\[1\]<<3) + (In\[2\]<<2) + (In\[3\]<<1) + In\[4\];  
    ByteToBit(Out, &S\_Box\[i\]\[j\]\[k\], 4);  
    }  
}  
void Transform(bool \*Out, bool \*In, const char \*Table, int len)  
{  
    for(int i=0; i<len; ++i)  
        Tmp\[i\] = In\[ Table\[i\]-1 \];  
    memcpy(Out, Tmp, len);  
}  
void Xor(bool \*InA, const bool \*InB, int len)  
{  
    for(int i=0; i<len; ++i)  
        InA\[i\] ^= InB\[i\];  
}  
void RotateL(bool \*In, int len, int loop)  
{  
    memcpy(Tmp, In, loop);  
    memcpy(In, In+loop, len-loop);  
    memcpy(In+len-loop, Tmp, loop);  
}  
void ByteToBit(bool \*Out, const char \*In, int bits)  
{  
    for(int i=0; i<bits; ++i)  
        Out\[i\] = (In\[i>>3\]>>(i&7)) & 1;  
}  
void BitToByte(char \*Out, const bool \*In, int bits)  
{  
    memset(Out, 0, bits>>3);  
    for(int i=0; i<bits; ++i)  
        Out\[i>>3\] |= In\[i\]<<(i&7);  
}  
//////////////////////////////////////////////////////////////////////////  
// Code ends at Line 231  
//////////////////////////////////////////////////////////////////////////  

 4dd466d0e1e0cc123afbe504da2462ca