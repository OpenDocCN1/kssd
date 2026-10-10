

# 某网络验证软件的初解析

> 原文：[`www.pediy.com/kssd/pediy08/pediy8-285.htm`](https://www.pediy.com/kssd/pediy08/pediy8-285.htm)

【文章标题】某网络验证程序的初跟踪  
【文章作者】误入楼台  
【破解工具】Peid0.94\\OD1.10CCDEBUG 版  
【破解平台】WinXP   
【软件大小】1.9M  
【原版下载】不提供  
【保护方式】网络会员验证  
【软件简介】无  
某程序是进行网络验证的做这个程序的不用说做这个程序的肯定是破解高手，但是也有百密一疏漏的时候，  
就被我不注意给找到了解密地方了用 PEID 查 aspack2.12 这个壳是好脱的无论手脱还是脱壳机都比较好搞，脱  
壳后运行正常。  
OD 载入运行提示输入帐号和密码且要求 3-12 位，我们随便输入慢慢跟踪到下面这里  
  
00408A30  /$  83EC 70       SUB ESP,70  
00408A33  |.  A1 FC884400   MOV EAX,DWORD PTR DS:\[4488FC\]  
00408A38  |.  53            PUSH EBX  
00408A39  |.  8B5C24 78     MOV EBX,DWORD PTR SS:\[ESP+78\]  
00408A3D  |.  894424 70     MOV DWORD PTR SS:\[ESP+70\],EAX  
00408A41  |.  55            PUSH EBP  
00408A42  |.  8BC3          MOV EAX,EBX  
00408A44  |.  56            PUSH ESI  
00408A45  |.  8BE9          MOV EBP,ECX  
00408A47  |.  8D50 01       LEA EDX,DWORD PTR DS:\[EAX+1\]  
00408A4A  |.  8D9B 00000000 LEA EBX,DWORD PTR DS:\[EBX\]  
00408A50  |>  8A08          /MOV CL,BYTE PTR DS:\[EAX\]  
00408A52  |.  40            |INC EAX  
00408A53  |.  84C9          |TEST CL,CL  
00408A55  |.^ 75 F9         \\JNZ SHORT xxxxx.00408A50  
00408A57  |.  2BC2          SUB EAX,EDX                     ;计算帐户的位数  
00408A59  |.  83F8 0C       CMP EAX,0C  
00408A5C  |.  0F87 8C030000 JA xxxxx.00408DEE               ；比较大于 12 位跳出错  
00408A62  |.  8BC3          MOV EAX,EBX  
00408A64  |.  8D50 01       LEA EDX,DWORD PTR DS:\[EAX+1\]  
00408A67  |>  8A08          /MOV CL,BYTE PTR DS:\[EAX\]  
00408A69  |.  40            |INC EAX  
00408A6A  |.  84C9          |TEST CL,CL  
00408A6C  |.^ 75 F9         \\JNZ SHORT xxxxx.00408A67  
00408A6E  |.  2BC2          SUB EAX,EDX                       
00408A70  |.  83F8 03       CMP EAX,3                          
00408A73  |.  0F82 75030000 JB xxxxx.00408DEE               ；小于 3 位出错  
00408A79  |.  8BB424 840000>MOV ESI,DWORD PTR SS:\[ESP+84\]  
00408A80  |.  8BC6          MOV EAX,ESI  
00408A82  |.  8D50 01       LEA EDX,DWORD PTR DS:\[EAX+1\]  
00408A85  |>  8A08          /MOV CL,BYTE PTR DS:\[EAX\]  
00408A87  |.  40            |INC EAX  
00408A88  |.  84C9          |TEST CL,CL  
00408A8A  |.^ 75 F9         \\JNZ SHORT xxxxx.00408A85  
00408A8C  |.  2BC2          SUB EAX,EDX                     ；密码的位数  
00408A8E  |.  83F8 0C       CMP EAX,0C                        
00408A91  |.  0F87 57030000 JA xxxxx.00408DEE               ；大于 12 位跳  
00408A97  |.  8BC3          MOV EAX,EBX                           
00408A99  |.  8D50 01       LEA EDX,DWORD PTR DS:\[EAX+1\]  
00408A9C  |.  8D6424 00     LEA ESP,DWORD PTR SS:\[ESP\]  
00408AA0  |>  8A08          /MOV CL,BYTE PTR DS:\[EAX\]  
00408AA2  |.  40            |INC EAX  
00408AA3  |.  84C9          |TEST CL,CL  
00408AA5  |.^ 75 F9         \\JNZ SHORT xxxxx.00408AA0  
00408AA7  |.  2BC2          SUB EAX,EDX  
00408AA9  |.  83F8 03       CMP EAX,3                       ；小于 3 位出错  
00408AAC  |.  0F82 3C030000 JB xxxxx.00408DEE  
00408AB2  |.  53            PUSH EBX  
00408AB3  |.  E8 38330000   CALL xxxxx.0040BDF0             ；这个 CALL 判断有无非法字符  
00408AB8  |.  83C4 04       ADD ESP,4  
00408ABB  |.  84C0          TEST AL,AL  
00408ABD  |.  0F85 2B030000 JNZ xxxxx.00408DEE  
00408AC3  |.  56            PUSH ESI  
00408AC4  |.  E8 27330000   CALL xxxxx.0040BDF0  
00408AC9  |.  83C4 04       ADD ESP,4  
00408ACC  |.  84C0          TEST AL,AL  
00408ACE  |.  0F85 1A030000 JNZ xxxxx.00408DEE  
00408AD4  |.  8B45 04       MOV EAX,DWORD PTR SS:\[EBP+4\]  
00408AD7  |.  83F8 01       CMP EAX,1  
00408ADA  |.  75 14         JNZ SHORT xxxxx.00408AF0  
00408ADC  |.  5E            POP ESI  
00408ADD  |.  5D            POP EBP  
00408ADE  |.  33C0          XOR EAX,EAX  
00408AE0  |.  5B            POP EBX  
00408AE1  |.  8B4C24 6C     MOV ECX,DWORD PTR SS:\[ESP+6C\]  
00408AE5  |.  E8 BFE00000   CALL xxxxx.00416BA9            ；不跳则提示网络不通，主要是防破解提示用的  
00408AEA  |.  83C4 70       ADD ESP,70  
00408AED  |.  C2 1C00       RETN 1C  
  
00408AF0  |>  83F8 02       CMP EAX,2  
00408AF3  |.  75 17         JNZ SHORT xxxxx.00408B0C       ；不跳直接网络不通  
00408AF5  |.  5E            POP ESI  
00408AF6  |.  5D            POP EBP  
00408AF7  |.  B8 FCFFFFFF   MOV EAX,-4  
00408AFC  |.  5B            POP EBX  
00408AFD  |.  8B4C24 6C     MOV ECX,DWORD PTR SS:\[ESP+6C\]  
00408B01  |.  E8 A3E00000   CALL xxxxx.00416BA9  
00408B06  |.  83C4 70       ADD ESP,70  
00408B09  |.  C2 1C00       RETN 1C  
  
00408B0C  |>  57            PUSH EDI  
00408B0D  |.  33C0          XOR EAX,EAX  
00408B0F  |.  B9 0C000000   MOV ECX,0C  
00408B14  |.  8D7C24 4C     LEA EDI,DWORD PTR SS:\[ESP+4C\]  
00408B18  |.  F3:AB         REP STOS DWORD PTR ES:\[EDI\]  
00408B1A  |.  8B45 10       MOV EAX,DWORD PTR SS:\[EBP+10\] ；程序版本号传给 EAX  
00408B1D  |.  894424 50     MOV DWORD PTR SS:\[ESP+50\],EAX  
00408B21  |.  8BC3          MOV EAX,EBX                    ；传帐号给 EAX  
00408B23  |.  C745 04 02000>MOV DWORD PTR SS:\[EBP+4\],2  
00408B2A  |.  66:C74424 4C >MOV WORD PTR SS:\[ESP+4C\],101  
00408B31  |.  8D50 01       LEA EDX,DWORD PTR DS:\[EAX+1\]  
00408B34  |>  8A08          /MOV CL,BYTE PTR DS:\[EAX\]  
00408B36  |.  40            |INC EAX  
00408B37  |.  84C9          |TEST CL,CL  
00408B39  |.^ 75 F9         \\JNZ SHORT xxxxx.00408B34  
00408B3B  |.  2BC2          SUB EAX,EDX                      ；帐号位数  
00408B3D  |.  8BC8          MOV ECX,EAX  
00408B3F  |.  8BD1          MOV EDX,ECX  
00408B41  |.  C1E9 02       SHR ECX,2                        ；左移 2 位  
00408B44  |.  8BF3          MOV ESI,EBX  
00408B46  |.  8D7C24 54     LEA EDI,DWORD PTR SS:\[ESP+54\]      
00408B4A  |.  F3:A5         REP MOVS DWORD PTR ES:\[EDI\],DWORD PTR DS>  
00408B4C  |.  8BCA          MOV ECX,EDX  
00408B4E  |.  83E1 03       AND ECX,3  
00408B51  |.  F3:A4         REP MOVS BYTE PTR ES:\[EDI\],BYTE PTR DS:\[>  
00408B53  |.  8B35 4C924300 MOV ESI,DWORD PTR DS:\[<&KERNEL32.GetTick>;  kernel32.GetTickCount  
00408B59  |.  FFD6          CALL ESI                                 ; 取系统运行时间，返回值是毫秒  
00408B5B  |.  50            PUSH EAX  
00408B5C  |.  E8 48FA0000   CALL xxxxx.004185A9                      ；保存时间  
00408B61  |.  83C4 04       ADD ESP,4  
00408B64  |>  E8 4DFA0000   /CALL xxxxx.004185B6                     ；对系统运行时间进行运算得到数 A  
00408B69  |.  85C0          |TEST EAX,EAX  
00408B6B  |.  894424 70     |MOV DWORD PTR SS:\[ESP+70\],EAX           ；保存数 A  
00408B6F  |.^ 74 F3         \\JE SHORT xxxxx.00408B64  
00408B71  |.  8B9424 880000>MOV EDX,DWORD PTR SS:\[ESP+88\]            ；取密码  
00408B78  |.  8BBC24 9C0000>MOV EDI,DWORD PTR SS:\[ESP+9C\]            ；EDI 清 0  
00408B7F  |.  33C0          XOR EAX,EAX                              ；EAX 清 0  
00408B81  |.  894424 10     MOV DWORD PTR SS:\[ESP+10\],EAX            
00408B85  |.  894424 14     MOV DWORD PTR SS:\[ESP+14\],EAX              
00408B89  |.  8D4C24 10     LEA ECX,DWORD PTR SS:\[ESP+10\]  
00408B8D  |.  894424 18     MOV DWORD PTR SS:\[ESP+18\],EAX  
00408B91  |.  51            PUSH ECX  
00408B92  |.  894424 20     MOV DWORD PTR SS:\[ESP+20\],EAX  
00408B96  |.  52            PUSH EDX  
00408B97  |.  89BC24 800000>MOV DWORD PTR SS:\[ESP+80\],EDI  
00408B9E  |.  884424 28     MOV BYTE PTR SS:\[ESP+28\],AL  
00408BA2  |.  E8 99330000   CALL xxxxx.0040BF40  
00408BA7  |.  8B4424 18     MOV EAX,DWORD PTR SS:\[ESP+18\]  
00408BAB  |.  8B4C24 1C     MOV ECX,DWORD PTR SS:\[ESP+1C\]  
00408BAF  |.  8B5424 20     MOV EDX,DWORD PTR SS:\[ESP+20\]  
00408BB3  |.  894424 68     MOV DWORD PTR SS:\[ESP+68\],EAX  
00408BB7  |.  8B4424 24     MOV EAX,DWORD PTR SS:\[ESP+24\]  
00408BBB  |.  83C4 08       ADD ESP,8  
00408BBE  |.  894C24 64     MOV DWORD PTR SS:\[ESP+64\],ECX  
00408BC2  |.  895424 68     MOV DWORD PTR SS:\[ESP+68\],EDX  
00408BC6  |.  894424 6C     MOV DWORD PTR SS:\[ESP+6C\],EAX  
00408BCA  |.  FFD6          CALL ESI                              ；取系统运行时间  
00408BCC  |.  50            PUSH EAX                             
00408BCD  |.  E8 D7F90000   CALL xxxxx.004185A9                     
00408BD2  |.  83C4 04       ADD ESP,4  
00408BD5  |>  E8 DCF90000   /CALL xxxxx.004185B6                   ；对取得的系统运行时间运算得一值记为 B  
00408BDA  |.  85C0          |TEST EAX,EAX  
00408BDC  |.  894424 74     |MOV DWORD PTR SS:\[ESP+74\],EAX  
00408BE0  |.^ 74 F3         \\JE SHORT xxxxx.00408BD5  
00408BE2  |.  33F6          XOR ESI,ESI  
00408BE4  |.  56            PUSH ESI  
00408BE5  |.  6A 18         PUSH 18  
00408BE7  |.  8D4C24 2C     LEA ECX,DWORD PTR SS:\[ESP+2C\]  
00408BEB  |.  51            PUSH ECX  
00408BEC  |.  6A 30         PUSH 30  
00408BEE  |.  8D5424 5C     LEA EDX,DWORD PTR SS:\[ESP+5C\]  
00408BF2  |.  52            PUSH EDX  
00408BF3  |.  8BCD          MOV ECX,EBP  
00408BF5  |.  E8 86FCFFFF   CALL xxxxx.00408880                   ；这里跟进进行网络验证数据的流通  
00408BFA  |.  85C0          TEST EAX,EAX  
00408BFC  |.  74 1B         JE SHORT xxxxx.00408C19  
00408BFE  |>  8975 04       MOV DWORD PTR SS:\[EBP+4\],ESI  
00408C01  |>  5F            POP EDI  
00408C02  |.  5E            POP ESI  
00408C03  |.  5D            POP EBP  
00408C04  |.  B8 F9FFFFFF   MOV EAX,-7  
00408C09  |.  5B            POP EBX  
00408C0A  |.  8B4C24 6C     MOV ECX,DWORD PTR SS:\[ESP+6C\]  
00408C0E  |.  E8 96DF0000   CALL xxxxx.00416BA9  
00408C13  |.  83C4 70       ADD ESP,70  
00408C16  |.  C2 1C00       RETN 1C  
  
00408880  /$  51            PUSH ECX                                 ;  xxxxx.00449A80  
00408881  |.  53            PUSH EBX  
00408882  |.  8B5C24 10     MOV EBX,DWORD PTR SS:\[ESP+10\]  
00408886  |.  55            PUSH EBP  
00408887  |.  8BE9          MOV EBP,ECX  
00408889  |.  8D4B 03       LEA ECX,DWORD PTR DS:\[EBX+3\]  
0040888C  |.  B8 56555555   MOV EAX,55555556  
00408891  |.  F7E9          IMUL ECX  
00408893  |.  56            PUSH ESI  
00408894  |.  8BF2          MOV ESI,EDX  
00408896  |.  C1EE 1F       SHR ESI,1F  
00408899  |.  03F2          ADD ESI,EDX  
0040889B  |.  57            PUSH EDI  
0040889C  |.  C1E6 02       SHL ESI,2  
0040889F  |.  56            PUSH ESI  
004088A0  |.  896C24 14     MOV DWORD PTR SS:\[ESP+14\],EBP  
004088A4  |.  E8 400D0200   CALL xxxxx.004295E9  
004088A9  |.  8B4C24 1C     MOV ECX,DWORD PTR SS:\[ESP+1C\]  
004088AD  |.  8BF8          MOV EDI,EAX  
004088AF  |.  8D46 FF       LEA EAX,DWORD PTR DS:\[ESI-1\]  
004088B2  |.  50            PUSH EAX  
004088B3  |.  53            PUSH EBX  
004088B4  |.  57            PUSH EDI  
004088B5  |.  51            PUSH ECX  
004088B6  |.  897C24 30     MOV DWORD PTR SS:\[ESP+30\],EDI  
004088BA  |.  E8 C1120000   CALL xxxxx.00409B80                       ；这里跟进进行用户名密码等数据加密准备传输  
004088BF  |.  83C4 14       ADD ESP,14  
004088C2  |.  6A 00         PUSH 0                                   ; /Protocol = IPPROTO\_IP  
004088C4  |.  6A 01         PUSH 1                                   ; |Type = SOCK\_STREAM  
004088C6  |.  6A 02         PUSH 2                                   ; |Family = AF\_INET  
跟进 4088BA 的 CALLL 到这里  
00409B80  /$  51            PUSH ECX  
00409B81  |.  8B4C24 10     MOV ECX,DWORD PTR SS:\[ESP+10\]  
00409B85  |.  56            PUSH ESI  
00409B86  |.  57            PUSH EDI  
00409B87  |.  33FF          XOR EDI,EDI  
00409B89  |.  33F6          XOR ESI,ESI  
00409B8B  |.  32C0          XOR AL,AL  
00409B8D  |.  3BCF          CMP ECX,EDI  
00409B8F  |.  897C24 08     MOV DWORD PTR SS:\[ESP+8\],EDI  
00409B93  |.  0F8E 86000000 JLE xxxxx.00409C1F  
00409B99  |.  55            PUSH EBP  
00409B9A  |.  8B6C24 18     MOV EBP,DWORD PTR SS:\[ESP+18\]  
00409B9E  |.  53            PUSH EBX  
00409B9F  |.  90            NOP  
00409BA0  |>  3B7424 24     /CMP ESI,DWORD PTR SS:\[ESP+24\]  
00409BA4  |.  7D 63         |JGE SHORT xxxxx.00409C09  
00409BA6  |.  8B4C24 18     |MOV ECX,DWORD PTR SS:\[ESP+18\]  
00409BAA  |.  8B5424 10     |MOV EDX,DWORD PTR SS:\[ESP+10\]  
00409BAE  |.  8A140A        |MOV DL,BYTE PTR DS:\[EDX+ECX\]              
；取用户名字等的字符一个一个取，其中开始取的次序为版本号，帐户，密码，帐户和密码取 12 位不够的补的是 0，然后取系统  
运行时间分别加密的后的数字 A，数字 B    
00409BB1  |.  8D5F 02       |LEA EBX,DWORD PTR DS:\[EDI+2\]  
00409BB4  |.  885424 1C     |MOV BYTE PTR SS:\[ESP+1C\],DL  
00409BB8  |.  8ACB          |MOV CL,BL  
00409BBA  |.  D2EA          |SHR DL,CL  
00409BBC  |.  B9 06000000   |MOV ECX,6  
00409BC1  |.  2BCF          |SUB ECX,EDI  
00409BC3  |.  8BFB          |MOV EDI,EBX  
00409BC5  |.  0AD0          |OR DL,AL  
00409BC7  |.  0FB64424 1C   |MOVZX EAX,BYTE PTR SS:\[ESP+1C\]  
00409BCC  |.  D3E0          |SHL EAX,CL  
00409BCE  |.  80E2 3F       |AND DL,3F  
00409BD1  |.  80C2 3C       |ADD DL,3C  
00409BD4  |.  88142E        |MOV BYTE PTR DS:\[ESI+EBP\],DL  
00409BD7  |.  C1F8 02       |SAR EAX,2  
00409BDA  |.  24 3F         |AND AL,3F  
00409BDC  |.  83FF 06       |CMP EDI,6  
00409BDF  |.  7D 03         |JGE SHORT xxxxx.00409BE4  
00409BE1  |.  46            |INC ESI  
00409BE2  |.  EB 14         |JMP SHORT xxxxx.00409BF8  
00409BE4  |>  8B4C24 24     |MOV ECX,DWORD PTR SS:\[ESP+24\]  
00409BE8  |.  49            |DEC ECX  
00409BE9  |.  3BF1          |CMP ESI,ECX  
00409BEB  |.  7D 06         |JGE SHORT xxxxx.00409BF3  
00409BED  |.  46            |INC ESI  
00409BEE  |.  04 3C         |ADD AL,3C  
00409BF0  |.  88042E        |MOV BYTE PTR DS:\[ESI+EBP\],AL  
00409BF3  |>  46            |INC ESI  
00409BF4  |.  33FF          |XOR EDI,EDI  
00409BF6  |.  32C0          |XOR AL,AL  
00409BF8  |>  8B4C24 10     |MOV ECX,DWORD PTR SS:\[ESP+10\]  
00409BFC  |.  8B5424 20     |MOV EDX,DWORD PTR SS:\[ESP+20\]  
00409C00  |.  41            |INC ECX  
00409C01  |.  3BCA          |CMP ECX,EDX  
00409C03  |.  894C24 10     |MOV DWORD PTR SS:\[ESP+10\],ECX  
00409C07  |.^ 7C 97         \\JL SHORT xxxxx.00409BA0               ;上面一共进行 64 次运行得到 64 位加密字符串  
00409C09  |>  85FF          TEST EDI,EDI  
00409C0B  |.  5B            POP EBX  
00409C0C  |.  7E 06         JLE SHORT xxxxx.00409C14  
00409C0E  |.  04 3C         ADD AL,3C  
00409C10  |.  88042E        MOV BYTE PTR DS:\[ESI+EBP\],AL  
00409C13  |.  46            INC ESI  
00409C14  |>  C6042E 00     MOV BYTE PTR DS:\[ESI+EBP\],0  
00409C18  |.  5D            POP EBP  
00409C19  |.  5F            POP EDI  
00409C1A  |.  8BC6          MOV EAX,ESI  
00409C1C  |.  5E            POP ESI  
00409C1D  |.  59            POP ECX  
00409C1E  |.  C3            RETN                                 ；返回调用 CALL 继续 F8  
  
004088C8  |.  FF15 9C954300 CALL DWORD PTR DS:\[<&WS2\_32.#23>\]        ; \\socket  
004088CE  |.  8BD8          MOV EBX,EAX  
004088D0  |.  83FB FF       CMP EBX,-1  
004088D3  |.  75 1D         JNZ SHORT xxxxx.004088F2  
004088D5  |.  57            PUSH EDI  
004088D6  |.  E8 090D0200   CALL xxxxx.004295E4                       
004088DB  |.  83C4 04       ADD ESP,4  
004088DE  |.  53            PUSH EBX                                 ; /Socket  
004088DF  |.  FF15 A0954300 CALL DWORD PTR DS:\[<&WS2\_32.#3>\]         ; \\closesocket  
004088E5  |.  5F            POP EDI  
004088E6  |.  5E            POP ESI  
004088E7  |.  5D            POP EBP  
004088E8  |.  B8 F9FFFFFF   MOV EAX,-7  
004088ED  |.  5B            POP EBX  
004088EE  |.  59            POP ECX  
004088EF  |.  C2 1400       RETN 14  
004088F2  |>  837C24 28 01  CMP DWORD PTR SS:\[ESP+28\],1  
004088F7  |.  75 0B         JNZ SHORT xxxxx.00408904  
004088F9  |.  8B55 0C       MOV EDX,DWORD PTR SS:\[EBP+C\]  
004088FC  |.  52            PUSH EDX  
004088FD  |.  68 08A34300   PUSH xxxxx.0043A308                      ;  66.36.243.164  
00408902  |.  EB 08         JMP SHORT xxxxx.0040890C  
00408904  |>  8B45 0C       MOV EAX,DWORD PTR SS:\[EBP+C\]  
00408907  |.  50            PUSH EAX  
00408908  |.  83C5 3A       ADD EBP,3A  
0040890B  |.  55            PUSH EBP  
0040890C  |>  53            PUSH EBX  
0040890D  |.  E8 7E420000   CALL xxxxx.0040CB90                      ；这个 CALL 进行网络连接连接上返回一数值到 EAX 中  
00408912  |.  83C4 0C       ADD ESP,0C  
00408915  |.  83F8 FF       CMP EAX,-1                               ；只要不等于-1 即 FFFFFFFF 就发送数据  
00408918  |.  74 0F         JE SHORT xxxxx.00408929  
0040891A  |.  6A 00         PUSH 0                                   ; /Flags = 0  
0040891C  |.  56            PUSH ESI                                 ; |DataSize  
0040891D  |.  57            PUSH EDI                                 ; |Data  
0040891E  |.  53            PUSH EBX                                 ; |Socket  
0040891F  |.  FF15 A4954300 CALL DWORD PTR DS:\[<&WS2\_32.#19>\]        ; \\send  
00408925  |.  3BC6          CMP EAX,ESI  
00408927  |.  74 1D         JE SHORT xxxxx.00408946  
00408929  |>  57            PUSH EDI  
0040892A  |.  E8 B50C0200   CALL xxxxx.004295E4  
0040892F  |.  83C4 04       ADD ESP,4  
00408932  |.  53            PUSH EBX                                 ; /Socket  
00408933  |.  FF15 A0954300 CALL DWORD PTR DS:\[<&WS2\_32.#3>\]         ; \\closesocket  
00408939  |.  5F            POP EDI  
0040893A  |.  5E            POP ESI  
0040893B  |.  5D            POP EBP  
0040893C  |.  B8 F9FFFFFF   MOV EAX,-7  
00408941  |.  5B            POP EBX  
00408942  |.  59            POP ECX  
00408943  |.  C2 1400       RETN 14  
00408946  |>  8B4424 24     MOV EAX,DWORD PTR SS:\[ESP+24\]  
0040894A  |.  85C0          TEST EAX,EAX  
0040894C  |.  75 1A         JNZ SHORT xxxxx.00408968  
0040894E  |.  57            PUSH EDI  
0040894F  |.  E8 900C0200   CALL xxxxx.004295E4  
00408954  |.  83C4 04       ADD ESP,4  
00408957  |.  53            PUSH EBX                                 ; /Socket  
00408958  |.  FF15 A0954300 CALL DWORD PTR DS:\[<&WS2\_32.#3>\]         ; \\closesocket  
0040895E  |.  5F            POP EDI  
0040895F  |.  5E            POP ESI  
00408960  |.  5D            POP EBP  
00408961  |.  33C0          XOR EAX,EAX  
00408963  |.  5B            POP EBX  
00408964  |.  59            POP ECX  
00408965  |.  C2 1400       RETN 14  
  
  
00408946  |> \\8B4424 24     MOV EAX,DWORD PTR SS:\[ESP+24\]  
0040894A  |.  85C0          TEST EAX,EAX  
0040894C  |.  75 1A         JNZ SHORT xxxxx.00408968  
0040894E  |.  57            PUSH EDI  
0040894F  |.  E8 900C0200   CALL xxxxx.004295E4  
00408954  |.  83C4 04       ADD ESP,4  
00408957  |.  53            PUSH EBX                                 ; /Socket  
00408958  |.  FF15 A0954300 CALL DWORD PTR DS:\[<&WS2\_32.#3>\]         ; \\closesocket  
0040895E  |.  5F            POP EDI  
0040895F  |.  5E            POP ESI  
00408960  |.  5D            POP EBP  
00408961  |.  33C0          XOR EAX,EAX  
00408963  |.  5B            POP EBX  
00408964  |.  59            POP ECX  
00408965  |.  C2 1400       RETN 14  
00408968  |>  8D48 03       LEA ECX,DWORD PTR DS:\[EAX+3\]  
0040896B  |.  B8 56555555   MOV EAX,55555556  
00408970  |.  F7E9          IMUL ECX  
00408972  |.  8BF2          MOV ESI,EDX  
00408974  |.  C1EE 1F       SHR ESI,1F  
00408977  |.  03F2          ADD ESI,EDX  
00408979  |.  C1E6 02       SHL ESI,2  
0040897C  |.  56            PUSH ESI  
0040897D  |.  E8 670C0200   CALL xxxxx.004295E9  
00408982  |.  8BE8          MOV EBP,EAX  
00408984  |.  8BCE          MOV ECX,ESI  
00408986  |.  8BD1          MOV EDX,ECX  
00408988  |.  C1E9 02       SHR ECX,2  
0040898B  |.  33C0          XOR EAX,EAX  
0040898D  |.  8BFD          MOV EDI,EBP  
0040898F  |.  F3:AB         REP STOS DWORD PTR ES:\[EDI\]  
00408991  |.  8BCA          MOV ECX,EDX  
00408993  |.  83E1 03       AND ECX,3  
00408996  |.  F3:AA         REP STOS BYTE PTR ES:\[EDI\]  
00408998  |.  8B4424 14     MOV EAX,DWORD PTR SS:\[ESP+14\]  
0040899C  |.  8B48 14       MOV ECX,DWORD PTR DS:\[EAX+14\]  
0040899F  |.  51            PUSH ECX  
004089A0  |.  56            PUSH ESI  
004089A1  |.  55            PUSH EBP  
004089A2  |.  53            PUSH EBX  
004089A3  |.  E8 68410000   CALL xxxxx.0040CB10  
004089A8  |.  83C4 14       ADD ESP,14  
004089AB  |.  3BC6          CMP EAX,ESI  
004089AD  |.  74 27         JE SHORT xxxxx.004089D6  
004089AF  |.  8B5424 1C     MOV EDX,DWORD PTR SS:\[ESP+1C\]  
004089B3  |.  52            PUSH EDX  
004089B4  |.  E8 2B0C0200   CALL xxxxx.004295E4  
004089B9  |.  55            PUSH EBP  
004089BA  |.  E8 250C0200   CALL xxxxx.004295E4  
004089BF  |.  83C4 08       ADD ESP,8  
004089C2  |.  53            PUSH EBX                                 ; /Socket  
004089C3  |.  FF15 A0954300 CALL DWORD PTR DS:\[<&WS2\_32.#3>\]         ; \\closesocket  
004089C9  |.  5F            POP EDI  
004089CA  |.  5E            POP ESI  
004089CB  |.  5D            POP EBP  
004089CC  |.  B8 F9FFFFFF   MOV EAX,-7  
004089D1  |.  5B            POP EBX  
004089D2  |.  59            POP ECX  
004089D3  |.  C2 1400       RETN 14  
004089D6  |>  8B4424 24     MOV EAX,DWORD PTR SS:\[ESP+24\]  
004089DA  |.  8B4C24 20     MOV ECX,DWORD PTR SS:\[ESP+20\]  
004089DE  |.  50            PUSH EAX  
004089DF  |.  56            PUSH ESI  
004089E0  |.  51            PUSH ECX  
004089E1  |.  55            PUSH EBP  
004089E2  |.  E8 49120000   CALL xxxxx.00409C30                      ；这里跟进是进行返回数据初解密的  
004089E7  |.  83C4 10       ADD ESP,10  
004089EA  |.  53            PUSH EBX                                 ; /Socket  
004089EB  |.  FF15 A0954300 CALL DWORD PTR DS:\[<&WS2\_32.#3>\]         ; \\closesocket  
004089F1  |.  8B5424 1C     MOV EDX,DWORD PTR SS:\[ESP+1C\]  
004089F5  |.  52            PUSH EDX  
004089F6  |.  E8 E90B0200   CALL xxxxx.004295E4  
004089FB  |.  55            PUSH EBP  
004089FC  |.  E8 E30B0200   CALL xxxxx.004295E4  
00408A01  |.  83C4 08       ADD ESP,8  
00408A04  |.  5F            POP EDI  
00408A05  |.  5E            POP ESI  
00408A06  |.  5D            POP EBP  
00408A07  |.  33C0          XOR EAX,EAX  
00408A09  |.  5B            POP EBX  
00408A0A  |.  59            POP ECX  
00408A0B  \\.  C2 1400       RETN 14  
4089E2 根进 CALL 后的数据  
00409C30  /$  8B5424 0C     MOV EDX,DWORD PTR SS:\[ESP+C\]  
00409C34  |.  57            PUSH EDI  
00409C35  |.  8BCA          MOV ECX,EDX  
00409C37  |.  33FF          XOR EDI,EDI  
00409C39  |.  33C0          XOR EAX,EAX  
00409C3B  |.  81E1 03000080 AND ECX,80000003  
00409C41  |.  79 05         JNS SHORT xxxxx.00409C48  
00409C43  |.  49            DEC ECX  
00409C44  |.  83C9 FC       OR ECX,FFFFFFFC  
00409C47  |.  41            INC ECX  
00409C48  |>  83F9 01       CMP ECX,1  
00409C4B  |.  75 05         JNZ SHORT xxxxx.00409C52              ；这里必须跳去解密字符串  
00409C4D  |.  83C8 FF       OR EAX,FFFFFFFF  
00409C50  |.  5F            POP EDI  
00409C51  |.  C3            RETN  
00409C52  |>  85D2          TEST EDX,EDX  
00409C54  |.  56            PUSH ESI  
00409C55  |.  8B7424 0C     MOV ESI,DWORD PTR SS:\[ESP+C\]  
00409C59  |.  7E 0E         JLE SHORT xxxxx.00409C69  
00409C5B  |.  EB 03         JMP SHORT xxxxx.00409C60  
00409C5D  |   8D49 00       LEA ECX,DWORD PTR DS:\[ECX\]  
00409C60  |>  800430 C4     /ADD BYTE PTR DS:\[EAX+ESI\],0C4  
00409C64  |.  40            |INC EAX  
00409C65  |.  3BC2          |CMP EAX,EDX  
00409C67  |.^ 7C F7         \\JL SHORT xxxxx.00409C60  
00409C69  |>  33C9          XOR ECX,ECX  
00409C6B  |.  85D2          TEST EDX,EDX  
00409C6D  |.  7E 72         JLE SHORT xxxxx.00409CE1  
00409C6F  |.  55            PUSH EBP  
00409C70  |.  8B6C24 14     MOV EBP,DWORD PTR SS:\[ESP+14\]  
00409C74  |.  53            PUSH EBX  
00409C75  |>  3B7C24 20     /CMP EDI,DWORD PTR SS:\[ESP+20\]  
00409C79  |.  7D 64         |JGE SHORT xxxxx.00409CDF  
00409C7B  |.  8BC1          |MOV EAX,ECX  
00409C7D  |.  25 03000080   |AND EAX,80000003  
00409C82  |.  79 05         |JNS SHORT xxxxx.00409C89  
00409C84  |.  48            |DEC EAX  
00409C85  |.  83C8 FC       |OR EAX,FFFFFFFC  
00409C88  |.  40            |INC EAX  
00409C89  |>  83E8 00       |SUB EAX,0                               ;  Switch (cases 0..2)  
00409C8C  |.  74 35         |JE SHORT xxxxx.00409CC3  
00409C8E  |.  48            |DEC EAX  
00409C8F  |.  74 1F         |JE SHORT xxxxx.00409CB0  
00409C91  |.  48            |DEC EAX  
00409C92  |.  75 47         |JNZ SHORT xxxxx.00409CDB  
00409C94  |.  0FB64431 01   |MOVZX EAX,BYTE PTR DS:\[ECX+ESI+1\]       ;  Case 2 of switch 00409C89  
00409C99  |.  8A1C31        |MOV BL,BYTE PTR DS:\[ECX+ESI\]  
00409C9C  |.  C1E0 02       |SHL EAX,2  
00409C9F  |.  C1F8 02       |SAR EAX,2  
00409CA2  |.  C0E3 06       |SHL BL,6  
00409CA5  |.  0AC3          |OR AL,BL  
00409CA7  |.  88042F        |MOV BYTE PTR DS:\[EDI+EBP\],AL  
00409CAA  |.  47            |INC EDI  
00409CAB  |.  83C1 02       |ADD ECX,2  
00409CAE  |.  EB 2B         |JMP SHORT xxxxx.00409CDB  
00409CB0  |>  0FB64431 01   |MOVZX EAX,BYTE PTR DS:\[ECX+ESI+1\]       ;  Case 1 of switch 00409C89  
00409CB5  |.  8A1C31        |MOV BL,BYTE PTR DS:\[ECX+ESI\]  
00409CB8  |.  C1E0 02       |SHL EAX,2  
00409CBB  |.  C1F8 04       |SAR EAX,4  
00409CBE  |.  C0E3 04       |SHL BL,4  
00409CC1  |.  EB 11         |JMP SHORT xxxxx.00409CD4  
00409CC3  |>  0FB64431 01   |MOVZX EAX,BYTE PTR DS:\[ECX+ESI+1\]       ;  Case 0 of switch 00409C89  
00409CC8  |.  8A1C31        |MOV BL,BYTE PTR DS:\[ECX+ESI\]  
00409CCB  |.  C1E0 02       |SHL EAX,2  
00409CCE  |.  C1F8 06       |SAR EAX,6  
00409CD1  |.  C0E3 02       |SHL BL,2  
00409CD4  |>  0AC3          |OR AL,BL  
00409CD6  |.  88042F        |MOV BYTE PTR DS:\[EDI+EBP\],AL  
00409CD9  |.  47            |INC EDI  
00409CDA  |.  41            |INC ECX  
00409CDB  |>  3BCA          |CMP ECX,EDX                             ;  Default case of switch 00409C89  
00409CDD  |.^ 7C 96         \\JL SHORT xxxxx.00409C75  
00409CDF  |>  5B            POP EBX  
00409CE0  |.  5D            POP EBP  
00409CE1  |>  5E            POP ESI  
00409CE2  |.  8BC7          MOV EAX,EDI  
00409CE4  |.  5F            POP EDI  
00409CE5  \\.  C3            RETN                                     ；这里返回，一直 F8 到返回到 408BFA  
  
数据通信返回后从 408BFC 跳到这里  
  
00408C19  |>  66:817C24 24 >CMP WORD PTR SS:\[ESP+24\],201  
00408C20  |.^ 75 DC         JNZ SHORT xxxxx.00408BFE                 ；跳转不然网络不通 爆破点 2  
00408C22  |.  8B4424 26     MOV EAX,DWORD PTR SS:\[ESP+26\]  
00408C26  |.  8B55 0C       MOV EDX,DWORD PTR SS:\[EBP+C\]  
00408C29  |.  8B4C24 2A     MOV ECX,DWORD PTR SS:\[ESP+2A\]  
00408C2D  |.  894424 3C     MOV DWORD PTR SS:\[ESP+3C\],EAX  
00408C31  |.  3B15 1CA34300 CMP EDX,DWORD PTR DS:\[43A31C\]  
00408C37  |.  894C24 40     MOV DWORD PTR SS:\[ESP+40\],ECX  
00408C3B  |.  75 11         JNZ SHORT xxxxx.00408C4E  
00408C3D  |.  8D4424 44     LEA EAX,DWORD PTR SS:\[ESP+44\]  
00408C41  |.  50            PUSH EAX  
00408C42  |.  68 70A64300   PUSH xxxxx.0043A670                      ;  unlimitedhero  
00408C47  |.  8D4C24 44     LEA ECX,DWORD PTR SS:\[ESP+44\]  
00408C4B  |.  51            PUSH ECX  
00408C4C  |.  EB 0F         JMP SHORT xxxxx.00408C5D  
00408C4E  |>  8D5424 44     LEA EDX,DWORD PTR SS:\[ESP+44\]  
00408C52  |.  52            PUSH EDX  
00408C53  |.  68 60A64300   PUSH xxxxx.0043A660                      ;  herobillforreal  
00408C58  |.  8D4424 44     LEA EAX,DWORD PTR SS:\[ESP+44\]  
00408C5C  |.  50            PUSH EAX  
00408C5D  |>  E8 7E330000   CALL xxxxx.0040BFE0                      ；进行返回数据的解密看帐户还有没有余额和帐户信息是否正确  
00408C62  |.  8B8C24 980000>MOV ECX,DWORD PTR SS:\[ESP+98\]  
00408C69  |.  8B7424 3C     MOV ESI,DWORD PTR SS:\[ESP+3C\]  
00408C6D  |.  8B5424 50     MOV EDX,DWORD PTR SS:\[ESP+50\]  
00408C71  |.  8B4424 54     MOV EAX,DWORD PTR SS:\[ESP+54\]  
00408C75  |.  8931          MOV DWORD PTR DS:\[ECX\],ESI  
00408C77  |.  8B7424 40     MOV ESI,DWORD PTR SS:\[ESP+40\]  
00408C7B  |.  8B8C24 9C0000>MOV ECX,DWORD PTR SS:\[ESP+9C\]  
00408C82  |.  8931          MOV DWORD PTR DS:\[ECX\],ESI  
00408C84  |.  8BB424 800000>MOV ESI,DWORD PTR SS:\[ESP+80\]            ；这里把前面的系统运行时间计算后的值传给 ESI  
00408C8B  |.  83C4 0C       ADD ESP,0C  
00408C8E  |.  3BD6          CMP EDX,ESI                              ；比较解密后的值，看是不是截取的封包骗取的客户端  
00408C90  |.  895424 26     MOV DWORD PTR SS:\[ESP+26\],EDX  
00408C94  |.  894424 2A     MOV DWORD PTR SS:\[ESP+2A\],EAX  
00408C98  |.  74 0C         JE SHORT xxxxx.00408CA6                  ；出现不等于就说明外挂被破解，有人用模拟信息骗取程序的登陆  
这里我们也可以看到程序的帐户信息加密和系统运行时间有关从而进行了动态加密，只要我们把系统返回的运行时间就可以把加密信息固定，那么  
返回的信息就可以不会被客户端认为是非法用户了，同时就可以进行截取数据进行本地验证了。  
00408C9A  |.  C745 04 00000>MOV DWORD PTR SS:\[EBP+4\],0  
00408CA1  |.^ E9 5BFFFFFF   JMP xxxxx.00408C01  
00408CA6  |>  3D 57040000   CMP EAX,457  
00408CAB  |.  0F85 9B000000 JNZ xxxxx.00408D4C                       ；比较 EAX 是不是等于 457 不等于则帐户没余额或者帐户信息错误 ，爆破点                        
00408CB1  |.  33D2          XOR EDX,EDX  
00408CB3  |.  85FF          TEST EDI,EDI  
00408CB5  |.  0F95C2        SETNE DL  
00408CB8  |.  8955 04       MOV DWORD PTR SS:\[EBP+4\],EDX  
00408CBB  |.  8B01          MOV EAX,DWORD PTR DS:\[ECX\]  
00408CBD  |.  8D55 20       LEA EDX,DWORD PTR SS:\[EBP+20\]  
00408CC0  |.  8945 08       MOV DWORD PTR SS:\[EBP+8\],EAX  
00408CC3  |.  8BC3          MOV EAX,EBX  
00408CC5  |.  2BD3          SUB EDX,EBX  
00408CC7  |>  8A08          /MOV CL,BYTE PTR DS:\[EAX\]  
00408CC9  |.  880C02        |MOV BYTE PTR DS:\[EDX+EAX\],CL  
00408CCC  |.  40            |INC EAX  
00408CCD  |.  84C9          |TEST CL,CL  
00408CCF  |.^ 75 F6         \\JNZ SHORT xxxxx.00408CC7  
00408CD1  |.  8B8424 880000>MOV EAX,DWORD PTR SS:\[ESP+88\]  
00408CD8  |.  8D55 2D       LEA EDX,DWORD PTR SS:\[EBP+2D\]  
00408CDB  |.  2BD0          SUB EDX,EAX  
00408CDD  |.  8D49 00       LEA ECX,DWORD PTR DS:\[ECX\]  
00408CE0  |>  8A08          /MOV CL,BYTE PTR DS:\[EAX\]  
00408CE2  |.  880C02        |MOV BYTE PTR DS:\[EDX+EAX\],CL  
00408CE5  |.  40            |INC EAX  
00408CE6  |.  84C9          |TEST CL,CL  
00408CE8  |.^ 75 F6         \\JNZ SHORT xxxxx.00408CE0  
00408CEA  |.  8B5424 24     MOV EDX,DWORD PTR SS:\[ESP+24\]  
00408CEE  |.  8B4424 28     MOV EAX,DWORD PTR SS:\[ESP+28\]  
00408CF2  |.  8D7D 50       LEA EDI,DWORD PTR SS:\[EBP+50\]  
00408CF5  |.  B9 0C000000   MOV ECX,0C  
00408CFA  |.  8D7424 4C     LEA ESI,DWORD PTR SS:\[ESP+4C\]  
00408CFE  |.  F3:A5         REP MOVS DWORD PTR ES:\[EDI\],DWORD PTR DS>  
00408D00  |.  8D8D 80000000 LEA ECX,DWORD PTR SS:\[EBP+80\]  
00408D06  |.  8911          MOV DWORD PTR DS:\[ECX\],EDX  
00408D08  |.  8B5424 2C     MOV EDX,DWORD PTR SS:\[ESP+2C\]  
00408D0C  |.  8941 04       MOV DWORD PTR DS:\[ECX+4\],EAX  
00408D0F  |.  8B4424 30     MOV EAX,DWORD PTR SS:\[ESP+30\]  
00408D13  |.  8951 08       MOV DWORD PTR DS:\[ECX+8\],EDX  
00408D16  |.  8B5424 34     MOV EDX,DWORD PTR SS:\[ESP+34\]  
00408D1A  |.  8941 0C       MOV DWORD PTR DS:\[ECX+C\],EAX  
00408D1D  |.  8B4424 38     MOV EAX,DWORD PTR SS:\[ESP+38\]  
00408D21  |.  8951 10       MOV DWORD PTR DS:\[ECX+10\],EDX  
00408D24  |.  8B9424 980000>MOV EDX,DWORD PTR SS:\[ESP+98\]  
00408D2B  |.  8941 14       MOV DWORD PTR DS:\[ECX+14\],EAX  
00408D2E  |.  8B4C24 70     MOV ECX,DWORD PTR SS:\[ESP+70\]  
00408D32  |.  5F            POP EDI  
00408D33  |.  5E            POP ESI  
00408D34  |.  894D 1C       MOV DWORD PTR SS:\[EBP+1C\],ECX  
00408D37  |.  5D            POP EBP  
00408D38  |.  8902          MOV DWORD PTR DS:\[EDX\],EAX  
00408D3A  |.  33C0          XOR EAX,EAX  
00408D3C  |.  5B            POP EBX  
00408D3D  |.  8B4C24 6C     MOV ECX,DWORD PTR SS:\[ESP+6C\]  
00408D41  |.  E8 63DE0000   CALL xxxxx.00416BA9  
00408D46  |.  83C4 70       ADD ESP,70  
00408D49  |.  C2 1C00       RETN 1C  
00408D4C  |>  3D 050D0000   CMP EAX,0D05  
00408D51  |.  C745 04 00000>MOV DWORD PTR SS:\[EBP+4\],0  
00408D58  |.  75 25         JNZ SHORT xxxxx.00408D7F  
00408D5A  |.  8B8424 940000>MOV EAX,DWORD PTR SS:\[ESP+94\]  
00408D61  |.  5F            POP EDI  
00408D62  |.  5E            POP ESI  
00408D63  |.  5D            POP EBP  
00408D64  |.  C700 050D0000 MOV DWORD PTR DS:\[EAX\],0D05  
00408D6A  |.  B8 FBFFFFFF   MOV EAX,-5  
00408D6F  |.  5B            POP EBX  
00408D70  |.  8B4C24 6C     MOV ECX,DWORD PTR SS:\[ESP+6C\]  
00408D74  |.  E8 30DE0000   CALL xxxxx.00416BA9  
00408D79  |.  83C4 70       ADD ESP,70  
00408D7C  |.  C2 1C00       RETN 1C  
00408D7F  |>  3D 611E0000   CMP EAX,1E61  
00408D84  |.  75 18         JNZ SHORT xxxxx.00408D9E  
00408D86  |.  5F            POP EDI  
00408D87  |.  5E            POP ESI  
00408D88  |.  5D            POP EBP  
00408D89  |.  B8 F8FFFFFF   MOV EAX,-8  
00408D8E  |.  5B            POP EBX  
00408D8F  |.  8B4C24 6C     MOV ECX,DWORD PTR SS:\[ESP+6C\]  
00408D93  |.  E8 11DE0000   CALL xxxxx.00416BA9  
00408D98  |.  83C4 70       ADD ESP,70  
00408D9B  |.  C2 1C00       RETN 1C  
00408D9E  |>  83F8 01       CMP EAX,1  
00408DA1  |.  75 21         JNZ SHORT xxxxx.00408DC4  
00408DA3  |.  8B8C24 940000>MOV ECX,DWORD PTR SS:\[ESP+94\]  
00408DAA  |.  5F            POP EDI  
00408DAB  |.  5E            POP ESI  
00408DAC  |.  5D            POP EBP  
00408DAD  |.  8901          MOV DWORD PTR DS:\[ECX\],EAX  
00408DAF  |.  B8 FBFFFFFF   MOV EAX,-5  
00408DB4  |.  5B            POP EBX  
00408DB5  |.  8B4C24 6C     MOV ECX,DWORD PTR SS:\[ESP+6C\]  
00408DB9  |.  E8 EBDD0000   CALL xxxxx.00416BA9  
00408DBE  |.  83C4 70       ADD ESP,70  
00408DC1  |.  C2 1C00       RETN 1C  
00408DC4  |>  83F8 08       CMP EAX,8  
00408DC7  |.^ 0F85 34FEFFFF JNZ xxxxx.00408C01  
00408DCD  |.  8B9424 940000>MOV EDX,DWORD PTR SS:\[ESP+94\]  
00408DD4  |.  5F            POP EDI  
00408DD5  |.  5E            POP ESI  
00408DD6  |.  5D            POP EBP  
00408DD7  |.  8902          MOV DWORD PTR DS:\[EDX\],EAX  
00408DD9  |.  B8 FBFFFFFF   MOV EAX,-5  
00408DDE  |.  5B            POP EBX  
00408DDF  |.  8B4C24 6C     MOV ECX,DWORD PTR SS:\[ESP+6C\]  
00408DE3  |.  E8 C1DD0000   CALL xxxxx.00416BA9  
00408DE8  |.  83C4 70       ADD ESP,70  
00408DEB  |.  C2 1C00       RETN 1C  
00408DEE  |>  8B4C24 78     MOV ECX,DWORD PTR SS:\[ESP+78\]  
00408DF2  |.  5E            POP ESI  
00408DF3  |.  5D            POP EBP  
00408DF4  |.  B8 FDFFFFFF   MOV EAX,-3  
00408DF9  |.  5B            POP EBX  
00408DFA  |.  E8 AADD0000   CALL xxxxx.00416BA9  
00408DFF  |.  83C4 70       ADD ESP,70  
00408E02  \\.  C2 1C00       RETN 1C  
  
总结：此软件的加密与用户名字密码，和系统运行时间有关，解密的时候和用户名、密码无关但是与系统运行时间有关。  
加密和解密算法，关键处我已经列出就不再描述了。  
  
文章写得乱，希望各位别丢砖头，其中有很多漏的地方请大家指正。  
\------------------------------------------------------------------------  
【版权声明】本文纯属技术交流, 转载请注明作者信息并保持文章的完整, 谢谢! 

 4dd466d0e1e0cc123afbe504da2462ca