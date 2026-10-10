

# Flash ScreenSaver Maker 1.3

> 原文：[`www.pediy.com/kssd/pediy08/pediy8-149.htm`](https://www.pediy.com/kssd/pediy08/pediy8-149.htm)

【文章标题】: 屏保软件算法分析(MD5)  
【文章作者】: ww990  
【作者邮箱】: ww990@126.com  
【软件名称】: Flash ScreenSaver Maker 1.3  
【下载地址】: http://www.screensaver-maker.biz/  
【加壳方式】: upx  
【保护方式】: 注册码  
【编写语言】: delphi  
【使用工具】: peid ollyice   
【操作平台】: XP  
【软件介绍】: Flash ScreenSaver Maker is a screen saver editor  
【作者声明】: 只是感兴趣，没有其他目的。失误之处敬请诸位大侠赐教!  
\--------------------------------------------------------------------------------  
【详细过程】  
  前些天到天空看到这个软件就下了，看着喜欢呗。  
  首先 peid 一下，upx 的壳，简单搞定，运行一下没有校检，狂喜。  
  再用 kanal 插件分析一下竟然有 ADLER32、CRC32、MD5 晕  
  不管运行注册一下，有注册提示，开心 ^\_^ 。  
  od 加载，下断 bp MessageBoxA，运行点注册输入注册名 WW9900(至少六位)，注册码 123-4567-890  
  断在这里  
  77D504EA >  8BFF            MOV EDI,EDI  
  77D504EC    55              PUSH EBP  
  77D504ED    8BEC            MOV EBP,ESP  
  77D504EF    833D BC04D777 0>CMP DWORD PTR DS:\[77D704BC\],0  
  77D504F6    74 24           JE SHORT USER32.77D5051C  
  ALT+F9 返回看看  
  004B7A24   .  8D45 F4       LEA EAX,DWORD PTR SS:\[EBP-C\]  
  004B7A27   .  E8 ECD2F4FF   CALL FlashScr.00404D18  
  004B7A2C   .  8B15 3C1D4C00 MOV EDX,DWORD PTR DS:\[4C1D3C\]            ;  FlashScr.004C18BC  
  004B7A32   .  8B12          MOV EDX,DWORD PTR DS:\[EDX\]  
  004B7A34   .  8A52 03       MOV DL,BYTE PTR DS:\[EDX+3\]               ;    
  004B7A37   .  8810          MOV BYTE PTR DS:\[EAX\],DL  
  004B7A39   .  8D45 F4       LEA EAX,DWORD PTR SS:\[EBP-C\]  
  004B7A3C   .  E8 D7D2F4FF   CALL FlashScr.00404D18  
  004B7A41   .  8B15 3C1D4C00 MOV EDX,DWORD PTR DS:\[4C1D3C\]            ;  FlashScr.004C18BC  
  004B7A47   .  8B12          MOV EDX,DWORD PTR DS:\[EDX\]  
  004B7A49   .  8A52 01       MOV DL,BYTE PTR DS:\[EDX+1\]               ;    
  004B7A4C   .  8850 01       MOV BYTE PTR DS:\[EAX+1\],DL  
  004B7A4F   .  8D45 F4       LEA EAX,DWORD PTR SS:\[EBP-C\]  
  004B7A52   .  E8 C1D2F4FF   CALL FlashScr.00404D18  
  004B7A57   .  8B15 3C1D4C00 MOV EDX,DWORD PTR DS:\[4C1D3C\]            ;  FlashScr.004C18BC  
  004B7A5D   .  8B12          MOV EDX,DWORD PTR DS:\[EDX\]  
  004B7A5F   .  8A52 02       MOV DL,BYTE PTR DS:\[EDX+2\]               ;    
  004B7A62   .  8850 02       MOV BYTE PTR DS:\[EAX+2\],DL  
  004B7A65   .  8D45 F4       LEA EAX,DWORD PTR SS:\[EBP-C\]  
  004B7A68   .  E8 ABD2F4FF   CALL FlashScr.00404D18  
  004B7A6D   .  8B15 3C1D4C00 MOV EDX,DWORD PTR DS:\[4C1D3C\]            ;  FlashScr.004C18BC  
  004B7A73   .  8B12          MOV EDX,DWORD PTR DS:\[EDX\]  
  004B7A75   .  8A52 05       MOV DL,BYTE PTR DS:\[EDX+5\]               ;    
  004B7A78   .  8850 03       MOV BYTE PTR DS:\[EAX+3\],DL  
  004B7A7B   .  8D45 F4       LEA EAX,DWORD PTR SS:\[EBP-C\]  
  004B7A7E   .  E8 95D2F4FF   CALL FlashScr.00404D18  
  004B7A83   .  8B15 3C1D4C00 MOV EDX,DWORD PTR DS:\[4C1D3C\]            ;  FlashScr.004C18BC  
  004B7A89   .  8B12          MOV EDX,DWORD PTR DS:\[EDX\]  
  004B7A8B   .  8A52 06       MOV DL,BYTE PTR DS:\[EDX+6\]  
  004B7A8E   .  8850 04       MOV BYTE PTR DS:\[EAX+4\],DL               ;   
  004B7A91   .  8D45 F4       LEA EAX,DWORD PTR SS:\[EBP-C\]  
  004B7A94   .  E8 7FD2F4FF   CALL FlashScr.00404D18  
  004B7A99   .  8B15 3C1D4C00 MOV EDX,DWORD PTR DS:\[4C1D3C\]            ;  FlashScr.004C18BC  
  004B7A9F   .  8B12          MOV EDX,DWORD PTR DS:\[EDX\]  
  004B7AA1   .  8A52 08       MOV DL,BYTE PTR DS:\[EDX+8\]               ;    
  004B7AA4   .  8850 05       MOV BYTE PTR DS:\[EAX+5\],DL  
  004B7AA7   .^ E9 BEFEFFFF   JMP FlashScr.004B796A  
  004B7AAC   >  BA 02000000   MOV EDX,2                                ;    
  004B7AB1   .  8D45 DC       LEA EAX,DWORD PTR SS:\[EBP-24\]            ;    
  004B7AB4   >  8B4D F4       MOV ECX,DWORD PTR SS:\[EBP-C\]             ;    
  004B7AB7   .  8A4C11 FF     MOV CL,BYTE PTR DS:\[ECX+EDX-1\]           ;    
  004B7ABB   .  8B18          MOV EBX,DWORD PTR DS:\[EAX\]               ;    
  004B7ABD   .  8B75 F8       MOV ESI,DWORD PTR SS:\[EBP-8\]  
  004B7AC0   .  3A4C1E FF     CMP CL,BYTE PTR DS:\[ESI+EBX-1\]           ;  
  004B7AC4      74 53         JE SHORT FlashScr.004B7B19  
  004B7AC6   .  6A 10         PUSH 10  
  004B7AC8   .  68 4C7C4B00   PUSH FlashScr.004B7C4C                   ;  ASCII "error"  
  004B7ACD   .  68 547C4B00   PUSH FlashScr.004B7C54                   ;  ASCII "Register FAILED, please check User name and Serial number."  
  004B7AD2   .  8B45 FC       MOV EAX,DWORD PTR SS:\[EBP-4\]  
  004B7AD5   .  E8 6603FBFF   CALL FlashScr.00467E40  
  004B7ADA   .  50            PUSH EAX                                 ; |hOwner  
  004B7ADB   .  E8 34F9F4FF   CALL <JMP.&user32.MessageBoxA>           ; \\MessageBoxA  
  004B7AE0   .  A1 B41D4C00   MOV EAX,DWORD PTR DS:\[4C1DB4\]              停止此处  
  重新运行程序我们在\*处设断                                  
  004B791C   .  68 3C7C4B00   PUSH FlashScr.004B7C3C                     \*  此处设断  
  004B7921   .  64:FF30       PUSH DWORD PTR FS:\[EAX\]  
  运行，再次输入用户名和注册码，确定后断下  
  004B791C   .  68 3C7C4B00   PUSH FlashScr.004B7C3C  
  004B7921   .  64:FF30       PUSH DWORD PTR FS:\[EAX\]  
  004B7924   .  64:8920       MOV DWORD PTR FS:\[EAX\],ESP  
  004B7927   .  8B45 FC       MOV EAX,DWORD PTR SS:\[EBP-4\]  
  004B792A   .  8B80 2C030000 MOV EAX,DWORD PTR DS:\[EAX+32C\]  
  004B7930   .  33D2          XOR EDX,EDX  
  004B7932   .  E8 F1F0F7FF   CALL FlashScr.00436A28  
  004B7937   .  8B0D 301F4C00 MOV ECX,DWORD PTR DS:\[4C1F30\]            ;  FlashScr.004C13F8  
  004B793D   .  8B09          MOV ECX,DWORD PTR DS:\[ECX\]               ;  FlashScr.004B324C  
  004B793F   .  8B15 B41D4C00 MOV EDX,DWORD PTR DS:\[4C1DB4\]            ;  FlashScr.004C18B8  
  004B7945   .  8B12          MOV EDX,DWORD PTR DS:\[EDX\]  
  004B7947   .  8D45 C4       LEA EAX,DWORD PTR SS:\[EBP-3C\]  
  004B794A   .  E8 BDD1F4FF   CALL FlashScr.00404B0C  
  004B794F   .  8B45 C4       MOV EAX,DWORD PTR SS:\[EBP-3C\]            ;  注册名+Flash ScreenSaver Maker  
  004B7952   .  8D55 C8       LEA EDX,DWORD PTR SS:\[EBP-38\]  
  004B7955   .  E8 8AEFFFFF   CALL FlashScr.004B68E4  
  004B795A   .  8D45 C8       LEA EAX,DWORD PTR SS:\[EBP-38\]  
  004B795D   .  8D55 F8       LEA EDX,DWORD PTR SS:\[EBP-8\]  
  004B7960   .  E8 F3EFFFFF   CALL FlashScr.004B6958                   :  此处调用是将“WW9900Flash ScreenSaver Maker”字符进行 MD5 加密  
  004B7965   .  E9 AD000000   JMP FlashScr.004B7A17                       生成密文 9817c30edd06f75bea902cc508ece106，供伪码对比。高兴的话可进去看看。  
    
  向下 F8 来到  
  004B7A1A   .  BA 06000000   MOV EDX,6  
  004B7A1F   .  E8 28D4F4FF   CALL FlashScr.00404E4C  
  004B7A24   .  8D45 F4       LEA EAX,DWORD PTR SS:\[EBP-C\]  
  004B7A27   .  E8 ECD2F4FF   CALL FlashScr.00404D18  
  004B7A2C   .  8B15 3C1D4C00 MOV EDX,DWORD PTR DS:\[4C1D3C\]            ;  FlashScr.004C18BC  
  004B7A32   .  8B12          MOV EDX,DWORD PTR DS:\[EDX\]  
  004B7A34   .  8A52 03       MOV DL,BYTE PTR DS:\[EDX+3\]               ;  取输入码的第四位  
  004B7A37   .  8810          MOV BYTE PTR DS:\[EAX\],DL  
  004B7A39   .  8D45 F4       LEA EAX,DWORD PTR SS:\[EBP-C\]  
  004B7A3C   .  E8 D7D2F4FF   CALL FlashScr.00404D18  
  004B7A41   .  8B15 3C1D4C00 MOV EDX,DWORD PTR DS:\[4C1D3C\]            ;  FlashScr.004C18BC  
  004B7A47   .  8B12          MOV EDX,DWORD PTR DS:\[EDX\]  
  004B7A49   .  8A52 01       MOV DL,BYTE PTR DS:\[EDX+1\]               ;  取输入码的第二位  
  004B7A4C   .  8850 01       MOV BYTE PTR DS:\[EAX+1\],DL  
  004B7A4F   .  8D45 F4       LEA EAX,DWORD PTR SS:\[EBP-C\]  
  004B7A52   .  E8 C1D2F4FF   CALL FlashScr.00404D18  
  004B7A57   .  8B15 3C1D4C00 MOV EDX,DWORD PTR DS:\[4C1D3C\]            ;  FlashScr.004C18BC  
  004B7A5D   .  8B12          MOV EDX,DWORD PTR DS:\[EDX\]  
  004B7A5F   .  8A52 02       MOV DL,BYTE PTR DS:\[EDX+2\]               ;  取输入码的第三位  
  004B7A62   .  8850 02       MOV BYTE PTR DS:\[EAX+2\],DL  
  004B7A65   .  8D45 F4       LEA EAX,DWORD PTR SS:\[EBP-C\]  
  004B7A68   .  E8 ABD2F4FF   CALL FlashScr.00404D18  
  004B7A6D   .  8B15 3C1D4C00 MOV EDX,DWORD PTR DS:\[4C1D3C\]            ;  FlashScr.004C18BC  
  004B7A73   .  8B12          MOV EDX,DWORD PTR DS:\[EDX\]  
  004B7A75   .  8A52 05       MOV DL,BYTE PTR DS:\[EDX+5\]               ;  取输入码的第六位  
  004B7A78   .  8850 03       MOV BYTE PTR DS:\[EAX+3\],DL  
  004B7A7B   .  8D45 F4       LEA EAX,DWORD PTR SS:\[EBP-C\]  
  004B7A7E   .  E8 95D2F4FF   CALL FlashScr.00404D18  
  004B7A83   .  8B15 3C1D4C00 MOV EDX,DWORD PTR DS:\[4C1D3C\]            ;  FlashScr.004C18BC  
  004B7A89   .  8B12          MOV EDX,DWORD PTR DS:\[EDX\]  
  004B7A8B   .  8A52 06       MOV DL,BYTE PTR DS:\[EDX+6\]  
  004B7A8E   .  8850 04       MOV BYTE PTR DS:\[EAX+4\],DL               ;  取输入码的第七位  
  004B7A91   .  8D45 F4       LEA EAX,DWORD PTR SS:\[EBP-C\]  
  004B7A94   .  E8 7FD2F4FF   CALL FlashScr.00404D18  
  004B7A99   .  8B15 3C1D4C00 MOV EDX,DWORD PTR DS:\[4C1D3C\]            ;  FlashScr.004C18BC  
  004B7A9F   .  8B12          MOV EDX,DWORD PTR DS:\[EDX\]  
  004B7AA1   .  8A52 08       MOV DL,BYTE PTR DS:\[EDX+8\]               ;  取输入码的第九位  
  004B7AA4   .  8850 05       MOV BYTE PTR DS:\[EAX+5\],DL  
  004B7AA7   .^ E9 BEFEFFFF   JMP FlashScr.004B796A                       跳到下面  
    
  004B7978   .  8D45 C0       LEA EAX,DWORD PTR SS:\[EBP-40\]              ;  (初始 cpu 选择)  
  004B797B   .  8B55 F4       MOV EDX,DWORD PTR SS:\[EBP-C\]  
  004B797E   .  8A12          MOV DL,BYTE PTR DS:\[EDX\]  
  004B7980   .  E8 63D0F4FF   CALL FlashScr.004049E8  
  004B7985   .  8B45 C0       MOV EAX,DWORD PTR SS:\[EBP-40\]  
  004B7988   .  E8 5F15F5FF   CALL FlashScr.00408EEC  
  004B798D   .  8D0440        LEA EAX,DWORD PTR DS:\[EAX+EAX\*2\]  
  004B7990   .  8B15 241B4C00 MOV EDX,DWORD PTR DS:\[4C1B24\]              ;  FlashScr.004C4854  
  004B7996   .  8D34C2        LEA ESI,DWORD PTR DS:\[EDX+EAX\*8\]  
  004B7999   .  8D7D D8       LEA EDI,DWORD PTR SS:\[EBP-28\]  
  004B799C   .  B9 06000000   MOV ECX,6  
  004B79A1   .  F3:A5         REP MOVS DWORD PTR ES:\[EDI\],DWORD PTR DS:\[>  
  004B79A3   .  33C0          XOR EAX,EAX  
  004B79A5   .  5A            POP EDX  
  004B79A6   .  59            POP ECX  
  004B79A7   .  59            POP ECX  
  004B79A8   .  64:8910       MOV DWORD PTR FS:\[EAX\],EDX  
  004B79AB   . /E9 FC000000   JMP FlashScr.004B7AAC                       :  此时 eax 显示假码“423679”  
    
  不管继续 F8 来到此处  
  004B7AAC   > \\BA 02000000   MOV EDX,2                                              ;  edx＝2  
  004B7AB1   .  8D45 DC       LEA EAX,DWORD PTR SS:\[EBP-24\]              ;  eax=10  
  004B7AB4   >  8B4D F4       MOV ECX,DWORD PTR SS:\[EBP-C\]               ;  按规律取的 6 位假码  
  004B7AB7   .  8A4C11 FF     MOV CL,BYTE PTR DS:\[ECX+EDX-1\]             ;  第一次取假码 2，第二次取 3，第三次取 6，第四次取 7,第五次取 9  
  004B7ABB   .  8B18          MOV EBX,DWORD PTR DS:\[EAX\]                 ;  ebx=10  
  004B7ABD   .  8B75 F8       MOV ESI,DWORD PTR SS:\[EBP-8\]  
  004B7AC0   .  3A4C1E FF     CMP CL,BYTE PTR DS:\[ESI+EBX-1\]             ; 比较所取假码的值与密文的值是否相等，此时可改 CL 值与 DS:\[XXXXXXXX\]的值相同即  
  
可继续分析  
                                                                           第一次取密文的第 16 位，第二次取第 1 位，第三次取第 22 位，第四次取第 9 位，第五次取倒数第 4 位   
  004B7AC4      74 53         JE SHORT FlashScr.004B7B19                  不等就不跳(死)，跳则循环（共判断五次）   
  004B7AC6   .  6A 10         PUSH 10  
  004B7AC8   .  68 4C7C4B00   PUSH FlashScr.004B7C4C                     ;  error  
  004B7ACD   .  68 547C4B00   PUSH FlashScr.004B7C54                     ;  register failed, please check user name and serial number.  
  004B7AD2   .  8B45 FC       MOV EAX,DWORD PTR SS:\[EBP-4\]  
  004B7AD5   .  E8 6603FBFF   CALL FlashScr.00467E40  
  004B7ADA   .  50            PUSH EAX                                   ; |hOwner  
  004B7ADB   .  E8 34F9F4FF   CALL <JMP.&user32.MessageBoxA>             ; \\MessageBoxA  
  004B7AE0   .  A1 B41D4C00   MOV EAX,DWORD PTR DS:\[4C1DB4\]  
  004B7AE5   .  E8 16CDF4FF   CALL FlashScr.00404800  
  004B7AEA   .  A1 3C1D4C00   MOV EAX,DWORD PTR DS:\[4C1D3C\]  
  004B7AEF   .  E8 0CCDF4FF   CALL FlashScr.00404800  
  004B7AF4   .  8B45 FC       MOV EAX,DWORD PTR SS:\[EBP-4\]  
  004B7AF7   .  8B80 FC020000 MOV EAX,DWORD PTR DS:\[EAX+2FC\]  
  004B7AFD   .  66:BE B5FF    MOV SI,0FFB5  
  004B7B01   .  E8 C2C0F4FF   CALL FlashScr.00403BC8  
  004B7B06   .  8B45 FC       MOV EAX,DWORD PTR SS:\[EBP-4\]  
  004B7B09   .  8B80 FC020000 MOV EAX,DWORD PTR DS:\[EAX+2FC\]  
  004B7B0F   .  E8 E4E2F9FF   CALL FlashScr.00455DF8  
  004B7B14   .  E9 FB000000   JMP FlashScr.004B7C14  
  004B7B19   >  42            INC EDX                                    ;  edx+1  
  004B7B1A   .  83C0 04       ADD EAX,4                                  ;  eax+4  
  004B7B1D   .  83FA 07       CMP EDX,7  
  004B7B20   .^ 75 92         JNZ SHORT FlashScr.004B7AB4                ;  循环判断  
  004B7B22   .  B2 01         MOV DL,1  
  004B7B20   .^\\75 92         JNZ SHORT FlashScr.004B7AB4                ;  循环判断  
  004B7B22   .  B2 01         MOV DL,1  
  004B7B24   .  A1 A0AD4300   MOV EAX,DWORD PTR DS:\[43ADA0\]  
  004B7B29   .  E8 7233F8FF   CALL FlashScr.0043AEA0  
  004B7B2E   .  8945 F0       MOV DWORD PTR SS:\[EBP-10\],EAX  
  004B7B31   .  33C0          XOR EAX,EAX  
  004B7B33   .  55            PUSH EBP  
  004B7B34   .  68 AA7B4B00   PUSH FlashScr.004B7BAA  
  004B7B39   .  64:FF30       PUSH DWORD PTR FS:\[EAX\]  
  004B7B3C   .  64:8920       MOV DWORD PTR FS:\[EAX\],ESP  
  004B7B3F   .  BA 02000080   MOV EDX,80000002  
  004B7B44   .  8B45 F0       MOV EAX,DWORD PTR SS:\[EBP-10\]  
  004B7B47   .  E8 F433F8FF   CALL FlashScr.0043AF40  
  004B7B4C   .  8B15 6C1B4C00 MOV EDX,DWORD PTR DS:\[4C1B6C\]              ;  FlashScr.004C18B0  
  004B7B52   .  8B12          MOV EDX,DWORD PTR DS:\[EDX\]  
  004B7B54   .  B1 01         MOV CL,1  
  004B7B56   .  8B45 F0       MOV EAX,DWORD PTR SS:\[EBP-10\]  
  004B7B59   .  E8 2235F8FF   CALL FlashScr.0043B080  
  004B7B5E   .  84C0          TEST AL,AL  
  004B7B60   .  74 32         JE SHORT FlashScr.004B7B94  
  004B7B62   .  8B0D B41D4C00 MOV ECX,DWORD PTR DS:\[4C1DB4\]              ;  FlashScr.004C18B8  
  004B7B68   .  8B09          MOV ECX,DWORD PTR DS:\[ECX\]  
  004B7B6A   .  BA 987C4B00   MOV EDX,FlashScr.004B7C98                  ;  username(在注册表中写入注册名)  
  004B7B6F   .  8B45 F0       MOV EAX,DWORD PTR SS:\[EBP-10\]  
  004B7B72   .  E8 A536F8FF   CALL FlashScr.0043B21C  
  004B7B77   .  8B0D 3C1D4C00 MOV ECX,DWORD PTR DS:\[4C1D3C\]              ;  FlashScr.004C18BC  
  004B7B7D   .  8B09          MOV ECX,DWORD PTR DS:\[ECX\]  
  004B7B7F   .  BA AC7C4B00   MOV EDX,FlashScr.004B7CAC                  ;  serialnumber(在注册表中写入注册码)  
    
    
  
\--------------------------------------------------------------------------------  
【经验总结】  
  注册算法(比较简单)：  
  1、密文＝MD5 加密（注册名+固定字符“Flash ScreenSaver Maker”）  
  2、注册码的第 2 位应等于密文的第 16 位  
  3、注册码的第 3 位应等于密文的第 1 位  
  4、注册码的第六位应等于密文的第 22 位  
  5、注册码的第 7 位应等于密文的第 9 位  
  6、注册码的第 9 位应等于密文的倒数第 4 位  
  7、注册码的第 4 位、对应软件的 4 个的权限，数值不同，算法各异，可自行分析，以上算法是注册码第 4 位为 4 的情况。  
  8、注册机就不写了，用个 MD5 的计算工具，依次选一下就行了  
  可用的注册 User Name:ww9900 Serial Number:1b9-45cd-8e0  
    
                        ww990  
    
\--------------------------------------------------------------------------------  
【版权声明】: 本文原创于看雪技术论坛, 转载请注明作者并保持文章的完整, 谢谢!  
  
                                                       2006 年 05 月 19 日 14:51:45  
  
BWT：放个可用的注册机 delhpi 源代码 

源码清单打包！![](img/45777b5dbd6b901752ab7df42fce6892.jpg)   
  
'窗体部分：  
  
Private Sub Command1\_Click()  
Dim s1, s2, s3, s4, s5  
Dim xmd5  
Dim S, char  
  
'用户名与固定字符串加密变形  
  
Set c1 = New clsMD5  
xmd5 = c1.Md5\_String\_Calc(UCase(Text1.Text) + "Flash ScreenSaver Maker")  
  
'计算注册码的第 2 位、第 3 位、第 6 位、第 7 位、第 9 位  
  
s1 = Mid(xmd5, 16, 1)    '第 2 位  
s2 = Mid(xmd5, 1, 1)     '第 3 位  
s3 = Mid(xmd5, 22, 1)    '第 6 位  
s4 = Mid(xmd5, 9, 1)     '第 7 位  
s5 = Mid(xmd5, 29, 1)      '第 9 位  
  
'注册码的第 1 位、第 4 位、第 5 位、第 8 位、第 10 位可为任意  
  
S = "8"  
  
'连接符号“-”  
  
char = "-"  
  
'注册码类型：XXX-XXXX-XXX  
  
Text2.Text = LCase(S & s1 & s2 & char & S & S & s3 & s4 & char & S & s5 & S)  
  
End Sub  
  
'类模块部分：\[名称:clsMD5\]  
  
Option Explicit  
  
' Visual Basic MD5 Implementation  
' Robert Hubley and David Midkiff  
' modify by simonyan, Support chinese  
' Standard MD5 implementation optimised for the Visual Basic environment.  
' Conforms to all standards and can be used in digital signature or password  
' protection related schemes.  
  
Private Const OFFSET\_4 = 4294967296#  
Private Const MAXINT\_4 = 2147483647  
Private State(4) As Long  
Private ByteCounter As Long  
Private ByteBuffer(63) As Byte  
Private Const S11 = 7  
Private Const S12 = 12  
Private Const S13 = 17  
Private Const S14 = 22  
Private Const S21 = 5  
Private Const S22 = 9  
Private Const S23 = 14  
Private Const S24 = 20  
Private Const S31 = 4  
Private Const S32 = 11  
Private Const S33 = 16  
Private Const S34 = 23  
Private Const S41 = 6  
Private Const S42 = 10  
Private Const S43 = 15  
Private Const S44 = 21  
Property Get RegisterA() As String  
    RegisterA = State(1)  
End Property  
Property Get RegisterB() As String  
    RegisterB = State(2)  
End Property  
  
Property Get RegisterC() As String  
    RegisterC = State(3)  
End Property  
  
Property Get RegisterD() As String  
    RegisterD = State(4)  
End Property  
Public Function Md5\_String\_Calc(SourceString As String) As String  
    MD5Init  
    MD5Update LenB(StrConv(SourceString, vbFromUnicode)), StringToArray(SourceString)  
    MD5Final  
    Md5\_String\_Calc = GetValues  
End Function  
Public Function Md5\_File\_Calc(InFile As String) As String  
  
GoSub begin  
  
begin:  
    Dim FileO As Integer  
    FileO = FreeFile  
    Call FileLen(InFile)  
    Open InFile For Binary Access Read As #FileO  
    MD5Init  
    Do While Not EOF(FileO)  
        Get #FileO, , ByteBuffer  
        If Loc(FileO) < LOF(FileO) Then  
            ByteCounter = ByteCounter + 64  
            MD5Transform ByteBuffer  
        End If  
    Loop  
    ByteCounter = ByteCounter + (LOF(FileO) Mod 64)  
    Close #FileO  
    MD5Final  
    Md5\_File\_Calc = GetValues  
End Function  
Private Function StringToArray(InString As String) As Byte()  
    Dim I As Integer, bytBuffer() As Byte  
    ReDim bytBuffer(LenB(StrConv(InString, vbFromUnicode)))  
    bytBuffer = StrConv(InString, vbFromUnicode)  
    StringToArray = bytBuffer  
End Function  
Public Function GetValues() As String  
    GetValues = LongToString(State(1)) & LongToString(State(2)) & LongToString(State(3)) & LongToString(State(4))  
End Function  
Private Function LongToString(Num As Long) As String  
        Dim A As Byte, B As Byte, C As Byte, D As Byte  
        A = Num And &HFF&  
        If A < 16 Then LongToString = "0" & Hex(A) Else LongToString = Hex(A)  
        B = (Num And &HFF00&) \\ 256  
        If B < 16 Then LongToString = LongToString & "0" & Hex(B) Else LongToString = LongToString & Hex(B)  
        C = (Num And &HFF0000) \\ 65536  
        If C < 16 Then LongToString = LongToString & "0" & Hex(C) Else LongToString = LongToString & Hex(C)  
        If Num < 0 Then D = ((Num And &H7F000000) \\ 16777216) Or &H80& Else D = (Num And &HFF000000) \\ 16777216  
        If D < 16 Then LongToString = LongToString & "0" & Hex(D) Else LongToString = LongToString & Hex(D)  
End Function  
  
Public Sub MD5Init()  
    ByteCounter = 0  
    State(1) = UnsignedToLong(1732584193#)  
    State(2) = UnsignedToLong(4023233417#)  
    State(3) = UnsignedToLong(2562383102#)  
    State(4) = UnsignedToLong(271733878#)  
End Sub  
  
Public Sub MD5Final()  
    Dim dblBits As Double, padding(72) As Byte, lngBytesBuffered As Long  
    padding(0) = &H80  
    dblBits = ByteCounter \* 8  
    lngBytesBuffered = ByteCounter Mod 64  
    If lngBytesBuffered <= 56 Then MD5Update 56 - lngBytesBuffered, padding Else MD5Update 120 - ByteCounter, padding  
    padding(0) = UnsignedToLong(dblBits) And &HFF&  
    padding(1) = UnsignedToLong(dblBits) \\ 256 And &HFF&  
    padding(2) = UnsignedToLong(dblBits) \\ 65536 And &HFF&  
    padding(3) = UnsignedToLong(dblBits) \\ 16777216 And &HFF&  
    padding(4) = 0  
    padding(5) = 0  
    padding(6) = 0  
    padding(7) = 0  
    MD5Update 8, padding  
End Sub  
Public Sub MD5Update(InputLen As Long, InputBuffer() As Byte)  
    Dim II As Integer, I As Integer, J As Integer, K As Integer, lngBufferedBytes As Long, lngBufferRemaining As Long, lngRem As Long  
  
    lngBufferedBytes = ByteCounter Mod 64  
    lngBufferRemaining = 64 - lngBufferedBytes  
    ByteCounter = ByteCounter + InputLen  
  
    If InputLen >= lngBufferRemaining Then  
        For II = 0 To lngBufferRemaining - 1  
            ByteBuffer(lngBufferedBytes + II) = InputBuffer(II)  
        Next II  
        MD5Transform ByteBuffer  
        lngRem = (InputLen) Mod 64  
        For I = lngBufferRemaining To InputLen - II - lngRem Step 64  
            For J = 0 To 63  
                ByteBuffer(J) = InputBuffer(I + J)  
            Next J  
            MD5Transform ByteBuffer  
        Next I  
        lngBufferedBytes = 0  
    Else  
      I = 0  
    End If  
    For K = 0 To InputLen - I - 1  
        ByteBuffer(lngBufferedBytes + K) = InputBuffer(I + K)  
    Next K  
End Sub  
Private Sub MD5Transform(Buffer() As Byte)  
    Dim x(16) As Long, A As Long, B As Long, C As Long, D As Long  
      
    A = State(1)  
    B = State(2)  
    C = State(3)  
    D = State(4)  
    Decode 64, x, Buffer  
    FF A, B, C, D, x(0), S11, -680876936  
    FF D, A, B, C, x(1), S12, -389564586  
    FF C, D, A, B, x(2), S13, 606105819  
    FF B, C, D, A, x(3), S14, -1044525330  
    FF A, B, C, D, x(4), S11, -176418897  
    FF D, A, B, C, x(5), S12, 1200080426  
    FF C, D, A, B, x(6), S13, -1473231341  
    FF B, C, D, A, x(7), S14, -45705983  
    FF A, B, C, D, x(8), S11, 1770035416  
    FF D, A, B, C, x(9), S12, -1958414417  
    FF C, D, A, B, x(10), S13, -42063  
    FF B, C, D, A, x(11), S14, -1990404162  
    FF A, B, C, D, x(12), S11, 1804603682  
    FF D, A, B, C, x(13), S12, -40341101  
    FF C, D, A, B, x(14), S13, -1502002290  
    FF B, C, D, A, x(15), S14, 1236535329  
  
    GG A, B, C, D, x(1), S21, -165796510  
    GG D, A, B, C, x(6), S22, -1069501632  
    GG C, D, A, B, x(11), S23, 643717713  
    GG B, C, D, A, x(0), S24, -373897302  
    GG A, B, C, D, x(5), S21, -701558691  
    GG D, A, B, C, x(10), S22, 38016083  
    GG C, D, A, B, x(15), S23, -660478335  
    GG B, C, D, A, x(4), S24, -405537848  
    GG A, B, C, D, x(9), S21, 568446438  
    GG D, A, B, C, x(14), S22, -1019803690  
    GG C, D, A, B, x(3), S23, -187363961  
    GG B, C, D, A, x(8), S24, 1163531501  
    GG A, B, C, D, x(13), S21, -1444681467  
    GG D, A, B, C, x(2), S22, -51403784  
    GG C, D, A, B, x(7), S23, 1735328473  
    GG B, C, D, A, x(12), S24, -1926607734  
  
    HH A, B, C, D, x(5), S31, -378558  
    HH D, A, B, C, x(8), S32, -2022574463  
    HH C, D, A, B, x(11), S33, 1839030562  
    HH B, C, D, A, x(14), S34, -35309556  
    HH A, B, C, D, x(1), S31, -1530992060  
    HH D, A, B, C, x(4), S32, 1272893353  
    HH C, D, A, B, x(7), S33, -155497632  
    HH B, C, D, A, x(10), S34, -1094730640  
    HH A, B, C, D, x(13), S31, 681279174  
    HH D, A, B, C, x(0), S32, -358537222  
    HH C, D, A, B, x(3), S33, -722521979  
    HH B, C, D, A, x(6), S34, 76029189  
    HH A, B, C, D, x(9), S31, -640364487  
    HH D, A, B, C, x(12), S32, -421815835  
    HH C, D, A, B, x(15), S33, 530742520  
    HH B, C, D, A, x(2), S34, -995338651  
  
    II A, B, C, D, x(0), S41, -198630844  
    II D, A, B, C, x(7), S42, 1126891415  
    II C, D, A, B, x(14), S43, -1416354905  
    II B, C, D, A, x(5), S44, -57434055  
    II A, B, C, D, x(12), S41, 1700485571  
    II D, A, B, C, x(3), S42, -1894986606  
    II C, D, A, B, x(10), S43, -1051523  
    II B, C, D, A, x(1), S44, -2054922799  
    II A, B, C, D, x(8), S41, 1873313359  
    II D, A, B, C, x(15), S42, -30611744  
    II C, D, A, B, x(6), S43, -1560198380  
    II B, C, D, A, x(13), S44, 1309151649  
    II A, B, C, D, x(4), S41, -145523070  
    II D, A, B, C, x(11), S42, -1120210379  
    II C, D, A, B, x(2), S43, 718787259  
    II B, C, D, A, x(9), S44, -343485551  
  
    State(1) = LongOverflowAdd(State(1), A)  
    State(2) = LongOverflowAdd(State(2), B)  
    State(3) = LongOverflowAdd(State(3), C)  
    State(4) = LongOverflowAdd(State(4), D)  
End Sub  
  
Private Sub Decode(Length As Integer, OutputBuffer() As Long, InputBuffer() As Byte)  
    Dim intDblIndex As Integer, intByteIndex As Integer, dblSum As Double  
    For intByteIndex = 0 To Length - 1 Step 4  
        dblSum = InputBuffer(intByteIndex) + InputBuffer(intByteIndex + 1) \* 256# + InputBuffer(intByteIndex + 2) \* 65536# + InputBuffer(intByteIndex + 3) \* 16777216#  
        OutputBuffer(intDblIndex) = UnsignedToLong(dblSum)  
        intDblIndex = intDblIndex + 1  
    Next intByteIndex  
End Sub  
Private Function FF(A As Long, B As Long, C As Long, D As Long, x As Long, S As Long, ac As Long) As Long  
    A = LongOverflowAdd4(A, (B And C) Or (Not (B) And D), x, ac)  
    A = LongLeftRotate(A, S)  
    A = LongOverflowAdd(A, B)  
End Function  
Private Function GG(A As Long, B As Long, C As Long, D As Long, x As Long, S As Long, ac As Long) As Long  
    A = LongOverflowAdd4(A, (B And D) Or (C And Not (D)), x, ac)  
    A = LongLeftRotate(A, S)  
    A = LongOverflowAdd(A, B)  
End Function  
Private Function HH(A As Long, B As Long, C As Long, D As Long, x As Long, S As Long, ac As Long) As Long  
    A = LongOverflowAdd4(A, B Xor C Xor D, x, ac)  
    A = LongLeftRotate(A, S)  
    A = LongOverflowAdd(A, B)  
End Function  
Private Function II(A As Long, B As Long, C As Long, D As Long, x As Long, S As Long, ac As Long) As Long  
    A = LongOverflowAdd4(A, C Xor (B Or Not (D)), x, ac)  
    A = LongLeftRotate(A, S)  
    A = LongOverflowAdd(A, B)  
End Function  
  
Function LongLeftRotate(value As Long, Bits As Long) As Long  
    Dim lngSign As Long, lngI As Long  
    Bits = Bits Mod 32  
    If Bits = 0 Then LongLeftRotate = value: Exit Function  
    For lngI = 1 To Bits  
        lngSign = value And &HC0000000  
        value = (value And &H3FFFFFFF) \* 2  
        value = value Or ((lngSign < 0) And 1) Or (CBool(lngSign And &H40000000) And &H80000000)  
    Next  
    LongLeftRotate = value  
End Function  
Private Function LongOverflowAdd(Val1 As Long, Val2 As Long) As Long  
    Dim lngHighWord As Long, lngLowWord As Long, lngOverflow As Long  
    lngLowWord = (Val1 And &HFFFF&) + (Val2 And &HFFFF&)  
    lngOverflow = lngLowWord \\ 65536  
    lngHighWord = (((Val1 And &HFFFF0000) \\ 65536) + ((Val2 And &HFFFF0000) \\ 65536) + lngOverflow) And &HFFFF&  
    LongOverflowAdd = UnsignedToLong((lngHighWord \* 65536#) + (lngLowWord And &HFFFF&))  
End Function  
Private Function LongOverflowAdd4(Val1 As Long, Val2 As Long, val3 As Long, val4 As Long) As Long  
    Dim lngHighWord As Long, lngLowWord As Long, lngOverflow As Long  
    lngLowWord = (Val1 And &HFFFF&) + (Val2 And &HFFFF&) + (val3 And &HFFFF&) + (val4 And &HFFFF&)  
    lngOverflow = lngLowWord \\ 65536  
    lngHighWord = (((Val1 And &HFFFF0000) \\ 65536) + ((Val2 And &HFFFF0000) \\ 65536) + ((val3 And &HFFFF0000) \\ 65536) + ((val4 And &HFFFF0000) \\ 65536) + lngOverflow) And &HFFFF&  
    LongOverflowAdd4 = UnsignedToLong((lngHighWord \* 65536#) + (lngLowWord And &HFFFF&))  
End Function  
  
Private Function UnsignedToLong(value As Double) As Long  
    If value < 0 Or value >= OFFSET\_4 Then Error 6  
    If value <= MAXINT\_4 Then UnsignedToLong = value Else UnsignedToLong = value - OFFSET\_4  
End Function  
Private Function LongToUnsigned(value As Long) As Double  
    If value < 0 Then LongToUnsigned = value + OFFSET\_4 Else LongToUnsigned = value  
End Function