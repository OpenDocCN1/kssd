

# 菜鸟初学 Crack 破解，学习 MD5 算法【原创】

> 原文：[`www.pediy.com/kssd/pediy08/pediy8-156.htm`](https://www.pediy.com/kssd/pediy08/pediy8-156.htm)

【破解软件】Arial CD Ripper 1.4.8   
【下载地址】http://www.onlinedown.net/soft/31096.htm  
【运行环境】Win9x/Me/NT/2000/XP  
【软件类别】国外软件/共享版/音频工具  
【保护方式】注册码   
【作者声明】初学 Crack，只是感兴趣，消遣业余时间，错误之处敬请诸位前辈不吝赐教。  
【调试环境】Winxp、OllyDBD、PEiD  
【软件信息】抓音轨和音频转换工具，能够把 CD 转换成 MP3，WAV，OGG，FLAC，APE 等文件格式，你可以在不损失质量的前提下只转换一  
  
　　　　　　条音轨或者转换整个光盘，软机同时具有在不同的音频格式之间互相转换的功能。  
【破解过程】学习《加密与解密》第二版第六章 6.1.1 的内容，分析这个软件，对照书本做练习。  
  
 PEiD 查壳：Borland Delphi 6.0 - 7.0　用插件 KANAL 查看是: BASE64 + MD5  
 OD 载入程序查找字串参考，找到：“register successfully!  thank you for your support!”  
双击来到:00573A3C 处，向上翻三十几行代码，在 005739DC 处下断，F9 运行时有三次提示：“　……入口点代码超出……”估计是在程  
  
序安装目录里有三个 Dll 文件加了壳的缘故，不过不影响找到注册码。在注册框里填用户名：wzwgp　注册码：12345678　点“OK”　  
    
005739DC  /.  55            PUSH EBP　　　　　　　　　　　　　　 ; 断在此  
005739DD  |.  8BEC          MOV EBP,ESP  
005739DF  |.  6A 00         PUSH 0  
005739E1  |.  6A 00         PUSH 0  
005739E3  |.  53            PUSH EBX  
005739E4  |.  8BD8          MOV EBX,EAX                            
005739E6  |.  33C0          XOR EAX,EAX  
005739E8  |.  55            PUSH EBP  
005739E9  |.  68 BF3A5700   PUSH Arial\_CD.00573ABF  
005739EE  |.  64:FF30       PUSH DWORD PTR FS:\[EAX\]  
005739F1  |.  64:8920       MOV DWORD PTR FS:\[EAX\],ESP  
005739F4  |.  8D55 FC       LEA EDX,DWORD PTR SS:\[EBP-4\]  
005739F7  |.  8B83 14030000 MOV EAX,DWORD PTR DS:\[EBX+314\]  
005739FD  |.  E8 3262EDFF   CALL Arial\_CD.00449C34  
00573A02  |.  8D55 F8       LEA EDX,DWORD PTR SS:\[EBP-8\]  
00573A05  |.  8B83 1C030000 MOV EAX,DWORD PTR DS:\[EBX+31C\]  
00573A0B  |.  E8 2462EDFF   CALL Arial\_CD.00449C34  
00573A10  |.  A1 78B05800   MOV EAX,DWORD PTR DS:\[58B078\]  
00573A15  |.  8B00          MOV EAX,DWORD PTR DS:\[EAX\]  
00573A17  |.  8B4D F8       MOV ECX,DWORD PTR SS:\[EBP-8\]         ;  假码地址入 ECX  
00573A1A  |.  8B55 FC       MOV EDX,DWORD PTR SS:\[EBP-4\]         ;  用户名地址入 EDX  
00573A1D  |.  E8 823F0100   CALL Arial\_CD.005879A4               ;  比较真假码　F7 跟进  
00573A22  |.  84C0          TEST AL,AL                           ;  00587A23 处返回　AL=0 失败　AL=1 成功  
00573A24  |.  74 7E         JE SHORT Arial\_CD.00573AA4           ;  跳失败　不跳成功  
00573A26  |.  A1 78B05800   MOV EAX,DWORD PTR DS:\[58B078\]  
00573A2B  |.  8B00          MOV EAX,DWORD PTR DS:\[EAX\]  
00573A2D  |.  8B55 FC       MOV EDX,DWORD PTR SS:\[EBP-4\]  
00573A30  |.  E8 CB420100   CALL Arial\_CD.00587D00  
00573A35  |.  6A 40         PUSH 40  
00573A37  |.  B9 CC3A5700   MOV ECX,Arial\_CD.00573ACC        ;  congratulations!  
00573A3C  |.  BA E03A5700   MOV EDX,Arial\_CD.00573AE0        ;  register successfully!  thank you for your support!  
00573A41  |.  A1 9CB35800   MOV EAX,DWORD PTR DS:\[58B39C\]  
00573A46  |.  8B00          MOV EAX,DWORD PTR DS:\[EAX\]  
  
 00573A1D 处 F7 来到(比较真假码)：  
  
 005879A4  /$  55            PUSH EBP  
 005879A5  |.  8BEC          MOV EBP,ESP  
 005879A7  |.  83C4 E4       ADD ESP,-1C  
 005879AA  |.  53            PUSH EBX  
 005879AB  |.  33DB          XOR EBX,EBX                            
 005879AD  |.  895D F4       MOV DWORD PTR SS:\[EBP-C\],EBX  
 005879B0  |.  894D F8       MOV DWORD PTR SS:\[EBP-8\],ECX         ;  假码入\[EBP-8\]  
 005879B3  |.  8955 FC       MOV DWORD PTR SS:\[EBP-4\],EDX         ;  用户名入\[EBP-4\]  
 005879B6  |.  8B45 FC       MOV EAX,DWORD PTR SS:\[EBP-4\]         ;  用户名入 EAX  
 005879B9  |.  E8 AAD8E7FF   CALL Arial\_CD.00405268  
 005879BE  |.  8B45 F8       MOV EAX,DWORD PTR SS:\[EBP-8\]  
 005879C1  |.  E8 A2D8E7FF   CALL Arial\_CD.00405268  
 005879C6  |.  33C0          XOR EAX,EAX  
 005879C8  |.  55            PUSH EBP  
 005879C9  |.  68 167A5800   PUSH Arial\_CD.00587A16  
 005879CE  |.  64:FF30       PUSH DWORD PTR FS:\[EAX\]  
 005879D1  |.  64:8920       MOV DWORD PTR FS:\[EAX\],ESP  
 005879D4  |.  33DB          XOR EBX,EBX  
 005879D6  |.  8D55 E4       LEA EDX,DWORD PTR SS:\[EBP-1C\]  
 005879D9  |.  8B45 FC       MOV EAX,DWORD PTR SS:\[EBP-4\]  
 005879DC  |.  E8 672DFEFF   CALL Arial\_CD.0056A748               ;  F7 进入漫长的运算过程  
 005879E1  |.  8D45 E4       LEA EAX,DWORD PTR SS:\[EBP-1C\]        ;  0056A7B9 处返回  
 005879E4  |.  8D55 F4       LEA EDX,DWORD PTR SS:\[EBP-C\]  
 005879E7  |.  E8 D02DFEFF   CALL Arial\_CD.0056A7BC               ;  MD5 加密的 4 组数查表变成注册码  
 005879EC  |.  8B55 F4       MOV EDX,DWORD PTR SS:\[EBP-C\]         ;  0056A864 处返回,真码入 EDX(明码)  
 005879EF  |.  8B45 F8       MOV EAX,DWORD PTR SS:\[EBP-8\]         ;  假码入 EAX  
 005879F2  |.  E8 CDD7E7FF   CALL Arial\_CD.004051C4               ;  真假码比较  
 005879F7  |.  75 02         JNZ SHORT Arial\_CD.005879FB　　　　　;  真假码不等就跳  
 005879F9  |.  B3 01         MOV BL,1  
 005879FB  |>  33C0          XOR EAX,EAX  
 005879FD  |.  5A            POP EDX  
 005879FE  |.  59            POP ECX  
 005879FF  |.  59            POP ECX  
 00587A00  |.  64:8910       MOV DWORD PTR FS:\[EAX\],EDX  
 00587A03  |.  68 1D7A5800   PUSH Arial\_CD.00587A1D  
 00587A08  |>  8D45 F4       LEA EAX,DWORD PTR SS:\[EBP-C\]  
 00587A0B  |.  BA 03000000   MOV EDX,3  
 00587A10  |.  E8 D7D3E7FF   CALL Arial\_CD.00404DEC  
 00587A15  \\.  C3            RETN                                         ;  转到 00587A1D  
 00587A16   .^ E9 15CDE7FF   JMP Arial\_CD.00404730  
 00587A1B   .^ EB EB         JMP SHORT Arial\_CD.00587A08  
 00587A1D   .  8BC3          MOV EAX,EBX  
 00587A1F   .  5B            POP EBX  
 00587A20   .  8BE5          MOV ESP,EBP  
 00587A22   .  5D            POP EBP  
 00587A23   .  C3            RETN                                         ;  返回到 00573A22  
  
  005879DC 处 F7 来到(漫长的运算过程)：  
  
  0056A748  /$  55            PUSH EBP  
  0056A749  |.  8BEC          MOV EBP,ESP  
  0056A74B  |.  83C4 A4       ADD ESP,-5C  
  0056A74E  |.  53            PUSH EBX  
  0056A74F  |.  8BDA          MOV EBX,EDX  
  0056A751  |.  8945 FC       MOV DWORD PTR SS:\[EBP-4\],EAX  
  0056A754  |.  8B45 FC       MOV EAX,DWORD PTR SS:\[EBP-4\]  
  0056A757  |.  E8 0CABE9FF   CALL Arial\_CD.00405268  
  0056A75C  |.  33C0          XOR EAX,EAX  
  0056A75E  |.  55            PUSH EBP  
  0056A75F  |.  68 AEA75600   PUSH Arial\_CD.0056A7AE  
  0056A764  |.  64:FF30       PUSH DWORD PTR FS:\[EAX\]  
  0056A767  |.  64:8920       MOV DWORD PTR FS:\[EAX\],ESP  
  0056A76A  |.  8D45 A4       LEA EAX,DWORD PTR SS:\[EBP-5C\]  
  0056A76D  |.  E8 AEFEFFFF   CALL Arial\_CD.0056A620            ;  初始化变量　F7  
  0056A772  |.  8B45 FC       MOV EAX,DWORD PTR SS:\[EBP-4\]      ;  用户名地址入 EAX  
  0056A775  |.  E8 06A9E9FF   CALL Arial\_CD.00405080            ;  取用户名位数  
  0056A77A  |.  50            PUSH EAX  
  0056A77B  |.  8B45 FC       MOV EAX,DWORD PTR SS:\[EBP-4\]  
  0056A77E  |.  E8 F5AAE9FF   CALL Arial\_CD.00405278  
  0056A783  |.  8BD0          MOV EDX,EAX  
  0056A785  |.  8D45 A4       LEA EAX,DWORD PTR SS:\[EBP-5C\]  
  0056A788  |.  59            POP ECX  
  0056A789  |.  E8 C6FEFFFF   CALL Arial\_CD.0056A654            ;  用户名字符转 16 进制数存入堆栈  
  0056A78E  |.  8BD3          MOV EDX,EBX                       ;  EBX(假码位数地址)  
  0056A790  |.  8D45 A4       LEA EAX,DWORD PTR SS:\[EBP-5C\]     ;  \[EBP-5C\]4 个常数地址  
  0056A793  |.  E8 3CFFFFFF   CALL Arial\_CD.0056A6D4            ;  跟进  
  0056A798  |.  33C0          XOR EAX,EAX                       ;  0056A747 处返回  
  0056A79A  |.  5A            POP EDX  
  0056A79B  |.  59            POP ECX  
  0056A79C  |.  59            POP ECX  
  0056A79D  |.  64:8910       MOV DWORD PTR FS:\[EAX\],EDX  
  0056A7A0  |.  68 B5A75600   PUSH Arial\_CD.0056A7B5  
  0056A7A5  |>  8D45 FC       LEA EAX,DWORD PTR SS:\[EBP-4\]  
  0056A7A8  |.  E8 1BA6E9FF   CALL Arial\_CD.00404DC8  
  0056A7AD  \\.  C3            RETN                              ;  转到 0056A7B5  
  0056A7AE   .^ E9 7D9FE9FF   JMP Arial\_CD.00404730  
  0056A7B3   .^ EB F0         JMP SHORT Arial\_CD.0056A7A5  
  0056A7B5   .  5B            POP EBX  
  0056A7B6   .  8BE5          MOV ESP,EBP  
  0056A7B8   .  5D            POP EBP  
  0056A7B9   .  C3            RETN                              ;  返回到 005879E1  
  
   0056A76D　处 F7 来到(初始化变量)：  
  
   0056A620  /$  C700 01234567 MOV DWORD PTR DS:\[EAX\],67452301　　------> A  
   0056A626  |.  C740 04 89ABC>MOV DWORD PTR DS:\[EAX+4\],EFCDAB89  ------> B  
   0056A62D  |.  C740 08 FEDCB>MOV DWORD PTR DS:\[EAX+8\],98BADCFE  ------> C  
   0056A634  |.  C740 0C 76543>MOV DWORD PTR DS:\[EAX+C\],10325476  ------> D  
   0056A63B  |.  33D2          XOR EDX,EDX  
   0056A63D  |.  8950 10       MOV DWORD PTR DS:\[EAX+10\],EDX    　; 堆栈空出空间  
   0056A640  |.  33D2          XOR EDX,EDX  
   0056A642  |.  8950 14       MOV DWORD PTR DS:\[EAX+14\],EDX　　　; 堆栈空出空间  
   0056A645  |.  83C0 18       ADD EAX,18  
   0056A648  |.  BA 40000000   MOV EDX,40  
   0056A64D  |.  E8 86DDE9FF   CALL Arial\_CD.004083D8  
   0056A652  \\.  C3            RETN　　　　　　　　　　　　　　　 ;返回到 0056A772   
  
   0056A793　处 F7 来到：  
  
　 0056A6D4  /$  53            PUSH EBX  
   0056A6D5  |.  56            PUSH ESI  
   0056A6D6  |.  83C4 F8       ADD ESP,-8  
   0056A6D9  |.  8BF2          MOV ESI,EDX  
   0056A6DB  |.  8BD8          MOV EBX,EAX  
   0056A6DD  |.  8BD4          MOV EDX,ESP  
   0056A6DF  |.  8D43 10       LEA EAX,DWORD PTR DS:\[EBX+10\]  
   0056A6E2  |.  B9 02000000   MOV ECX,2  
   0056A6E7  |.  E8 C8F7FFFF   CALL Arial\_CD.00569EB4  
   0056A6EC  |.  8B43 10       MOV EAX,DWORD PTR DS:\[EBX+10\]       ;  \[EBX+10\]=28  
   0056A6EF  |.  C1E8 03       SHR EAX,3                           ;  EAX=28 -> 5  
   0056A6F2  |.  83E0 3F       AND EAX,3F  
   0056A6F5  |.  83F8 38       CMP EAX,38  
   0056A6F8  |.  73 0B         JNB SHORT Arial\_CD.0056A705  
   0056A6FA  |.  BA 38000000   MOV EDX,38  
   0056A6FF  |.  2BD0          SUB EDX,EAX                         ;  EDX=38-5=33  
   0056A701  |.  8BC2          MOV EAX,EDX  
   0056A703  |.  EB 09         JMP SHORT Arial\_CD.0056A70E  
   0056A705  |>  BA 78000000   MOV EDX,78  
   0056A70A  |.  2BD0          SUB EDX,EAX  
   0056A70C  |.  8BC2          MOV EAX,EDX  
   0056A70E  |>  BA 7CAD5800   MOV EDX,Arial\_CD.0058AD7C  
   0056A713  |.  8BCB          MOV ECX,EBX  
   0056A715  |.  91            XCHG EAX,ECX  
   0056A716  |.  E8 39FFFFFF   CALL Arial\_CD.0056A654  
   0056A71B  |.  8BD4          MOV EDX,ESP  
   0056A71D  |.  8BC3          MOV EAX,EBX  
   0056A71F  |.  B9 08000000   MOV ECX,8  
   0056A724  |.  E8 2BFFFFFF   CALL Arial\_CD.0056A654              ;  数据处理 F7  
   0056A729  |.  8BD6          MOV EDX,ESI                         ;  0056A6D3 处返回  
   0056A72B  |.  8BC3          MOV EAX,EBX  
   0056A72D  |.  B9 04000000   MOV ECX,4  
   0056A732  |.  E8 7DF7FFFF   CALL Arial\_CD.00569EB4  
   0056A737  |.  8BC3          MOV EAX,EBX  
   0056A739  |.  BA 58000000   MOV EDX,58  
   0056A73E  |.  E8 95DCE9FF   CALL Arial\_CD.004083D8  
   0056A743  |.  59            POP ECX  
   0056A744  |.  5A            POP EDX  
   0056A745  |.  5E            POP ESI  
   0056A746  |.  5B            POP EBX  
   0056A747  \\.  C3            RETN　　　　　　　　　　　　　　　  ;  返回到 0056A798   
  
    0056A724 处 F7 来到(数据处理)：  
  
    0056A654  /$  53            PUSH EBX  
    0056A655  |.  56            PUSH ESI  
    0056A656  |.  57            PUSH EDI  
    0056A657  |.  55            PUSH EBP  
    0056A658  |.  8BF9          MOV EDI,ECX  
    0056A65A  |.  8BEA          MOV EBP,EDX  
    0056A65C  |.  8BF0          MOV ESI,EAX  
    0056A65E  |.  8B46 10       MOV EAX,DWORD PTR DS:\[ESI+10\]                ;  28  
    0056A661  |.  C1E8 03       SHR EAX,3                                    ;  EAX=28 -> 5  
    0056A664  |.  83E0 3F       AND EAX,3F  
    0056A667  |.  8BD7          MOV EDX,EDI  
    0056A669  |.  C1E2 03       SHL EDX,3                                    ;  EDX=33 -> 198  
    0056A66C  |.  0156 10       ADD DWORD PTR DS:\[ESI+10\],EDX                ;  \[ESI+10\]=28 add 198=1C0  
    0056A66F  |.  3B56 10       CMP EDX,DWORD PTR DS:\[ESI+10\]                ;  198 < 1C0  
    0056A672  |.  76 03         JBE SHORT Arial\_CD.0056A677  
    0056A674  |.  FF46 14       INC DWORD PTR DS:\[ESI+14\]  
    0056A677  |>  8BD7          MOV EDX,EDI  
    0056A679  |.  C1EA 1D       SHR EDX,1D                                   ;  EDX=33 -> 0  
    0056A67C  |.  0156 14       ADD DWORD PTR DS:\[ESI+14\],EDX  
    0056A67F  |.  BB 40000000   MOV EBX,40  
    0056A684  |.  2BD8          SUB EBX,EAX                                  ;  EBX=40-5=3B  
    0056A686  |.  3BDF          CMP EBX,EDI                                  ;  3B > 33  
    0056A688  |.  77 32         JA SHORT Arial\_CD.0056A6BC  
    0056A68A  |.  8D4406 18     LEA EAX,DWORD PTR DS:\[ESI+EAX+18\]  
    0056A68E  |.  8BCB          MOV ECX,EBX  
    0056A690  |.  8BD5          MOV EDX,EBP  
    0056A692  |.  E8 39DDE9FF   CALL Arial\_CD.004083D0  
    0056A697  |.  8BD6          MOV EDX,ESI  
    0056A699  |.  8D46 18       LEA EAX,DWORD PTR DS:\[ESI+18\]  
    0056A69C  |.  E8 4FF8FFFF   CALL Arial\_CD.00569EF0                       ;  数据处理 F7  
    0056A6A1  |. /EB 0E         JMP SHORT Arial\_CD.0056A6B1       　　　　　 ;  数据处理完后返回到此  
    0056A6A3  |>  8BD6          /MOV EDX,ESI  
    0056A6A5  |.  8D441D 00     |LEA EAX,DWORD PTR SS:\[EBP+EBX\]  
    0056A6A9  |.  E8 42F8FFFF   |CALL Arial\_CD.00569EF0  
    0056A6AE  |.  83C3 40       |ADD EBX,40  
    0056A6B1  |>  8D43 3F        LEA EAX,DWORD PTR DS:\[EBX+3F\]  
    0056A6B4  |.  3BF8          |CMP EDI,EAX  
    0056A6B6  |.^ 77 EB         \\JA SHORT Arial\_CD.0056A6A3  
    0056A6B8  |.  33C0          XOR EAX,EAX  
    0056A6BA  |.  EB 02         JMP SHORT Arial\_CD.0056A6BE  
    0056A6BC  |>  33DB          XOR EBX,EBX  
    0056A6BE  |>  8D4406 18     LEA EAX,DWORD PTR DS:\[ESI+EAX+18\]  
    0056A6C2  |.  8BCF          MOV ECX,EDI  
    0056A6C4  |.  2BCB          SUB ECX,EBX  
    0056A6C6  |.  8D541D 00     LEA EDX,DWORD PTR SS:\[EBP+EBX\]  
    0056A6CA  |.  E8 01DDE9FF   CALL Arial\_CD.004083D0  
    0056A6CF  |.  5D            POP EBP  
    0056A6D0  |.  5F            POP EDI  
    0056A6D1  |.  5E            POP ESI  
    0056A6D2  |.  5B            POP EBX  
    0056A6D3  \\.  C3            RETN　　　　　　　　　　　　　　　　　　　　　;  返回到 0056A729  
  
  
     0056A69C  处 F7 来到(数据处理):  
 　　MD5 加密用户名，wzwgp 的 16 进制数是 (77 7A 77 67 70)  
  
     00569EF0  /$  53            PUSH EBX  
     00569EF1  |.  56            PUSH ESI  
     00569EF2  |.  57            PUSH EDI  
     00569EF3  |.  55            PUSH EBP  
     00569EF4  |.  83C4 A8       ADD ESP,-58  
     00569EF7  |.  895424 04     MOV DWORD PTR SS:\[ESP+4\],EDX  
     00569EFB  |.  890424        MOV DWORD PTR SS:\[ESP\],EAX  
     00569EFE  |.  8D5C24 08     LEA EBX,DWORD PTR SS:\[ESP+8\]  
     00569F02  |.  8D7424 0C     LEA ESI,DWORD PTR SS:\[ESP+C\]  
     00569F06  |.  8D7C24 10     LEA EDI,DWORD PTR SS:\[ESP+10\]  
     00569F0A  |.  8D6C24 14     LEA EBP,DWORD PTR SS:\[ESP+14\]  
     00569F0E  |.  8D5424 18     LEA EDX,DWORD PTR SS:\[ESP+18\]  
     00569F12  |.  B9 40000000   MOV ECX,40  
     00569F17  |.  8B0424        MOV EAX,DWORD PTR SS:\[ESP\]  
     00569F1A  |.  E8 5DFFFFFF   CALL Arial\_CD.00569E7C              ;  取用户名地址  
     00569F1F  |.  8B4424 04     MOV EAX,DWORD PTR SS:\[ESP+4\]  
     00569F23  |.  8B00          MOV EAX,DWORD PTR DS:\[EAX\]          ;  \[EAX\]=A  
     00569F25  |.  8903          MOV DWORD PTR DS:\[EBX\],EAX  
     00569F27  |.  8B4424 04     MOV EAX,DWORD PTR SS:\[ESP+4\]  
     00569F2B  |.  8B40 04       MOV EAX,DWORD PTR DS:\[EAX+4\]        ;  \[EAX+4\]=B  
     00569F2E  |.  8906          MOV DWORD PTR DS:\[ESI\],EAX  
     00569F30  |.  8B4424 04     MOV EAX,DWORD PTR SS:\[ESP+4\]  
     00569F34  |.  8B40 08       MOV EAX,DWORD PTR DS:\[EAX+8\]        ;  \[EAX+8\]=C  
     00569F37  |.  8907          MOV DWORD PTR DS:\[EDI\],EAX  
     00569F39  |.  8B4424 04     MOV EAX,DWORD PTR SS:\[ESP+4\]  
     00569F3D  |.  8B40 0C       MOV EAX,DWORD PTR DS:\[EAX+C\]        ;  \[EAX+C\]=D  
     00569F40  |.  8945 00       MOV DWORD PTR SS:\[EBP\],EAX  
     00569F43  |.  8B45 00       MOV EAX,DWORD PTR SS:\[EBP\]  
     00569F46  |.  50            PUSH EAX                            ; Arg4=D  
     00569F47  |.  8B4424 1C     MOV EAX,DWORD PTR SS:\[ESP+1C\]       ; \[ESP+1C\]=用户名前 4 位 16 进制数  
     00569F4B  |.  50            PUSH EAX                            ; Arg3 = 67777A77  
     00569F4C  |.  6A 07         PUSH 7                              ; Arg2 = 00000007  
     00569F4E  |.  68 78A46AD7   PUSH D76AA478                       ; Arg1 = D76AA478  
     00569F53  |.  8BC3          MOV EAX,EBX                         ;   
     00569F55  |.  8B0F          MOV ECX,DWORD PTR DS:\[EDI\]          ; \[EDI\]=C  
     00569F57  |.  8B16          MOV EDX,DWORD PTR DS:\[ESI\]          ; \[ESI\]=B  
     00569F59  |.  E8 4EFEFFFF   CALL Arial\_CD.00569DAC              ; 第 1 次数据处理  
     00569F5E  |.  8B07          MOV EAX,DWORD PTR DS:\[EDI\]          ;  EAX=C  
     00569F60  |.  50            PUSH EAX                            ; /Arg4=C  
     00569F61  |.  8B4424 20     MOV EAX,DWORD PTR SS:\[ESP+20\]       ; |EAX=8070  
     00569F65  |.  50            PUSH EAX                            ; |Arg3  
     00569F66  |.  6A 0C         PUSH 0C                             ; |Arg2 = 0000000C  
     00569F68  |.  68 56B7C7E8   PUSH E8C7B756                       ; |Arg1 = E8C7B756  
     00569F6D  |.  8BC5          MOV EAX,EBP                         ; |  
     00569F6F  |.  8B0E          MOV ECX,DWORD PTR DS:\[ESI\]          ; |ECX=C2  
     00569F71  |.  8B13          MOV EDX,DWORD PTR DS:\[EBX\]          ; |EDX=S1=60DD22A8  
     00569F73  |.  E8 34FEFFFF   CALL Arial\_CD.00569DAC              ; \\2 次  
  
\----------------------------------------------------------------------------------------------  
  CALL Arial\_CD.00569DAC  第 1 次数据处理  
  
00569DAC  /$  55            PUSH EBP  
00569DAD  |.  8BEC          MOV EBP,ESP  
00569DAF  |.  53            PUSH EBX  
00569DB0  |.  56            PUSH ESI  
00569DB1  |.  57            PUSH EDI  
00569DB2  |.  8BF9          MOV EDI,ECX  
00569DB4  |.  8BF2          MOV ESI,EDX  
00569DB6  |.  8BD8          MOV EBX,EAX  
00569DB8  |.  8B4D 14       MOV ECX,DWORD PTR SS:\[EBP>  
00569DBB  |.  8BD7          MOV EDX,EDI  
00569DBD  |.  8BC6          MOV EAX,ESI  
00569DBF  |.  E8 9CFFFFFF   CALL Arial\_CD.00569D60    ;  F7 进入  
00569DC4  |.  0345 10       ADD EAX,DWORD PTR SS:\[EBP>;  EAX=C+67777A77=325775  
00569DC7  |.  0345 08       ADD EAX,DWORD PTR SS:\[EBP>;  EAX=325775+D76AA478=D79CFBED  
00569DCA  |.  0103          ADD DWORD PTR DS:\[EBX\],EA>;  \[EBX\]=67452301+D79CFBED=3EE21EEE  
00569DCC  |.  8BC3          MOV EAX,EBX  
00569DCE  |.  8A55 0C       MOV DL,BYTE PTR SS:\[EBP+C>;  DL=FE -> 07 (EDX=C -> 98BADC07)  
00569DD1  |.  E8 B6FFFFFF   CALL Arial\_CD.00569D8C    ;  F7 进入  
00569DD6  |.  0133          ADD DWORD PTR DS:\[EBX\],ES>;  \[EBX\]=710F771F+98BADCFE=60DD22A8   
00569DD8  |.  5F            POP EDI  
00569DD9  |.  5E            POP ESI  
00569DDA  |.  5B            POP EBX  
00569DDB  |.  5D            POP EBP  
00569DDC  \\.  C2 1000       RETN 10  
  
　00569DBF  处 F7            
  
　00569D60  /$  23D0          AND EDX,EAX                    ;  EDX=C and  B =88888888  
　00569D62  |.  F7D0          NOT EAX                        ;  EAX=B(取反)＝10325476 ---> D  
　00569D64  |.  23C8          AND ECX,EAX                    ;  ECX=D and D =10325476  
　00569D66  |.  0BD1          OR EDX,ECX                     ;  EDX=88888888 or D =98BADCFE ---> C  
　00569D68  |.  8BC2          MOV EAX,EDX  
　00569D6A  \\.  C3            RETN  
  
　00569DBF 处 F7  
           
　00569D8C  /$  53            PUSH EBX  
　00569D8D  |.  33C9          XOR ECX,ECX  
　00569D8F  |.  8ACA          MOV CL,DL  
　00569D91  |.  51            PUSH ECX  
　00569D92  |.  B9 20000000   MOV ECX,20  
　00569D97  |.  5B            POP EBX  
　00569D98  |.  2BCB          SUB ECX,EBX                  ;  ECX=20-7=19  
　00569D9A  |.  8B18          MOV EBX,DWORD PTR DS:\[EAX\]   ;  \[EAX\]=3EE21EEE  
　00569D9C  |.  D3EB          SHR EBX,CL                   ;  EBX=3EE21EEE -> 1F  
　00569D9E  |.  8BCA          MOV ECX,EDX                  ;  ECX=98BADC07  
　00569DA0  |.  8B10          MOV EDX,DWORD PTR DS:\[EAX\]   ;  EDX=3EE21EEE  
　00569DA2  |.  D3E2          SHL EDX,CL                   ;  EDX=710F7700(CL=07)  
　00569DA4  |.  0BDA          OR EBX,EDX                   ;  EBX=1F or 710F7700=710F771F  
　00569DA6  |.  8918          MOV DWORD PTR DS:\[EAX\],EBX   ;  保存 EBX -> \[12F560\]  
　00569DA8  |.  5B            POP EBX  
　00569DA9  \\.  C3            RETN  
\-----------------------------------------------------------------------------------------------------------  
  
             --------------中间省略第 3-63 次数据处理代码------------------------  
  
     0056A5DA  |.  50            PUSH EAX                          ; /Arg4 = 476F80D4  
     0056A5DB  |.  8B4424 40     MOV EAX,DWORD PTR SS:\[ESP+40\]     ; |  
     0056A5DF  |.  50            PUSH EAX                          ; |Arg3 = 00000000  
     0056A5E0  |.  6A 15         PUSH 15                           ; |Arg2 = 00000015  
     0056A5E2  |.  68 91D386EB   PUSH EB86D391                     ; |Arg1 = EB86D391  
     0056A5E7  |.  8BC6          MOV EAX,ESI                       ; |  
     0056A5E9  |.  8B4D 00       MOV ECX,DWORD PTR SS:\[EBP\]        ; |  
     0056A5EC  |.  8B17          MOV EDX,DWORD PTR DS:\[EDI\]        ; |  
     0056A5EE  |.  E8 55F8FFFF   CALL Arial\_CD.00569E48            ; \\第 64 次数据处理  
     0056A5F3  |.  8B4424 04     MOV EAX,DWORD PTR SS:\[ESP+4\]  
     0056A5F7  |.  8B13          MOV EDX,DWORD PTR DS:\[EBX\]      ;  \[EBX\]=476F80D4  
     0056A5F9  |.  0110          ADD DWORD PTR DS:\[EAX\],EDX      ;  \[EAX\]=A+476F80D4=AEB4A3D5　 ----1  
     0056A5FB  |.  8B4424 04     MOV EAX,DWORD PTR SS:\[ESP+4\]  
     0056A5FF  |.  8B16          MOV EDX,DWORD PTR DS:\[ESI\]  
     0056A601  |.  0150 04       ADD DWORD PTR DS:\[EAX+4\],EDX    ;  \[EAX+4\]=B+37DB229E=27A8CE27 ----2  
     0056A604  |.  8B4424 04     MOV EAX,DWORD PTR SS:\[ESP+4\]  
     0056A608  |.  8B17          MOV EDX,DWORD PTR DS:\[EDI\]  
     0056A60A  |.  0150 08       ADD DWORD PTR DS:\[EAX+8\],EDX    ;  \[EAX+8\]=C+15AF0A78=AE69E776 ----3  
     0056A60D  |.  8B4424 04     MOV EAX,DWORD PTR SS:\[ESP+4\]  
     0056A611  |.  8B55 00       MOV EDX,DWORD PTR SS:\[EBP\]  
     0056A614  |.  0150 0C       ADD DWORD PTR DS:\[EAX+C\],EDX    ;  \[EAX+C\]=D+1999B819=29CC0C8F ----4  
     0056A617  |.  83C4 58       ADD ESP,58                      ;  到此 MD5 加密的 4 组数(32 个)终于出来了  
     0056A61A  |.  5D            POP EBP  
     0056A61B  |.  5F            POP EDI  
     0056A61C  |.  5E            POP ESI  
     0056A61D  |.  5B            POP EBX  
     0056A61E  \\.  C3            RETN                            ;  返回到 0056A6A1  
  
  
 005879E7 处 F7 来到(MD5 加密的 4 组数查表变成注册码):                
  
 0056A7BC  /$  55            PUSH EBP  
 0056A7BD  |.  8BEC          MOV EBP,ESP  
 0056A7BF  |.  83C4 E8       ADD ESP,-18  
 0056A7C2  |.  53            PUSH EBX  
 0056A7C3  |.  56            PUSH ESI  
 0056A7C4  |.  57            PUSH EDI  
 0056A7C5  |.  33C9          XOR ECX,ECX  
 0056A7C7  |.  894D EC       MOV DWORD PTR SS:\[EBP-14\],ECX  
 0056A7CA  |.  894D E8       MOV DWORD PTR SS:\[EBP-18\],ECX  
 0056A7CD  |.  8BF0          MOV ESI,EAX  
 0056A7CF  |.  8D7D F0       LEA EDI,DWORD PTR SS:\[EBP-10\]  
 0056A7D2  |.  A5            MOVS DWORD PTR ES:\[EDI\],DWORD PTR DS:\[ESI\]   |   
 0056A7D3  |.  A5            MOVS DWORD PTR ES:\[EDI\],DWORD PTR DS:\[ESI\]   |传送 4 组处理的数据  
 0056A7D4  |.  A5            MOVS DWORD PTR ES:\[EDI\],DWORD PTR DS:\[ESI\]   |  
 0056A7D5  |.  A5            MOVS DWORD PTR ES:\[EDI\],DWORD PTR DS:\[ESI\]   |  
 0056A7D6  |.  8BFA          MOV EDI,EDX  
 0056A7D8  |.  33C0          XOR EAX,EAX  
 0056A7DA  |.  55            PUSH EBP  
 0056A7DB  |.  68 57A85600   PUSH Arial\_CD.0056A857  
 0056A7E0  |.  64:FF30       PUSH DWORD PTR FS:\[EAX\]  
 0056A7E3  |.  64:8920       MOV DWORD PTR FS:\[EAX\],ESP  
 0056A7E6  |.  8BC7          MOV EAX,EDI  
 0056A7E8  |.  E8 DBA5E9FF   CALL Arial\_CD.00404DC8  
 0056A7ED  |.  B3 10         MOV BL,10  
 0056A7EF  |.  8D75 F0       LEA ESI,DWORD PTR SS:\[EBP-10\]  
 0056A7F2  |>  FF37          /PUSH DWORD PTR DS:\[EDI\]                       
 0056A7F4  |.  8D45 EC       |LEA EAX,DWORD PTR SS:\[EBP-14\]  
 0056A7F7  |.  33D2          |XOR EDX,EDX  
 0056A7F9  |.  8A16          |MOV DL,BYTE PTR DS:\[ESI\]                    ;  每次取 2 位 MD5 加密的数  
 0056A7FB  |.  C1EA 04       |SHR EDX,4                                   ;  逻辑右移 4 位  
 0056A7FE  |.  83E2 0F       |AND EDX,0F                                  ;  保留前 1 位数用于查表  
 0056A801  |.  8A92 BCAD5800 |MOV DL,BYTE PTR DS:\[EDX+58ADBC\]             ;  查表 (0123456789abcdef?)  
 0056A807  |.  E8 9CA7E9FF   |CALL Arial\_CD.00404FA8  
 0056A80C  |.  FF75 EC       |PUSH DWORD PTR SS:\[EBP-14\]  
 0056A80F  |.  8D45 E8       |LEA EAX,DWORD PTR SS:\[EBP-18\]  
 0056A812  |.  8A16          |MOV DL,BYTE PTR DS:\[ESI\]                    ;  再取一遍  
 0056A814  |.  80E2 0F       |AND DL,0F                                   ;  保留后 1 位数用于查表  
 0056A817  |.  81E2 FF000000 |AND EDX,0FF  
 0056A81D  |.  8A92 BCAD5800 |MOV DL,BYTE PTR DS:\[EDX+58ADBC\]             ;  查表 (0123456789abcdef?)  
 0056A823  |.  E8 80A7E9FF   |CALL Arial\_CD.00404FA8  
 0056A828  |.  FF75 E8       |PUSH DWORD PTR SS:\[EBP-18\]  
 0056A82B  |.  8BC7          |MOV EAX,EDI  
 0056A82D  |.  BA 03000000   |MOV EDX,3  
 0056A832  |.  E8 09A9E9FF   |CALL Arial\_CD.00405140                      ;  保存查表结果  
 0056A837  |.  46            |INC ESI  
 0056A838  |.  FECB          |DEC BL                                      ;  BL 计数器  
 0056A83A  |.^ 75 B6         \\JNZ SHORT Arial\_CD.0056A7F2                 ;  循环  
 0056A83C  |.  33C0          XOR EAX,EAX  
 0056A83E  |.  5A            POP EDX  
 0056A83F  |.  59            POP ECX  
 0056A840  |.  59            POP ECX  
 0056A841  |.  64:8910       MOV DWORD PTR FS:\[EAX\],EDX  
 0056A844  |.  68 5EA85600   PUSH Arial\_CD.0056A85E  
 0056A849  |>  8D45 E8       LEA EAX,DWORD PTR SS:\[EBP-18\]  
 0056A84C  |.  BA 02000000   MOV EDX,2  
 0056A851  |.  E8 96A5E9FF   CALL Arial\_CD.00404DEC  
 0056A856  \\.  C3            RETN                                         ;  转到 0056A85E  
 0056A857   .^ E9 D49EE9FF   JMP Arial\_CD.00404730  
 0056A85C   .^ EB EB         JMP SHORT Arial\_CD.0056A849  
 0056A85E   .  5F            POP EDI  
 0056A85F   .  5E            POP ESI  
 0056A860   .  5B            POP EBX  
 0056A861   .  8BE5          MOV ESP,EBP  
 0056A863   .  5D            POP EBP  
 0056A864   .  C3            RETN                                         ;  返回到 005879EC  
  
    D5A3B4AE27CEA82776E769AE8F0CCC29 -- 查表 --> d5a3b4ae27cea82776e769ae8f0ccc29   
  
  
  终于跟完了，一层一层的 Call 转得头晕,有些地方还没有明白，跟书上讲的一样，搞得这么复杂最后却用明码比较。用《加密与解密》光盘上的 MD5calculator.exe 计算用户名，再将计算结果里的字母由大写改成小写，就可得到注册码。  
  
我的用户名：wzwgp  
　　注册码：d5a3b4ae27cea82776e769ae8f0ccc29  
  
注册信息保存在注册表：HKEY\_USERS\\S-1-5-21-1123561945-492894223-1060284298-1002\\Software\\Arial CD Ripper  
项下的　username  
  
　破文写得象老太太的裹脚布，感谢你看完。 

 4dd466d0e1e0cc123afbe504da2462ca