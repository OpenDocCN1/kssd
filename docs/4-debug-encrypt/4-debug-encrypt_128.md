

# 在 rsatools 中已知(Q-1)\*(P-1),E,求 D 的反汇编过程及分析

> 原文：[`www.pediy.com/kssd/pediy09/pediy09-19.htm`](https://www.pediy.com/kssd/pediy09/pediy09-19.htm)

> 使用工具 softice  
> 调试环境：windwos2000  
> 原 RSATOOLS.exe 在论坛的工具中  
> 调试时使用的数据：  
> D2F1F1429A4A565657B25A75341392C5;　　　　　　　　　　　　　　　　　＝Ｑ  
> D2F1F1429A4A565657B25A75341392D7;　　　　　　　　　　　　　　　　　＝Ｐ  
> ADD1D47967E6ED701852554F88EEE22C416B71A26F3961AE922CFD6ECEC39D73;　＝Ｎ  
> D2F1F1429A4A565657B25A75341392CD  
> 79C5ABBE161D81306BC99A7CAAE8F847734867E9C940C4927AA767AE6218B219;  ＝Ｄ  
> \=====================打这里Ｑ　Ｐ　Ｅ 已经得到============================  
> 00405B67  6A00                PUSH      00  
> 00405B69  E8B2200000          CALL      00407C20   ；申请内存  
> 00405B6E  6A00                PUSH      00  
> 00405B70  8BE8                MOV       EBP,EAX  
> 00405B72  E8A9200000          CALL      00407C20   ；申请内存  
> 00405B77  50                  PUSH      EAX  
> 00405B78  57                  PUSH      EDI  
> 00405B79  56                  PUSH      ESI  
> 00405B7A  89442424            MOV       \[ESP+24\],EAX  
> 00405B7E  E81D390000          CALL      004094A0   ；EDI\*ESI-->EAX  
> 00405B83  56                  PUSH      ESI        ；相当于 Q\*P  
> 00405B84  6A01                PUSH      01  
> 00405B86  56                  PUSH      ESI  
> 00405B87  E8944A0000          CALL      0040A620   ；ESI-1-->ESI=\[Q-1\]  
> 00405B8C  57                  PUSH      EDI  
> 00405B8D  6A01                PUSH      01  
> 00405B8F  57                  PUSH      EDI  
> 00405B90  E88B4A0000          CALL      0040A620   ；EDI-1-->EDI=\[P-1\]  
> 00405B95  55                  PUSH      EBP  
> 00405B96  57                  PUSH      EDI  
> 00405B97  56                  PUSH      ESI  
> 00405B98  E803390000          CALL      004094A0   ；相当于(Q-1)\*(P-1)  
> 00405B9D  57                  PUSH      EDI  
> 00405B9E  56                  PUSH      ESI  
> 00405B9F  56                  PUSH      ESI  
> 00405BA0  55                  PUSH      EBP-------->(Q-1)\*(P-1)  
> 00405BA1  53                  PUSH      EBX-------->共钥 E  
> 00405BA2  E879540000          CALL      0040B020   ；根据(Q-1)\*(P-1)，共钥 E，计算私钥 D  
> 00405BA7  83C44C              ADD       ESP,4C  
> 00405BAA  83F801              CMP       EAX,01     ；无法计算出私钥  
> 00405BAD  0F85C2000000        JNZ       00405C75  
> 00405BB3  53                  PUSH      EBX  
> 00405BB4  53                  PUSH      EBX  
> 00405BB5  53                  PUSH      EBX  
> 00405BB6  55                  PUSH      EBP-------->(Q-1)\*(P-1)  
> 00405BB7  53                  PUSH      EBX-------->共钥 E  
> 00405BB8  E863540000          CALL      0040B020   ；根据(Q-1)\*(P-1)，共钥 E，计算私钥 D  
> 00405BBD  A100E44100          MOV       EAX,\[0041E400\]  
> 00405BC2  83C414              ADD       ESP,14  
> 00405BC5  83F803              CMP       EAX,03  
> 00405BC8  7526                JNZ       00405BF0  
> 00405BCA  8D9424F0050000      LEA       EDX,\[ESP+000005F0\]  
> 00405BD1  52                  PUSH      EDX  
> 00405BD2  53                  PUSH      EBX  
> 00405BD3  E8881C0000          CALL      00407860  
> 00405BD8  50                  PUSH      EAX  
> 00405BD9  8D442420            LEA       EAX,\[ESP+20\]  
> 00405BDD  8D8C24FC050000      LEA       ECX,\[ESP+000005FC\]  
> 00405BE4  50                  PUSH      EAX  
> 00405BE5  51                  PUSH      ECX  
> 00405BE6  E865B5FFFF          CALL      00401150  
> 00405BEB  83C414              ADD       ESP,14  
> 00405BEE  EB0E                JMP       00405BFE  
> 00405BF0  8D542414            LEA       EDX,\[ESP+14\]  
> 00405BF4  52                  PUSH      EDX  
> 00405BF5  53                  PUSH      EBX  
> 00405BF6  E8651C0000          CALL      00407860----->把数据转化为可显示的字符串  
> 00405BFB  83C408              ADD       ESP,08  
> 00405BFE  8B8C24D00B0000      MOV       ECX,\[ESP+00000BD0\]  
> 00405C05  8D442414            LEA       EAX,\[ESP+14\]  
> 00405C09  50                  PUSH      EAX   --------->已经算出来的Ｄ  
> 00405C0A  68F5030000          PUSH      000003F5  
> 00405C0F  51                  PUSH      ECX  
> 00405C10  FF1520614100        CALL      \[USER32!SetDlgItemTextA\]---显示  
> 00405C16  833D00E4410003      CMP       DWORD PTR \[0041E400\],03  
> 00405C1D  752A                JNZ       00405C49  
> 00405C1F  8B442410            MOV       EAX,\[ESP+10\]  
>   
> 数据的储格式如下：第一个 dword 的是数据的位数， 其中的 D31  
> 位表示这个数的正负,以 100000000H 为进值表示的  
> DD EBX l24----->E 共钥  
> 0089D5F8 00000001  00010001  00000000  00000000      ................  
> DD EBP l24----->(Q-1)\*(P-1)  
> 0089A718 00000008  669C77D8  E2C84884  3AA4B501      .....w.f.H.....:  
> 0089A728 9B878F1D  88EEE22A  1852554F  67E6ED70      ....\*...OUR.p..g  
> 0089A738 ADD1D479  00000000  00000000  00000000      y...............  
> DD ESI l24----->Q-1  
> 0089DAE0 00000004  341392C4  57B25A75  9A4A5656      .......4uZ.WVVJ.  
> 0089DAF0 D2F1F142  00000000  00000000  00000000      B...............  
> DD EDI l24----->P-1  
> 0089A230 00000004  341392D6  57B25A75  9A4A5656      .......4uZ.WVVJ.  
> 0089A240 D2F1F142  00000000  00000000  00000000      B...............  
> \[ESI+000001D8\]------>X0 用于保存临时的计算结果  
> \[ESI+000001DC\]------>X1 初始化时等于 E  
> \[ESI+000001E0\]------>X2 初始化时等于 (Q-1)\*(P-1)  
> \[ESI+000001E4\]------>X3 初始化时等于 1----->最后结果就是私钥  
> \[ESI+000001E8\]------>X4 初始化时等于 0  
> \[ESI+000001EC\]------>X5 用于保存临时的计算结果  
> \[EBP-04\]------------>Y0 初始化时等于 1   Y3-Y0\*商  
> \[EBP-08\]------------>Y1 初始化时等于 0   Y2-Y1\*商  
> \[EBP-14\]------------>Y2 初始化时等于 1   Y1-->Y2  
> \[EBP-18\]------------>Y3 初始化时等于 0   Y0-->Y3  
>   
> \=========验证是否能得到私有密钥\[上面是调用该程序时积存器所指内存的值================  
> 0040B020  55                  PUSH      EBP  
> 0040B021  8BEC                MOV       EBP,ESP  
> 0040B023  83EC24              SUB       ESP,24  
> 0040B026  56                  PUSH      ESI  
> 0040B027  57                  PUSH      EDI  
> 0040B028  E8C3C8FFFF          CALL      004078F0------>取得一个地址  
> 0040B02D  8BF0                MOV       ESI,EAX      ；该地址存了一些计算时需要的，已经申请的  
> 0040B02F  F8                  CLC                    ；内存  
> 0040B030  8B8630020000        MOV       EAX,\[ESI+00000230\]  
> 0040B036  33FF                XOR       EDI,EDI  
> 0040B038  3BC7                CMP       EAX,EDI  
> 0040B03A  7408                JZ        0040B044  
> 0040B03C  5F                  POP       EDI  
> 0040B03D  33C0                XOR       EAX,EAX  
> 0040B03F  5E                  POP       ESI  
> 0040B040  8BE5                MOV       ESP,EBP  
> 0040B042  5D                  POP       EBP  
> 0040B043  C3                  RET  
> 0040B044  8B86DC010000        MOV       EAX,\[ESI+000001DC\] X1  
> 0040B04A  8B4D08              MOV       ECX,\[EBP+08\]  
> 0040B04D  53                  PUSH      EBX  
> 0040B04E  50                  PUSH      EAX  
> 0040B04F  51                  PUSH      ECX  
> 0040B050  E8CBD5FFFF          CALL      00408620--->共钥Ｅ--->X1  
> 0040B055  8B96E0010000        MOV       EDX,\[ESI+000001E0\] X2  
> 0040B05B  8B450C              MOV       EAX,\[EBP+0C\]  
> 0040B05E  52                  PUSH      EDX  
> 0040B05F  50                  PUSH      EAX  
> 0040B060  E8BBD5FFFF          CALL      00408620--->(Q-1)\*(P-1)--->X2  
> 0040B065  8B8EDC010000        MOV       ECX,\[ESI+000001DC\] X1  
> 0040B06B  51                  PUSH      ECX  
> 0040B06C  E89FD3FFFF          CALL      00408410--->检查数的正负  
> 0040B071  8B96DC010000        MOV       EDX,\[ESI+000001DC\] X1  
> 0040B077  8945DC              MOV       \[EBP-24\],EAX----->存符号  
> 0040B07A  52                  PUSH      EDX  
> 0040B07B  6A01                PUSH      01  
> 0040B07D  E8AED3FFFF          CALL      00408430--->清掉Ｄ31 位\[求绝对值\]  
> 0040B082  8B86E0010000        MOV       EAX,\[ESI+000001E0\] X2  
> 0040B088  50                  PUSH      EAX  
> 0040B089  6A01                PUSH      01  
> 0040B08B  E8A0D3FFFF          CALL      00408430--->清掉Ｄ31 位\[求绝对值\]  
> 0040B090  8B8EE4010000        MOV       ECX,\[ESI+000001E4\] X3  
> 0040B096  51                  PUSH      ECX  
> 0040B097  6A01                PUSH      01  
> 0040B099  E862CAFFFF          CALL      00407B00--->初始化为 1  
> 0040B09E  8B96E8010000        MOV       EDX,\[ESI+000001E8\] X4  
> 0040B0A4  52                  PUSH      EDX  
> 0040B0A5  E836CAFFFF          CALL      00407AE0--->清零  
> 0040B0AA  8B86E0010000        MOV       EAX,\[ESI+000001E0\] X2  
> 0040B0B0  897DE0              MOV       \[EBP-20\],EDI  
> 0040B0B3  50                  PUSH      EAX  
> 0040B0B4  897DFC              MOV       \[EBP-04\],EDI  ；初始化为零 Y0  
> 0040B0B7  897DF8              MOV       \[EBP-08\],EDI  ；初始化为零 Y1  
> 0040B0BA  33DB                XOR       EBX,EBX  
> 0040B0BC  E88FD7FFFF          CALL      00408850--->检查数据的正负［如果小于 40000000 返回这个数据\]  
> 0040B0C1  83C434              ADD       ESP,34  
> 0040B0C4  85C0                TEST      EAX,EAX  
> 0040B0C6  0F8417030000        JZ        0040B3E3  
> 0040B0CC  EB06                JMP       0040B0D4  
> 0040B0CE  8B5DEC              MOV       EBX,\[EBP-14\]       Y2  
> 0040B0D1  8B7DE8              MOV       EDI,\[EBP-18\]       Y3  
> 0040B0D4  85FF                TEST      EDI,EDI  
> 0040B0D6  0F8580000000        JNZ       0040B15C  
> 0040B0DC  8B8EEC010000        MOV       ECX,\[ESI+000001EC\] X5  
> 0040B0E2  8B96E0010000        MOV       EDX,\[ESI+000001E0\] X2  
> 0040B0E8  8B86DC010000        MOV       EAX,\[ESI+000001DC\] X1  
> 0040B0EE  51                  PUSH      ECX-->X5  
> 0040B0EF  52                  PUSH      EDX-->X2  
> 0040B0F0  50                  PUSH      EAX-->X1    ;X1 / X2-->X5  
> 0040B0F1  E8DAE7FFFF          CALL      004098D0--->;X1 % X2-->X1  
> 0040B0F6  8B86DC010000        MOV       EAX,\[ESI+000001DC\] X1  
> 0040B0FC  8B8EE0010000        MOV       ECX,\[ESI+000001E0\] X2  
> 0040B102  8B96D8010000        MOV       EDX,\[ESI+000001D8\] X0　  
> 0040B108  8986E0010000        MOV       \[ESI+000001E0\],EAX X1-->X2　  
> 0040B10E  8B86EC010000        MOV       EAX,\[ESI+000001EC\] X5  
> 0040B114  898EDC010000        MOV       \[ESI+000001DC\],ECX X2-->X1  
> 0040B11A  8B8EE8010000        MOV       ECX,\[ESI+000001E8\] X4  
> 0040B120  52                  PUSH      EDX-->X0  
> 0040B121  50                  PUSH      EAX-->X5  
> 0040B122  51                  PUSH      ECX-->X4  
> 0040B123  E878E3FFFF          CALL      004094A0--->X4\*X5-->X0  
> 0040B128  8B86E4010000        MOV       EAX,\[ESI+000001E4\] X3  
> 0040B12E  8B96D8010000        MOV       EDX,\[ESI+000001D8\] X0  
> 0040B134  50                  PUSH      EAX-->X3  
> 0040B135  52                  PUSH      EDX-->X0  
> 0040B136  50                  PUSH      EAX-->X3  
> 0040B137  E864F4FFFF          CALL      0040A5A0--->X3-X0-->X3  
> 0040B13C  8B86E4010000        MOV       EAX,\[ESI+000001E4\] X3  
> 0040B142  8B8EE8010000        MOV       ECX,\[ESI+000001E8\] X4  
> 0040B148  83C424              ADD       ESP,24  
> 0040B14B  898EE4010000        MOV       \[ESI+000001E4\],ECX X4--->X3  
> 0040B151  8986E8010000        MOV       \[ESI+000001E8\],EAX X3--->X4  
> 0040B157  E9EA000000          JMP       0040B246  
> 0040B15C  8B96EC010000        MOV       EDX,\[ESI+000001EC\] X5  
> 0040B162  8B45F8              MOV       EAX,\[EBP-08\]       Y1  
> 0040B165  8B8EDC010000        MOV       ECX,\[ESI+000001DC\] X1  
> 0040B16B  52                  PUSH      EDX--->X5  
> 0040B16C  50                  PUSH      EAX--->Y1  
> 0040B16D  51                  PUSH      ECX--->X1  
> 0040B16E  E8EDDFFFFF          CALL      00409160--->X1\*Y1-->X5  
> 0040B173  8B86DC010000        MOV       EAX,\[ESI+000001DC\] X1  
> 0040B179  50                  PUSH      EAX--->X1  
> 0040B17A  53                  PUSH      EBX--->Y2  
> 0040B17B  50                  PUSH      EAX--->X1  
> 0040B17C  E8DFDFFFFF          CALL      00409160--->X0\*Y2-->X2  
> 0040B181  8B96D8010000        MOV       EDX,\[ESI+000001D8\] X0  
> 0040B187  8B86E0010000        MOV       EAX,\[ESI+000001E0\] X2  
> 0040B18D  52                  PUSH      EDX--->X0  
> 0040B18E  57                  PUSH      EDI--->Y3  
> 0040B18F  50                  PUSH      EAX--->X2  
> 0040B190  E8CBDFFFFF          CALL      00409160--->X2\*Y3-->X0  
> 0040B195  8B86E0010000        MOV       EAX,\[ESI+000001E0\] X2  
> 0040B19B  8B4DFC              MOV       ECX,\[EBP-04\]       Y0  
> 0040B19E  50                  PUSH      EAX-->X2  
> 0040B19F  51                  PUSH      ECX-->Y0  
> 0040B1A0  50                  PUSH      EAX-->X2  
> 0040B1A1  E8BADFFFFF          CALL      00409160--->X2\*Y0-->X2  
> 0040B1A6  8B86DC010000        MOV       EAX,\[ESI+000001DC\] X1  
> 0040B1AC  8B96D8010000        MOV       EDX,\[ESI+000001D8\] X0  
> 0040B1B2  50                  PUSH      EAX-->X1  
> 0040B1B3  52                  PUSH      EDX-->X0  
> 0040B1B4  50                  PUSH      EAX-->X1  
> 0040B1B5  E896F2FFFF          CALL      0040A450--->X1+X0-->X1  
> 0040B1BA  8B86E0010000        MOV       EAX,\[ESI+000001E0\] X2  
> 0040B1C0  8B8EEC010000        MOV       ECX,\[ESI+000001EC\] X5  
> 0040B1C6  50                  PUSH      EAX-->X2  
> 0040B1C7  51                  PUSH      ECX-->X5  
> 0040B1C8  50                  PUSH      EAX-->X2  
> 0040B1C9  E882F2FFFF          CALL      0040A450--->X2+X5-->X2  
> 0040B1CE  8B96EC010000        MOV       EDX,\[ESI+000001EC\] X5  
> 0040B1D4  8B45F8              MOV       EAX,\[EBP-08\]       Y1  
> 0040B1D7  8B8EE4010000        MOV       ECX,\[ESI+000001E4\] X3  
> 0040B1DD  83C448              ADD       ESP,48  
> 0040B1E0  52                  PUSH      EDX-->X5  
> 0040B1E1  50                  PUSH      EAX-->Y1  
> 0040B1E2  51                  PUSH      ECX-->X3  
> 0040B1E3  E878DFFFFF          CALL      00409160--->X3\*Y1-->X5  
> 0040B1E8  8B86E4010000        MOV       EAX,\[ESI+000001E4\] X3  
> 0040B1EE  50                  PUSH      EAX-->X3  
> 0040B1EF  53                  PUSH      EBX-->Y2  
> 0040B1F0  50                  PUSH      EAX-->X3  
> 0040B1F1  E86ADFFFFF          CALL      00409160--->X3\*Y2-->X3  
> 0040B1F6  8B96D8010000        MOV       EDX,\[ESI+000001D8\] X0  
> 0040B1FC  8B86E8010000        MOV       EAX,\[ESI+000001E8\] X4  
> 0040B202  52                  PUSH      EDX-->X0  
> 0040B203  57                  PUSH      EDI-->Y3  
> 0040B204  50                  PUSH      EAX-->X4  
> 0040B205  E856DFFFFF          CALL      00409160--->X4\*Y3-->X0  
> 0040B20A  8B86E8010000        MOV       EAX,\[ESI+000001E8\] X4  
> 0040B210  8B4DFC              MOV       ECX,\[EBP-04\]       Y0  
> 0040B213  50                  PUSH      EAX-->X4  
> 0040B214  51                  PUSH      ECX-->Y0  
> 0040B215  50                  PUSH      EAX-->X4  
> 0040B216  E845DFFFFF          CALL      00409160--->X4\*Y0-->X4  
> 0040B21B  8B86E4010000        MOV       EAX,\[ESI+000001E4\] X3  
> 0040B221  8B96D8010000        MOV       EDX,\[ESI+000001D8\] X0  
> 0040B227  50                  PUSH      EAX-->X3  
> 0040B228  52                  PUSH      EDX-->X0  
> 0040B229  50                  PUSH      EAX-->X3  
> 0040B22A  E821F2FFFF          CALL      0040A450--->X3+X0-->X3  
> 0040B22F  8B86E8010000        MOV       EAX,\[ESI+000001E8\] X4  
> 0040B235  8B8EEC010000        MOV       ECX,\[ESI+000001EC\] X5  
> 0040B23B  50                  PUSH      EAX-->X4  
> 0040B23C  51                  PUSH      ECX-->X5  
> 0040B23D  50                  PUSH      EAX-->X4  
> 0040B23E  E80DF2FFFF          CALL      0040A450--->X4+X5-->X4  
> 0040B243  83C448              ADD       ESP,48  
> 0040B246  8B8630020000        MOV       EAX,\[ESI+00000230\]  
> 0040B24C  33DB                XOR       EBX,EBX  
> 0040B24E  3BC3                CMP       EAX,EBX  
> 0040B250  0F858D010000        JNZ       0040B3E3  
> 0040B256  8B96E0010000        MOV       EDX,\[ESI+000001E0\] X2  
> 0040B25C  52                  PUSH      EDX  
> 0040B25D  E8EED5FFFF          CALL      00408850--->检查数据的正负［如果小于 40000000 返回这个数据\]  
> 0040B262  83C404              ADD       ESP,04  
> 0040B265  85C0                TEST      EAX,EAX  
> 0040B267  0F8476010000        JZ        0040B3E3  
> 0040B26D  8B86DC010000        MOV       EAX,\[ESI+000001DC\] X1  
> 0040B273  B901000000          MOV       ECX,00000001  
> 0040B278  894DEC              MOV       \[EBP-14\],ECX  ；初始化为 1-->Y2  
> 0040B27B  895DE8              MOV       \[EBP-18\],EBX  ；初始化为 0-->Y3  
> 0040B27E  8B38                MOV       EDI,\[EAX\]  
> 0040B280  895DF8              MOV       \[EBP-08\],EBX  ；初始化为 0-->Y1  
> 0040B283  3BF9                CMP       EDI,ECX  
> 0040B285  894DFC              MOV       \[EBP-04\],ECX  ；初始化为 1-->Y0  
> 0040B288  7517                JNZ       0040B2A1  
> 0040B28A  8B5804              MOV       EBX,\[EAX+04\]  
> 0040B28D  8B86E0010000        MOV       EAX,\[ESI+000001E0\] X2  
> 0040B293  894DE0              MOV       \[EBP-20\],ECX  
> 0040B296  895DF0              MOV       \[EBP-10\],EBX  
> 0040B299  8B7804              MOV       EDI,\[EAX+04\]  
> 0040B29C  E995000000          JMP       0040B336  
> 0040B2A1  8B1CB8              MOV       EBX,\[EDI\*4+EAX\]  
> 0040B2A4  8B16                MOV       EDX,\[ESI\]  
> 0040B2A6  85D2                TEST      EDX,EDX  
> 0040B2A8  8D4B01              LEA       ECX,\[EBX+01\]  
> 0040B2AB  894DE4              MOV       \[EBP-1C\],ECX  
> 0040B2AE  7549                JNZ       0040B2F9  
> 0040B2B0  85C9                TEST      ECX,ECX  
> 0040B2B2  750E                JNZ       0040B2C2  
> 0040B2B4  8B8EE0010000        MOV       ECX,\[ESI+000001E0\] X2  
> 0040B2BA  895DF0              MOV       \[EBP-10\],EBX  
> 0040B2BD  8B3CB9              MOV       EDI,\[EDI\*4+ECX\]  
> 0040B2C0  EB74                JMP       0040B336  
> 0040B2C2  8B44B8FC            MOV       EAX,\[EDI\*4+EAX-04\]  
> 0040B2C6  8D55F4              LEA       EDX,\[EBP-0C\]  
> 0040B2C9  52                  PUSH      EDX  
> 0040B2CA  51                  PUSH      ECX  
> 0040B2CB  50                  PUSH      EAX  
> 0040B2CC  53                  PUSH      EBX  
> 0040B2CD  E8CE270000          CALL      0040DAA0--->  
> 0040B2D2  8B8EE0010000        MOV       ECX,\[ESI+000001E0\] X2  
> 0040B2D8  8BD8                MOV       EBX,EAX  
> 0040B2DA  8B45E4              MOV       EAX,\[EBP-1C\]  
> 0040B2DD  8D55F4              LEA       EDX,\[EBP-0C\]  
> 0040B2E0  8D3CB9              LEA       EDI,\[EDI\*4+ECX\]  
> 0040B2E3  52                  PUSH      EDX  
> 0040B2E4  50                  PUSH      EAX  
> 0040B2E5  895DF0              MOV       \[EBP-10\],EBX  
> 0040B2E8  8B4FFC              MOV       ECX,\[EDI-04\]  
> 0040B2EB  8B17                MOV       EDX,\[EDI\]  
> 0040B2ED  51                  PUSH      ECX  
> 0040B2EE  52                  PUSH      EDX  
> 0040B2EF  E8AC270000          CALL      0040DAA0--->  
> 0040B2F4  83C420              ADD       ESP,20  
> 0040B2F7  EB3B                JMP       0040B334  
> 0040B2F9  8B44B8FC            MOV       EAX,\[EDI\*4+EAX-04\]  
> 0040B2FD  8D55F4              LEA       EDX,\[EBP-0C\]  
> 0040B300  52                  PUSH      EDX  
> 0040B301  51                  PUSH      ECX  
> 0040B302  50                  PUSH      EAX  
> 0040B303  8B06                MOV       EAX,\[ESI\]  
> 0040B305  50                  PUSH      EAX  
> 0040B306  53                  PUSH      EBX  
> 0040B307  E874270000          CALL      0040DA80--->  
> 0040B30C  8B8EE0010000        MOV       ECX,\[ESI+000001E0\] X2  
> 0040B312  8BD8                MOV       EBX,EAX  
> 0040B314  8B45E4              MOV       EAX,\[EBP-1C\]  
> 0040B317  8D55F4              LEA       EDX,\[EBP-0C\]  
> 0040B31A  8D3CB9              LEA       EDI,\[EDI\*4+ECX\]  
> 0040B31D  52                  PUSH      EDX  
> 0040B31E  8B16                MOV       EDX,\[ESI\]  
> 0040B320  50                  PUSH      EAX  
> 0040B321  8B4FFC              MOV       ECX,\[EDI-04\]  
> 0040B324  8B07                MOV       EAX,\[EDI\]  
> 0040B326  51                  PUSH      ECX  
> 0040B327  52                  PUSH      EDX  
> 0040B328  50                  PUSH      EAX  
> 0040B329  895DF0              MOV       \[EBP-10\],EBX  
> 0040B32C  E84F270000          CALL      0040DA80--->  
> 0040B331  83C428              ADD       ESP,28  
> 0040B334  8BF8                MOV       EDI,EAX  
> 0040B336  8B45E0              MOV       EAX,\[EBP-20\]  
> 0040B339  85C0                TEST      EAX,EAX  
> 0040B33B  7412                JZ        0040B34F  
> 0040B33D  85FF                TEST      EDI,EDI  
> 0040B33F  0F8487000000        JZ        0040B3CC  
> 0040B345  8BC3                MOV       EAX,EBX  
> 0040B347  33D2                XOR       EDX,EDX  
> 0040B349  F7F7                DIV       EDI  
> 0040B34B  8BC8                MOV       ECX,EAX  
> 0040B34D  EB30                JMP       0040B37F  
> 0040B34F  8B4DF8              MOV       ECX,\[EBP-08\]       Y1  
> 0040B352  03CF                ADD       ECX,EDI  
> 0040B354  85C9                TEST      ECX,ECX  
> 0040B356  7474                JZ        0040B3CC  
> 0040B358  8B55FC              MOV       EDX,\[EBP-04\]       Y0  
> 0040B35B  8D0417              LEA       EAX,\[EDX+EDI\]  
> 0040B35E  85C0                TEST      EAX,EAX  
> 0040B360  8945E4              MOV       \[EBP-1C\],EAX  
> 0040B363  7467                JZ        0040B3CC  
> 0040B365  8B45EC              MOV       EAX,\[EBP-14\]       Y2  
> 0040B368  33D2                XOR       EDX,EDX  
> 0040B36A  03C3                ADD       EAX,EBX  
> 0040B36C  F7F1                DIV       ECX  
> 0040B36E  8B55E8              MOV       EDX,\[EBP-18\]       Y3  
> 0040B371  8BC8                MOV       ECX,EAX  
> 0040B373  8D0413              LEA       EAX,\[EDX+EBX\]  
> 0040B376  33D2                XOR       EDX,EDX  
> 0040B378  F775E4              DIV       DWORD PTR \[EBP-1C\]  
> 0040B37B  3BC8                CMP       ECX,EAX  
> 0040B37D  754D                JNZ       0040B3CC  
> 0040B37F  8B5DFC              MOV       EBX,\[EBP-04\]       Y0  
> 0040B382  85DB                TEST      EBX,EBX  
> 0040B384  7D02                JGE       0040B388  
> 0040B386  F7DB                NEG       EBX  
> 0040B388  B800000040          MOV       EAX,40000000  
> 0040B38D  99                  CDQ  
> 0040B38E  F7FB                IDIV      EBX  
> 0040B390  3BC8                CMP       ECX,EAX  
> 0040B392  7338                JAE       0040B3CC  
> 0040B394  8B55F8              MOV       EDX,\[EBP-08\]       Y1  
> 0040B397  8BD9                MOV       EBX,ECX  
> 0040B399  0FAFDA              IMUL      EBX,EDX  
> 0040B39C  8B45EC              MOV       EAX,\[EBP-14\]       Y2  
> 0040B39F  8955EC              MOV       \[EBP-14\],EDX   --->Y2  
> 0040B3A2  8B55FC              MOV       EDX,\[EBP-04\]       Y0  
> 0040B3A5  2BC3                SUB       EAX,EBX  
> 0040B3A7  8BD9                MOV       EBX,ECX  
> 0040B3A9  0FAFCF              IMUL      ECX,EDI  
> 0040B3AC  0FAFDA              IMUL      EBX,EDX  
> 0040B3AF  8945F8              MOV       \[EBP-08\],EAX   --->Y1  
> 0040B3B2  8B45E8              MOV       EAX,\[EBP-18\]       Y3  
> 0040B3B5  2BC3                SUB       EAX,EBX  
> 0040B3B7  8BDF                MOV       EBX,EDI  
> 0040B3B9  8945FC              MOV       \[EBP-04\],EAX   --->Y0  
> 0040B3BC  8B45F0              MOV       EAX,\[EBP-10\]  
> 0040B3BF  8955E8              MOV       \[EBP-18\],EDX   --->Y3  
> 0040B3C2  2BC1                SUB       EAX,ECX  
> 0040B3C4  895DF0              MOV       \[EBP-10\],EBX  
> 0040B3C7  E968FFFFFF          JMP       0040B334  
> 0040B3CC  8B86E0010000        MOV       EAX,\[ESI+000001E0\] X2  
> 0040B3D2  50                  PUSH      EAX  
> 0040B3D3  E878D4FFFF          CALL      00408850--->检查数据的正负［如果小于 40000000 返回这个数据\]  
> 0040B3D8  83C404              ADD       ESP,04  
> 0040B3DB  85C0                TEST      EAX,EAX  
> 0040B3DD  0F85EBFCFFFF        JNZ       0040B0CE  
> 0040B3E3  837DDCFF            CMP       DWORD PTR \[EBP-24\],-01  
> 0040B3E7  7510                JNZ       0040B3F9  
> 0040B3E9  8B86E4010000        MOV       EAX,\[ESI+000001E4\] X3  
> 0040B3EF  50                  PUSH      EAX  
> 0040B3F0  50                  PUSH      EAX  
> 0040B3F1  E89AD2FFFF          CALL      00408690--->  
> 0040B3F6  83C408              ADD       ESP,08  
> 0040B3F9  8B8EE4010000        MOV       ECX,\[ESI+000001E4\] X3  
> 0040B3FF  51                  PUSH      ECX  
> 0040B400  E84BD4FFFF          CALL      00408850--->检查数据的正负［如果小于 40000000 返回这个数据\]  
> 0040B405  83C404              ADD       ESP,04  
> 0040B408  85C0                TEST      EAX,EAX  
> 0040B40A  7F14                JG        0040B420  
> 0040B40C  8B86E4010000        MOV       EAX,\[ESI+000001E4\] X3  
> 0040B412  8B550C              MOV       EDX,\[EBP+0C\]  
> 0040B415  50                  PUSH      EAX  
> 0040B416  52                  PUSH      EDX  
> 0040B417  50                  PUSH      EAX  
> 0040B418  E833F0FFFF          CALL      0040A450--->  
> 0040B41D  83C40C              ADD       ESP,0C  
> 0040B420  8B5D10              MOV       EBX,\[EBP+10\]  
> 0040B423  8B7D14              MOV       EDI,\[EBP+14\]  
> 0040B426  3BDF                CMP       EBX,EDI  
> 0040B428  7446                JZ        0040B470  
> 0040B42A  8B86E0010000        MOV       EAX,\[ESI+000001E0\] X2  
> 0040B430  8B4D08              MOV       ECX,\[EBP+08\]  
> 0040B433  50                  PUSH      EAX  
> 0040B434  51                  PUSH      ECX  
> 0040B435  E856D2FFFF          CALL      00408690--->  
> 0040B43A  8B86E8010000        MOV       EAX,\[ESI+000001E8\] X4  
> 0040B440  8B550C              MOV       EDX,\[EBP+0C\]  
> 0040B443  8B8EE4010000        MOV       ECX,\[ESI+000001E4\] X3  
> 0040B449  50                  PUSH      EAX  
> 0040B44A  50                  PUSH      EAX  
> 0040B44B  8B86DC010000        MOV       EAX,\[ESI+000001DC\] X1  
> 0040B451  52                  PUSH      EDX  
> 0040B452  8B96E0010000        MOV       EDX,\[ESI+000001E0\] X2  
> 0040B458  50                  PUSH      EAX  
> 0040B459  51                  PUSH      ECX  
> 0040B45A  52                  PUSH      EDX  
> 0040B45B  E8B0ECFFFF          CALL      0040A110--->  
> 0040B460  8B86E8010000        MOV       EAX,\[ESI+000001E8\] X4  
> 0040B466  57                  PUSH      EDI  
> 0040B467  50                  PUSH      EAX  
> 0040B468  E8B3D1FFFF          CALL      00408620--->  
> 0040B46D  83C428              ADD       ESP,28  
> 0040B470  8B8EE4010000        MOV       ECX,\[ESI+000001E4\] X3  
> 0040B476  53                  PUSH      EBX  
> 0040B477  51                  PUSH      ECX  
> 0040B478  E8A3D1FFFF          CALL      00408620--->  
> 0040B47D  8B4518              MOV       EAX,\[EBP+18\]  
> 0040B480  83C408              ADD       ESP,08  
> 0040B483  3BC3                CMP       EAX,EBX  
> 0040B485  5B                  POP       EBX  
> 0040B486  7414                JZ        0040B49C  
> 0040B488  3BC7                CMP       EAX,EDI  
> 0040B48A  7410                JZ        0040B49C  
> 0040B48C  8B96DC010000        MOV       EDX,\[ESI+000001DC\] X1  
> 0040B492  50                  PUSH      EAX  
> 0040B493  52                  PUSH      EDX  
> 0040B494  E887D1FFFF          CALL      00408620--->  
> 0040B499  83C408              ADD       ESP,08  
> 0040B49C  8B86DC010000        MOV       EAX,\[ESI+000001DC\] X1  
> 0040B4A2  50                  PUSH      EAX  
> 0040B4A3  E8A8D3FFFF          CALL      00408850--->检查数据的正负［如果小于 40000000 返回这个数据\]  
> 0040B4A8  83C404              ADD       ESP,04  
> 0040B4AB  5F                  POP       EDI  
> 0040B4AC  5E                  POP       ESI  
> 0040B4AD  8BE5                MOV       ESP,EBP  
> 0040B4AF  5D                  POP       EBP  
> 0040B4B0  C3                  RET 

 4dd466d0e1e0cc123afbe504da2462ca