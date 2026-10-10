

# 简单打狗文章一二

> 原文：[`www.pediy.com/kssd/pediy10/87831.html`](https://www.pediy.com/kssd/pediy10/87831.html)

简单打狗文章一二  
【应用平台】Win2000  
【作者邮箱】chubing6143@sina.com  
【使用工具】 peid, OllyDbg1.10  
【软件限制】狗  
【破解工具】OllyDbg v1.10  
   
 采用 USB 狗加密这种方式的软件很多,例如 PAWS 等,还有些软件是狗与 Flexlm 加密综合使用的,我打狗棒法未曾学精,但也打过一两个简单狗,见坛子上大家学习打狗棒法热情也很高,就将自己的浅薄经验拿出来与大家分享.高手飘过！  
一、超级简单的打狗  
这样的软件基本上是通过一个函数对狗的有无进行验证，然后一个关键跳转，错误弹出对话框，正确继续执行，下面是某程序的例子：  
0042ADF0   > \\6A FF         PUSH -1  
0042ADF2   .  68 2F465200   PUSH Eb.0052462F                                 ;  SE handler installation  
0042ADF7   .  64:A1 0000000>MOV EAX,DWORD PTR FS:\[0\]  
0042ADFD   .  50            PUSH EAX  
0042ADFE   .  64:8925 00000>MOV DWORD PTR FS:\[0\],ESP  
0042AE05   .  81EC C8000000 SUB ESP,0C8  
0042AE0B   .  55            PUSH EBP  
0042AE0C   .  56            PUSH ESI  
0042AE0D   .  8BE9          MOV EBP,ECX  
0042AE0F   .  6A 00         PUSH 0  
0042AE11   .  896C24 10     MOV DWORD PTR SS:\[ESP+10\],EBP  
0042AE15   .  E8 C2510A00   CALL <JMP.&MFC42.#561>  
0042AE1A   .  8DB5 C4000000 LEA ESI,DWORD PTR SS:\[EBP+C4\]  
0042AE20   .  C78424 D80000>MOV DWORD PTR SS:\[ESP+D8\],0  
0042AE2B   .  8BCE          MOV ECX,ESI  
0042AE2D   .  FF15 48255300 CALL DWORD PTR DS:\[<&customui.??0CBCGWorkspace@@>;  customui.??0CBCGWorkspace@@QAE@XZ  
0042AE33   .  8D8D DC000000 LEA ECX,DWORD PTR SS:\[EBP+DC\]  
0042AE39   .  C68424 D80000>MOV BYTE PTR SS:\[ESP+D8\],1  
0042AE41   .  FF15 24255300 CALL DWORD PTR DS:\[<&customui.??0CBCGKeyboardMan>;  customui.??0CBCGKeyboardManager@@QAE@XZ  
0042AE47   .  8D8D E0000000 LEA ECX,DWORD PTR SS:\[EBP+E0\]  
0042AE4D   .  C68424 D80000>MOV BYTE PTR SS:\[ESP+D8\],2  
0042AE55   .  FF15 A8265300 CALL DWORD PTR DS:\[<&customui.??0CBCGMouseManage>;  customui.??0CBCGMouseManager@@QAE@XZ  
0042AE5B   .  8D8D 38010000 LEA ECX,DWORD PTR SS:\[EBP+138\]  
0042AE61   .  C68424 D80000>MOV BYTE PTR SS:\[ESP+D8\],3  
0042AE69   .  FF15 B0255300 CALL DWORD PTR DS:\[<&customui.??0CBCGContextMenu>;  customui.??0CBCGContextMenuManager@@QAE@XZ  
0042AE6F   .  8D8D 74010000 LEA ECX,DWORD PTR SS:\[EBP+174\]  
0042AE75   .  C68424 D80000>MOV BYTE PTR SS:\[ESP+D8\],4  
0042AE7D   .  E8 54510A00   CALL <JMP.&MFC42.#459>  
0042AE82   .  8D4424 10     LEA EAX,DWORD PTR SS:\[ESP+10\]  
0042AE86   .  C68424 D80000>MOV BYTE PTR SS:\[ESP+D8\],5  
0042AE8E   .  50            PUSH EAX  
0042AE8F   .  C745 00 48705>MOV DWORD PTR SS:\[EBP\],Eb.00537048  
0042AE96   .  C706 10705300 MOV DWORD PTR DS:\[ESI\],Eb.00537010  
0042AE9C   .  E8 BFF3FFFF   CALL Eb.0042A260  
0042AEA1   .  83C4 04       ADD ESP,4  
0042AEA4   .  85C0          TEST EAX,EAX  
0042AEA6   .  75 6A         JNZ SHORT Eb.0042AF12                            ;  关键跳转,前面函数检查加密狗是否存在,此处必须跳转  
0042AEA8   .  E8 33310000   CALL Eb.0042DFE0  
0042AEAD   .  66:85C0       TEST AX,AX  
0042AEB0   .  BE 63000000   MOV ESI,63  
0042AEB5   .  74 1A         JE SHORT Eb.0042AED1  
0042AEB7   .  6A 01         PUSH 1  
0042AEB9   .  C705 789A5500>MOV DWORD PTR DS:\[559A78\],460  
0042AEC3   .  8935 749A5500 MOV DWORD PTR DS:\[559A74\],ESI  
0042AEC9   .  E8 5241FEFF   CALL Eb.0040F020  
0042AECE   .  83C4 04       ADD ESP,4  
0042AED1   >  E8 9AFDFFFF   CALL Eb.0042AC70  
0042AED6   .  85C0          TEST EAX,EAX  
0042AED8   .  74 1A         JE SHORT Eb.0042AEF4  
0042AEDA   .  6A 03         PUSH 3  
0042AEDC   .  C705 789A5500>MOV DWORD PTR DS:\[559A78\],474  
0042AEE6   .  8935 749A5500 MOV DWORD PTR DS:\[559A74\],ESI  
0042AEEC   .  E8 2F41FEFF   CALL Eb.0040F020  
0042AEF1   .  83C4 04       ADD ESP,4  
0042AEF4   >  E8 27310000   CALL Eb.0042E020  
0042AEF9   .  8BC5          MOV EAX,EBP  
0042AEFB   .  5E            POP ESI  
0042AEFC   .  5D            POP EBP  
0042AEFD   .  8B8C24 C80000>MOV ECX,DWORD PTR SS:\[ESP+C8\]  
0042AF04   .  64:890D 00000>MOV DWORD PTR FS:\[0\],ECX  
0042AF0B   .  81C4 D4000000 ADD ESP,0D4  
0042AF11   .  C3            RETN  
0042AF12   >  57            PUSH EDI  
0042AF13   .  8D4C24 0C     LEA ECX,DWORD PTR SS:\[ESP+C\]  
0042AF17   .  E8 B6420A00   CALL <JMP.&MFC42.#540>  
0042AF1C   .  68 98B45500   PUSH Eb.0055B498                                 ;  "授权使用"  
0042AF21   .  8D4C24 10     LEA ECX,DWORD PTR SS:\[ESP+10\]  
0042AF25   .  C68424 E00000>MOV BYTE PTR SS:\[ESP+E0\],6  
0042AF2D   .  E8 D6420A00   CALL <JMP.&MFC42.#860>  
0042AF32   .  8B7C24 0C     MOV EDI,DWORD PTR SS:\[ESP+C\]  
0042AF36   .  83C9 FF       OR ECX,FFFFFFFF  
0042AF39   .  33C0          XOR EAX,EAX  
0042AF3B   .  8D5424 54     LEA EDX,DWORD PTR SS:\[ESP+54\]  
0042AF3F   .  F2:AE         REPNE SCAS BYTE PTR ES:\[EDI\]  
0042AF41   .  F7D1          NOT ECX  
0042AF43   .  2BF9          SUB EDI,ECX  
0042AF45   .  8BC1          MOV EAX,ECX  
0042AF47   .  8BF7          MOV ESI,EDI  
0042AF49   .  8BFA          MOV EDI,EDX  
0042AF4B   .  8D5424 54     LEA EDX,DWORD PTR SS:\[ESP+54\]  
0042AF4F   .  C1E9 02       SHR ECX,2  
0042AF52   .  F3:A5         REP MOVS DWORD PTR ES:\[EDI\],DWORD PTR DS:\[ESI\]  
0042AF54   .  8BC8          MOV ECX,EAX  
0042AF56   .  33C0          XOR EAX,EAX  
0042AF58   .  83E1 03       AND ECX,3  
0042AF5B   .  50            PUSH EAX                                         ; /Style => MB\_OK|MB\_APPLMODAL  
0042AF5C   .  F3:A4         REP MOVS BYTE PTR ES:\[EDI\],BYTE PTR DS:\[ESI\]     ; |  
0042AF5E   .  8D7C24 18     LEA EDI,DWORD PTR SS:\[ESP+18\]                    ; |  
0042AF62   .  83C9 FF       OR ECX,FFFFFFFF                                  ; |  
0042AF65   .  F2:AE         REPNE SCAS BYTE PTR ES:\[EDI\]                     ; |  
0042AF67   .  F7D1          NOT ECX                                          ; |  
0042AF69   .  2BF9          SUB EDI,ECX                                      ; |  
0042AF6B   .  68 98B45500   PUSH Eb.0055B498                                 ; |Title = "授权使用"  
0042AF70   .  8BF7          MOV ESI,EDI                                      ; |  
0042AF72   .  8BFA          MOV EDI,EDX                                      ; |  
0042AF74   .  8BD1          MOV EDX,ECX                                      ; |  
0042AF76   .  83C9 FF       OR ECX,FFFFFFFF                                  ; |  
0042AF79   .  F2:AE         REPNE SCAS BYTE PTR ES:\[EDI\]                     ; |  
0042AF7B   .  8BCA          MOV ECX,EDX                                      ; |  
0042AF7D   .  4F            DEC EDI                                          ; |  
0042AF7E   .  C1E9 02       SHR ECX,2                                        ; |  
0042AF81   .  F3:A5         REP MOVS DWORD PTR ES:\[EDI\],DWORD PTR DS:\[ESI\]   ; |  
0042AF83   .  8BCA          MOV ECX,EDX                                      ; |  
0042AF85   .  8D4424 5C     LEA EAX,DWORD PTR SS:\[ESP+5C\]                    ; |  
0042AF89   .  83E1 03       AND ECX,3                                        ; |  
0042AF8C   .  50            PUSH EAX                                         ; |Text  
0042AF8D   .  F3:A4         REP MOVS BYTE PTR ES:\[EDI\],BYTE PTR DS:\[ESI\]     ; |  
0042AF8F   .  6A 00         PUSH 0                                           ; |hOwner = NULL  
0042AF91   .  FF15 38235300 CALL DWORD PTR DS:\[<&USER32.MessageBoxA>\]        ; \\MessageBoxA  
0042AF97   .  8D4C24 0C     LEA ECX,DWORD PTR SS:\[ESP+C\]  
0042AF9B   .  C68424 DC0000>MOV BYTE PTR SS:\[ESP+DC\],5  
0042AFA3   .  E8 48420A00   CALL <JMP.&MFC42.#800>  
0042AFA8   .  8B8C24 D40000>MOV ECX,DWORD PTR SS:\[ESP+D4\]  
0042AFAF   .  5F            POP EDI  
0042AFB0   .  8BC5          MOV EAX,EBP  
0042AFB2   .  5E            POP ESI  
0042AFB3   .  5D            POP EBP  
0042AFB4   .  64:890D 00000>MOV DWORD PTR FS:\[0\],ECX  
0042AFBB   .  81C4 D4000000 ADD ESP,0D4  
0042AFC1   .  C3            RETN  
  
看到没有 0042AEA6 处就是一个函数的判断，然后跟着判断跳转，只需要爆破就可。当然这样爆破弹出的“授权使用”的名称的是乱码，为此，可以自己 DIY 了。  
  
为了使得弹出的"授权使用"的对话框能够显示我自己的名称,我利用 PEID 打开程序，点 EP 段后面的那个 > 号，随便选择一个区段右击，  
选“搜索全 0 处”（原版好像是 cave 什么的）：找到 RVA 为 130025,偏移为 130025,长度为 0xFDB 的全 0 处.利用 UltraEdit 编辑 130025 处为  
"授权使用 laowang!",然后利用 Hiew 编辑代码  
0042AF85   .  8D4424 5C     LEA EAX,DWORD PTR SS:\[ESP+5C\]                    ; |  
0042AF89   .  83E1 03       AND ECX,3                                        ; |  
0042AF8C   .  50            PUSH EAX                                         ; |Text  
  
修改为  
0042AF85   .  68 25005300   PUSH 复件\_Eb.00530025                      ; |Text = "授权使用 laowang!"  
0042AF8A   .  90            NOP                                      ; |  
0042AF8B   .  90            NOP                                      ; |  
0042AF8C   .  90            NOP                                      ; |  
  
这样,就能弹出授权 laowang 的对话框了.大功告成.  
  
二、略微复杂的打狗  
某软件的破解主要针对 RYC\_OPEN,RYC\_READ 等函数即可.具体需要修改多处.为了某种需要将软件名称用“\*\*\*\*”代替，其实这是脱壳之后的流程。  
文中通过“-----”表示一级函数调用，“========”表示二级函数调用，请大家参看地址读该软件流程。  
00402D60     .  55                  push ebp  
00402D61     .  8B6C24 08           mov ebp,dword ptr ss:\[esp+8\]  
00402D65     .  56                  push esi  
00402D66     .  8B7424 10           mov esi,dword ptr ss:\[esp+10\]  
00402D6A     .  57                  push edi  
00402D6B     .  8BF9                mov edi,ecx  
00402D6D     .  8B47 0C             mov eax,dword ptr ds:\[edi+C\]  
00402D70     .  85C0                test eax,eax  
00402D72     .  896F 04             mov dword ptr ds:\[edi+4\],ebp  
00402D75     .  8977 08             mov dword ptr ds:\[edi+8\],esi  
00402D78     .  75 06               jnz short \*\*\*\*.00402D80  
00402D7A     .  5F                  pop edi  
00402D7B     .  5E                  pop esi  
00402D7C     .  5D                  pop ebp  
00402D7D     .  C2 0800             retn 8  
00402D80     >  53                  push ebx  
00402D81     .  8B18                mov ebx,dword ptr ds:\[eax\]  
00402D83     .  837B 14 00          cmp dword ptr ds:\[ebx+14\],0  
00402D87     .  74 05               je short \*\*\*\*.00402D8E  
00402D89     .  E8 D2C90000         call \*\*\*\*.0040F760                        ;  判断是否插入了 USB 狗  
\--------------------------------------------------------------------------------------------------------  
此处 F7 进去：  
0040F760    /$  56                  push esi  
0040F761    |.  57                  push edi  
0040F762    |.  33FF                xor edi,edi  
0040F764    |.  8D73 04             lea esi,dword ptr ds:\[ebx+4\]  
0040F767    |>  833E 00             /cmp dword ptr ds:\[esi\],0  
0040F76A    |.  74 0D               |je short \*\*\*\*.0040F779  
0040F76C    |.  8B0E                |mov ecx,dword ptr ds:\[esi\]  
0040F76E    |.  8B01                |mov eax,dword ptr ds:\[ecx\]  
0040F770    |.  8B50 04             |mov edx,dword ptr ds:\[eax+4\]  
0040F773    |.  FFD2                |call edx                                     ;  当循环到 edi==3 时,查询是否插入 Rockey USB 狗的关键函数,此时 F7 进去  
\=========================================================================================================  
此处 F7 进去：  
00412700     .  81EC 08020000       sub esp,208  
00412706     .  A1 8C924800         mov eax,dword ptr ds:\[48928C\]  
0041270B     .  33C4                xor eax,esp  
0041270D     .  898424 04020000     mov dword ptr ss:\[esp+204\],eax  
00412714     .  56                  push esi  
00412715     .  8BF1                mov esi,ecx  
00412717     .  837E 30 00          cmp dword ptr ds:\[esi+30\],0  
0041271B     .  0F84 95000000       je \*\*\*\*.004127B6  
00412721     .  837E 0C 00          cmp dword ptr ds:\[esi+C\],0  
00412725     .  74 4E               je short \*\*\*\*.00412775  
00412727     .  8B46 18             mov eax,dword ptr ds:\[esi+18\]  
0041272A     .  FFD0                call eax                                      ;  调用 RY2\_Find 函数  
0041272C        85C0                test eax,eax                                  ;  改为 xor eax,eax  
0041272E     .  0F8E 82000000       jle \*\*\*\*.004127B6                         ;  nop 掉  
00412734     .  8B56 1C             mov edx,dword ptr ds:\[esi+1C\]  
00412737     .  8D4C24 04           lea ecx,dword ptr ss:\[esp+4\]  
0041273B     .  51                  push ecx  
0041273C     .  68 2DEE9384         push 8493EE2D  
00412741     .  6A 01               push 1  
00412743     .  FFD2                call edx                                      ;  调用 Rockye2.RY2\_Open 函数  
00412745     .  85C0                test eax,eax                                  ;  改为 xor eax,eax  
00412747     .  7C 6D               jl short \*\*\*\*.004127B6                    ;  nop 掉  
00412749     .  8B4C24 04           mov ecx,dword ptr ss:\[esp+4\]  
0041274D     .  894E 08             mov dword ptr ds:\[esi+8\],ecx  
00412750     >  8946 04             mov dword ptr ds:\[esi+4\],eax  
00412753     .  C746 10 01000000    mov dword ptr ds:\[esi+10\],1  
0041275A     >  B8 01000000         mov eax,1  
0041275F     .  5E                  pop esi  
00412760     .  8B8C24 04020000     mov ecx,dword ptr ss:\[esp+204\]  
00412767     .  33CC                xor ecx,esp  
00412769     .  E8 ACFC0400         call \*\*\*\*.0046241A  
0041276E     .  81C4 08020000       add esp,208  
00412774     .  C3                  retn  
00412775     >  837E 10 00          cmp dword ptr ds:\[esi+10\],0  
00412779     .  75 27               jnz short \*\*\*\*.004127A2  
0041277B     .  8B56 18             mov edx,dword ptr ds:\[esi+18\]  
0041277E     .  FFD2                call edx  
00412780     .  85C0                test eax,eax  
00412782     .  7E 32               jle short \*\*\*\*.004127B6  
00412784     .  8B4E 1C             mov ecx,dword ptr ds:\[esi+1C\]  
00412787     .  8D4424 04           lea eax,dword ptr ss:\[esp+4\]  
0041278B     .  50                  push eax  
0041278C     .  68 2DEE9384         push 8493EE2D  
00412791     .  6A 01               push 1  
00412793     .  FFD1                call ecx  
00412795     .  85C0                test eax,eax  
00412797     .  7C 1D               jl short \*\*\*\*.004127B6  
00412799     .  8B5424 04           mov edx,dword ptr ss:\[esp+4\]  
0041279D     .  8956 08             mov dword ptr ds:\[esi+8\],edx  
004127A0     .^ EB AE               jmp short \*\*\*\*.00412750  
004127A2     >  8B4E 04             mov ecx,dword ptr ds:\[esi+4\]  
004127A5     .  8B56 28             mov edx,dword ptr ds:\[esi+28\]  
004127A8     .  8D4424 08           lea eax,dword ptr ss:\[esp+8\]  
004127AC     .  50                  push eax  
004127AD     .  6A 00               push 0  
004127AF     .  51                  push ecx  
004127B0     .  FFD2                call edx  
004127B2     .  85C0                test eax,eax  
004127B4     .^ 7D A4               jge short \*\*\*\*.0041275A  
004127B6     >  8B8C24 08020000     mov ecx,dword ptr ss:\[esp+208\]  
004127BD     .  5E                  pop esi  
004127BE     .  33CC                xor ecx,esp  
004127C0     .  33C0                xor eax,eax  
004127C2     .  E8 53FC0400         call \*\*\*\*.0046241A  
004127C7     .  81C4 08020000       add esp,208  
004127CD     .  C3                  retn  
004127CE        CC                  int3  
004127CF        CC                  int3  
004127D0     .  33C0                xor eax,eax  
004127D2     .  3941 30             cmp dword ptr ds:\[ecx+30\],eax  
004127D5     .  0F95C0              setne al  
004127D8     .  C3                  retn  
\=========================================================================================================  
0040F775    |.  85C0                |test eax,eax  
0040F777    |.  75 15               |jnz short \*\*\*\*.0040F78E  
0040F779    |>  83C7 01             |add edi,1  
0040F77C    |.  83C6 04             |add esi,4  
0040F77F    |.  83FF 04             |cmp edi,4  
0040F782    |.^ 7C E3               \\jl short \*\*\*\*.0040F767  
0040F784    |.  5F                  pop edi  
0040F785    |.  C743 18 FFFFFFFF    mov dword ptr ds:\[ebx+18\],-1  
0040F78C    |.  5E                  pop esi  
0040F78D    |.  C3                  retn  
0040F78E    |>  897B 18             mov dword ptr ds:\[ebx+18\],edi  
0040F791    |.  5F                  pop edi  
0040F792    |.  5E                  pop esi  
0040F793    \\.  C3                  retn  
\--------------------------------------------------------------------------------------------------------  
00402D8E     >  837B 18 FF          cmp dword ptr ds:\[ebx+18\],-1  
00402D92     .  0F84 0A010000       je \*\*\*\*.00402EA2                          ;  错误跳转 1  
00402D98     .  8B43 18             mov eax,dword ptr ds:\[ebx+18\]  
00402D9B     .  8B4C83 04           mov ecx,dword ptr ds:\[ebx+eax\*4+4\]  
00402D9F     .  8B11                mov edx,dword ptr ds:\[ecx\]  
00402DA1     .  8B42 04             mov eax,dword ptr ds:\[edx+4\]  
00402DA4     .  FFD0                call eax                                      ;  此处与 402DB9 一样再次判断是否插入了 USB 狗,前面的爆破搞定  
00402DA6     .  85C0                test eax,eax  
00402DA8     .  0F84 F4000000       je \*\*\*\*.00402EA2                          ;  错误跳转 2  
00402DAE     .  8B47 0C             mov eax,dword ptr ds:\[edi+C\]  
00402DB1     .  8D4C24 14           lea ecx,dword ptr ss:\[esp+14\]  
00402DB5     .  51                  push ecx  
00402DB6     .  E8 552C0000         call \*\*\*\*.00405A10                        ;  读取狗中数据,进行验证  
\--------------------------------------------------------------------------------------------------------  
此处 F7 进去：  
00405A10    /$  51                  push ecx  
00405A11    |.  53                  push ebx  
00405A12    |.  8B18                mov ebx,dword ptr ds:\[eax\]  
00405A14    |.  837B 14 00          cmp dword ptr ds:\[ebx+14\],0  
00405A18    |.  74 05               je short \*\*\*\*.00405A1F  
00405A1A    |.  E8 419D0000         call \*\*\*\*.0040F760                        ;  判断是否插入了 USB 狗,前面处理过了  
00405A1F    |>  837B 18 FF          cmp dword ptr ds:\[ebx+18\],-1  
00405A23    |.  74 20               je short \*\*\*\*.00405A45  
00405A25    |.  8B4B 18             mov ecx,dword ptr ds:\[ebx+18\]  
00405A28    |.  8B4C8B 04           mov ecx,dword ptr ds:\[ebx+ecx\*4+4\]  
00405A2C    |.  8B11                mov edx,dword ptr ds:\[ecx\]  
00405A2E    |.  8B52 0C             mov edx,dword ptr ds:\[edx+C\]  
00405A31    |.  8D4424 04           lea eax,dword ptr ss:\[esp+4\]  
00405A35    |.  50                  push eax  
00405A36    |.  6A 04               push 4  
00405A38    |.  6A 30               push 30  
00405A3A    |.  68 E2850000         push 85E2  
00405A3F    |.  FFD2                call edx                                      ;  读狗数据  
\=========================================================================================================  
004127E0     .  81EC 04020000       sub esp,204  
004127E6     .  A1 8C924800         mov eax,dword ptr ds:\[48928C\]  
004127EB     .  33C4                xor eax,esp  
004127ED     .  898424 00020000     mov dword ptr ss:\[esp+200\],eax  
004127F4     .  56                  push esi  
004127F5     .  8BF1                mov esi,ecx  
004127F7     .  837E 30 00          cmp dword ptr ds:\[esi+30\],0  
004127FB     .  57                  push edi  
004127FC     .  8BBC24 1C020000     mov edi,dword ptr ss:\[esp+21C\]  
00412803     .  75 04               jnz short \*\*\*\*.00412809  
00412805     >  33C0                xor eax,eax  
00412807     .  EB 4C               jmp short \*\*\*\*.00412855  
00412809     >  68 00020000         push 200                                      ; /n = 200 (512.)  
0041280E     .  8D4424 0C           lea eax,dword ptr ss:\[esp+C\]                  ; |  
00412812     .  6A 00               push 0                                        ; |c = 00  
00412814     .  50                  push eax                                      ; |s  
00412815     .  E8 9EFC0400         call <jmp.&MSVCR80.memset>                    ; \\memset  
0041281A     .  8B56 04             mov edx,dword ptr ds:\[esi+4\]  
0041281D     .  8B46 28             mov eax,dword ptr ds:\[esi+28\]  
00412820     .  83C4 0C             add esp,0C  
00412823     .  8D4C24 08           lea ecx,dword ptr ss:\[esp+8\]  
00412827     .  51                  push ecx  
00412828     .  6A 00               push 0  
0041282A     .  52                  push edx  
0041282B     .  FFD0                call eax                                      ;  调用 Rockey2.RY2\_Read 函数,读取狗中的数据进行验证  
0041282D     .  85C0                test eax,eax                                  ;  爆破为 xor eax,eax  
0041282F     .^\\7C D4               jl short \*\*\*\*.00412805  
00412831     .  0FBF8C24 18020000   movsx ecx,word ptr ss:\[esp+218\]  
00412839     .  0FBF9424 14020000   movsx edx,word ptr ss:\[esp+214\]  
00412841     .  51                  push ecx                                      ; /n  
00412842     .  8D4414 0C           lea eax,dword ptr ss:\[esp+edx+C\]              ; |  
00412846     .  50                  push eax                                      ; |src  
00412847     .  57                  push edi                                      ; |dest  
00412848     .  E8 B5FC0400         call <jmp.&MSVCR80.memcpy>                    ; \\memcpy  
0041284D     .  83C4 0C             add esp,0C  
00412850     .  B8 01000000         mov eax,1  
00412855     >  8B8C24 08020000     mov ecx,dword ptr ss:\[esp+208\]  
0041285C     .  5F                  pop edi  
0041285D     .  5E                  pop esi  
0041285E     .  33CC                xor ecx,esp  
00412860     .  E8 B5FB0400         call \*\*\*\*.0046241A  
00412865     .  81C4 04020000       add esp,204  
0041286B     .  C2 1000             retn 10  
\=========================================================================================================  
00405A41    |.  85C0                test eax,eax  
00405A43    |.  75 0A               jnz short \*\*\*\*.00405A4F  
00405A45    |>  B8 01000000         mov eax,1  
00405A4A    |.  5B                  pop ebx  
00405A4B    |.  59                  pop ecx  
00405A4C    |.  C2 0400             retn 4  
00405A4F    |> \\8B4424 04           mov eax,dword ptr ss:\[esp+4\]  
00405A53    |.  8B4C24 0C           mov ecx,dword ptr ss:\[esp+C\]  
00405A57        8901                mov dword ptr ds:\[ecx\],eax                    ;  为了 402DC3 处的比较，爆破为 mov dword ptr ds:\[ecx\],esi  
00405A59    |.  33C0                xor eax,eax  
00405A5B    |.  5B                  pop ebx  
00405A5C    |.  59                  pop ecx  
00405A5D    \\.  C2 0400             retn 4  
\--------------------------------------------------------------------------------------------------------  
00402DBB     .  85C0                test eax,eax  
00402DBD     .  0F85 DF000000       jnz \*\*\*\*.00402EA2                         ;  错误跳转 3  
00402DC3     .  397424 14           cmp dword ptr ss:\[esp+14\],esi  
00402DC7     .  0F85 D5000000       jnz \*\*\*\*.00402EA2                         ;  错误跳转 4  
00402DCD     .  8B47 0C             mov eax,dword ptr ds:\[edi+C\]  
00402DD0     .  8D5424 14           lea edx,dword ptr ss:\[esp+14\]  
00402DD4     .  33DB                xor ebx,ebx  
00402DD6     .  52                  push edx  
00402DD7     .  895C24 18           mov dword ptr ss:\[esp+18\],ebx  
00402DDB     .  E8 C02B0000         call \*\*\*\*.004059A0  
00402DE0     .  85C0                test eax,eax  
00402DE2     .  75 09               jnz short \*\*\*\*.00402DED  
00402DE4     .  66:817C24 14 0408   cmp word ptr ss:\[esp+14\],804  
00402DEB     .  75 12               jnz short \*\*\*\*.00402DFF  
00402DED     >  81FD 04080000       cmp ebp,804  
00402DF3     .  74 0A               je short \*\*\*\*.00402DFF  
00402DF5     .  8B7F 2C             mov edi,dword ptr ds:\[edi+2C\]  
00402DF8     .  3BFB                cmp edi,ebx  
00402DFA     .  E9 A8000000         jmp \*\*\*\*.00402EA7  
00402DFF     >  8B4F 0C             mov ecx,dword ptr ds:\[edi+C\]  
00402E02     .  8D77 18             lea esi,dword ptr ds:\[edi+18\]  
00402E05     .  895F 10             mov dword ptr ds:\[edi+10\],ebx  
00402E08     .  E8 D32A0000         call \*\*\*\*.004058E0                        ;  验证函数  
\--------------------------------------------------------------------------------------------------------  
此处 F7 进去：  
004058E0    /$  83EC 1C             sub esp,1C  
004058E3    |.  A1 8C924800         mov eax,dword ptr ds:\[48928C\]  
004058E8    |.  33C4                xor eax,esp  
004058EA    |.  894424 18           mov dword ptr ss:\[esp+18\],eax  
004058EE    |.  33C0                xor eax,eax  
004058F0    |.  53                  push ebx  
004058F1    |.  8B19                mov ebx,dword ptr ds:\[ecx\]  
004058F3    |.  894424 10           mov dword ptr ss:\[esp+10\],eax  
004058F7    |.  894424 14           mov dword ptr ss:\[esp+14\],eax  
004058FB    |.  884424 18           mov byte ptr ss:\[esp+18\],al  
004058FF    |.  3943 14             cmp dword ptr ds:\[ebx+14\],eax  
00405902    |.  74 05               je short \*\*\*\*.00405909  
00405904    |.  E8 579E0000         call \*\*\*\*.0040F760                        ;  查询 USB 狗是否存在，前面爆破了  
00405909    |>  837B 18 FF          cmp dword ptr ds:\[ebx+18\],-1  
0040590D    |.  74 20               je short \*\*\*\*.0040592F  
0040590F    |.  8B43 18             mov eax,dword ptr ds:\[ebx+18\]  
00405912    |.  8B4C83 04           mov ecx,dword ptr ds:\[ebx+eax\*4+4\]  
00405916    |.  8B11                mov edx,dword ptr ds:\[ecx\]  
00405918    |.  8B52 0C             mov edx,dword ptr ds:\[edx+C\]  
0040591B    |.  8D4424 10           lea eax,dword ptr ss:\[esp+10\]  
0040591F    |.  50                  push eax  
00405920    |.  6A 08               push 8  
00405922    |.  6A 24               push 24  
00405924    |.  68 E2850000         push 85E2  
00405929    |.  FFD2                call edx                                      ;  读 USB 狗数据，前面爆破了  
0040592B    |.  85C0                test eax,eax  
0040592D    |.  75 15               jnz short \*\*\*\*.00405944  
0040592F    |>  B8 01000000         mov eax,1  
00405934    |.  5B                  pop ebx  
00405935    |.  8B4C24 18           mov ecx,dword ptr ss:\[esp+18\]  
00405939    |.  33CC                xor ecx,esp  
0040593B    |.  E8 DACA0500         call \*\*\*\*.0046241A  
00405940    |.  83C4 1C             add esp,1C  
00405943    |.  C3                  retn  
00405944    |>  33C0                xor eax,eax  
00405946    |.  894424 04           mov dword ptr ss:\[esp+4\],eax  
0040594A    |.  894424 08           mov dword ptr ss:\[esp+8\],eax  
0040594E    |.  884424 0C           mov byte ptr ss:\[esp+C\],al  
00405952    |.  8D4424 04           lea eax,dword ptr ss:\[esp+4\]  
00405956    |.  50                  push eax  
00405957    |.  B8 70C64800         mov eax,\*\*\*\*.0048C670  
0040595C    |.  8D4C24 14           lea ecx,dword ptr ss:\[esp+14\]  
00405960    |.  E8 3BAC0000         call \*\*\*\*.004105A0  
00405965    |.  8B4C24 08           mov ecx,dword ptr ss:\[esp+8\]  
00405969    |.  81C1 03250000       add ecx,2503  
0040596F    |.  F7D1                not ecx  
00405971    |.  83C4 04             add esp,4  
00405974    |.  66:394C24 08        cmp word ptr ss:\[esp+8\],cx  
00405979    |.^ 75 B4               jnz short \*\*\*\*.0040592F                   ;  nop 掉  
0040597B    |.  0FB75424 04         movzx edx,word ptr ss:\[esp+4\]  
00405980    |.  8B4C24 1C           mov ecx,dword ptr ss:\[esp+1C\]  
00405984    |.  5B                  pop ebx  
00405985    |.  33CC                xor ecx,esp  
00405987    |.  8916                mov dword ptr ds:\[esi\],edx  
00405989    |.  33C0                xor eax,eax  
0040598B    |.  E8 8ACA0500         call \*\*\*\*.0046241A  
00405990    |.  83C4 1C             add esp,1C  
00405993    \\.  C3                  retn  
\--------------------------------------------------------------------------------------------------------  
00402E0D     .  85C0                test eax,eax  
00402E0F     .  74 0A               je short \*\*\*\*.00402E1B                    ;  错误跳转 5  
00402E11     .  8B7F 2C             mov edi,dword ptr ds:\[edi+2C\]  
00402E14     .  3BFB                cmp edi,ebx  
00402E16     .  E9 8C000000         jmp \*\*\*\*.00402EA7  
00402E1B     > \\8B4F 0C             mov ecx,dword ptr ds:\[edi+C\]  
00402E1E     .  E8 2D260000         call \*\*\*\*.00405450                        ;  验证函数 2  
\--------------------------------------------------------------------------------------------------------  
此处 F7 进去：  
00405450    /$  83EC 1C             sub esp,1C  
00405453    |.  A1 8C924800         mov eax,dword ptr ds:\[48928C\]  
00405458    |.  33C4                xor eax,esp  
0040545A    |.  894424 18           mov dword ptr ss:\[esp+18\],eax  
0040545E    |.  33C0                xor eax,eax  
00405460    |.  53                  push ebx  
00405461    |.  8B19                mov ebx,dword ptr ds:\[ecx\]  
00405463    |.  894424 10           mov dword ptr ss:\[esp+10\],eax  
00405467    |.  894424 14           mov dword ptr ss:\[esp+14\],eax  
0040546B    |.  884424 18           mov byte ptr ss:\[esp+18\],al  
0040546F    |.  3943 14             cmp dword ptr ds:\[ebx+14\],eax  
00405472    |.  74 05               je short \*\*\*\*.00405479  
00405474    |.  E8 E7A20000         call \*\*\*\*.0040F760                        ;  查询 USB 狗是否存在  
00405479    |>  837B 18 FF          cmp dword ptr ds:\[ebx+18\],-1  
0040547D    |.  74 20               je short \*\*\*\*.0040549F  
0040547F    |.  8B43 18             mov eax,dword ptr ds:\[ebx+18\]  
00405482    |.  8B4C83 04           mov ecx,dword ptr ds:\[ebx+eax\*4+4\]  
00405486    |.  8B11                mov edx,dword ptr ds:\[ecx\]  
00405488    |.  8B52 0C             mov edx,dword ptr ds:\[edx+C\]  
0040548B    |.  8D4424 10           lea eax,dword ptr ss:\[esp+10\]  
0040548F    |.  50                  push eax  
00405490    |.  6A 08               push 8  
00405492    |.  6A 14               push 14  
00405494    |.  68 E2850000         push 85E2  
00405499    |.  FFD2                call edx                                      ;  读取 USB 狗数据  
0040549B    |.  85C0                test eax,eax  
0040549D    |.  75 15               jnz short \*\*\*\*.004054B4  
0040549F    |>  B8 01000000         mov eax,1  
004054A4    |.  5B                  pop ebx  
004054A5    |.  8B4C24 18           mov ecx,dword ptr ss:\[esp+18\]  
004054A9    |.  33CC                xor ecx,esp  
004054AB    |.  E8 6ACF0500         call \*\*\*\*.0046241A  
004054B0    |.  83C4 1C             add esp,1C  
004054B3    |.  C3                  retn  
004054B4    |>  33C0                xor eax,eax  
004054B6    |.  894424 04           mov dword ptr ss:\[esp+4\],eax  
004054BA    |.  894424 08           mov dword ptr ss:\[esp+8\],eax  
004054BE    |.  884424 0C           mov byte ptr ss:\[esp+C\],al  
004054C2    |.  8D4424 04           lea eax,dword ptr ss:\[esp+4\]  
004054C6    |.  50                  push eax  
004054C7    |.  B8 70C64800         mov eax,\*\*\*\*.0048C670  
004054CC    |.  8D4C24 14           lea ecx,dword ptr ss:\[esp+14\]  
004054D0    |.  E8 CBB00000         call \*\*\*\*.004105A0  
004054D5    |.  8B4424 08           mov eax,dword ptr ss:\[esp+8\]  
004054D9    |.  8D88 29090000       lea ecx,dword ptr ds:\[eax+929\]  
004054DF    |.  F7D1                not ecx  
004054E1    |.  83C4 04             add esp,4  
004054E4    |.  3B4C24 08           cmp ecx,dword ptr ss:\[esp+8\]  
004054E8    |.^ 75 B5               jnz short \*\*\*\*.0040549F                   ;  nop 掉  
004054EA    |.  8B4C24 1C           mov ecx,dword ptr ss:\[esp+1C\]  
004054EE    |.  33D2                xor edx,edx  
004054F0    |.  3D 73743612         cmp eax,12367473  
004054F5    |.  0F95C2              setne dl  
004054F8    |.  5B                  pop ebx  
004054F9    |.  33CC                xor ecx,esp  
004054FB    |.  8BC2                mov eax,edx  
004054FD    |.  E8 18CF0500         call \*\*\*\*.0046241A  
00405502    |.  83C4 1C             add esp,1C  
00405505    \\.  C3                  retn  
\--------------------------------------------------------------------------------------------------------  
00402E23     .  85C0                test eax,eax  
00402E25     .  75 03               jnz short \*\*\*\*.00402E2A                   ;  错误跳转 6  
00402E27     .  895F 14             mov dword ptr ds:\[edi+14\],ebx  
00402E2A     >  395F 14             cmp dword ptr ds:\[edi+14\],ebx  
00402E2D     .  74 67               je short \*\*\*\*.00402E96  
00402E2F     .  8B4F 0C             mov ecx,dword ptr ds:\[edi+C\]  
00402E32     .  8D7424 14           lea esi,dword ptr ss:\[esp+14\]  
00402E36     .  E8 F5280000         call \*\*\*\*.00405730                        ;  验证函数 3  
\--------------------------------------------------------------------------------------------------------  
此处 F7 进去：  
00405730    /$  83EC 1C             sub esp,1C  
00405733    |.  A1 8C924800         mov eax,dword ptr ds:\[48928C\]  
00405738    |.  33C4                xor eax,esp  
0040573A    |.  894424 18           mov dword ptr ss:\[esp+18\],eax  
0040573E    |.  33C0                xor eax,eax  
00405740    |.  53                  push ebx  
00405741    |.  8B19                mov ebx,dword ptr ds:\[ecx\]  
00405743    |.  894424 10           mov dword ptr ss:\[esp+10\],eax  
00405747    |.  894424 14           mov dword ptr ss:\[esp+14\],eax  
0040574B    |.  884424 18           mov byte ptr ss:\[esp+18\],al  
0040574F    |.  3943 14             cmp dword ptr ds:\[ebx+14\],eax  
00405752    |.  74 05               je short \*\*\*\*.00405759  
00405754    |.  E8 07A00000         call \*\*\*\*.0040F760  
00405759    |>  837B 18 FF          cmp dword ptr ds:\[ebx+18\],-1  
0040575D    |.  74 20               je short \*\*\*\*.0040577F  
0040575F    |.  8B43 18             mov eax,dword ptr ds:\[ebx+18\]  
00405762    |.  8B4C83 04           mov ecx,dword ptr ds:\[ebx+eax\*4+4\]  
00405766    |.  8B11                mov edx,dword ptr ds:\[ecx\]  
00405768    |.  8B52 0C             mov edx,dword ptr ds:\[edx+C\]  
0040576B    |.  8D4424 10           lea eax,dword ptr ss:\[esp+10\]  
0040576F    |.  50                  push eax  
00405770    |.  6A 08               push 8  
00405772    |.  6A 1C               push 1C  
00405774    |.  68 E2850000         push 85E2  
00405779    |.  FFD2                call edx  
0040577B    |.  85C0                test eax,eax  
0040577D    |.  75 15               jnz short \*\*\*\*.00405794  
0040577F    |>  B8 01000000         mov eax,1  
00405784    |.  5B                  pop ebx  
00405785    |.  8B4C24 18           mov ecx,dword ptr ss:\[esp+18\]  
00405789    |.  33CC                xor ecx,esp  
0040578B    |.  E8 8ACC0500         call \*\*\*\*.0046241A  
00405790    |.  83C4 1C             add esp,1C  
00405793    |.  C3                  retn  
00405794    |>  33C0                xor eax,eax  
00405796    |.  894424 04           mov dword ptr ss:\[esp+4\],eax  
0040579A    |.  894424 08           mov dword ptr ss:\[esp+8\],eax  
0040579E    |.  884424 0C           mov byte ptr ss:\[esp+C\],al  
004057A2    |.  8D4424 04           lea eax,dword ptr ss:\[esp+4\]  
004057A6    |.  50                  push eax  
004057A7    |.  B8 70C64800         mov eax,\*\*\*\*.0048C670  
004057AC    |.  8D4C24 14           lea ecx,dword ptr ss:\[esp+14\]  
004057B0    |.  E8 EBAD0000         call \*\*\*\*.004105A0  
004057B5    |.  8B4424 08           mov eax,dword ptr ss:\[esp+8\]  
004057B9    |.  8D88 AC200000       lea ecx,dword ptr ds:\[eax+20AC\]  
004057BF    |.  F7D1                not ecx  
004057C1    |.  83C4 04             add esp,4  
004057C4    |.  3B4C24 08           cmp ecx,dword ptr ss:\[esp+8\]  
004057C8    |.^ 75 B5               jnz short \*\*\*\*.0040577F                   ;  nop 掉  
004057CA    |.  8B4C24 1C           mov ecx,dword ptr ss:\[esp+1C\]  
004057CE    |.  5B                  pop ebx  
004057CF    |.  8906                mov dword ptr ds:\[esi\],eax  
004057D1    |.  33CC                xor ecx,esp  
004057D3    |.  33C0                xor eax,eax  
004057D5    |.  E8 40CC0500         call \*\*\*\*.0046241A  
004057DA    |.  83C4 1C             add esp,1C  
004057DD    \\.  C3                  retn  
\--------------------------------------------------------------------------------------------------------  
00402E3B     .  85C0                test eax,eax  
00402E3D     .  74 07               je short \*\*\*\*.00402E46                    ;  错误跳转 7  
00402E3F     .  8B7F 2C             mov edi,dword ptr ds:\[edi+2C\]  
00402E42     .  3BFB                cmp edi,ebx  
00402E44     .  EB 61               jmp short \*\*\*\*.00402EA7  
00402E46     >  8B4424 14           mov eax,dword ptr ss:\[esp+14\]  
00402E4A     .  8D48 FF             lea ecx,dword ptr ds:\[eax-1\]  
00402E4D     .  81F9 0E0E0000       cmp ecx,0E0E  
00402E53     .  77 19               ja short \*\*\*\*.00402E6E                    ;  是否过期判断  
00402E55     .  8B7F 28             mov edi,dword ptr ds:\[edi+28\]  
00402E58     .  3BFB                cmp edi,ebx  
00402E5A     .  74 3A               je short \*\*\*\*.00402E96  
00402E5C     .  50                  push eax  
00402E5D     .  FFD7                call edi  
00402E5F     .  83C4 04             add esp,4  
00402E62     .  5B                  pop ebx  
00402E63     .  5F                  pop edi  
00402E64     .  5E                  pop esi  
00402E65     .  B8 01000000         mov eax,1  
00402E6A     .  5D                  pop ebp  
00402E6B     .  C2 0800             retn 8  
00402E6E     >  3BC3                cmp eax,ebx  
00402E70     .  75 24               jnz short \*\*\*\*.00402E96  
00402E72     .  8B47 30             mov eax,dword ptr ds:\[edi+30\]  
00402E75     .  3BC3                cmp eax,ebx  
00402E77     .  74 02               je short \*\*\*\*.00402E7B  
00402E79     .  FFD0                call eax  
00402E7B     >  8B17                mov edx,dword ptr ds:\[edi\]  
00402E7D     .  8B42 18             mov eax,dword ptr ds:\[edx+18\]  
00402E80     .  8BCF                mov ecx,edi  
00402E82     .  FFD0                call eax  
00402E84     .  8B4F 0C             mov ecx,dword ptr ds:\[edi+C\]  
00402E87     .  E8 C4250000         call \*\*\*\*.00405450  
00402E8C     .  85C0                test eax,eax  
00402E8E     .  75 1B               jnz short \*\*\*\*.00402EAB  
00402E90     .  895F 14             mov dword ptr ds:\[edi+14\],ebx  
00402E93     .  895F 10             mov dword ptr ds:\[edi+10\],ebx  
00402E96     >  5B                  pop ebx  
00402E97     .  5F                  pop edi  
00402E98     .  5E                  pop esi  
00402E99     .  B8 01000000         mov eax,1  
00402E9E     .  5D                  pop ebp  
00402E9F     .  C2 0800             retn 8  
00402EA2     >  8B7F 2C             mov edi,dword ptr ds:\[edi+2C\]  
00402EA5     .  85FF                test edi,edi  
00402EA7     >  74 02               je short \*\*\*\*.00402EAB  
00402EA9     .  FFD7                call edi  
00402EAB     >  5B                  pop ebx  
00402EAC     .  5F                  pop edi  
00402EAD     .  5E                  pop esi  
00402EAE     .  33C0                xor eax,eax  
00402EB0     .  5D                  pop ebp  
00402EB1     .  C2 0800             retn 8  
  
总结可以发现软件打狗主要需要针对 RYC\_OPEN,RYC\_READ 函数调用进行爆破，具体爆破需要根据软件流程进行爆破，很难三言两语进行交待，我等菜鸟凑合看吧！  
高手飘过。