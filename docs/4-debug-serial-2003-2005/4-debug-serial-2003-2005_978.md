
# 中学 CAI 课件、积件管理 V1.0 的注册

> 原文：[`www.pediy.com/kssd/pediy07/pediy7-201.htm`](https://www.pediy.com/kssd/pediy07/pediy7-201.htm)

【破文作者】 rdsnow[PYG]  
【 E-mail 】 RDSNOW@163.COM  
【 作者 QQ 】 83757177  
【文章题目】 中学 CAI 课件、积件管理 V1.0 的注册  
【软件名称】 中学 CAI 课件、积件管理 V1.0  
【下载地址】 [`www.softreg.com.cn/shareware...5-45CEAC8CDE5F/`](http://www.softreg.com.cn/shareware_view.aspx?id=/65FB0BC4-A791-45A4-8545-45CEAC8CDE5F/)  

----------------------------------------------------------------------------------------------  

【加密方式】 注册码  
【破解工具】 `FLYOD V1.10`、`W32DASM V8.93`  
【软件限制】 功能限制  
【破解平台】 `XP SP2`  

----------------------------------------------------------------------------------------------  

【软件简介】 该软件提供了易用的课件（积件）收集、分类、浏览、调用等功能，通过它就可以将分散的课件、积件很好地管理起来，使广大教师能够充分地利用这些课件和积件资源。  

【文章简介】  

作者为这个软件的注册写了很多代码，但是却采用了明码比较，这就可以让破解者在短短几分钟内得到注册码！这是软件作者的失误，另外，注册时虽然有注册信息码，而且计算过程中也有对信息码的处理，但是却不影响到最后的注册码！让我们来看看这个软件的注册。  

----------------------------------------------------------------------------------------------  

【破解过程】  

`OD` 载入，输入用户名"Rdsnow"，假注册码"987A-8765B-7654C"，来到这里：

```
004F60EA  |>  \8D55 E4       LEA EDX,DWORD PTR SS:[EBP-1C]  
004F60ED  |.   8B83 14030000 MOV EAX,DWORD PTR DS:[EBX+314]  
004F60F3  |.   E8 4C5AF5FF   CALL Unpacked.0044BB44             ;  取得注册信息码  
004F60F8  |.   8B45 E4       MOV EAX,DWORD PTR SS:[EBP-1C]  
004F60FB  |.   50           PUSH EAX  
004F60FC  |.   8D55 E0       LEA EDX,DWORD PTR SS:[EBP-20]  
004F60FF  |.   8B83 10030000 MOV EAX,DWORD PTR DS:[EBX+310]  
004F6105  |.   E8 3A5AF5FF   CALL Unpacked.0044BB44             ;  取得用户名  
004F610A  |.   8B45 E0       MOV EAX,DWORD PTR SS:[EBP-20]  
004F610D  |.   8D4D E8       LEA ECX,DWORD PTR SS:[EBP-18]  
004F6110  |.   5A           POP EDX  
004F6111  |.   E8 42F4FFFF   CALL Unpacked.004F5558             ;  算注册码  
004F6116  |.   8B45 E8       MOV EAX,DWORD PTR SS:[EBP-18]  
004F6119  |.   50           PUSH EAX  
004F611A  |.   8D55 D8       LEA EDX,DWORD PTR SS:[EBP-28]  
004F611D  |.   8B83 18030000 MOV EAX,DWORD PTR DS:[EBX+318]  
004F6123  |.   E8 1C5AF5FF   CALL Unpacked.0044BB44             ;  取出输入的假注册码的第一段  
004F6128  |.   FF75 D8       PUSH DWORD PTR SS:[EBP-28]  
004F612B  |.   8D55 D4       LEA EDX,DWORD PTR SS:[EBP-2C]  
004F612E  |.   8B83 1C030000 MOV EAX,DWORD PTR DS:[EBX+31C]  
004F6134  |.   E8 0B5AF5FF   CALL Unpacked.0044BB44             ;  取出输入的假注册码的第二段  
004F6139  |.   FF75 D4       PUSH DWORD PTR SS:[EBP-2C]  
004F613C  |.   8D55 D0       LEA EDX,DWORD PTR SS:[EBP-30]  
004F613F  |.   8B83 20030000 MOV EAX,DWORD PTR DS:[EBX+320]  
004F6145  |.   E8 FA59F5FF   CALL Unpacked.0044BB44             ;  取出输入的假注册码的第三段  
004F614A  |.   FF75 D0       PUSH DWORD PTR SS:[EBP-30]  
004F614D  |.   8D45 DC       LEA EAX,DWORD PTR SS:[EBP-24]  
004F6150  |.   BA 03000000   MOV EDX,3  
004F6155  |.   E8 5EEBF0FF   CALL Unpacked.00404CB8             ;  连接三段假码(不包括'-')  
004F615A  |.   8B45 DC       MOV EAX,DWORD PTR SS:[EBP-24]  
004F615D  |.   5A           POP EDX  
004F615E  |.   E8 E12DF1FF   CALL Unpacked.00408F44             ;  比较真假注册码  
004F6163  |.   85C0         TEST EAX,EAX  
004F6165  |.   0F85 6D010000 JNZ Unpacked.004F62D8              ;  不相等就跳  
```

如果不想分析注册代码，在 `004F615E` 地址上中断，做内存注册机，或者将 `004F6165 JNZ 004F62D8` 直接 `NOP` 掉就可以爆破了。

----------------------------------------------------------------------------------------------

想看注册代码，进入 `004F6111 CALL 004F5558`

```
004F5558  $  55           PUSH EBP  
004F5559  |. 8BEC         MOV EBP,ESP  
004F555B  |. 51           PUSH ECX  
004F555C  |. B9 0D000000  MOV ECX,0D  
004F5561  |>  6A 00       PUSH 0  
004F5563  |. 6A 00        |PUSH 0  
004F5565  |. 49           |DEC ECX  
004F5566  |.^ 75 F9      \|JNZ SHORT Unpacked.004F5561  
004F5568  |. 51           PUSH ECX  
004F5569  |. 874D FC      XCHG DWORD PTR SS:[EBP-4],ECX  
004F556C  |. 53           PUSH EBX  
004F556D  |. 56           PUSH ESI  
004F556E  |. 57           PUSH EDI  
004F556F  |. 894D F4      MOV DWORD PTR SS:[EBP-C],ECX  
004F5572  |. 8955 F8      MOV DWORD PTR SS:[EBP-8],EDX  
004F5575  |. 8945 FC      MOV DWORD PTR SS:[EBP-4],EAX  
004F5578  |. 8B45 FC      MOV EAX,DWORD PTR SS:[EBP-4]  
004F557B  |. E8 60F8F0FF  CALL Unpacked.00404DE0  
004F5580  |. 8B45 F8      MOV EAX,DWORD PTR SS:[EBP-8]  
004F5583  |. E8 58F8F0FF  CALL Unpacked.00404DE0  
004F5588  |. 33C0         XOR EAX,EAX  
004F558A  |. 55           PUSH EBP  
004F558B  |. 68 66584F00  PUSH Unpacked.004F5866  
004F5590  |. 64:FF30      PUSH DWORD PTR FS:[EAX]  
004F5593  |. 64:8920      MOV DWORD PTR FS:[EAX],ESP  
004F5596  |. 8D45 E4      LEA EAX,DWORD PTR SS:[EBP-1C]  
004F5599  |. E8 A2F3F0FF  CALL Unpacked.00404940  
004F559E  |. 8D45 D4      LEA EAX,DWORD PTR SS:[EBP-2C]  
004F55A1  |. 50           PUSH EAX  
004F55A2  |. 8B45 FC      MOV EAX,DWORD PTR SS:[EBP-4]  
004F55A5  |. E8 4EF6F0FF  CALL Unpacked.00404BF8             ;  取得用户名"Rdsnow"的长度  
004F55AA  |. 8BC8         MOV ECX,EAX  
004F55AC  |. 83E9 03      SUB ECX,3  
004F55AF  |. BA 04000000  MOV EDX,4  
004F55B4  |. 8B45 FC      MOV EAX,DWORD PTR SS:[EBP-4]  
004F55B7  |. E8 94F8F0FF  CALL Unpacked.00404E50             ;  从第四个字符开始取用户名后面的字符，得到"now"  
004F55BC  |. 8B45 D4      MOV EAX,DWORD PTR SS:[EBP-2C]  
004F55BF  |. 50           PUSH EAX  
004F55C0  |. 8D45 CC      LEA EAX,DWORD PTR SS:[EBP-34]  
004F55C3  |. 8B55 FC      MOV EDX,DWORD PTR SS:[EBP-4]  
004F55C6  |. 8A12         MOV DL,BYTE PTR DS:[EDX]  
004F55C8  |. 8850 01      MOV BYTE PTR DS:[EAX+1],DL  
004F55CB  |. C600 01      MOV BYTE PTR DS:[EAX],1  
004F55CE  |. 8D55 CC      LEA EDX,DWORD PTR SS:[EBP-34]  
004F55D1  |. 8D45 C8      LEA EAX,DWORD PTR SS:[EBP-38]  
004F55D4  |. E8 A7D9F0FF  CALL Unpacked.00402F80  
004F55D9  |. BA 74584F00  MOV EDX,Unpacked.004F5874  
004F55DE  |. 8D45 C8      LEA EAX,DWORD PTR SS:[EBP-38]  
004F55E1  |. B1 02        MOV CL,2  
004F55E3  |. E8 68D9F0FF  CALL Unpacked.00402F50  
004F55E8  |. 8D55 C8      LEA EDX,DWORD PTR SS:[EBP-38]  
004F55EB  |. 8D45 C4      LEA EAX,DWORD PTR SS:[EBP-3C]  
004F55EE  |. E8 8DD9F0FF  CALL Unpacked.00402F80  
004F55F3  |. 8D45 C0      LEA EAX,DWORD PTR SS:[EBP-40]  
004F55F6  |. 8B55 FC      MOV EDX,DWORD PTR SS:[EBP-4]  
004F55F9  |. 8A52 01      MOV DL,BYTE PTR DS:[EDX+1]  
004F55FC  |. 8850 01      MOV BYTE PTR DS:[EAX+1],DL  
004F55FF  |. C600 01      MOV BYTE PTR DS:[EAX],1  
004F5602  |. 8D55 C0      LEA EDX,DWORD PTR SS:[EBP-40]  
004F5605  |. 8D45 C4      LEA EAX,DWORD PTR SS:[EBP-3C]  
004F5608  |. B1 03        MOV CL,3  
004F560A  |. E8 41D9F0FF  CALL Unpacked.00402F50  
004F560F  |. 8D55 C4      LEA EDX,DWORD PTR SS:[EBP-3C]  
004F5612  |. 8D45 B8      LEA EAX,DWORD PTR SS:[EBP-48]  
004F5615  |. E8 66D9F0FF  CALL Unpacked.00402F80  
004F561A  |. BA 78584F00  MOV EDX,Unpacked.004F5878  
004F561F  |. 8D45 B8      LEA EAX,DWORD PTR SS:[EBP-48]  
004F5622  |. B1 04        MOV CL,4  
004F5624  |. E8 27D9F0FF  CALL Unpacked.00402F50  
004F5629  |. 8D55 B8      LEA EDX,DWORD PTR SS:[EBP-48]  
004F562C  |. 8D45 B0      LEA EAX,DWORD PTR SS:[EBP-50]  
004F562F  |. E8 4CD9F0FF  CALL Unpacked.00402F80  
004F5634  |. 8D45 C0      LEA EAX,DWORD PTR SS:[EBP-40]  
004F5637  |. 8B55 FC      MOV EDX,DWORD PTR SS:[EBP-4]  
004F563A  |. 8A52 02      MOV DL,BYTE PTR DS:[EDX+2]  
004F563D  |. 8850 01      MOV BYTE PTR DS:[EAX+1],DL  
004F5640  |. C600 01      MOV BYTE PTR DS:[EAX],1  
004F5643  |. 8D55 C0      LEA EDX,DWORD PTR SS:[EBP-40]  
004F5646  |. 8D45 B0      LEA EAX,DWORD PTR SS:[EBP-50]  
004F5649  |. B1 05        MOV CL,5  
004F564B  |. E8 00D9F0FF  CALL Unpacked.00402F50  
```



```
004F5650  |.  8D55 B0       LEA EDX,DWORD PTR SS:[EBP-50]  
004F5653  |.  8D45 A8       LEA EAX,DWORD PTR SS:[EBP-58]  
004F5656  |.  E8 25D9F0FF   CALL Unpacked.00402F80  
004F565B  |.  BA 7C584F00   MOV EDX,Unpacked.004F587C  
004F5660  |.  8D45 A8       LEA EAX,DWORD PTR SS:[EBP-58]  
004F5663  |.  B1 06         MOV CL,6  
004F5665  |.  E8 E6D8F0FF   CALL Unpacked.00402F50  
004F566A  |.  8D55 A8       LEA EDX,DWORD PTR SS:[EBP-58]  
004F566D  |.  8D45 D0       LEA EAX,DWORD PTR SS:[EBP-30]  
004F5670  |.  E8 27F5F0FF   CALL Unpacked.00404B9C              ;  以上用户名前三个字符"Rds"中依次插入字符'z'、'j'、'y'得到"Rzdjsy"  
004F5675  |.  8B55 D0       MOV EDX,DWORD PTR SS:[EBP-30]  
004F5678  |.  8D45 EC       LEA EAX,DWORD PTR SS:[EBP-14]  
004F567B  |.  59            POP ECX  
004F567C  |.  E8 C3F5F0FF   CALL Unpacked.00404C44              ;  再跟后面的"now"合并得到变形的用户名"Rzdjsynow"  
004F5681  |.  8B45 FC       MOV EAX,DWORD PTR SS:[EBP-4]  
004F5684  |.  E8 6FF5F0FF   CALL Unpacked.00404BF8              ;  取得原用户名"Rdsnow"的长度  
004F5689  |.  8BF8          MOV EDI,EAX  
004F568B  |.  57            PUSH EDI  
004F568C  |.  8D45 F0       LEA EAX,DWORD PTR SS:[EBP-10]  
004F568F  |.  B9 01000000   MOV ECX,1  
004F5694  |.  8B15 38554F00 MOV EDX,DWORD PTR DS:[4F5538]     ;  Unpacked.004F553C  
004F569A  |.  E8 890AF1FF   CALL Unpacked.00406128  
004F569F  |.  83C4 04       ADD ESP,4  
004F56A2  |.  8BF7          MOV ESI,EDI                         ;  以原用户名的长度作为循环次数  
004F56A4  |.  85F6          TEST ESI,ESI  
004F56A6  |.  7E 32         JLE SHORT Unpacked.004F56DA  
004F56A8  |.  BB 01000000   MOV EBX,1                           ;  i=1，准备循环  
004F56AD  |>  8D45 A0       LEA EAX,DWORD PTR SS:[EBP-60]  
004F56B0  |.  8B55 EC       |MOV EDX,DWORD PTR SS:[EBP-14]  
004F56B3  |.  8A541A FF     |MOV DL,BYTE PTR DS:[EDX+EBX-1]  
004F56B7  |.  E8 64F4F0FF   |CALL Unpacked.00404B20  
004F56BC  |.  8B45 A0       |MOV EAX,DWORD PTR SS:[EBP-60]  
004F56BF  |.  8D55 A4       |LEA EDX,DWORD PTR SS:[EBP-5C]  
004F56C2  |.  E8 B5F3FFFF   |CALL Unpacked.004F4A7C  
004F56C7  |.  8B55 A4       |MOV EDX,DWORD PTR SS:[EBP-5C]  
004F56CA  |.  8B45 F0       |MOV EAX,DWORD PTR SS:[EBP-10]  
004F56CD  |.  8D4498 FC     |LEA EAX,DWORD PTR DS:[EAX+EBX*4-4]  
004F56D1  |.  E8 BEF2F0FF   |CALL Unpacked.00404994  
004F56D6  |.  43            |INC EBX  
004F56D7  |.  4E            |DEC ESI  
004F56D8  |.^ 75 D3         \JNZ SHORT Unpacked.004F56AD        ;  以上循环把"Rzdjsynow"去掉后面三位的各个字符去查表，每个字符对应四位数字  
004F56DA  |>  8BF7          MOV ESI,EDI  
004F56DC  |.  4E            DEC ESI  
004F56DD  |.  85F6          TEST ESI,ESI  
004F56DF  |.  7C 15         JL SHORT Unpacked.004F56F6  
004F56E1  |.  46            INC ESI  
004F56E2  |.  33DB          XOR EBX,EBX  
004F56E4  |>  8D45 E4       LEA EAX,DWORD PTR SS:[EBP-1C]  
004F56E7  |.  8B55 F0       |MOV EDX,DWORD PTR SS:[EBP-10]  
004F56EA  |.  8B149A        |MOV EDX,DWORD PTR DS:[EDX+EBX*4]  
004F56ED  |.  E8 0EF5F0FF   |CALL Unpacked.00404C00  
004F56F2  |.  43            |INC EBX  
004F56F3  |.  4E            |DEC ESI  
004F56F4  |.^ 75 EE         \JNZ SHORT Unpacked.004F56E4        ;  以上循环把刚才查到的数字连接成一个字符串，得到"859953476317764394215669"  
004F56F6  |>  8D45 E0       LEA EAX,DWORD PTR SS:[EBP-20]  
004F56F9  |.  E8 42F2F0FF   CALL Unpacked.00404940  
004F56FE  |.  8B45 E4       MOV EAX,DWORD PTR SS:[EBP-1C]  
004F5701  |.  E8 F2F4F0FF   CALL Unpacked.00404BF8              ;  取字符串的长度  
004F5706  |.  8BF0          MOV ESI,EAX  
004F5708  |.  85F6          TEST ESI,ESI  
004F570A  |.  7E 3D         JLE SHORT Unpacked.004F5749  
004F570C  |.  BB 01000000   MOV EBX,1  
004F5711  |>  FF75 E0       PUSH DWORD PTR SS:[EBP-20]  
004F5714  |.  8D45 9C       |LEA EAX,DWORD PTR SS:[EBP-64]  
004F5717  |.  8B55 E4       |MOV EDX,DWORD PTR SS:[EBP-1C]  
004F571A  |.  8A541A FF     |MOV DL,BYTE PTR DS:[EDX+EBX-1]  
004F571E  |.  E8 FDF3F0FF   |CALL Unpacked.00404B20  
004F5723  |.  FF75 9C       |PUSH DWORD PTR SS:[EBP-64]  
004F5726  |.  8D45 98       |LEA EAX,DWORD PTR SS:[EBP-68]  
004F5729  |.  8B55 F8       |MOV EDX,DWORD PTR SS:[EBP-8]  
004F572C  |.  8A541A FF     |MOV DL,BYTE PTR DS:[EDX+EBX-1]  
004F5730  |.  E8 EBF3F0FF   |CALL Unpacked.00404B20  
004F5735  |.  FF75 98       |PUSH DWORD PTR SS:[EBP-68]  
004F5738  |.  8D45 E0       |LEA EAX,DWORD PTR SS:[EBP-20]  
004F573B  |.  BA 03000000   |MOV EDX,3  
004F5740  |.  E8 73F5F0FF   |CALL Unpacked.00404CB8  
004F5745  |.  43            |INC EBX  
004F5746  |.  4E            |DEC ESI  
004F5747  |.^ 75 C8         \JNZ SHORT Unpacked.004F5711        ;  以上循环把机器码的各个数字依次插入字符串，并且去掉后面多余部分得到"85589998533044706"不过这个后面也没有用到，这个循环这儿做什么用，偶也不太清楚  
004F5749  |>  8D55 DC       LEA EDX,DWORD PTR SS:[EBP-24]  
004F574C  |.  8B45 E4       MOV EAX,DWORD PTR SS:[EBP-1C]  
004F574F  |.  E8 10FEF4FF   CALL Unpacked.00445564              ;  将没有插入机器码前的字符串倒序  
004F5754  |.  8D45 D8       LEA EAX,DWORD PTR SS:[EBP-28]  
004F5757  |.  E8 E4F1F0FF   CALL Unpacked.00404940  
004F575C  |.  8B45 DC       MOV EAX,DWORD PTR SS:[EBP-24]  
004F575F  |.  E8 94F4F0FF   CALL Unpacked.00404BF8              ;  取得位数  
004F5764  |.  8BF0          MOV ESI,EAX  
004F5766  |.  85F6          TEST ESI,ESI  
004F5768  |.  7E 7E         JLE SHORT Unpacked.004F57E8  
004F576A  |.  BB 01000000   MOV EBX,1                           ;  i=1，准备循环  
004F576F  |>  8D45 94       LEA EAX,DWORD PTR SS:[EBP-6C]  
004F5772  |.  8B55 DC       |MOV EDX,DWORD PTR SS:[EBP-24]  
004F5775  |.  8A541A FF     |MOV DL,BYTE PTR DS:[EDX+EBX-1]  
004F5779  |.  E8 A2F3F0FF   |CALL Unpacked.00404B20  
004F577E  |.  8B45 94       |MOV EAX,DWORD PTR SS:[EBP-6C]  
004F5781  |.  E8 163DF1FF   |CALL Unpacked.0040949C             ;  取出各位数字 St[i-1]，转换为整数  
004F5786  |.  8BC8          |MOV ECX,EAX  
004F5788  |.  8BC1          |MOV EAX,ECX  
004F578A  |.  F7E9          |IMUL ECX  
004F578C  |.  F7E9          |IMUL ECX                             ;  St[i-1]的三次方  
004F578E  |.  8D51 14       |LEA EDX,DWORD PTR DS:[ECX+14]       ;  St[i-1]+20  
004F5791  |.  8BFA          |MOV EDI,EDX  
004F5793  |.  99            |CDQ  
004F5794  |.  F7FF          |IDIV EDI                             ;  相除,取得余数  
004F5796  |.  8BFA          |MOV EDI,EDX  
004F5798  |.  03F9          |ADD EDI,ECX                            ;  再加上 St[i-1]得到结果一  
004F579A  |.  8BC1          |MOV EAX,ECX  
004F579C  |.  F7E9          |IMUL ECX                             ;  St[i-1]的平方  
004F579E  |.  8D51 0A       |LEA EDX,DWORD PTR DS:[ECX+A]         ;  St[i-1]+10  
004F57A1  |.  51            |PUSH ECX  
004F57A2  |.  8BCA          |MOV ECX,EDX  
004F57A4  |.  99            |CDQ  
004F57A5  |.  F7F9          |IDIV ECX                             ;  相除,取得余数  
004F57A7  |.  59            |POP ECX  
004F57A8  |.  03FA          |ADD EDI,EDX                            ;  与结果一相加得到结果二  
004F57AA  |.  8D0449        |LEA EAX,DWORD PTR DS:[ECX+ECX*2]  
004F57AD  |.  03F8          |ADD EDI,EAX  
004F57AF  |.  03FB          |ADD EDI,EBX                            ;  再加上 St[i-1]*3+i  
004F57B1  |.  83FF 30       |CMP EDI,30  
004F57B4  |.  7C 05         |JL SHORT Unpacked.004F57BB  
004F57B6  |.  83FF 39       |CMP EDI,39  
004F57B9  |.  7E 14         |JLE SHORT Unpacked.004F57CF  
004F57BB  |>  83FF 41       |CMP EDI,41  
004F57BE  |.  7C 05         |JL SHORT Unpacked.004F57C5  
004F57C0  |.  83FF 5A       |CMP EDI,5A  
004F57C3  |.  7E 0A         |JLE SHORT Unpacked.004F57CF  
004F57C5  |>  83FF 60       |CMP EDI,60  
004F57C8  |.  7C 1A         |JL SHORT Unpacked.004F57E4  
004F57CA  |.  83FF 7F       |CMP EDI,7F  
004F57CD  |.  7F 15         |JG SHORT Unpacked.004F57E4           ;  以上是看看结果是不是数字或字母的 ASC，不是就抛弃  
004F57CF  |>  8D45 90       |LEA EAX,DWORD PTR SS:[EBP-70]  
004F57D2  |.  8BD7          |MOV EDX,EDI  
```



### 反汇编代码片段

```
004F57D4  |.  E8 47F3F0FF  |CALL Unpacked.00404B20  
004F57D9  |.  8B55 90      |MOV EDX,DWORD PTR SS:[EBP-70]  
004F57DC  |.  8D45 E8      |LEA EAX,DWORD PTR SS:[EBP-18]  
004F57DF  |.  E8 1CF4F0FF  |CALL Unpacked.00404C00             ;  是字母或数字就保留  
004F57E4  |>  43           |INC EBX  
004F57E5  |.  4E           |DEC ESI  
004F57E6  |.^ 75 87        \\JNZ SHORT Unpacked.004F576F  
004F57E8  |>  8B45 E8      |MOV EAX,DWORD PTR SS:[EBP-18]      ;  经过以上循环得到"5JK4O42BC5J"  
004F57EB  |.  E8 08F4F0FF  |CALL Unpacked.00404BF8  
004F57F0  |.  83F8 10      |CMP EAX,10                         ;  检查结果有没有 16 位  
004F57F3  |.  7D 0D        |JGE SHORT Unpacked.004F5802  
004F57F5  |.  8D45 E8      |LEA EAX,DWORD PTR SS:[EBP-18]  
004F57F8  |.  BA 88584F00  |MOV EDX,Unpacked.004F5888          ;  ASCII "ZFJSYZGNYEJDSUFOHYOJUZDZ"  
004F57FD  |.  E8 FEF3F0FF  |CALL Unpacked.00404C00             ;  没有就跟以上字符串相连  
004F5802  |>  8D45 D8      |LEA EAX,DWORD PTR SS:[EBP-28]  
004F5805  |.  50           |PUSH EAX  
004F5806  |.  B9 0F000000  |MOV ECX,0F  
004F580B  |.  BA 02000000  |MOV EDX,2  
004F5810  |.  8B45 E8      |MOV EAX,DWORD PTR SS:[EBP-18]  
004F5813  |.  E8 38F6F0FF  |CALL Unpacked.00404E50             ;  从第二位开始取 15 的字符，得到"JK4O42BC5JZFJSY",加上'-'就是注册码了  
004F5818  |.  8B45 F4      |MOV EAX,DWORD PTR SS:[EBP-C]  
004F581B  |.  8B55 D8      |MOV EDX,DWORD PTR SS:[EBP-28]  
004F581E  |.  E8 71F1F0FF  |CALL Unpacked.00404994  
004F5823  |.  33C0         |XOR EAX,EAX  
004F5825  |.  5A           |POP EDX  
004F5826  |.  59           |POP ECX  
004F5827  |.  59           |POP ECX  
004F5828  |.  64:8910      |MOV DWORD PTR FS:[EAX],EDX  
004F582B  |.  68 6D584F00  |PUSH Unpacked.004F586D  
004F5830  |>  8D45 90      |LEA EAX,DWORD PTR SS:[EBP-70]  
004F5833  |.  BA 06000000  |MOV EDX,6  
004F5838  |.  E8 27F1F0FF  |CALL Unpacked.00404964  
004F583D  |.  8D45 D0      |LEA EAX,DWORD PTR SS:[EBP-30]  
004F5840  |.  BA 08000000  |MOV EDX,8  
004F5845  |.  E8 1AF1F0FF  |CALL Unpacked.00404964  
004F584A  |.  8D45 F0      |LEA EAX,DWORD PTR SS:[EBP-10]  
004F584D  |.  8B15 38554F00|MOV EDX,DWORD PTR DS:[4F5538]     ;  Unpacked.004F553C  
004F5853  |.  E8 F009F1FF  |CALL Unpacked.00406248  
004F5858  |.  8D45 F8      |LEA EAX,DWORD PTR SS:[EBP-8]  
004F585B  |.  BA 02000000  |MOV EDX,2  
004F5860  |.  E8 FFF0F0FF  |CALL Unpacked.00404964  
004F5865  |\.  C3          |RETN  
004F5866  .^  E9 1DEAF0FF |JMP Unpacked.00404288  
004F586B  .^  EB C3        |JMP SHORT Unpacked.004F5830  
004F586D  .  5F           |POP EDI  
004F586E  .  5E           |POP ESI  
004F586F  .  5B           |POP EBX  
004F5870  .  8BE5         |MOV ESP,EBP  
004F5872  .  5D           |POP EBP  
004F5873  .  C3           |RETN  
```

### 破解心得

注册代码写得再好，计算在复杂，如果采用了明码比较，等于把注册码送给别人；另外既然注册信息码已经插入字符串，为什么不用，让人觉得很费解(看上面代码的红字处)；

注册码是这样生成的：用户名长度大于 `5`，小于 `13`

`"Rdsnow"` 插入字符得到 `"Rzdjsynow"`

然后去掉最后三位查表，对照表如下：好像都是一些质数

`1-1697` `2-1699` `3-1709` `4-1721` `5-1723` `6-1733` `7-1741` `8-1747` `9-1759` `0-1777` `Aa-6299` `Bb-6301` `Cc-6311` `Dd-6317` `Ee-6323` `Ff-6329` `Gg-7591` `Hh-7621` `Ii-7639` `Jj-7643` `Kk-7649` `Ll-7669` `Mm-8543` `Nn-8563` `Oo-8573` `Pp-8581` `Qq-8597` `Rr-8599` `Ss-9421` `Tt-9431` `Uu-9437` `Vv-9439` `Ww-9461` `Xx-9463` `Yy-5669` `Zz-5347`

查表得到 `"859953476317764394215669"`

对个数字采用公式 `St[i]³mod(St[i]+20) + St[i]²mod(St[i]+10) + St[i]*4 + i + 1`

(重新写的公式，`i` 初值为 `0`)得到 ASC 留下字母数字。

最后接上`"ZFJSYZGNYEJDSUFOHYOJUZDZ"`从第二位开始取十五个字符，每五个插入`'-'`，就是注册码了。

注册码分三段保存在注册表中：

第一段存在 `HKCU\Control Panel\Keyboard\CAI_m1`

第二段：`HKCU\SOFTWARE\Microsoft\Shared\HTML\Old Default Editor\shell\CAI_m2`

第三段：`HKLM\SYSTEM\ControlSet001\Hardware Profiles\0001\SOFTWARE\Microsoft\WAB\Export\PAB\CAI_m3`

删除其中的任何一段程序就改为未注册版了

### 注册机源码

编程偶不会，不知道怎样提高代码的效率，希望高手能指点一二

```cpp
void CKEYDlg::OnOK()  
{  
  // TODO: Add extra validation here  
  //CDialog::OnOK();  
  //变量定义  
  char cTable[250]={"1777-1697-1699-1709-1721-1723-1733-1741-1747-1759-0000-0000-0000-0000-0000-0000-0000-6299-6301-6311-6317-6323-6329-7591-7621-7639-7643-7649-7669-8543-8563-8573-8581-8597-8599-9421-9431-9437-9439-9461-9463-5669-5347"};  
  int i,j,n;  
  CString sUsename;  
  char cUsename[16],cNumber[80],cSn;  
  UpdateData(true);  
  m_Edit2="";  
  sUsename=m_Edit1;  
  sUsename.MakeUpper ();//注册码跟大小写无关，转为大写是为了便于查表  
  //检查用户名的长度  
  n=sUsename.GetLength ();  
  if (n<6 || n>12) {  
    MessageBox("用户名的长度必须在 6 到 12 之间","提示",MB_OK);  
    return;  
  }  
  //插入字符，检查用户名是否全失字母数字  
  sUsename.Insert (1,'Z');  
  sUsename.Insert (3,'J');  
  sUsename.Insert (5,'Y');  
  strcpy(cUsename,sUsename);  
  for (i=0;i<n+3;i++){  
    if(cUsename[i]<0x30||(cUsename[i]>0x39 && cUsename[i]<0x41)||cUsename[i]>0x5A){  
      MessageBox("用户名只能由 A-Z，a-z，0-9 组成，请不要使用其它字符","提示",MB_OK);  
      return;  
    }  
  }  
  //查表  
  for (i=0;i<n;i++){  
     j=cUsename[i]-0x30;  
    cNumber[i*4]=cTable[j*5];  
    cNumber[i*4+1]=cTable[5*j+1];  
    cNumber[i*4+2]=cTable[5*j+2];  
    cNumber[i*4+3]=cTable[5*j+3];  
    cNumber[i*4+4]=0;  
  }  
  //运用公式将数字逆序运算，取出有用的字符  
  for (i=4*n;i>0;i--){  
    j=cNumber[i-1]-0x30;  
    j=(j*j*j)%(j+20)+(j*j)%(j+10)+j*4+4*n-i+1;  
    if((j>0x2F && j<0x3A)||(j>0x40 && j<0x5B)||(j>0x60 && j<0x7B)){  
      cSn=j;  
      m_Edit2 += cSn;  
    }  
  }  
  //最终形成注册码  
  m_Edit2 += "ZFJSYZGNYEJDSUFOHYOJUZDZ";  
  m_Edit2=m_Edit2.Mid (1,15);  
  m_Edit2.Insert (5,'-');  
  m_Edit2.Insert (11,'-');  
  UpdateData(false);  
}  
```

注册机就不贴上来了。

### 声明

**【破解声明】** 我是一只小菜鸟，偶得一点心得，愿与大家分享:)

**【版权声明】** 本文纯属技术交流, 转载请注明作者并保持文章的完整, 谢谢!

文章写于 2005-5-2 22:56:20

`4dd466d0e1e0cc123afbe504da2462ca`
