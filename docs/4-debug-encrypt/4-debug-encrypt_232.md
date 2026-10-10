

# 流星网络电视 V1.9.2.0 的注册

> 原文：[`www.pediy.com/kssd/pediy07/pediy7-367.htm`](https://www.pediy.com/kssd/pediy07/pediy7-367.htm)

【破文作者】   rdsnow\[BCG\]\[PYG\]\[D.4s\]  
【作者主页】   http://rdsnow.ys168.com  
【 E-mail 】   rdsnow@163.com  
【 作者 QQ 】   83757177  
【文章题目】   流星网络电视 V1.9.2.0 的注册  
【软件名称】   流星网络电视 1.9.2.0  
【下载地址】   http://www.jesen.cn  
  
【文章简介】  
  
这个程序原先版本加了 Asprotect 2.11 的加密外壳的，一直没有能够脱壳，不知道什么原因，新版本的程序换成 Aspack 2.12 的压缩壳，手脱或用工具都可以轻松搞定。  
  
程序采用了现在流行的重启验证的方式。  
  
加密过程中主要使用到了两个加密算法：MD5 和 DES，都没有变形。其中 DES 有两个函数，DES\_EN( )和 DES\_DE( )，分别用来加密信息和解密信息，使用同一个密钥。  
  
\----------------------------------------------------------------------------------------------  
【破解过程】  
  
启动程序，输入注册名和假码，来到这里：(注册码的长度是 16、20、24 或 32)  
  
005BD888  |.  55              PUSH EBP  
005BD889  |.  68 F5DA5B00     PUSH unpacked.005BDAF5  
005BD88E  |.  64:FF30         PUSH DWORD PTR FS:\[EAX\]  
005BD891  |.  64:8920         MOV DWORD PTR FS:\[EAX\],ESP  
005BD894  |.  8D55 EC         LEA EDX,DWORD PTR SS:\[EBP-14\]  
005BD897  |.  8B83 14030000   MOV EAX,DWORD PTR DS:\[EBX+314\]  
005BD89D  |.  E8 7275E9FF     CALL unpacked.00454E14           ;  取出输入的用户名  
005BD8A2  |.  8B45 EC         MOV EAX,DWORD PTR SS:\[EBP-14\]  
005BD8A5  |.  8D55 F0         LEA EDX,DWORD PTR SS:\[EBP-10\]  
005BD8A8  |.  E8 CFB5E4FF     CALL unpacked.00408E7C  
005BD8AD  |.  837D F0 00      CMP DWORD PTR SS:\[EBP-10\],0      ;  检查是否成功取出  
005BD8B1  |.  0F84 E9010000   JE unpacked.005BDAA0  
005BD8B7  |.  8D55 E4         LEA EDX,DWORD PTR SS:\[EBP-1C\]  
005BD8BA  |.  8B83 18030000   MOV EAX,DWORD PTR DS:\[EBX+318\]  
005BD8C0  |.  E8 4F75E9FF     CALL unpacked.00454E14           ;  取出输入的注册码  
005BD8C5  |.  8B45 E4         MOV EAX,DWORD PTR SS:\[EBP-1C\]  
005BD8C8  |.  8D55 E8         LEA EDX,DWORD PTR SS:\[EBP-18\]  
005BD8CB  |.  E8 ACB5E4FF     CALL unpacked.00408E7C  
005BD8D0  |.  837D E8 00      CMP DWORD PTR SS:\[EBP-18\],0      ;  检查是否成功取出  
005BD8D4  |.  0F84 C6010000   JE unpacked.005BDAA0  
005BD8DA  |.  8D55 E0         LEA EDX,DWORD PTR SS:\[EBP-20\]  
005BD8DD  |.  8B83 14030000   MOV EAX,DWORD PTR DS:\[EBX+314\]  
005BD8E3  |.  E8 2C75E9FF     CALL unpacked.00454E14           ;  计算用户名的长度  
005BD8E8  |.  8B45 E0         MOV EAX,DWORD PTR SS:\[EBP-20\]  
005BD8EB  |.  8D55 F8         LEA EDX,DWORD PTR SS:\[EBP-8\]  
005BD8EE  |.  E8 89B5E4FF     CALL unpacked.00408E7C  
005BD8F3  |.  8D55 D8         LEA EDX,DWORD PTR SS:\[EBP-28\]    ;  下面开始对注册码变换  
005BD8F6  |.  8B83 18030000   MOV EAX,DWORD PTR DS:\[EBX+318\]  
005BD8FC  |.  E8 1375E9FF     CALL unpacked.00454E14           ;  取出注册码的长度  
005BD901  |.  8B45 D8         MOV EAX,DWORD PTR SS:\[EBP-28\]  
005BD904  |.  8D55 DC         LEA EDX,DWORD PTR SS:\[EBP-24\]  
005BD907  |.  E8 70B5E4FF     CALL unpacked.00408E7C  
005BD90C  |.  8B45 DC         MOV EAX,DWORD PTR SS:\[EBP-24\]  
005BD90F  |.  8D55 FC         LEA EDX,DWORD PTR SS:\[EBP-4\]  
005BD912  |.  E8 11B3E4FF     CALL unpacked.00408C28           ;  注册码大写转为小写  
005BD917  |.  8D55 D4         LEA EDX,DWORD PTR SS:\[EBP-2C\]  
005BD91A  |.  8B45 FC         MOV EAX,DWORD PTR SS:\[EBP-4\]  
005BD91D  |.  E8 52A5EBFF     CALL unpacked.00477E74           ;  对注册码采用 MD5 运算  
005BD922  |.  8B45 D4         MOV EAX,DWORD PTR SS:\[EBP-2C\]  
005BD925  |.  8D4D F4         LEA ECX,DWORD PTR SS:\[EBP-C\]  
005BD928  |.  BA 0CDB5B00     MOV EDX,unpacked.005BDB0C        ;  ASCII "dream"  
005BD92D  |.  E8 5697EBFF     CALL unpacked.00477088           ;  再经过 DES 加密，Key＝"dream"因为密钥至少要有 64 位，不够用 0x00 替代  
005BD932  |.  8B45 FC         MOV EAX,DWORD PTR SS:\[EBP-4\]  
005BD935  |.  E8 D66DE4FF     CALL unpacked.00404710           ;  计算输入注册码的长度  
005BD93A  |.  83E8 10         SUB EAX,10                       ;  Switch (cases 10..20)  
005BD93D  |.  74 11           JE SHORT unpacked.005BD950  
005BD93F  |.  83E8 04         SUB EAX,4  
005BD942  |.  74 26           JE SHORT unpacked.005BD96A  
005BD944  |.  83E8 04         SUB EAX,4  
005BD947  |.  74 3B           JE SHORT unpacked.005BD984  
005BD949  |.  83E8 08         SUB EAX,8  
005BD94C  |.  74 50           JE SHORT unpacked.005BD99E  
005BD94E  |.  EB 68           JMP SHORT unpacked.005BD9B8  
005BD950  |>  A1 586A5C00     MOV EAX,DWORD PTR DS:\[5C6A58\]    ;  Case 10 of switch 005BD93A  
005BD955  |.  50              PUSH EAX  
005BD956  |.  B9 0A000000     MOV ECX,0A  
005BD95B  |.  BA 06000000     MOV EDX,6  
005BD960  |.  8B45 F4         MOV EAX,DWORD PTR SS:\[EBP-C\]  
005BD963  |.  E8 0870E4FF     CALL unpacked.00404970           ;  如果输入 16 位注册码，从 DES 结果的第 6 位起取 10 个字符  
005BD968  |.  EB 6B           JMP SHORT unpacked.005BD9D5  
005BD96A  |>  A1 586A5C00     MOV EAX,DWORD PTR DS:\[5C6A58\]    ;  Case 14 of switch 005BD93A  
005BD96F  |.  50              PUSH EAX  
005BD970  |.  B9 0E000000     MOV ECX,0E  
005BD975  |.  BA 03000000     MOV EDX,3  
005BD97A  |.  8B45 F4         MOV EAX,DWORD PTR SS:\[EBP-C\]  
005BD97D  |.  E8 EE6FE4FF     CALL unpacked.00404970           ;  如果输入 20 位注册码，从 DES 结果的第 3 位起取 14 个字符  
005BD982  |.  EB 51           JMP SHORT unpacked.005BD9D5  
005BD984  |>  A1 586A5C00     MOV EAX,DWORD PTR DS:\[5C6A58\]    ;  Case 18 of switch 005BD93A  
005BD989  |.  50              PUSH EAX  
005BD98A  |.  B9 10000000     MOV ECX,10  
005BD98F  |.  BA 05000000     MOV EDX,5  
005BD994  |.  8B45 F4         MOV EAX,DWORD PTR SS:\[EBP-C\]  
005BD997  |.  E8 D46FE4FF     CALL unpacked.00404970           ;  如果输入 24 位注册码，从 DES 结果的第 5 位起取 16 个字符  
005BD99C  |.  EB 37           JMP SHORT unpacked.005BD9D5  
005BD99E  |>  A1 586A5C00     MOV EAX,DWORD PTR DS:\[5C6A58\]    ;  Case 20 of switch 005BD93A  
005BD9A3  |.  50              PUSH EAX  
005BD9A4  |.  B9 0C000000     MOV ECX,0C  
005BD9A9  |.  BA 0C000000     MOV EDX,0C  
005BD9AE  |.  8B45 F4         MOV EAX,DWORD PTR SS:\[EBP-C\]  
005BD9B1  |.  E8 BA6FE4FF     CALL unpacked.00404970           ;  如果输入 32 位注册码，从 DES 结果的第 12 位起取 32 个字符  
005BD9B6  |.  EB 1D           JMP SHORT unpacked.005BD9D5      ;  以上已经完成了对注册码的处理，取出的字符等待存入注册表  
005BD9B8  |>  6A 30           PUSH 30                          ;  Default case of switch 005BD93A  
005BD9BA  |.  B9 14DB5B00     MOV ECX,unpacked.005BDB14        ;  如果输入的注册码不是以上长度，就到这里，OVER 了  
005BD9BF  |.  BA 20DB5B00     MOV EDX,unpacked.005BDB20  
005BD9C4  |.  A1 A46C5C00     MOV EAX,DWORD PTR DS:\[5C6CA4\]  
005BD9C9  |.  8B00            MOV EAX,DWORD PTR DS:\[EAX\]  
005BD9CB  |.  E8 A477EBFF     CALL unpacked.00475174  
005BD9D0  |.  E9 CB000000     JMP unpacked.005BDAA0            ;  这个 JMP，跳向注册码错误的对话框  
005BD9D5  |>  B2 01           MOV DL,1  
005BD9D7  |.  A1 FCD34300     MOV EAX,DWORD PTR DS:\[43D3FC\]  
005BD9DC  |.  E8 1BFBE7FF     CALL unpacked.0043D4FC  
005BD9E1  |.  8BD8            MOV EBX,EAX  
005BD9E3  |.  BA 02000080     MOV EDX,80000002  
005BD9E8  |.  8BC3            MOV EAX,EBX  
005BD9EA  |.  E8 ADFBE7FF     CALL unpacked.0043D59C  
005BD9EF  |.  B1 01           MOV CL,1  
005BD9F1  |.  BA 48DB5B00     MOV EDX,unpacked.005BDB48        ;  ASCII "\\SOFTWARE\\Microsoft\\Windows\\CurrentVersion\\zlb"  
005BD9F6  |.  8BC3            MOV EAX,EBX  
005BD9F8  |.  E8 07FCE7FF     CALL unpacked.0043D604  
005BD9FD  |.  84C0            TEST AL,AL  
005BD9FF  |.  74 14           JE SHORT unpacked.005BDA15  
005BDA01  |.  8B0D 586A5C00   MOV ECX,DWORD PTR DS:\[5C6A58\]    ;  unpacked.005C92E0  
005BDA07  |.  8B09            MOV ECX,DWORD PTR DS:\[ECX\]  
005BDA09  |.  BA 0CDB5B00     MOV EDX,unpacked.005BDB0C        ;  ASCII "dream"  
005BDA0E  |.  8BC3            MOV EAX,EBX  
005BDA10  |.  E8 3300E8FF     CALL unpacked.0043DA48           ;  下面代码是将注册码的处理结果存入注册表中  
005BDA15  |>  8BC3            MOV EAX,EBX  
005BDA17  |.  E8 50FBE7FF     CALL unpacked.0043D56C  
005BDA1C  |.  8BC3            MOV EAX,EBX  
005BDA1E  |.  E8 E15BE4FF     CALL unpacked.00403604  
  
点击 OK 后重新启动程序，找到字符串"dream"或"whole"来到这儿：  
  
005BAB6C   .  B1 01           MOV CL,1  
005BAB6E   .  BA C0B15B00     MOV EDX,unpacked.005BB1C0        ;  ASCII "\\SOFTWARE\\Microsoft\\Windows\\CurrentVersion\\zlb"  
005BAB73   .  8BC3            MOV EAX,EBX  
005BAB75   .  E8 8A2AE8FF     CALL unpacked.0043D604  
005BAB7A   .  84C0            TEST AL,AL  
005BAB7C   .  74 1F           JE SHORT unpacked.005BAB9D  
005BAB7E   .  8D4D D4         LEA ECX,DWORD PTR SS:\[EBP-2C\]  
005BAB81   .  BA F8B15B00     MOV EDX,unpacked.005BB1F8        ;  ASCII "dream"  
005BAB86   .  8BC3            MOV EAX,EBX  
005BAB88   .  E8 E72EE8FF     CALL unpacked.0043DA74           ;  从注册表中取出保存的注册码的信息  
005BAB8D   .  8B55 D4         MOV EDX,DWORD PTR SS:\[EBP-2C\]  
005BAB90   .  8B45 FC         MOV EAX,DWORD PTR SS:\[EBP-4\]  
005BAB93   .  05 00040000     ADD EAX,400  
005BAB98   .  E8 EB98E4FF     CALL unpacked.00404488  
005BAB9D   >  B1 01           MOV CL,1  
005BAB9F   .  BA 08B25B00     MOV EDX,unpacked.005BB208        ;  ASCII "\\SOFTWARE\\Microsoft\\Internet Explorer\\ActiveX Compatibility"  
005BABA4   .  8BC3            MOV EAX,EBX  
005BABA6   .  E8 592AE8FF     CALL unpacked.0043D604  
005BABAB   .  84C0            TEST AL,AL  
005BABAD   .  74 1C           JE SHORT unpacked.005BABCB  
005BABAF   .  BA 4CB25B00     MOV EDX,unpacked.005BB24C        ;  ASCII "{6BF52A52-394A-11D3-B153-00C04F79FAA6}"  
005BABB4   .  8BC3            MOV EAX,EBX  
005BABB6   .  E8 A530E8FF     CALL unpacked.0043DC60  
005BABBB   .  84C0            TEST AL,AL  
005BABBD   .  74 0C           JE SHORT unpacked.005BABCB  
005BABBF   .  BA 4CB25B00     MOV EDX,unpacked.005BB24C        ;  ASCII "{6BF52A52-394A-11D3-B153-00C04F79FAA6}"  
005BABC4   .  8BC3            MOV EAX,EBX  
005BABC6   .  E8 5D2BE8FF     CALL unpacked.0043D728  
005BABCB   >  BA 01000080     MOV EDX,80000001  
005BABD0   .  8BC3            MOV EAX,EBX  
005BABD2   .  E8 C529E8FF     CALL unpacked.0043D59C  
005BABD7   .  B1 01           MOV CL,1  
005BABD9   .  BA 7CB25B00     MOV EDX,unpacked.005BB27C        ;  ASCII "\\SoftWare\\Microsoft\\Windows\\CurrentVersion\\polly"  
005BABDE   .  8BC3            MOV EAX,EBX  
005BABE0   .  E8 1F2AE8FF     CALL unpacked.0043D604  
005BABE5   .  84C0            TEST AL,AL  
005BABE7   .  74 1F           JE SHORT unpacked.005BAC08  
005BABE9   .  8D4D D0         LEA ECX,DWORD PTR SS:\[EBP-30\]  
005BABEC   .  BA B8B25B00     MOV EDX,unpacked.005BB2B8        ;  ASCII "whole"  
005BABF1   .  8BC3            MOV EAX,EBX  
005BABF3   .  E8 7C2EE8FF     CALL unpacked.0043DA74           ;  从注册表中取出保存用户名的信息  
005BABF8   .  8B55 D0         MOV EDX,DWORD PTR SS:\[EBP-30\]  
005BABFB   .  8B45 FC         MOV EAX,DWORD PTR SS:\[EBP-4\]  
005BABFE   .  05 04040000     ADD EAX,404  
005BAC03   .  E8 8098E4FF     CALL unpacked.00404488  
005BAC08   >  8BC3            MOV EAX,EBX  
005BAC0A   .  E8 5D29E8FF     CALL unpacked.0043D56C  
005BAC0F   .  8BC3            MOV EAX,EBX  
005BAC11   .  E8 EE89E4FF     CALL unpacked.00403604  
005BAC16   .  8D4D CC         LEA ECX,DWORD PTR SS:\[EBP-34\]  
005BAC19   .  8B45 FC         MOV EAX,DWORD PTR SS:\[EBP-4\]  
005BAC1C   .  8B80 04040000   MOV EAX,DWORD PTR DS:\[EAX+404\]  
005BAC22   .  BA C8B25B00     MOV EDX,unpacked.005BB2C8        ;  ASCII "zlbpolly"  
005BAC27   .  E8 44C6EBFF     CALL unpacked.00477270           ;  用 DES\_DE( )解密用户名，将用户名还原出来  
005BAC2C   .  8B55 CC         MOV EDX,DWORD PTR SS:\[EBP-34\]  
005BAC2F   .  8B45 FC         MOV EAX,DWORD PTR SS:\[EBP-4\]  
005BAC32   .  05 04040000     ADD EAX,404  
005BAC37   .  E8 4C98E4FF     CALL unpacked.00404488  
005BAC3C   .  8B45 FC         MOV EAX,DWORD PTR SS:\[EBP-4\]  
005BAC3F   .  8B80 04040000   MOV EAX,DWORD PTR DS:\[EAX+404\]  
005BAC45   .  BA DCB25B00     MOV EDX,unpacked.005BB2DC        ;  丁世龙  
005BAC4A   .  E8 0D9CE4FF     CALL unpacked.0040485C  
005BAC4F   .  74 69           JE SHORT unpacked.005BACBA  
005BAC51   .  8B45 FC         MOV EAX,DWORD PTR SS:\[EBP-4\]  
005BAC54   .  8B80 04040000   MOV EAX,DWORD PTR DS:\[EAX+404\]  
005BAC5A   .  BA ECB25B00     MOV EDX,unpacked.005BB2EC        ;  ASCII "8348006"  
005BAC5F   .  E8 F89BE4FF     CALL unpacked.0040485C  
005BAC64   .  74 54           JE SHORT unpacked.005BACBA  
005BAC66   .  8B45 FC         MOV EAX,DWORD PTR SS:\[EBP-4\]  
005BAC69   .  8B80 04040000   MOV EAX,DWORD PTR DS:\[EAX+404\]  
005BAC6F   .  BA FCB25B00     MOV EDX,unpacked.005BB2FC        ;  ASCII "zhaoguihu"  
005BAC74   .  E8 E39BE4FF     CALL unpacked.0040485C  
005BAC79   .  74 3F           JE SHORT unpacked.005BACBA  
005BAC7B   .  8B45 FC         MOV EAX,DWORD PTR SS:\[EBP-4\]  
005BAC7E   .  8B80 04040000   MOV EAX,DWORD PTR DS:\[EAX+404\]  
005BAC84   .  BA 10B35B00     MOV EDX,unpacked.005BB310        ;  ASCII "wwwjjj"  
005BAC89   .  E8 CE9BE4FF     CALL unpacked.0040485C  
005BAC8E   .  74 2A           JE SHORT unpacked.005BACBA  
005BAC90   .  8B45 FC         MOV EAX,DWORD PTR SS:\[EBP-4\]  
005BAC93   .  8B80 04040000   MOV EAX,DWORD PTR DS:\[EAX+404\]  
005BAC99   .  BA 20B35B00     MOV EDX,unpacked.005BB320        ;  吴小龙  
005BAC9E   .  E8 B99BE4FF     CALL unpacked.0040485C  
005BACA3   .  74 15           JE SHORT unpacked.005BACBA  
005BACA5   .  8B45 FC         MOV EAX,DWORD PTR SS:\[EBP-4\]  
005BACA8   .  8B80 04040000   MOV EAX,DWORD PTR DS:\[EAX+404\]  
005BACAE   .  BA 30B35B00     MOV EDX,unpacked.005BB330        ;  刘光军  
005BACB3   .  E8 A49BE4FF     CALL unpacked.0040485C  
005BACB8   .  75 1A           JNZ SHORT unpacked.005BACD4  
005BACBA   >  8B45 FC         MOV EAX,DWORD PTR SS:\[EBP-4\]  
005BACBD   .  E8 EAFCFFFF     CALL unpacked.005BA9AC  
005BACC2   .  8B45 FC         MOV EAX,DWORD PTR SS:\[EBP-4\]  
005BACC5   .  05 04040000     ADD EAX,404  
005BACCA   .  BA 40B35B00     MOV EDX,unpacked.005BB340  
005BACCF   .  E8 B497E4FF     CALL unpacked.00404488  
005BACD4   >  8D55 C0         LEA EDX,DWORD PTR SS:\[EBP-40\]    ;  上面的名字都是一些黑名单，对比后跳到这里继续对用户名处理  
005BACD7   .  8B45 FC         MOV EAX,DWORD PTR SS:\[EBP-4\]  
005BACDA   .  8B80 04040000   MOV EAX,DWORD PTR DS:\[EAX+404\]  
005BACE0   .  E8 8FD1EBFF     CALL unpacked.00477E74           ;  MD5(用户名)  
005BACE5   .  8B45 C0         MOV EAX,DWORD PTR SS:\[EBP-40\]  
005BACE8   .  8D4D C4         LEA ECX,DWORD PTR SS:\[EBP-3C\]  
005BACEB   .  8B55 FC         MOV EDX,DWORD PTR SS:\[EBP-4\]  
005BACEE   .  8B92 08040000   MOV EDX,DWORD PTR DS:\[EDX+408\]  
005BACF4   .  E8 8FC3EBFF     CALL unpacked.00477088           ;  DES\_EN()，Key＝机器码  
005BACF9   .  8B45 C4         MOV EAX,DWORD PTR SS:\[EBP-3C\]  
005BACFC   .  8D55 C8         LEA EDX,DWORD PTR SS:\[EBP-38\]  
005BACFF   .  E8 70D1EBFF     CALL unpacked.00477E74           ;  MD5( )  
005BAD04   .  8B55 C8         MOV EDX,DWORD PTR SS:\[EBP-38\]  
005BAD07   .  8B45 FC         MOV EAX,DWORD PTR SS:\[EBP-4\]  
005BAD0A   .  05 10040000     ADD EAX,410  
005BAD0F   .  E8 7497E4FF     CALL unpacked.00404488  
005BAD14   .  8B45 FC         MOV EAX,DWORD PTR SS:\[EBP-4\]  
005BAD17   .  8B80 00040000   MOV EAX,DWORD PTR DS:\[EAX+400\]  
005BAD1D   .  E8 EE99E4FF     CALL unpacked.00404710           ;  判断注册表中保存注册码信息的长度，应该有四种可能：10，14，16，12 分别对应注册码的长度是 16、20、24、32  
005BAD22   .  83E8 0A         SUB EAX,0A                       ;  Switch (cases A..10)  
005BAD25   .  74 7E           JE SHORT unpacked.005BADA5  
005BAD27   .  83E8 02         SUB EAX,2  
005BAD2A   .  74 17           JE SHORT unpacked.005BAD43  
005BAD2C   .  83E8 02         SUB EAX,2  
005BAD2F   .  0F84 52010000   JE unpacked.005BAE87  
005BAD35   .  83E8 02         SUB EAX,2  
005BAD38   .  0F84 2B020000   JE unpacked.005BAF69  
005BAD3E   .  E9 E6030000     JMP unpacked.005BB129  
005BAD43   >  8B45 FC         MOV EAX,DWORD PTR SS:\[EBP-4\]     ;  Case C of switch 005BAD22  
005BAD46   .  05 0C040000     ADD EAX,40C                      ;  如果长度位 12，即当时填入 32 位假码，就跳到这里  
005BAD4B   .  8B55 FC         MOV EDX,DWORD PTR SS:\[EBP-4\]  
005BAD4E   .  8B92 10040000   MOV EDX,DWORD PTR DS:\[EDX+410\]  
005BAD54   .  E8 2F97E4FF     CALL unpacked.00404488  
005BAD59   .  68 E0925C00     PUSH unpacked.005C92E0  
005BAD5E   .  8D55 B8         LEA EDX,DWORD PTR SS:\[EBP-48\]  
005BAD61   .  8B45 FC         MOV EAX,DWORD PTR SS:\[EBP-4\]  
005BAD64   .  8B80 0C040000   MOV EAX,DWORD PTR DS:\[EAX+40C\]  
005BAD6A   .  E8 05D1EBFF     CALL unpacked.00477E74           ;  MD( )如果输入 32 个字符的假码，就在这里做内存注册机吧  
005BAD6F   .  8B45 B8         MOV EAX,DWORD PTR SS:\[EBP-48\]  
005BAD72   .  8D4D BC         LEA ECX,DWORD PTR SS:\[EBP-44\]  
005BAD75   .  BA F8B15B00     MOV EDX,unpacked.005BB1F8        ;  ASCII "dream"  
005BAD7A   .  E8 09C3EBFF     CALL unpacked.00477088           ;  DES\_EN( ),Key="dream"  
005BAD7F   .  8B45 BC         MOV EAX,DWORD PTR SS:\[EBP-44\]  
005BAD82   .  B9 0C000000     MOV ECX,0C  
005BAD87   .  BA 0C000000     MOV EDX,0C  
005BAD8C   .  E8 DF9BE4FF     CALL unpacked.00404970           ;  第 12 位起取 12 个字符  
005BAD91   .  33C0            XOR EAX,EAX  
005BAD93   .  A3 E8925C00     MOV DWORD PTR DS:\[5C92E8\],EAX  
005BAD98   .  8D45 F0         LEA EAX,DWORD PTR SS:\[EBP-10\]  
005BAD9B   .  E8 9496E4FF     CALL unpacked.00404434  
005BADA0   .  E9 B9020000     JMP unpacked.005BB05E  
005BADA5   >  8D45 B4         LEA EAX,DWORD PTR SS:\[EBP-4C\]    ;  Case A of switch 005BAD22  
005BADA8   .  50              PUSH EAX                         ;  如果长度位 10，即当时填入 16 位假码，就跳到这里  
005BADA9   .  8B45 FC         MOV EAX,DWORD PTR SS:\[EBP-4\]  
005BADAC   .  8B80 10040000   MOV EAX,DWORD PTR DS:\[EAX+410\]  
005BADB2   .  B9 05000000     MOV ECX,5  
005BADB7   .  BA 10000000     MOV EDX,10  
005BADBC   .  E8 AF9BE4FF     CALL unpacked.00404970           ;  第 16 位起取 5 个字符  
005BADC1   .  FF75 B4         PUSH DWORD PTR SS:\[EBP-4C\]  
005BADC4   .  8D45 B0         LEA EAX,DWORD PTR SS:\[EBP-50\]  
005BADC7   .  50              PUSH EAX  
005BADC8   .  8B45 FC         MOV EAX,DWORD PTR SS:\[EBP-4\]  
005BADCB   .  8B80 10040000   MOV EAX,DWORD PTR DS:\[EAX+410\]  
005BADD1   .  B9 04000000     MOV ECX,4  
005BADD6   .  BA 04000000     MOV EDX,4  
005BADDB   .  E8 909BE4FF     CALL unpacked.00404970           ;  第 4 位起取 4 个字符  
005BADE0   .  FF75 B0         PUSH DWORD PTR SS:\[EBP-50\]  
005BADE3   .  8D45 AC         LEA EAX,DWORD PTR SS:\[EBP-54\]  
005BADE6   .  50              PUSH EAX  
005BADE7   .  8B45 FC         MOV EAX,DWORD PTR SS:\[EBP-4\]  
005BADEA   .  8B80 10040000   MOV EAX,DWORD PTR DS:\[EAX+410\]  
005BADF0   .  B9 04000000     MOV ECX,4  
005BADF5   .  BA 12000000     MOV EDX,12  
005BADFA   .  E8 719BE4FF     CALL unpacked.00404970           ;  第 18 位起取 4 个字符  
005BADFF   .  FF75 AC         PUSH DWORD PTR SS:\[EBP-54\]  
005BAE02   .  8D45 A8         LEA EAX,DWORD PTR SS:\[EBP-58\]  
005BAE05   .  50              PUSH EAX  
005BAE06   .  8B45 FC         MOV EAX,DWORD PTR SS:\[EBP-4\]  
005BAE09   .  8B80 10040000   MOV EAX,DWORD PTR DS:\[EAX+410\]  
005BAE0F   .  B9 03000000     MOV ECX,3  
005BAE14   .  BA 1D000000     MOV EDX,1D  
005BAE19   .  E8 529BE4FF     CALL unpacked.00404970           ;  第 29 起取 3 个字符  
005BAE1E   .  FF75 A8         PUSH DWORD PTR SS:\[EBP-58\]  
005BAE21   .  8B45 FC         MOV EAX,DWORD PTR SS:\[EBP-4\]  
005BAE24   .  05 0C040000     ADD EAX,40C  
005BAE29   .  BA 04000000     MOV EDX,4  
005BAE2E   .  E8 9D99E4FF     CALL unpacked.004047D0           ;  合并取出的字符，这里也是一个长度是 16 的标准版的真注册码  
005BAE33   .  68 E0925C00     PUSH unpacked.005C92E0  
005BAE38   .  8D55 A0         LEA EDX,DWORD PTR SS:\[EBP-60\]  
005BAE3B   .  8B45 FC         MOV EAX,DWORD PTR SS:\[EBP-4\]  
005BAE3E   .  8B80 0C040000   MOV EAX,DWORD PTR DS:\[EAX+40C\]  
005BAE44   .  E8 2BD0EBFF     CALL unpacked.00477E74           ;  MD5( )如果输入 16 个字符的假码，就在这里做内存注册机吧  
005BAE49   .  8B45 A0         MOV EAX,DWORD PTR SS:\[EBP-60\]  
005BAE4C   .  8D4D A4         LEA ECX,DWORD PTR SS:\[EBP-5C\]  
005BAE4F   .  BA F8B15B00     MOV EDX,unpacked.005BB1F8        ;  ASCII "dream"  
005BAE54   .  E8 2FC2EBFF     CALL unpacked.00477088           ;  DES\_EN( )，key="dream"  
005BAE59   .  8B45 A4         MOV EAX,DWORD PTR SS:\[EBP-5C\]  
005BAE5C   .  B9 0A000000     MOV ECX,0A  
005BAE61   .  BA 06000000     MOV EDX,6  
005BAE66   .  E8 059BE4FF     CALL unpacked.00404970           ;  第 6 位起取 10 个字符  
005BAE6B   .  C705 E8925C00 0>MOV DWORD PTR DS:\[5C92E8\],1  
005BAE75   .  8D45 F0         LEA EAX,DWORD PTR SS:\[EBP-10\]  
005BAE78   .  BA 4CB35B00     MOV EDX,unpacked.005BB34C  
005BAE7D   .  E8 4A96E4FF     CALL unpacked.004044CC  
005BAE82   .  E9 D7010000     JMP unpacked.005BB05E  
005BAE87   >  8D45 9C         LEA EAX,DWORD PTR SS:\[EBP-64\]    ;  Case E of switch 005BAD22  
005BAE8A   .  50              PUSH EAX                         ;  如果长度位 14，即当时填入 20 位假码，就跳到这里  
005BAE8B   .  8B45 FC         MOV EAX,DWORD PTR SS:\[EBP-4\]  
005BAE8E   .  8B80 10040000   MOV EAX,DWORD PTR DS:\[EAX+410\]  
005BAE94   .  B9 05000000     MOV ECX,5  
005BAE99   .  BA 03000000     MOV EDX,3  
005BAE9E   .  E8 CD9AE4FF     CALL unpacked.00404970           ;  从上面用户名处理结果中第 3 位起取 5 个字符  
005BAEA3   .  FF75 9C         PUSH DWORD PTR SS:\[EBP-64\]  
005BAEA6   .  8D45 98         LEA EAX,DWORD PTR SS:\[EBP-68\]  
005BAEA9   .  50              PUSH EAX  
005BAEAA   .  8B45 FC         MOV EAX,DWORD PTR SS:\[EBP-4\]  
005BAEAD   .  8B80 10040000   MOV EAX,DWORD PTR DS:\[EAX+410\]  
005BAEB3   .  B9 02000000     MOV ECX,2  
005BAEB8   .  BA 1D000000     MOV EDX,1D  
005BAEBD   .  E8 AE9AE4FF     CALL unpacked.00404970           ;  第 29 位起取 2 个字符  
005BAEC2   .  FF75 98         PUSH DWORD PTR SS:\[EBP-68\]  
005BAEC5   .  8D45 94         LEA EAX,DWORD PTR SS:\[EBP-6C\]  
005BAEC8   .  50              PUSH EAX  
005BAEC9   .  8B45 FC         MOV EAX,DWORD PTR SS:\[EBP-4\]  
005BAECC   .  8B80 10040000   MOV EAX,DWORD PTR DS:\[EAX+410\]  
005BAED2   .  B9 06000000     MOV ECX,6  
005BAED7   .  BA 0C000000     MOV EDX,0C  
005BAEDC   .  E8 8F9AE4FF     CALL unpacked.00404970           ;  第 13 位起取 6 个字符  
005BAEE1   .  FF75 94         PUSH DWORD PTR SS:\[EBP-6C\]  
005BAEE4   .  8D45 90         LEA EAX,DWORD PTR SS:\[EBP-70\]  
005BAEE7   .  50              PUSH EAX  
005BAEE8   .  8B45 FC         MOV EAX,DWORD PTR SS:\[EBP-4\]  
005BAEEB   .  8B80 10040000   MOV EAX,DWORD PTR DS:\[EAX+410\]  
005BAEF1   .  B9 07000000     MOV ECX,7  
005BAEF6   .  BA 06000000     MOV EDX,6  
005BAEFB   .  E8 709AE4FF     CALL unpacked.00404970           ;  第 6 位起取 7 个字符  
005BAF00   .  FF75 90         PUSH DWORD PTR SS:\[EBP-70\]  
005BAF03   .  8B45 FC         MOV EAX,DWORD PTR SS:\[EBP-4\]  
005BAF06   .  05 0C040000     ADD EAX,40C  
005BAF0B   .  BA 04000000     MOV EDX,4  
005BAF10   .  E8 BB98E4FF     CALL unpacked.004047D0           ;  合并上面字符，得到一个长度位 20 的 VIP 版的真注册码  
005BAF15   .  68 E0925C00     PUSH unpacked.005C92E0  
005BAF1A   .  8D55 88         LEA EDX,DWORD PTR SS:\[EBP-78\]  
005BAF1D   .  8B45 FC         MOV EAX,DWORD PTR SS:\[EBP-4\]  
005BAF20   .  8B80 0C040000   MOV EAX,DWORD PTR DS:\[EAX+40C\]  
005BAF26   .  E8 49CFEBFF     CALL unpacked.00477E74           ;  MD5( )如果输入 20 个字符的假码，就在这里做内存注册机吧  
005BAF2B   .  8B45 88         MOV EAX,DWORD PTR SS:\[EBP-78\]  
005BAF2E   .  8D4D 8C         LEA ECX,DWORD PTR SS:\[EBP-74\]  
005BAF31   .  BA F8B15B00     MOV EDX,unpacked.005BB1F8        ;  ASCII "dream"  
005BAF36   .  E8 4DC1EBFF     CALL unpacked.00477088           ;  DES\_EN( )运算，Key="dream"  
005BAF3B   .  8B45 8C         MOV EAX,DWORD PTR SS:\[EBP-74\]  
005BAF3E   .  B9 0E000000     MOV ECX,0E  
005BAF43   .  BA 03000000     MOV EDX,3  
005BAF48   .  E8 239AE4FF     CALL unpacked.00404970           ;  从第 3 位起取 14 个字符  
005BAF4D   .  C705 E8925C00 0>MOV DWORD PTR DS:\[5C92E8\],2  
005BAF57   .  8D45 F0         LEA EAX,DWORD PTR SS:\[EBP-10\]  
005BAF5A   .  BA 60B35B00     MOV EDX,unpacked.005BB360  
005BAF5F   .  E8 6895E4FF     CALL unpacked.004044CC  
005BAF64   .  E9 F5000000     JMP unpacked.005BB05E  
005BAF69   >  8D45 84         LEA EAX,DWORD PTR SS:\[EBP-7C\]    ;  Case 10 of switch 005BAD22  
005BAF6C   .  50              PUSH EAX                         ;  如果长度位 16，即当时填入 24 位假码，就跳到这里  
005BAF6D   .  8B45 FC         MOV EAX,DWORD PTR SS:\[EBP-4\]  
005BAF70   .  8B80 10040000   MOV EAX,DWORD PTR DS:\[EAX+410\]  
005BAF76   .  B9 06000000     MOV ECX,6  
005BAF7B   .  BA 08000000     MOV EDX,8  
005BAF80   .  E8 EB99E4FF     CALL unpacked.00404970           ;  第 8 位起取 6 个字符  
005BAF85   .  FF75 84         PUSH DWORD PTR SS:\[EBP-7C\]  
005BAF88   .  8D45 80         LEA EAX,DWORD PTR SS:\[EBP-80\]  
005BAF8B   .  50              PUSH EAX  
005BAF8C   .  8B45 FC         MOV EAX,DWORD PTR SS:\[EBP-4\]  
005BAF8F   .  8B80 10040000   MOV EAX,DWORD PTR DS:\[EAX+410\]  
005BAF95   .  B9 07000000     MOV ECX,7  
005BAF9A   .  BA 12000000     MOV EDX,12  
005BAF9F   .  E8 CC99E4FF     CALL unpacked.00404970           ;  第 18 位起取 7 个字符  
005BAFA4   .  FF75 80         PUSH DWORD PTR SS:\[EBP-80\]  
005BAFA7   .  8D85 7CFFFFFF   LEA EAX,DWORD PTR SS:\[EBP-84\]  
005BAFAD   .  50              PUSH EAX  
005BAFAE   .  8B45 FC         MOV EAX,DWORD PTR SS:\[EBP-4\]  
005BAFB1   .  8B80 10040000   MOV EAX,DWORD PTR DS:\[EAX+410\]  
005BAFB7   .  B9 08000000     MOV ECX,8  
005BAFBC   .  BA 17000000     MOV EDX,17  
005BAFC1   .  E8 AA99E4FF     CALL unpacked.00404970           ;  第 23 位起取 8 个字符  
005BAFC6   .  FFB5 7CFFFFFF   PUSH DWORD PTR SS:\[EBP-84\]  
005BAFCC   .  8D85 78FFFFFF   LEA EAX,DWORD PTR SS:\[EBP-88\]  
005BAFD2   .  50              PUSH EAX  
005BAFD3   .  8B45 FC         MOV EAX,DWORD PTR SS:\[EBP-4\]  
005BAFD6   .  8B80 10040000   MOV EAX,DWORD PTR DS:\[EAX+410\]  
005BAFDC   .  B9 03000000     MOV ECX,3  
005BAFE1   .  BA 0D000000     MOV EDX,0D  
005BAFE6   .  E8 8599E4FF     CALL unpacked.00404970           ;  第 13 位起取 3 个字符  
005BAFEB   .  FFB5 78FFFFFF   PUSH DWORD PTR SS:\[EBP-88\]  
005BAFF1   .  8B45 FC         MOV EAX,DWORD PTR SS:\[EBP-4\]  
005BAFF4   .  05 0C040000     ADD EAX,40C  
005BAFF9   .  BA 04000000     MOV EDX,4  
005BAFFE   .  E8 CD97E4FF     CALL unpacked.004047D0           ;  合并得到钻石版的真注册码  
005BB003   .  68 E0925C00     PUSH unpacked.005C92E0  
005BB008   .  8D95 70FFFFFF   LEA EDX,DWORD PTR SS:\[EBP-90\]  
005BB00E   .  8B45 FC         MOV EAX,DWORD PTR SS:\[EBP-4\]  
005BB011   .  8B80 0C040000   MOV EAX,DWORD PTR DS:\[EAX+40C\]  
005BB017   .  E8 58CEEBFF     CALL unpacked.00477E74           ;  MD5( )如果输入 24 个字符的假码，就在这里做内存注册机吧  
005BB01C   .  8B85 70FFFFFF   MOV EAX,DWORD PTR SS:\[EBP-90\]  
005BB022   .  8D8D 74FFFFFF   LEA ECX,DWORD PTR SS:\[EBP-8C\]  
005BB028   .  BA F8B15B00     MOV EDX,unpacked.005BB1F8        ;  ASCII "dream"  
005BB02D   .  E8 56C0EBFF     CALL unpacked.00477088           ;  DES\_EN( ),Key="dream"  
005BB032   .  8B85 74FFFFFF   MOV EAX,DWORD PTR SS:\[EBP-8C\]  
005BB038   .  B9 10000000     MOV ECX,10  
005BB03D   .  BA 05000000     MOV EDX,5  
005BB042   .  E8 2999E4FF     CALL unpacked.00404970           ;  第 5 位起取 16 的字符  
005BB047   .  C705 E8925C00 0>MOV DWORD PTR DS:\[5C92E8\],3  
005BB051   .  8D45 F0         LEA EAX,DWORD PTR SS:\[EBP-10\]  
005BB054   .  BA 70B35B00     MOV EDX,unpacked.005BB370  
005BB059   .  E8 6E94E4FF     CALL unpacked.004044CC  
005BB05E   >  8D85 68FFFFFF   LEA EAX,DWORD PTR SS:\[EBP-98\]    ;  下面代码比较第 2 位起的 5 个字符  
005BB064   .  50              PUSH EAX  
005BB065   .  B9 05000000     MOV ECX,5  
005BB06A   .  BA 02000000     MOV EDX,2  
005BB06F   .  A1 E0925C00     MOV EAX,DWORD PTR DS:\[5C92E0\]  
005BB074   .  E8 F798E4FF     CALL unpacked.00404970           ;  第 2 位起取 5 个字符  
005BB079   .  8B85 68FFFFFF   MOV EAX,DWORD PTR SS:\[EBP-98\]  
005BB07F   .  8D95 6CFFFFFF   LEA EDX,DWORD PTR SS:\[EBP-94\]  
005BB085   .  E8 36F8FFFF     CALL unpacked.005BA8C0           ;  转换位 ASC 码+空格的形式  
005BB08A   .  8B85 6CFFFFFF   MOV EAX,DWORD PTR SS:\[EBP-94\]  
005BB090   .  50              PUSH EAX  
005BB091   .  8D85 60FFFFFF   LEA EAX,DWORD PTR SS:\[EBP-A0\]  
005BB097   .  50              PUSH EAX  
005BB098   .  8B45 FC         MOV EAX,DWORD PTR SS:\[EBP-4\]  
005BB09B   .  8B80 00040000   MOV EAX,DWORD PTR DS:\[EAX+400\]  
005BB0A1   .  B9 05000000     MOV ECX,5  
005BB0A6   .  BA 02000000     MOV EDX,2  
005BB0AB   .  E8 C098E4FF     CALL unpacked.00404970           ;  第 2 位起取 5 个字符  
005BB0B0   .  8B85 60FFFFFF   MOV EAX,DWORD PTR SS:\[EBP-A0\]  
005BB0B6   .  8D95 64FFFFFF   LEA EDX,DWORD PTR SS:\[EBP-9C\]  
005BB0BC   .  E8 FFF7FFFF     CALL unpacked.005BA8C0           ;  转换位 ASC 码+空格的形式  
005BB0C1   .  8B95 64FFFFFF   MOV EDX,DWORD PTR SS:\[EBP-9C\]  
005BB0C7   .  58              POP EAX  
005BB0C8   .  E8 8F97E4FF     CALL unpacked.0040485C           ;  比较  
005BB0CD   .  75 5A           JNZ SHORT unpacked.005BB129  
005BB0CF   .  C605 E0645C00 0>MOV BYTE PTR DS:\[5C64E0\],1  
005BB0D6   .  833D E8925C00 0>CMP DWORD PTR DS:\[5C92E8\],1  
005BB0DD   .  74 07           JE SHORT unpacked.005BB0E6  
005BB0DF   .  C605 E4645C00 0>MOV BYTE PTR DS:\[5C64E4\],1  
005BB0E6   >  8B45 FC         MOV EAX,DWORD PTR SS:\[EBP-4\]  
005BB0E9   .  8B80 AC030000   MOV EAX,DWORD PTR DS:\[EAX+3AC\]  
005BB0EF   .  33D2            XOR EDX,EDX  
  
DES 加密或解密过程中，因为密钥至少要有 64 位，如有几次密钥 Key="dream"，密钥只有 40 位，因此要填充 24 个 0  
  
\----------------------------------------------------------------------------------------------  
【破解心得】  
  
作者没有采用了注册码的明码比较。对用户名和注册码的处理过程大致是这样的；  
  
注册码——MD5( )——DES\_EN( ,Key="dream")——抽取字符——存入注册表——重启程序读注册表  
  
用户名——DES\_EN( ,Key="zlbpolly")——存入注册表——重启程序读注册表——DES\_DE( ,Key="zlbpolly")还原用户名——MD5( )——抽取字符形成真注册码——MD5( )——DES\_EN( Key="dream")——抽取字符  
  
经过上述过程后将用户名和注册码得到的最终结果比较其中的一部分字符。  
  
有没有发现用户名和注册码最后一段处理是一样的，其实真码的形成很简单：  
  
用户名——经过 MD5( )运算——DES\_EN( Key="机器码")——MD5( )——根据版本钻石版还是标准版抽取相应字符形成真码。  
  
编写算法注册机时要注意，MD5( )的结果以小写 16 进制输出，DES( )的结果以大写 16 禁止文本输出：  
  
如："rdsnow\[BCG\]\[PYG\]\[D.4s\]"的 MD5 输出为："28a5ec4ecec3e6227fbffdcedeec1bc4"  
  
"28a5ec4ecec3e6227fbffdcedeec1bc4"的 DES( key＝机器码)输出(我的 Key 是"67f2a822")为："B51E989F74EEE338B51ABC5A6AB315FD42B634367DDF2D96BEC3B6004B5C2422"  
  
再经过一次 MD5( )得到"b196b34a23783d94922e8b4ca5f92a7a"  
  
抽取不同位置的字符得到不同版本的注册码：  
  
用户名：rdsnow\[BCG\]\[PYG\]\[D.4s\]，机器码 "67f2a822"  
  
未知版：b196b34a23783d94922e8b4ca5f92a7a (32 个字符)(未能通过后面的长度验证)  
  
标准版：4922e6b3422e82a7 (16 个字符)  
  
VIP 版：96b342a83d94934a2378 (20 个字符)  
  
钻石版：a2378322e8b4c4ca5f92a3d9 (24 个字符)  
  
因为个人的机器码不一样，所以注册码也不一样。  
  
\----------------------------------------------------------------------------------------------  
【破解声明】   我是一只小菜鸟，偶得一点心得，愿与大家分享:)  
  
【版权声明】   本文纯属技术交流, 转载请注明作者并保持文章的完整, 谢谢!   
\----------------------------------------------------------------------------------------------  
                                                                     文章写于 2005-8-25 17:55:37 

上次跟踪了流星网络电视的注册过程，再贴上 DES 的跟踪笔记，跟大家共同学习一下，这是个标准的 Delphi 的 DES 源码：  
  
※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※  
  
加密和解密是同一个 CALL，即下面的 CALL unpacked.00476B38，通过调用参数决定是加密还是解密：  
  
00476DD0  |> /8B55 F8         /MOV EDX,DWORD PTR SS:\[EBP-8\]       ;  取 64 位的密钥  
00476DD3  |. |8A141A          |MOV DL,BYTE PTR DS:\[EDX+EBX\]  
00476DD6  |. |8810            |MOV BYTE PTR DS:\[EAX\],DL           ;  转存密钥  
00476DD8  |. |43              |INC EBX  
00476DD9  |. |40              |INC EAX  
00476DDA  |. |83FB 08         |CMP EBX,8  
00476DDD  |.^\\75 F1           \\JNZ SHORT unpacked.00476DD0        ;  分 8 轮取完，取完跳出循环  
00476DDF  |.  6A 0F           PUSH 0F                             ; /Arg1 = 0000000F  
00476DE1  |.  B9 0C8C5C00     MOV ECX,unpacked.005C8C0C           ; |  
00476DE6  |.  8D45 DC         LEA EAX,DWORD PTR SS:\[EBP-24\]       ; |  
00476DE9  |.  BA 07000000     MOV EDX,7                           ; |  
00476DEE  |.  E8 EDFAFFFF     CALL unpacked.004768E0              ; \\生成 16 个子密钥 KO、K1、……K15  
  
………………(省略 64 位信息分组的代码)  
00476E3C  |.  50              |PUSH EAX                           ; /Arg2  
00476E3D  |.  6A 07           |PUSH 7                             ; |Arg1 = 00000007  
00476E3F  |.  8D55 EC         |LEA EDX,DWORD PTR SS:\[EBP-14\]      ; |  
00476E42  |.  B9 07000000     |MOV ECX,7                          ; |  
00476E47  |.  33C0            |XOR EAX,EAX                        ; |AL＝0 表示加密，AL＝1 表示解密  
00476E49  |.  E8 EAFCFFFF     |CALL unpacked.00476B38             ; \\对分组得到的 64 位信息加密，跟进  
  
信息分组，即将很长的待加密信息分成 64 位一组，有多少组，看信息长度决定，然后分别对每一组加密，最后把每一组加密后的结果连接起来，就得到了最后的结果。  
  
※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※  
  
进入 CALL unpacked.00476B38，看看 DES 的关键代码(输入 64 位，输出 64 位)：  
  
00476B38  /$  55              PUSH EBP  
00476B39  |.  8BEC            MOV EBP,ESP  
00476B3B  |.  83C4 E8         ADD ESP,-18  
00476B3E  |.  53              PUSH EBX  
00476B3F  |.  56              PUSH ESI  
00476B40  |.  57              PUSH EDI  
00476B41  |.  8BD9            MOV EBX,ECX  
00476B43  |.  85DB            TEST EBX,EBX  
00476B45  |.  78 0A           JS SHORT unpacked.00476B51  
00476B47  |.  C1EB 02         SHR EBX,2  
00476B4A  |>  8B349A          /MOV ESI,DWORD PTR DS:\[EDX+EBX\*4\]  
00476B4D  |.  4B              |DEC EBX  
00476B4E  |.  56              |PUSH ESI  
00476B4F  |.^ 79 F9           \\JNS SHORT unpacked.00476B4A  
00476B51  |>  8BD4            MOV EDX,ESP  
00476B53  |.  8955 FC         MOV DWORD PTR SS:\[EBP-4\],EDX  
00476B56  |.  8BD8            MOV EBX,EAX  
00476B58  |.  C745 F8 0800000>MOV DWORD PTR SS:\[EBP-8\],8  
00476B5F  |.  8B45 FC         MOV EAX,DWORD PTR SS:\[EBP-4\]  
00476B62  |.  8B4D 0C         MOV ECX,DWORD PTR SS:\[EBP+C\]  
00476B65  |>  8A10            /MOV DL,BYTE PTR DS:\[EAX\]  
00476B67  |.  8811            |MOV BYTE PTR DS:\[ECX\],DL  
00476B69  |.  41              |INC ECX  
00476B6A  |.  40              |INC EAX  
00476B6B  |.  FF4D F8         |DEC DWORD PTR SS:\[EBP-8\]  
00476B6E  |.^ 75 F5           \\JNZ SHORT unpacked.00476B65  
00476B70  |.  8B45 0C         MOV EAX,DWORD PTR SS:\[EBP+C\]  
00476B73  |.  8B55 08         MOV EDX,DWORD PTR SS:\[EBP+8\]  
00476B76  |.  E8 81F9FFFF     CALL unpacked.004764FC              ;  初始换位  
00476B7B  |.  84DB            TEST BL,BL  
00476B7D  |.  0F85 B5000000   JNZ unpacked.00476C38               ;  如不跳，下面是加密代码，如果跳，则跳向解密代码  
00476B83  |.  C745 F8 1000000>MOV DWORD PTR SS:\[EBP-8\],10  
00476B8A  |.  C745 EC 0C8C5C0>MOV DWORD PTR SS:\[EBP-14\],unpacked.>  
00476B91  |>  B8 04000000     /MOV EAX,4  
00476B96  |.  8B55 0C         |MOV EDX,DWORD PTR SS:\[EBP+C\]  
00476B99  |.  8D75 F4         |LEA ESI,DWORD PTR SS:\[EBP-C\]       ;  下面开始对消息进行分组,把 64 位的消息分为 L 和 R  
00476B9C  |>  8A0A            |/MOV CL,BYTE PTR DS:\[EDX\]          ;  取 L(i)  
00476B9E  |.  880E            ||MOV BYTE PTR DS:\[ESI\],CL          ;  保存 L(i)出去  
00476BA0  |.  46              ||INC ESI  
00476BA1  |.  42              ||INC EDX  
00476BA2  |.  48              ||DEC EAX  
00476BA3  |.^ 75 F7           |\\JNZ SHORT unpacked.00476B9C       ;  取四个字节，即 32 位  
00476BA5  |.  B8 04000000     |MOV EAX,4  
00476BAA  |.  8B55 0C         |MOV EDX,DWORD PTR SS:\[EBP+C\]  
00476BAD  |.  83C2 04         |ADD EDX,4  
00476BB0  |>  8A0A            |/MOV CL,BYTE PTR DS:\[EDX\]          ;  取 R(i)  
00476BB2  |.  884A FC         ||MOV BYTE PTR DS:\[EDX-4\],CL        ;  保存 R(i),L(i+1)=R(i),这个循环起到了交换位置的作用  
00476BB5  |.  42              ||INC EDX  
00476BB6  |.  48              ||DEC EAX  
00476BB7  |.^ 75 F7           |\\JNZ SHORT unpacked.00476BB0       ;  取 4 个字节，即 32 位  
00476BB9  |.  6A 05           |PUSH 5                             ; /Arg3 = 00000005  
00476BBB  |.  8D45 F0         |LEA EAX,DWORD PTR SS:\[EBP-10\]      ; |  
00476BBE  |.  50              |PUSH EAX                           ; |Arg2  
00476BBF  |.  6A 03           |PUSH 3                             ; |Arg1 = 00000003  
00476BC1  |.  8B45 EC         |MOV EAX,DWORD PTR SS:\[EBP-14\]      ; |  
00476BC4  |.  8BC8            |MOV ECX,EAX                        ; |  
00476BC6  |.  8B45 0C         |MOV EAX,DWORD PTR SS:\[EBP+C\]       ; |  
00476BC9  |.  8B55 08         |MOV EDX,DWORD PTR SS:\[EBP+8\]       ; |  
00476BCC  |.  E8 3FFEFFFF     |CALL unpacked.00476A10             ; \\F(Ri,Ki)  
00476BD1  |.  B8 04000000     |MOV EAX,4  
00476BD6  |.  8D55 F4         |LEA EDX,DWORD PTR SS:\[EBP-C\]  
00476BD9  |.  8D75 F0         |LEA ESI,DWORD PTR SS:\[EBP-10\]  
00476BDC  |.  8B4D 0C         |MOV ECX,DWORD PTR SS:\[EBP+C\]  
00476BDF  |.  83C1 04         |ADD ECX,4  
00476BE2  |>  8A1A            |/MOV BL,BYTE PTR DS:\[EDX\]          ;  取 L(i)  
00476BE4  |.  321E            ||XOR BL,BYTE PTR DS:\[ESI\]          ;  L(i) ^ F(Ri,Ki)  
00476BE6  |.  8819            ||MOV BYTE PTR DS:\[ECX\],BL          ;  结果赋给 R(i)，即 R(i+1)=L(i) ^ F(Ri,Ki)  
00476BE8  |.  41              ||INC ECX  
00476BE9  |.  46              ||INC ESI  
00476BEA  |.  42              ||INC EDX  
00476BEB  |.  48              ||DEC EAX  
00476BEC  |.^ 75 F4           |\\JNZ SHORT unpacked.00476BE2       ;  循环四个字节，即 32 位  
00476BEE  |.  8345 EC 06      |ADD DWORD PTR SS:\[EBP-14\],6  
00476BF2  |.  FF4D F8         |DEC DWORD PTR SS:\[EBP-8\]  
00476BF5  |.^ 75 9A           \\JNZ SHORT unpacked.00476B91        ;  一共要经过 16 轮循环，由原先的 L0、R0 转变成了 L16、R16  
00476BF7  |.  B8 04000000     MOV EAX,4  
00476BFC  |.  8B55 0C         MOV EDX,DWORD PTR SS:\[EBP+C\]  
00476BFF  |.  83C2 04         ADD EDX,4  
00476C02  |.  8D4D F4         LEA ECX,DWORD PTR SS:\[EBP-C\]  
00476C05  |>  8A1A            /MOV BL,BYTE PTR DS:\[EDX\]           ;  取 R16  
00476C07  |.  8819            |MOV BYTE PTR DS:\[ECX\],BL           ;  转存 R16  
00476C09  |.  41              |INC ECX  
00476C0A  |.  42              |INC EDX  
00476C0B  |.  48              |DEC EAX  
00476C0C  |.^ 75 F7           \\JNZ SHORT unpacked.00476C05        ;  取四个字节，32 位,这个循环把 R16 保存出去  
00476C0E  |.  B8 04000000     MOV EAX,4  
00476C13  |.  8B55 0C         MOV EDX,DWORD PTR SS:\[EBP+C\]  
00476C16  |>  8A0A            /MOV CL,BYTE PTR DS:\[EDX\]           ;  取 L16  
00476C18  |.  884A 04         |MOV BYTE PTR DS:\[EDX+4\],CL         ;  R16=L16  
00476C1B  |.  42              |INC EDX  
00476C1C  |.  48              |DEC EAX  
00476C1D  |.^ 75 F7           \\JNZ SHORT unpacked.00476C16        ;  一共替换 4 字节 32 位，这个循环用 L16 覆盖 R16  
00476C1F  |.  B8 04000000     MOV EAX,4  
00476C24  |.  8D55 F4         LEA EDX,DWORD PTR SS:\[EBP-C\]  
00476C27  |.  8B4D 0C         MOV ECX,DWORD PTR SS:\[EBP+C\]  
00476C2A  |>  8A1A            /MOV BL,BYTE PTR DS:\[EDX\]           ;  取保存出去的 R16  
00476C2C  |.  8819            |MOV BYTE PTR DS:\[ECX\],BL           ;  L16=R16,即用保存出去的 R16 覆盖 L16  
00476C2E  |.  41              |INC ECX  
00476C2F  |.  42              |INC EDX  
00476C30  |.  48              |DEC EAX  
00476C31  |.^ 75 F7           \\JNZ SHORT unpacked.00476C2A        ;  一共替换 4 字节 32 位,上面三个小循环等于由交换了一下 L16 和 R16  
00476C33  |.  E9 BB000000     JMP unpacked.00476CF3               ;  跳向逆初始换位  
00476C38  |>  80FB 01         CMP BL,1  
00476C3B  |.  0F85 B2000000   JNZ unpacked.00476CF3  
00476C41  |.  C745 F8 F0FFFFF>MOV DWORD PTR SS:\[EBP-8\],-10  
00476C48  |.  BB 668C5C00     MOV EBX,unpacked.005C8C66  
00476C4D  |>  B8 04000000     /MOV EAX,4  
00476C52  |.  8B55 0C         |MOV EDX,DWORD PTR SS:\[EBP+C\]  
00476C55  |.  8D75 F4         |LEA ESI,DWORD PTR SS:\[EBP-C\]       ;  下面开始对消息进行分组,把 64 位的消息分为 L0 和 R0  
00476C58  |>  8A0A            |/MOV CL,BYTE PTR DS:\[EDX\]          ;  取 L(i)  
00476C5A  |.  880E            ||MOV BYTE PTR DS:\[ESI\],CL          ;  保存 L(i)出去  
00476C5C  |.  46              ||INC ESI  
00476C5D  |.  42              ||INC EDX  
00476C5E  |.  48              ||DEC EAX  
00476C5F  |.^ 75 F7           |\\JNZ SHORT unpacked.00476C58       ;  取四个字节，即 32 位  
00476C61  |.  B8 04000000     |MOV EAX,4  
00476C66  |.  8B55 0C         |MOV EDX,DWORD PTR SS:\[EBP+C\]  
00476C69  |.  83C2 04         |ADD EDX,4  
00476C6C  |>  8A0A            |/MOV CL,BYTE PTR DS:\[EDX\]          ;  取 R(i)  
00476C6E  |.  884A FC         ||MOV BYTE PTR DS:\[EDX-4\],CL        ;  保存 R(i),L(i+1)=R(i),这个循环起到了交换位置的作用  
00476C71  |.  42              ||INC EDX  
00476C72  |.  48              ||DEC EAX  
00476C73  |.^ 75 F7           |\\JNZ SHORT unpacked.00476C6C       ;  取四个字节，即 32 位  
00476C75  |.  6A 05           |PUSH 5                             ; /Arg3 = 00000005  
00476C77  |.  8D45 F0         |LEA EAX,DWORD PTR SS:\[EBP-10\]      ; |  
00476C7A  |.  50              |PUSH EAX                           ; |Arg2  
00476C7B  |.  6A 03           |PUSH 3                             ; |Arg1 = 00000003  
00476C7D  |.  8BCB            |MOV ECX,EBX                        ; |  
00476C7F  |.  8B45 0C         |MOV EAX,DWORD PTR SS:\[EBP+C\]       ; |  
00476C82  |.  8B55 08         |MOV EDX,DWORD PTR SS:\[EBP+8\]       ; |  
00476C85  |.  E8 86FDFFFF     |CALL unpacked.00476A10             ; \\F( Ri,K(15-i) )  
00476C8A  |.  B8 04000000     |MOV EAX,4  
00476C8F  |.  8D55 F4         |LEA EDX,DWORD PTR SS:\[EBP-C\]  
00476C92  |.  8D75 F0         |LEA ESI,DWORD PTR SS:\[EBP-10\]  
00476C95  |.  8B4D 0C         |MOV ECX,DWORD PTR SS:\[EBP+C\]  
00476C98  |.  83C1 04         |ADD ECX,4  
00476C9B  |.  894D E8         |MOV DWORD PTR SS:\[EBP-18\],ECX  
00476C9E  |>  8A0A            |/MOV CL,BYTE PTR DS:\[EDX\]          ;  取经过了 F( Ri,K(15-i) )的结果  
00476CA0  |.  320E            ||XOR CL,BYTE PTR DS:\[ESI\]          ;  L(i) ^ F( Ri,K(15-i) )  
00476CA2  |.  8B7D E8         ||MOV EDI,DWORD PTR SS:\[EBP-18\]     ;  结果赋给 R(i)，即 R(i+1)=L(i) ^ F(Ri,Ki)  
00476CA5  |.  880F            ||MOV BYTE PTR DS:\[EDI\],CL  
00476CA7  |.  FF45 E8         ||INC DWORD PTR SS:\[EBP-18\]  
00476CAA  |.  46              ||INC ESI  
00476CAB  |.  42              ||INC EDX  
00476CAC  |.  48              ||DEC EAX  
00476CAD  |.^ 75 EF           |\\JNZ SHORT unpacked.00476C9E       ;  循环四个字节，即 32 位  
00476CAF  |.  83EB 06         |SUB EBX,6  
00476CB2  |.  FF45 F8         |INC DWORD PTR SS:\[EBP-8\]  
00476CB5  |.^ 75 96           \\JNZ SHORT unpacked.00476C4D        ;  一共要经过 16 轮循环，由原先的 L0、R0 转变成了 L16、R16  
00476CB7  |.  B8 04000000     MOV EAX,4  
00476CBC  |.  8B55 0C         MOV EDX,DWORD PTR SS:\[EBP+C\]  
00476CBF  |.  83C2 04         ADD EDX,4  
00476CC2  |.  8D5D F4         LEA EBX,DWORD PTR SS:\[EBP-C\]  
00476CC5  |>  8A0A            /MOV CL,BYTE PTR DS:\[EDX\]           ;  取 R16  
00476CC7  |.  880B            |MOV BYTE PTR DS:\[EBX\],CL           ;  转存 R16  
00476CC9  |.  43              |INC EBX  
00476CCA  |.  42              |INC EDX  
00476CCB  |.  48              |DEC EAX  
00476CCC  |.^ 75 F7           \\JNZ SHORT unpacked.00476CC5        ;  取四个字节，32 位,这个循环把 R16 保存出去  
00476CCE  |.  B8 04000000     MOV EAX,4  
00476CD3  |.  8B5D 0C         MOV EBX,DWORD PTR SS:\[EBP+C\]  
00476CD6  |>  8A13            /MOV DL,BYTE PTR DS:\[EBX\]           ;  取 L16  
00476CD8  |.  8853 04         |MOV BYTE PTR DS:\[EBX+4\],DL         ;  R16=L16  
00476CDB  |.  43              |INC EBX  
00476CDC  |.  48              |DEC EAX  
00476CDD  |.^ 75 F7           \\JNZ SHORT unpacked.00476CD6        ;  一共替换 4 字节 32 位，这个循环用 L16 覆盖 R16  
00476CDF  |.  B8 04000000     MOV EAX,4  
00476CE4  |.  8D5D F4         LEA EBX,DWORD PTR SS:\[EBP-C\]  
00476CE7  |.  8B55 0C         MOV EDX,DWORD PTR SS:\[EBP+C\]  
00476CEA  |>  8A0B            /MOV CL,BYTE PTR DS:\[EBX\]           ;  取保存出去的 R16  
00476CEC  |.  880A            |MOV BYTE PTR DS:\[EDX\],CL           ;  L16=R16,即用保存出去的 R16 覆盖 L16  
00476CEE  |.  42              |INC EDX  
00476CEF  |.  43              |INC EBX  
00476CF0  |.  48              |DEC EAX  
00476CF1  |.^ 75 F7           \\JNZ SHORT unpacked.00476CEA        ;  一共替换 4 字节 32 位,上面三个小循环等于由交换了一下 L16 和 R16  
00476CF3  |>  8B45 0C         MOV EAX,DWORD PTR SS:\[EBP+C\]  
00476CF6  |.  8B55 08         MOV EDX,DWORD PTR SS:\[EBP+8\]  
00476CF9  |.  E8 82F8FFFF     CALL unpacked.00476580              ;  逆初始换位  
00476CFE  |.  8B7D DC         MOV EDI,DWORD PTR SS:\[EBP-24\]  
00476D01  |.  8B75 E0         MOV ESI,DWORD PTR SS:\[EBP-20\]  
00476D04  |.  8B5D E4         MOV EBX,DWORD PTR SS:\[EBP-1C\]  
00476D07  |.  8BE5            MOV ESP,EBP  
00476D09  |.  5D              POP EBP  
00476D0A  \\.  C2 0800         RETN 8  
  
小结：  
  
可见加密代码和解密代码几乎一样，不同的是 16 个 48 位的子密钥使用的顺序不同。  
  
我们主要看加密过程：  
  
待加密 64 位信息--->『初始变换』-->『16 轮循环处理』-->『逆初始变换』-->输出 64 位结果  
  
『初始变换』表：输入 64 位，输出 64 位(数据处理的第一步)  
  
58 50 42 34 26 18 10 02 60 52 44 36 28 20 12 04   
62 54 46 38 30 22 14 06 64 56 48 40 32 24 16 08   
57 49 41 33 25 17 09 01 59 51 43 35 27 19 11 03   
61 53 45 37 29 21 13 05 63 55 47 39 31 23 15 07  
  
『16 轮循环处理』(数据处理的中间一步)：  
  
64 位信息分为 32 的 L0 和 32 位的 R0  
  
For(i=0;i<16;i++){  
   Temp=Ri;  
   R(i+1)=Li ^ F(Ri,Ki);  
   L(i+1)=Temp;  
}  
  
经过 16 轮得到 L16,R16，再交换一下 L16,R16 输处 64 位信息  
  
『逆初始变换』表：输入 64 位，输出 64 位(数据处理的最后一步)  
  
40 08 48 16 56 24 64 32 39 07 47 15 55 23 63 31   
38 06 46 14 54 22 62 30 37 05 45 13 53 21 61 29   
36 04 44 12 52 20 60 28 35 03 43 11 51 19 59 27   
34 02 42 10 50 18 58 26 33 01 41 09 49 17 57 25  
  
※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※  
  
跟进 F(Ri,Ki)函数，过程如下：  
  
00476A10  /$  55             PUSH EBP  
00476A11  |.  8BEC           MOV EBP,ESP  
00476A13  |.  83C4 EC        ADD ESP,-14  
00476A16  |.  53             PUSH EBX  
00476A17  |.  56             PUSH ESI  
00476A18  |.  8B5D 10        MOV EBX,DWORD PTR SS:\[EBP+10\]  
00476A1B  |.  85DB           TEST EBX,EBX  
00476A1D  |.  78 0A          JS SHORT unpacked.00476A29  
00476A1F  |.  C1EB 02        SHR EBX,2  
00476A22  |>  8B3499         /MOV ESI,DWORD PTR DS:\[ECX+EBX\*4>  
00476A25  |.  4B             |DEC EBX  
00476A26  |.  56             |PUSH ESI  
00476A27  |.^ 79 F9          \\JNS SHORT unpacked.00476A22  
00476A29  |>  8BCC           MOV ECX,ESP  
00476A2B  |.  8BDA           MOV EBX,EDX  
00476A2D  |.  85DB           TEST EBX,EBX  
00476A2F  |.  78 0A          JS SHORT unpacked.00476A3B  
00476A31  |.  C1EB 02        SHR EBX,2  
00476A34  |>  8B3498         /MOV ESI,DWORD PTR DS:\[EAX+EBX\*4>  
00476A37  |.  4B             |DEC EBX  
00476A38  |.  56             |PUSH ESI  
00476A39  |.^ 79 F9          \\JNS SHORT unpacked.00476A34  
00476A3B  |>  8BC4           MOV EAX,ESP  
00476A3D  |.  894D FC        MOV DWORD PTR SS:\[EBP-4\],ECX  
00476A40  |.  6A 05          PUSH 5                           ; /Arg1 = 00000005  
00476A42  |.  8D4D F6        LEA ECX,DWORD PTR SS:\[EBP-A\]     ; |  
00476A45  |.  E8 BAFBFFFF    CALL unpacked.00476604           ; \\扩大换位将 32 位信息变为 48 位  
00476A4A  |.  BB 06000000    MOV EBX,6  
00476A4F  |.  8B45 FC        MOV EAX,DWORD PTR SS:\[EBP-4\]  
00476A52  |.  8D55 F6        LEA EDX,DWORD PTR SS:\[EBP-A\]  
00476A55  |>  8A08           /MOV CL,BYTE PTR DS:\[EAX\]  
00476A57  |.  300A           |XOR BYTE PTR DS:\[EDX\],CL        ;  48 位信息与 48 位密钥异或  
00476A59  |.  42             |INC EDX  
00476A5A  |.  40             |INC EAX  
00476A5B  |.  4B             |DEC EBX  
00476A5C  |.^ 75 F7          \\JNZ SHORT unpacked.00476A55     ;  六轮循环完成异或  
00476A5E  |.  8A45 F6        MOV AL,BYTE PTR SS:\[EBP-A\]       ;  下面开始将 48 位信息分成 8 个 6 位信息  
00476A61  |.  33D2           XOR EDX,EDX  
00476A63  |.  8AD0           MOV DL,AL  
00476A65  |.  C1EA 02        SHR EDX,2                        ;  得到第一个 6 位数据  
00476A68  |.  8855 EE        MOV BYTE PTR SS:\[EBP-12\],DL      ;  保存  
00476A6B  |.  24 03          AND AL,3  
00476A6D  |.  C1E0 04        SHL EAX,4  
00476A70  |.  8A55 F7        MOV DL,BYTE PTR SS:\[EBP-9\]  
00476A73  |.  33C9           XOR ECX,ECX  
00476A75  |.  8ACA           MOV CL,DL  
00476A77  |.  C1E9 04        SHR ECX,4  
00476A7A  |.  0AC1           OR AL,CL                         ;  得到第二个 6 位数据  
00476A7C  |.  8845 EF        MOV BYTE PTR SS:\[EBP-11\],AL      ;  保存  
00476A7F  |.  80E2 0F        AND DL,0F  
00476A82  |.  C1E2 02        SHL EDX,2  
00476A85  |.  8A45 F8        MOV AL,BYTE PTR SS:\[EBP-8\]  
00476A88  |.  33C9           XOR ECX,ECX  
00476A8A  |.  8AC8           MOV CL,AL  
00476A8C  |.  C1E9 06        SHR ECX,6  
00476A8F  |.  0AD1           OR DL,CL                         ;  得到第三个 6 位数据  
00476A91  |.  8855 F0        MOV BYTE PTR SS:\[EBP-10\],DL      ;  保存  
00476A94  |.  24 3F          AND AL,3F                        ;  得到第四个 6 位数据  
00476A96  |.  8845 F1        MOV BYTE PTR SS:\[EBP-F\],AL       ;  保存  
00476A99  |.  8A45 F9        MOV AL,BYTE PTR SS:\[EBP-7\]  
00476A9C  |.  33D2           XOR EDX,EDX  
00476A9E  |.  8AD0           MOV DL,AL  
00476AA0  |.  C1EA 02        SHR EDX,2                        ;  得到第五个 6 位数据  
00476AA3  |.  8855 F2        MOV BYTE PTR SS:\[EBP-E\],DL       ;  保存  
00476AA6  |.  24 03          AND AL,3  
00476AA8  |.  C1E0 04        SHL EAX,4  
00476AAB  |.  33D2           XOR EDX,EDX  
00476AAD  |.  8A55 FA        MOV DL,BYTE PTR SS:\[EBP-6\]  
00476AB0  |.  C1EA 04        SHR EDX,4  
00476AB3  |.  0AC2           OR AL,DL                         ;  得到第六个 6 位数据  
00476AB5  |.  8845 F3        MOV BYTE PTR SS:\[EBP-D\],AL       ;  保存  
00476AB8  |.  8A45 FA        MOV AL,BYTE PTR SS:\[EBP-6\]  
00476ABB  |.  24 0F          AND AL,0F  
00476ABD  |.  C1E0 02        SHL EAX,2  
00476AC0  |.  33D2           XOR EDX,EDX  
00476AC2  |.  8A55 FB        MOV DL,BYTE PTR SS:\[EBP-5\]  
00476AC5  |.  C1EA 06        SHR EDX,6  
00476AC8  |.  0AC2           OR AL,DL                         ;  得到第七个 6 位数据  
00476ACA  |.  8845 F4        MOV BYTE PTR SS:\[EBP-C\],AL       ;  保存  
00476ACD  |.  8A45 FB        MOV AL,BYTE PTR SS:\[EBP-5\]  
00476AD0  |.  24 3F          AND AL,3F                        ;  得到第八个 6 位数据  
00476AD2  |.  8845 F5        MOV BYTE PTR SS:\[EBP-B\],AL       ;  保存  
00476AD5  |.  33DB           XOR EBX,EBX  
00476AD7  |.  8D75 EE        LEA ESI,DWORD PTR SS:\[EBP-12\]  
00476ADA  |>  8BC3           /MOV EAX,EBX  
00476ADC  |.  8A16           |MOV DL,BYTE PTR DS:\[ESI\]        ;  取其中一个六位数据  
00476ADE  |.  E8 31FCFFFF    |CALL unpacked.00476714          ;  到 8 个 SBOX 相应的盒子中取 4 位的数据  
00476AE3  |.  8806           |MOV BYTE PTR DS:\[ESI\],AL        ;  保存  
00476AE5  |.  43             |INC EBX  
00476AE6  |.  46             |INC ESI  
00476AE7  |.  83FB 08        |CMP EBX,8  
00476AEA  |.^ 75 EE          \\JNZ SHORT unpacked.00476ADA     ;  循环八轮  
00476AEC  |.  BB 04000000    MOV EBX,4  
00476AF1  |.  8D45 EE        LEA EAX,DWORD PTR SS:\[EBP-12\]    ;  取出数据保存的地址  
00476AF4  |.  8D55 F6        LEA EDX,DWORD PTR SS:\[EBP-A\]  
00476AF7  |>  8A08           /MOV CL,BYTE PTR DS:\[EAX\]  
00476AF9  |.  C1E1 04        |SHL ECX,4  
00476AFC  |.  0A48 01        |OR CL,BYTE PTR DS:\[EAX+1\]       ;  两个 4 位数据合并成一个 8 位数据  
00476AFF  |.  880A           |MOV BYTE PTR DS:\[EDX\],CL        ;  保存  
00476B01  |.  42             |INC EDX  
00476B02  |.  83C0 02        |ADD EAX,2  
00476B05  |.  4B             |DEC EBX  
00476B06  |.^ 75 EF          \\JNZ SHORT unpacked.00476AF7     ;  四论循环，得到 32 位数据  
00476B08  |.  8D45 F6        LEA EAX,DWORD PTR SS:\[EBP-A\]  
00476B0B  |.  BA 05000000    MOV EDX,5  
00476B10  |.  E8 7FFBFFFF    CALL unpacked.00476694           ;  根据单纯换位表进行 32 位数据调整  
00476B15  |.  BB 04000000    MOV EBX,4  
00476B1A  |.  8D45 F6        LEA EAX,DWORD PTR SS:\[EBP-A\]  
00476B1D  |.  8B55 0C        MOV EDX,DWORD PTR SS:\[EBP+C\]  
00476B20  |>  8A08           /MOV CL,BYTE PTR DS:\[EAX\]  
00476B22  |.  880A           |MOV BYTE PTR DS:\[EDX\],CL        ;  保存结果  
00476B24  |.  42             |INC EDX  
00476B25  |.  40             |INC EAX  
00476B26  |.  4B             |DEC EBX  
00476B27  |.^ 75 F7          \\JNZ SHORT unpacked.00476B20     ;  四轮循环  
00476B29  |.  8B75 E4        MOV ESI,DWORD PTR SS:\[EBP-1C\]  
00476B2C  |.  8B5D E8        MOV EBX,DWORD PTR SS:\[EBP-18\]  
00476B2F  |.  8BE5           MOV ESP,EBP  
00476B31  |.  5D             POP EBP  
00476B32  \\.  C2 0C00        RETN 0C  
  
小结：  
  
选择函数 F(Ri,Ki)，输入 32 位，输出 32 位，它的大致过程：  
  
『数据膨胀』输入 32 位，输出 48 位：  
  
32 01 02 03 04 05 04 05 06 07 08 09 08 09 10 11  
12 13 12 13 14 15 16 17 16 17 18 19 20 21 20 21  
22 23 24 25 24 25 26 27 28 29 28 29 30 31 32 31  
  
『数据切割』输入 48 位，输出 8 个 6 位  
  
将 48 位数据，先跟 48 位子密钥异或，然后切割成 8 个六位输据  
  
『查 SBOX 表』输入 8 个 6 位，输出 8 个 4 位  
  
SBOX\[8,64\]是个二维数组，里面存放了固定的 4 位数据，用六位数据的第 1 位和第 6 位组成 i，第 2、3、4、5 位组成 j，然后取 SBOX\[i,j\]，将取出的 8 个 4 位数据组成 32 位数据。  
  
SBOX 数据太多，这里就不贴出来了。可以在论坛 Crack Tutorial 中找到。  
  
『单纯换位』输入 32 位，输出 32 位  
  
得到的 32 位数据，经单纯换位，既不膨胀，也不收缩，得到 32 位输出  
  
单纯换位表：  
  
16 07 20 21 29 12 28 17 01 15 23 26 05 18 31 10   
02 08 24 14 32 27 03 09 19 13 30 06 22 11 04 25  
  
※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※  
  
子密钥的生成过程：输入 64 位密钥，输出 16 个 48 位子密钥  
  
跟进 CALL unpacked.004768E0：  
  
004768E0  /$  55             PUSH EBP  
004768E1  |.  8BEC           MOV EBP,ESP  
004768E3  |.  83C4 E4        ADD ESP,-1C  
004768E6  |.  53             PUSH EBX  
004768E7  |.  56             PUSH ESI  
004768E8  |.  57             PUSH EDI  
004768E9  |.  8BDA           MOV EBX,EDX  
004768EB  |.  85DB           TEST EBX,EBX  
004768ED  |.  78 0A          JS SHORT unpacked.004768F9  
004768EF  |.  C1EB 02        SHR EBX,2  
004768F2  |>  8B3498         /MOV ESI,DWORD PTR DS:\[EAX+EBX\*4>  
004768F5  |.  4B             |DEC EBX  
004768F6  |.  56             |PUSH ESI  
004768F7  |.^ 79 F9          \\JNS SHORT unpacked.004768F2  
004768F9  |>  8BC4           MOV EAX,ESP  
004768FB  |.  894D FC        MOV DWORD PTR SS:\[EBP-4\],ECX  
004768FE  |.  8D75 F5        LEA ESI,DWORD PTR SS:\[EBP-B\]  
00476901  |.  6A 06          PUSH 6                           ; /Arg1 = 00000006  
00476903  |.  8BCE           MOV ECX,ESI                      ; |  
00476905  |.  E8 46FEFFFF    CALL unpacked.00476750           ; \\数据收缩 A，将 64 位密钥收缩位 56 位  
0047690A  |.  8A06           MOV AL,BYTE PTR DS:\[ESI\]         ;  下面开始将 56 位分为两个 28 位 C0 和 D0  
0047690C  |.  33D2           XOR EDX,EDX  
0047690E  |.  8AD0           MOV DL,AL  
00476910  |.  C1EA 04        SHR EDX,4  
00476913  |.  8855 F1        MOV BYTE PTR SS:\[EBP-F\],DL       ;  保存  
00476916  |.  C1E0 04        SHL EAX,4  
00476919  |.  8A56 01        MOV DL,BYTE PTR DS:\[ESI+1\]  
0047691C  |.  33C9           XOR ECX,ECX  
0047691E  |.  8ACA           MOV CL,DL  
00476920  |.  C1E9 04        SHR ECX,4  
00476923  |.  0AC1           OR AL,CL  
00476925  |.  8845 F2        MOV BYTE PTR SS:\[EBP-E\],AL       ;  保存  
00476928  |.  C1E2 04        SHL EDX,4  
0047692B  |.  8A46 02        MOV AL,BYTE PTR DS:\[ESI+2\]  
0047692E  |.  33C9           XOR ECX,ECX  
00476930  |.  8AC8           MOV CL,AL  
00476932  |.  C1E9 04        SHR ECX,4  
00476935  |.  0AD1           OR DL,CL  
00476937  |.  8855 F3        MOV BYTE PTR SS:\[EBP-D\],DL       ;  保存  
0047693A  |.  C1E0 04        SHL EAX,4  
0047693D  |.  8A56 03        MOV DL,BYTE PTR DS:\[ESI+3\]  
00476940  |.  33C9           XOR ECX,ECX  
00476942  |.  8ACA           MOV CL,DL  
00476944  |.  C1E9 04        SHR ECX,4  
00476947  |.  0AC1           OR AL,CL  
00476949  |.  8845 F4        MOV BYTE PTR SS:\[EBP-C\],AL       ;  保存，上面的四个保存，保存了左 28 位 C0  
0047694C  |.  80E2 0F        AND DL,0F  
0047694F  |.  8855 ED        MOV BYTE PTR SS:\[EBP-13\],DL      ;  保存  
00476952  |.  8A46 04        MOV AL,BYTE PTR DS:\[ESI+4\]  
00476955  |.  8845 EE        MOV BYTE PTR SS:\[EBP-12\],AL      ;  保存  
00476958  |.  8A46 05        MOV AL,BYTE PTR DS:\[ESI+5\]  
0047695B  |.  8845 EF        MOV BYTE PTR SS:\[EBP-11\],AL      ;  保存  
0047695E  |.  8A46 06        MOV AL,BYTE PTR DS:\[ESI+6\]  
00476961  |.  8845 F0        MOV BYTE PTR SS:\[EBP-10\],AL      ;  保存，上面的四个保存，保存了右 28 位 D0  
00476964  |.  BF 10000000    MOV EDI,10  
00476969  |.  BB 40055C00    MOV EBX,OFFSET <unpacked.移位表>    ;  取移位表地址  
0047696E  |.  8B75 FC        MOV ESI,DWORD PTR SS:\[EBP-4\]     ;  下面注释中,i 取 1～16  
00476971  |>  8D45 F1        /LEA EAX,DWORD PTR SS:\[EBP-F\]    ;  左 28 位 C(i-1)  
00476974  |.  8A0B           |MOV CL,BYTE PTR DS:\[EBX\]        ;  取移位表数据  
00476976  |.  BA 03000000    |MOV EDX,3  
0047697B  |.  E8 00FFFFFF    |CALL unpacked.00476880          ;  C(i-1)左移后生成 Ci  
00476980  |.  8D45 ED        |LEA EAX,DWORD PTR SS:\[EBP-13\]   ;  右 28 位 D(i-1)  
00476983  |.  8A0B           |MOV CL,BYTE PTR DS:\[EBX\]        ;  取移位表数据  
00476985  |.  BA 03000000    |MOV EDX,3  
0047698A  |.  E8 F1FEFFFF    |CALL unpacked.00476880          ;  D(i-1)移位生成 Di  
0047698F  |.  8A55 F1        |MOV DL,BYTE PTR SS:\[EBP-F\]  
00476992  |.  C1E2 04        |SHL EDX,4  
00476995  |.  8A45 F2        |MOV AL,BYTE PTR SS:\[EBP-E\]  
00476998  |.  33C9           |XOR ECX,ECX  
0047699A  |.  8AC8           |MOV CL,AL  
0047699C  |.  C1E9 04        |SHR ECX,4  
0047699F  |.  0AD1           |OR DL,CL  
004769A1  |.  8855 E6        |MOV BYTE PTR SS:\[EBP-1A\],DL     ;  保存  
004769A4  |.  C1E0 04        |SHL EAX,4  
004769A7  |.  33D2           |XOR EDX,EDX  
004769A9  |.  8A55 F3        |MOV DL,BYTE PTR SS:\[EBP-D\]  
004769AC  |.  C1EA 04        |SHR EDX,4  
004769AF  |.  0AC2           |OR AL,DL  
004769B1  |.  8845 E7        |MOV BYTE PTR SS:\[EBP-19\],AL     ;  保存  
004769B4  |.  8A55 F3        |MOV DL,BYTE PTR SS:\[EBP-D\]  
004769B7  |.  C1E2 04        |SHL EDX,4  
004769BA  |.  8A45 F4        |MOV AL,BYTE PTR SS:\[EBP-C\]  
004769BD  |.  33C9           |XOR ECX,ECX  
004769BF  |.  8AC8           |MOV CL,AL  
004769C1  |.  C1E9 04        |SHR ECX,4  
004769C4  |.  0AD1           |OR DL,CL  
004769C6  |.  8855 E8        |MOV BYTE PTR SS:\[EBP-18\],DL     ;  保存  
004769C9  |.  C1E0 04        |SHL EAX,4  
004769CC  |.  0A45 ED        |OR AL,BYTE PTR SS:\[EBP-13\]  
004769CF  |.  8845 E9        |MOV BYTE PTR SS:\[EBP-17\],AL     ;  保存  
004769D2  |.  8A45 EE        |MOV AL,BYTE PTR SS:\[EBP-12\]  
004769D5  |.  8845 EA        |MOV BYTE PTR SS:\[EBP-16\],AL     ;  保存  
004769D8  |.  8A45 EF        |MOV AL,BYTE PTR SS:\[EBP-11\]  
004769DB  |.  8845 EB        |MOV BYTE PTR SS:\[EBP-15\],AL     ;  保存  
004769DE  |.  8A45 F0        |MOV AL,BYTE PTR SS:\[EBP-10\]  
004769E1  |.  8845 EC        |MOV BYTE PTR SS:\[EBP-14\],AL     ;  保存，上面 7 个保存，将 Ci 和 Di 合并成 56 位数据  
004769E4  |.  6A 05          |PUSH 5                          ; /Arg1 = 00000005  
004769E6  |.  8BCE           |MOV ECX,ESI                     ; |  
004769E8  |.  8D45 E6        |LEA EAX,DWORD PTR SS:\[EBP-1A\]   ; |  
004769EB  |.  BA 06000000    |MOV EDX,6                       ; |  
004769F0  |.  E8 F3FDFFFF    |CALL unpacked.004767E8          ; \\缩位变换 B，将 56 位数决变换成 48 位的 Ki  
004769F5  |.  83C6 06        |ADD ESI,6  
004769F8  |.  43             |INC EBX  
004769F9  |.  4F             |DEC EDI  
004769FA  |.^ 0F85 71FFFFFF  \\JNZ unpacked.00476971           ;  循环 16 轮,生成 16 个子密钥  
00476A00  |.  8B7D D8        MOV EDI,DWORD PTR SS:\[EBP-28\]  
00476A03  |.  8B75 DC        MOV ESI,DWORD PTR SS:\[EBP-24\]  
00476A06  |.  8B5D E0        MOV EBX,DWORD PTR SS:\[EBP-20\]  
00476A09  |.  8BE5           MOV ESP,EBP  
00476A0B  |.  5D             POP EBP  
00476A0C  \\.  C2 0400        RETN 4  
  
小结：  
  
『缩位变换 A』输入 64 位密钥，输出 56 位数据  
  
57 49 41 33 25 17 09   
01 58 50 42 34 26 18   
10 02 59 51 43 35 27   
19 11 03 60 52 44 36   
63 55 47 39 31 23 15   
07 62 54 46 38 30 22   
14 06 61 53 45 37 29   
21 13 05 28 20 12 04  
  
56 位数据分为两个 28 位，进入 16 轮循环  
  
移位表：1 1 2 2 2 2 2 2 1 2 2 2 2 2 2 1  
  
『缩位变换 B』输入 56 位，输出 48 位 Ki  
  
14 17 11 24 01 05   
03 28 15 06 21 10   
23 19 12 04 26 08   
16 07 27 20 13 02   
41 52 31 37 47 55   
30 40 51 45 33 48   
44 49 39 56 34 53   
46 42 50 36 29 32  
  
※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※  
\----------------------------------------------------------------------------------------------  
【破解声明】   我是一只小菜鸟，偶得一点心得，愿与大家分享:)  
  
【版权声明】   本文纯属技术交流, 转载请注明作者并保持文章的完整, 谢谢!   
\----------------------------------------------------------------------------------------------  
                                                                     文章写于 2005-8-25 17:55:37 

 4dd466d0e1e0cc123afbe504da2462ca