
# PGWARE PCBoost 3.8.7.2006

> 原文：[`www.pediy.com/kssd/pediy08/pediy8-277.htm`](https://www.pediy.com/kssd/pediy08/pediy8-277.htm)

【破解文件】`PGWARE PCBoost 3.8.7.2006`

【下载地址】`http://www.newhua.com/soft/2295.htm`

【软件类别】国外软件/共享版/系统设置

【运行环境】`Win9x/Me/NT/2000/XP/2003`

【保护方式】`PECompact 2.x` + `Name` + `Serial`

【作者声明】只是感兴趣，消遣业余时间，错误之处敬请诸位前辈不吝赐教

【使用工具】`OllyDBD`、`PEiD`

【软件信息】帮您完成系统自动加速、私人档案保、手动个别设定调整，包括桌面、开始功能表、网路、硬　　　　　　体、光碟机、记忆体、关机的优化。

## 一、破解过程

`PEiD` 检查:`PECompact 2.x` -> `Jeremy Collake`

利用 `ESP` 定律可以顺利脱壳

脱壳后 `PEiD` 再查:`Borland Delphi 6.0 - 7.0`

超级字串参考找到：文本字串=`\\software\\pgware\\pcboost` 再找对该子程序的调用，

在 `0048D497` 处下断，`F9` 一次异常，`Shift+F9` 。

填入 `Name:wzwgp` `Serial:12345678`

```
0048D497   .  51            PUSH ECX                     ;  断下  
0048D498   .  53            PUSH EBX  
0048D499   .  56            PUSH ESI  
0048D49A   .  57            PUSH EDI  
0048D49B   .  8BD8          MOV EBX,EAX  
0048D49D   .  33C0          XOR EAX,EAX  
0048D49F   .  55            PUSH EBP  
0048D4A0   .  68 ADD64800   PUSH tk.0048D6AD  
0048D4A5   .  64:FF30      PUSH DWORD PTR FS:[EAX]  
0048D4A8   .  64:8920      MOV DWORD PTR FS:[EAX],ESP  
0048D4AB   .  8D55 FC      LEA EDX,DWORD PTR SS:[EBP-4]  
0048D4AE   .  8B83 38030000 MOV EAX,DWORD PTR DS:[EBX+338]  
0048D4B4   .  E8 43A1FDFF  CALL tk.004675FC             ;  得到用户名位数  
0048D4B9   .  837D FC 00    CMP DWORD PTR SS:[EBP-4],0    ;  [EBP-4]=用户名  
0048D4BD   .  0F84 98010000 JE tk.0048D65B  
0048D4C3   .  8D55 F8      LEA EDX,DWORD PTR SS:[EBP-8]  
0048D4C6   .  8B83 34030000 MOV EAX,DWORD PTR DS:[EBX+334]  
0048D4CC   .  E8 2BA1FDFF  CALL tk.004675FC  
0048D4D1   .  837D F8 00    CMP DWORD PTR SS:[EBP-8],0    ;  [EBP-8]=假码  
0048D4D5   .  0F84 80010000 JE tk.0048D65B  
0048D4DB   .  8D55 F0      LEA EDX,DWORD PTR SS:[EBP-10]  
0048D4DE   .  8B83 34030000 MOV EAX,DWORD PTR DS:[EBX+334]  
0048D4E4   .  E8 13A1FDFF  CALL tk.004675FC  
0048D4E9   .  8B45 F0      MOV EAX,DWORD PTR SS:[EBP-10]  
0048D4EC   .  8D55 F4      LEA EDX,DWORD PTR SS:[EBP-C]  
0048D4EF   .  E8 78AFF7FF  CALL tk.0040846C  
0048D4F4   .  8B45 F4      MOV EAX,DWORD PTR SS:[EBP-C]  
0048D4F7   .  50            PUSH EAX  
0048D4F8   .  8D55 E8      LEA EDX,DWORD PTR SS:[EBP-18]  
0048D4FB   .  8B83 38030000 MOV EAX,DWORD PTR DS:[EBX+338]  
0048D501   .  E8 F6A0FDFF  CALL tk.004675FC  
0048D506   .  8B45 E8      MOV EAX,DWORD PTR SS:[EBP-18]  
0048D509   .  8D55 EC      LEA EDX,DWORD PTR SS:[EBP-14]  
0048D50C   .  E8 5BAFF7FF  CALL tk.0040846C  
0048D511   .  8B45 EC      MOV EAX,DWORD PTR SS:[EBP-14]  
0048D514   .  5A            POP EDX  
0048D515   .  E8 C62D0000  CALL tk.004902E0             ;  验证注册码 F7 进入  
0048D51A   .  3C 01         CMP AL,1                     ;  AL=1 通过验证  
0048D51C     0F85 03010000 JNZ tk.0048D625              ;  跳验证失败  
0048D522   .  A1 F4724900  MOV EAX,DWORD PTR DS:[4972F4]  
0048D527   .  8B00          MOV EAX,DWORD PTR DS:[EAX]  
0048D529   .  8B80 38030000 MOV EAX,DWORD PTR DS:[EAX+338]  
0048D52F   .  33D2          XOR EDX,EDX  
0048D531   .  E8 E69FFDFF  CALL tk.0046751C  
0048D536   .  A1 F4724900  MOV EAX,DWORD PTR DS:[4972F4]  
0048D53B   .  8B00          MOV EAX,DWORD PTR DS:[EAX]  
0048D53D   .  8B80 2C030000 MOV EAX,DWORD PTR DS:[EAX+32C]  
0048D543   .  33D2          XOR EDX,EDX  
0048D545   .  E8 D29FFDFF  CALL tk.0046751C  
0048D54A   .  A1 A8714900  MOV EAX,DWORD PTR DS:[4971A8]  
0048D54F   .  8B00          MOV EAX,DWORD PTR DS:[EAX]  
0048D551   .  B2 06         MOV DL,6  
0048D553   .  E8 7042FFFF  CALL tk.004817C8  
0048D558   .  A1 A8714900  MOV EAX,DWORD PTR DS:[4971A8]  
0048D55D   .  8B00          MOV EAX,DWORD PTR DS:[EAX]  
0048D55F   .  8B10          MOV EDX,DWORD PTR DS:[EAX]  
0048D561   .  FF92 EC000000 CALL NEAR DWORD PTR DS:[EDX+EC]  ;  注册成功提示  
0048D567   .  48            DEC EAX  
0048D568   .  75 0A         JNZ SHORT tk.0048D574  
```

`0048D515` 处 `F7` 进入验证注册码

```
004902E0   $  55            PUSH EBP  
004902E1   .  8BEC          MOV EBP,ESP  
004902E3   .  B9 12000000  MOV ECX,12  
004902E8   >  6A 00        PUSH 0  
004902EA   .  6A 00        PUSH 0  
004902EC   .  49            DEC ECX  
004902ED   .^ 75 F9        JNZ SHORT tk.004902E8        ;  堆栈留出空间  
004902EF   .  51            PUSH ECX  
004902F0   .  53            PUSH EBX  
004902F1   .  56            PUSH ESI  
004902F2   .  57            PUSH EDI  
004902F3   .  8955 F8      MOV DWORD PTR SS:[EBP-8],EDX  
004902F6   .  8945 FC      MOV DWORD PTR SS:[EBP-4],EAX  
004902F9   .  8B45 FC      MOV EAX,DWORD PTR SS:[EBP-4]  
004902FC   .  E8 5B42F7FF  CALL tk.0040455C  
00490301   .  8B45 F8      MOV EAX,DWORD PTR SS:[EBP-8]  
00490304   .  E8 5342F7FF  CALL tk.0040455C  
00490309   .  33C0          XOR EAX,EAX  
0049030B   .  55            PUSH EBP  
0049030C   .  68 82094900  PUSH tk.00490982  
00490311   .  64:FF30      PUSH DWORD PTR FS:[EAX]  
00490314   .  64:8920      MOV DWORD PTR FS:[EAX],ESP  
00490317   .  33C0          XOR EAX,EAX  
00490319   .  55            PUSH EBP  
0049031A   .  68 33094900  PUSH tk.00490933  
0049031F   .  64:FF30      PUSH DWORD PTR FS:[EAX]  
00490322   .  64:8920      MOV DWORD PTR FS:[EAX],ESP  
00490325   .  837D FC 00    CMP DWORD PTR SS:[EBP-4],0     ;  注册框是否输入用户名  
00490329   .  74 73         JE SHORT tk.0049039E          ;  未输入跳到下面读注册表信息  
0049032B   .  837D F8 00    CMP DWORD PTR SS:[EBP-8],0     ;  [EBP-8]假码地址  
0049032F   .  74 6D         JE SHORT tk.0049039E          ;  未输入假码就跳走  
00490331   .  33C0          XOR EAX,EAX  
00490333   .  55            PUSH EBP  
00490334   .  68 92034900  PUSH tk.00490392  
00490339   .  64:FF30      PUSH DWORD PTR FS:[EAX]  
0049033C   .  64:8920      MOV DWORD PTR FS:[EAX],ESP  
0049033F   .  B2 01         MOV DL,1  
00490341   .  A1 F48E4300  MOV EAX,DWORD PTR DS:[438EF4]     
00490346   .  E8 A98CFAFF  CALL tk.00438FF4  
0049034B   .  8BD8          MOV EBX,EAX  
0049034D   .  BA 02000080  MOV EDX,80000002  
00490352   .  8BC3          MOV EAX,EBX  
00490354   .  E8 3B8DFAFF  CALL tk.00439094  
00490359   .  B1 01         MOV CL,1  
0049035B   .  BA 9C094900  MOV EDX,tk.0049099C            ;  注册信息地址\\software\\pgware\\pcboost  
00490360   .  8BC3          MOV EAX,EBX  
00490362   .  E8 918DFAFF  CALL tk.004390F8  
00490367   .  8B4D FC      MOV ECX,DWORD PTR SS:[EBP-4]   ;  [EBP-4]用户名  
0049036A   .  BA C0094900  MOV EDX,tk.004909C0            ;  name  
0049036F   .  8BC3          MOV EAX,EBX  
00490371   .  E8 3E8FFAFF  CALL tk.004392B4               ;  用户名写入注册表  
00490376   .  8B4D F8      MOV ECX,DWORD PTR SS:[EBP-8]   ;  [EBP-8]假码  
00490379   .  BA D0094900  MOV EDX,tk.004909D0            ;  serial  
0049037E   .  8BC3          MOV EAX,EBX  
00490380   .  E8 2F8FFAFF  CALL tk.004392B4               ;  假码写入注册表  
00490385   .  33C0          XOR EAX,EAX  
00490387   .  5A            POP EDX  
00490388   .  59            POP ECX  
00490389   .  59            POP ECX  
0049038A   .  64:8910      MOV DWORD PTR FS:[EAX],EDX  
0049038D   .  E9 89000000  JMP tk.0049041B                ;  注册框输入注册码就跳过读注册表  
```



```
00490392   .^ E9 C133F7FF        JMP tk.00403758  

00490397   .  E8 2437F7FF        CALL tk.00403AC0  

0049039C   .  EB 7D              JMP SHORT tk.0049041B  

0049039E   >  33C0              XOR EAX,EAX  

004903A0   .  55                PUSH EBP  

004903A1   .  68 11044900       PUSH tk.00490411  

004903A6   .  64:FF30           PUSH DWORD PTR FS:[EAX]  

004903A9   .  64:8920           MOV DWORD PTR FS:[EAX],ESP  

004903AC   .  B2 01             MOV DL,1  

004903AE   .  A1 F48E4300       MOV EAX,DWORD PTR DS:[438EF4]  

004903B3   .  E8 3C8CFAFF       CALL tk.00438FF4  

004903B8   .  8BD8              MOV EBX,EAX  

004903BA   .  BA 02000080       MOV EDX,80000002  

004903BF   .  8BC3              MOV EAX,EBX  

004903C1   .  E8 CE8CFAFF       CALL tk.00439094  

004903C6   .  C743 18 19000200  MOV DWORD PTR DS:[EBX+18],20019  

004903CD   .  33C9              XOR ECX,ECX  

004903CF   .  BA 9C094900       MOV EDX,tk.0049099C            ;  \\software\\pgware\\pcboost  

004903D4   .  8BC3              MOV EAX,EBX  

004903D6   .  E8 1D8DFAFF       CALL tk.004390F8  

004903DB   .  8D4D FC           LEA ECX,DWORD PTR SS:[EBP-4]  

004903DE   .  BA C0094900       MOV EDX,tk.004909C0            ;  name  

004903E3   .  8BC3              MOV EAX,EBX  

004903E5   .  E8 F68EFAFF       CALL tk.004392E0               ;  用户名  

004903EA   .  8D4D F8           LEA ECX,DWORD PTR SS:[EBP-8]  

004903ED   .  BA D0094900       MOV EDX,tk.004909D0            ;  serial  

004903F2   .  8BC3              MOV EAX,EBX  

004903F4   .  E8 E78EFAFF       CALL tk.004392E0               ;  注册码  

004903F9   .  8BC3              MOV EAX,EBX  

004903FB   .  E8 648CFAFF       CALL tk.00439064  

00490400   .  8BC3              MOV EAX,EBX  

00490402   .  E8 B12EF7FF       CALL tk.004032B8  

00490407   .  33C0              XOR EAX,EAX  

00490409   .  5A                POP EDX  

0049040A   .  59                POP ECX  

0049040B   .  59                POP ECX  

0049040C   .  64:8910           MOV DWORD PTR FS:[EAX],EDX  

0049040F   .  EB 0A             JMP SHORT tk.0049041B  

00490411   .^ E9 4233F7FF       JMP tk.00403758  

00490416   .  E8 A536F7FF       CALL tk.00403AC0  

0049041B   >  33C0              XOR EAX,EAX  

0049041D   .  55                PUSH EBP  

0049041E   .  68 1B094900       PUSH tk.0049091B  

00490423   .  64:FF30           PUSH DWORD PTR FS:[EAX]  

00490426   .  64:8920           MOV DWORD PTR FS:[EAX],ESP  

00490429   .  837D FC 00        CMP DWORD PTR SS:[EBP-4],0      ;  [EBP-4]=用户名  

0049042D   .  0F84 DE040000     JE tk.00490911  

00490433   .  837D F8 00        CMP DWORD PTR SS:[EBP-8],0      ;  [EBP-8]=注册码  

00490437   .  0F84 D4040000     JE tk.00490911  

0049043D   .  B8 FC9B4900       MOV EAX,tk.00499BFC  

00490442   .  8B55 FC           MOV EDX,DWORD PTR SS:[EBP-4]  

00490445   .  E8 B63CF7FF       CALL tk.00404100  

0049044A   .  68 009C4900       PUSH tk.00499C00  

0049044F   .  8D45 D8           LEA EAX,DWORD PTR SS:[EBP-28]  

00490452   .  50                PUSH EAX  

00490453   .  8B55 F8           MOV EDX,DWORD PTR SS:[EBP-8]  

00490456   .  B8 E0094900       MOV EAX,tk.004909E0             ;  +  

0049045B   .  E8 5042F7FF       CALL tk.004046B0                ;  检查注册码中是否有+(2B)  

00490460   .  40                INC EAX                           重新输入假码 S(s1+s2+s3)  

00490461   .  50                PUSH EAX                          (12345678+123+234)  

00490462   .  8B45 F8           MOV EAX,DWORD PTR SS:[EBP-8]  

00490465   .  E8 023FF7FF       CALL tk.0040436C                ;  得到假码位数  

0049046A   .  8BC8              MOV ECX,EAX  

0049046C   .  8B45 F8           MOV EAX,DWORD PTR SS:[EBP-8]  

0049046F   .  5A                POP EDX  

00490470   .  E8 5741F7FF       CALL tk.004045CC  

00490475   .  8B55 D8           MOV EDX,DWORD PTR SS:[EBP-28]   ;  [EBP-28]=s2+s3  

00490478   .  B8 E0094900       MOV EAX,tk.004909E0             ;  +  

0049047D   .  E8 2E42F7FF       CALL tk.004046B0                ;  检查第一个+号后的假码中是否还有+号  

00490482   .  48                DEC EAX  

00490483   .  50                PUSH EAX  

00490484   .  8D45 D4           LEA EAX,DWORD PTR SS:[EBP-2C]  

00490487   .  50                PUSH EAX  

00490488   .  8B55 F8           MOV EDX,DWORD PTR SS:[EBP-8]  

0049048B   .  B8 E0094900       MOV EAX,tk.004909E0             ;  +  

00490490   .  E8 1B42F7FF       CALL tk.004046B0                

00490495   .  40                INC EAX  

00490496   .  50                PUSH EAX  

00490497   .  8B45 F8           MOV EAX,DWORD PTR SS:[EBP-8]  

0049049A   .  E8 CD3EF7FF       CALL tk.0040436C  

0049049F   .  8BC8              MOV ECX,EAX  

004904A1   .  8B45 F8           MOV EAX,DWORD PTR SS:[EBP-8]  

004904A4   .  5A                POP EDX  

004904A5   .  E8 2241F7FF       CALL tk.004045CC  

004904AA   .  8B45 D4           MOV EAX,DWORD PTR SS:[EBP-2C]  

004904AD   .  BA 01000000       MOV EDX,1  

004904B2   .  59                POP ECX  

004904B3   .  E8 1441F7FF       CALL tk.004045CC  

004904B8   .  BB 01000000       MOV EBX,1                         ;  EBX 赋值 1  

004904BD   >  8D45 CC           LEA EAX,DWORD PTR SS:[EBP-34]  

004904C0   .  50                PUSH EAX  

004904C1   .  B9 01000000       MOV ECX,1  

004904C6   .  8BD3              MOV EDX,EBX  

004904C8   .  8B45 FC           MOV EAX,DWORD PTR SS:[EBP-4]     ;  设用户名为 N(n1、n2、…ni)  

004904CB   .  E8 FC40F7FF       CALL tk.004045CC                ;  逐位取出用户名 ni  

004904D0   .  8B45 CC           MOV EAX,DWORD PTR SS:[EBP-34]   ;  [EBP-34]ni 地址  

004904D3   .  0FB600            MOVZX EAX,BYTE PTR DS:[EAX]      ;  用户名 16 进制数入 EAX  

004904D6   .  F7EB              IMUL EBX                          ;  EBX=i  

004904D8   .  8945 C8           MOV DWORD PTR SS:[EBP-38],EAX  

004904DB   .  DB45 C8           FILD DWORD PTR SS:[EBP-38]        ;  ni*i 转成浮点数入 st(0)  

004904DE   .  E8 D925F7FF       CALL tk.00402ABC                ;  ni 入栈  

004904E3   .  8945 C0           MOV DWORD PTR SS:[EBP-40],EAX  

004904E6   .  8955 C4           MOV DWORD PTR SS:[EBP-3C],EDX  

004904E9   .  DF6D C0           FILD QWORD PTR SS:[EBP-40]       ;  ni*i 装入 st(0)  

004904EC   .  83C4 F4           ADD ESP,-0C  

004904EF   .  DB3C24            FSTP TBYTE PTR SS:[ESP]          ; |保存 ni 到[ESP]  

004904F2   .  9B                WAIT                              ; |  

004904F3   .  8D55 D0           LEA EDX,DWORD PTR SS:[EBP-30]    ; |  

004904F6   .  B8 EC094900       MOV EAX,tk.004909EC             ; |#  

004904FB   .  E8 3CA3F7FF       CALL tk.0040A83C                ; \\ni*i 转成 10 进制  

00490500   .  FF75 D0           PUSH DWORD PTR SS:[EBP-30]  

00490503   .  8D55 BC           LEA EDX,DWORD PTR SS:[EBP-44]  

00490506   .  8BC3              MOV EAX,EBX  

00490508   .  E8 2F83F7FF       CALL tk.0040883C  

0049050D   .  FF75 BC           PUSH DWORD PTR SS:[EBP-44]  

00490510   .  FF35 009C4900     PUSH DWORD PTR DS:[499C00]      ;  [499C00]=s2  

00490516   .  8D45 E4           LEA EAX,DWORD PTR SS:[EBP-1C]  

00490519   .  BA 03000000       MOV EDX,3  

0049051E   .  E8 093FF7FF       CALL tk.0040442C                ;  ni*i+i+s2(连接)  

00490523   .  8B45 E4           MOV EAX,DWORD PTR SS:[EBP-1C]   ;  [EBP-1C]=ni*i+i+s2  

00490526   .  E8 4D84F7FF       CALL tk.00408978                ;  转成 16 进制  

0049052B   .  8BF0              MOV ESI,EAX                     ;  EAX=122CD3  

0049052D   .  8B45 E4           MOV EAX,DWORD PTR SS:[EBP-1C]  

00490530   .  E8 4384F7FF       CALL tk.00408978  

00490535   .  03F0              ADD ESI,EAX                      ;  ESI=(ni+i+s2)相加,即乘 2  

00490537   .  8BC6              MOV EAX,ESI
```



## 00490539

`00490539   .   8D55 B8             LEA EDX,DWORD PTR SS:[EBP-48]`

`0049053C   .   E8 FB82F7FF        CALL tk.0040883C               ;  转成 10 进制`

`00490541   .   8B55 B8            MOV EDX,DWORD PTR SS:[EBP-48]   ;  [EBP-48]=(ni+i+s2)*2 (D)`

`00490544   .   8D45 E4            LEA EAX,DWORD PTR SS:[EBP-1C]   ;  [EBP-1C]=(ni*i+i+s2) (D)`

`00490547   .   E8 F83BF7FF        CALL tk.00404144`

`0049054C   .   43                 INC EBX                          ;  EBX + 1`

`0049054D   .   8B45 FC            MOV EAX,DWORD PTR SS:[EBP-4]     ;  [EBP-4]=wzwgp(用户名)`

`00490550   .   E8 173EF7FF        CALL tk.0040436C`

`00490555   .   40                 INC EAX                          ;  EAX=用户名位数`

`00490556   .   3BD8               CMP EBX,EAX`

`00490558   .^ 0F85 5FFFFFFF       JNZ tk.004904BD                  ;  用户名位数是循环次数`

`0049055E   .   6A 04              PUSH 4`

`00490560   .   68 FD4E5A09        PUSH 95A4EFD`

`00490565   .   8B45 E4            MOV EAX,DWORD PTR SS:[EBP-1C]   ;  [EBP-1C]=11210246(末位用户名计算结果)`

`00490568   .   E8 0B84F7FF        CALL tk.00408978               ;  EAX=AB0E06(转成 16 进制)`

`0049056D   .   99                 CDQ`

`0049056E   .   E8 614AF7FF        CALL tk.00404FD4               ;  运算用户名计算结果 F7`

F7 进入运算用户名计算结果：

`00404FD4  /$  52                 PUSH EDX`

`00404FD5  |.  50                 PUSH EAX`

`00404FD6  |.  8B4424 10           MOV EAX,DWORD PTR SS:[ESP+10]   ;  [ESP+10]=4(常数)`

`00404FDA  |.  F72424             MUL DWORD PTR SS:[ESP]           ;  [ESP]=AB0E06`

`00404FDD  |.  89C1               MOV ECX,EAX                      ;  EAX=AB0E06*4=2AC3818`

`00404FDF  |.  8B4424 04          MOV EAX,DWORD PTR SS:[ESP+4]     ;  [ESP+4]=0`

`00404FE3  |.  F76424 0C          MUL DWORD PTR SS:[ESP+C]         ;  [ESP+C]=095A4EFD(常数)`

`00404FE7  |.  01C1               ADD ECX,EAX`

`00404FE9  |.  8B0424             MOV EAX,DWORD PTR SS:[ESP]       ;  [ESP]=00AB0E06`

`00404FEC  |.  F76424 0C          MUL DWORD PTR SS:[ESP+C]         ;  EAX=AB0E06*95A4EFD=EB6EAFEE`

`00404FF0  |.  01CA               ADD EDX,ECX                      ;  EDX=63FD5(溢出部分)+2AC3818=2B277ED`

`00404FF2  |.  59                 POP ECX`

`00404FF3  |.  59                 POP ECX`

`00404FF4  \.  C2 0800            RETN 8                           ;  返回到 00490573`

`00490573   .   8945 C0            MOV DWORD PTR SS:[EBP-40],EAX   ;  EAX=EB6EAFEE`

`00490576   .   8955 C4            MOV DWORD PTR SS:[EBP-3C],EDX   ;  EDX=02B277ED`

`00490579   .   DF6D C0            FILD QWORD PTR SS:[EBP-40]      ;  [EBP-40]=2B277EDEB6EAFEE`

`0049057C   .   83C4 F4            ADD ESP,-0C`

`0049057F   .   DB3C24             FSTP TBYTE PTR SS:[ESP]         ;  |ST=1.9434959767120689400e+17`

`00490582   .   9B                 WAIT                             ;  |`

`00490583   .   8D55 B4            LEA EDX,DWORD PTR SS:[EBP-4C]    ;  |`

`00490586   .   B8 EC094900        MOV EAX,tk.004909EC             ;  #`

`0049058B   .   E8 ACA2F7FF        CALL tk.0040A83C                 ;  \tk.0040A83C`

`00490590   .   8B55 B4            MOV EDX,DWORD PTR SS:[EBP-4C]    ;  [EBP-4C]计算结果`

`00490593   .   8D45 E4            LEA EAX,DWORD PTR SS:[EBP-1C]`

`00490596   .   E8 A93BF7FF        CALL tk.00404144                 ;  EDX=11210246`

`0049059B   .   8D45 F0            LEA EAX,DWORD PTR SS:[EBP-10]`

`0049059E   .   50                 PUSH EAX`

`0049059F   .   8B55 F8            MOV EDX,DWORD PTR SS:[EBP-8]`

`004905A2   .   B8 E0094900        MOV EAX,tk.004909E0              ;  +`

`004905A7   .   E8 0441F7FF        CALL tk.004046B0`

`004905AC   .   8BC8               MOV ECX,EAX`

`004905AE   .   49                 DEC ECX`

`004905AF   .   BA 01000000        MOV EDX,1`

`004905B4   .   8B45 F8            MOV EAX,DWORD PTR SS:[EBP-8]`

`004905B7   .   E8 1040F7FF        CALL tk.004045CC                 ;  取出 s1`

`004905BC   .   8D45 B0            LEA EAX,DWORD PTR SS:[EBP-50]`

`004905BF   .   50                 PUSH EAX`

`004905C0   .   8B45 F0            MOV EAX,DWORD PTR SS:[EBP-10]`

`004905C3   .   E8 A43DF7FF        CALL tk.0040436C                 ;  s1 位数`

`004905C8   .   8BD8               MOV EBX,EAX`

`004905CA   .   A1 009C4900        MOV EAX,DWORD PTR DS:[499C00]    ;  [499C00]=s2`

`004905CF   .   E8 983DF7FF        CALL tk.0040436C                 ;  第二段假码位数`

`004905D4   .   03D8               ADD EBX,EAX                      ;  s1、s2 位数相加`

`004905D6   .   83C3 03            ADD EBX,3                        ;  再加 3`

`004905D9   .   53                 PUSH EBX`

`004905DA   .   8B45 F8            MOV EAX,DWORD PTR SS:[EBP-8]`

`004905DD   .   E8 8A3DF7FF        CALL tk.0040436C                 ;  假码位数`

`004905E2   .   8BC8               MOV ECX,EAX`

`004905E4   .   8B45 F8            MOV EAX,DWORD PTR SS:[EBP-8]`

`004905E7   .   5A                 POP EDX`

`004905E8   .   E8 DF3FF7FF        CALL tk.004045CC                 ;  s3 地址`

`004905ED   .   8B45 B0            MOV EAX,DWORD PTR SS:[EBP-50]    ;  [EBP-50]=s3`

`004905F0   .   8D55 EC            LEA EDX,DWORD PTR SS:[EBP-14]`

`004905F3   .   E8 747EF7FF        CALL tk.0040846C`

`004905F8   .   8B55 EC            MOV EDX,DWORD PTR SS:[EBP-14]`

`004905FB   .   B8 E0094900        MOV EAX,tk.004909E0              ;  +`

`00490600   .   E8 AB40F7FF        CALL tk.004046B0`

`00490605   .   85C0               TEST EAX,EAX`

`00490607   .   0F8E 6B010000      JLE tk.00490778                  ;  跳`

`0049060D   .   8D45 AC            LEA EAX,DWORD PTR SS:[EBP-54]`

`00490610   .   50                 PUSH EAX`

`---------------------------中间省略-------------------------------------`

`0049076B   .   8B55 90            MOV EDX,DWORD PTR SS:[EBP-70]`

`0049076E   .   8D45 E0            LEA EAX,DWORD PTR SS:[EBP-20]`

`00490771   .   E8 CE39F7FF        CALL tk.00404144`

`00490776   .   EB 1A              JMP SHORT tk.00490792`

`00490778   >   8D45 E8            LEA EAX,DWORD PTR SS:[EBP-18]    ;  跳到此`

`0049077B   .   BA F8094900        MOV EDX,tk.004909F8              ;  1`

`00490780   .   E8 BF39F7FF        CALL tk.00404144`

`00490785   .   8D45 E0            LEA EAX,DWORD PTR SS:[EBP-20]`

`00490788   .   BA F8094900        MOV EDX,tk.004909F8              ;  1`

`0049078D   .   E8 B239F7FF        CALL tk.00404144`

`00490792   >   8B45 F0            MOV EAX,DWORD PTR SS:[EBP-10]    ;  [EBP-10]=s1`

`00490795   .   E8 FAA0F7FF        CALL tk.0040A894                 ;  假码中无+号将出现异常`

`0049079A   .   DB7D 84            FSTP TBYTE PTR SS:[EBP-7C]       ;  [EBP-7C]假码(s1)`

`0049079D   .   9B                 WAIT`

`0049079E   .   8B45 E4            MOV EAX,DWORD PTR SS:[EBP-1C]    ;  [EBP-1C]=计算结果`

`004907A1   .   E8 EEA0F7FF        CALL tk.0040A894`

`004907A6   .   DB6D 84            FLD TBYTE PTR SS:[EBP-7C]`

`004907A9   .   DEE1               FSUBRP ST(1),ST                  ;  st(1)=假码(s1)减计算结果`

`004907AB   .   D81D FC094900      FCOMP DWORD PTR DS:[4909FC]       ;  比较是否相等`

`004907B1   .   DFE0               FSTSW AX                         ;  AX=100(保存状态字的值)`

`004907B3   .   9E                 SAHF                             ;  AH 装入标志寄存器`

`004907B4   .   0F87 53010000      JA tk.0049090D                   ;  大于跳`

`004907BA   .   8B45 E4            MOV EAX,DWORD PTR SS:[EBP-1C]    ;  [EBP-1C]=194349597671206894`

`004907BD   .   E8 D2A0F7FF        CALL tk.0040A894`

`004907C2   .   DBBD 78FFFFFF      FSTP TBYTE PTR SS:[EBP-88]`

`004907C8   .   9B                 WAIT`

`004907C9   .   8B45 F0            MOV EAX,DWORD PTR SS:[EBP-10]`

`004907CC   .   E8 C3A0F7FF        CALL tk.0040A894`

`004907D1   .   DBAD 78FFFFFF      FLD TBYTE PTR SS:[EBP-88]`

`004907D7   .   DEE1               FSUBRP ST(1),ST                  ;  计算结果减 s1`

`004907D9   .   D81D FC094900      FCOMP DWORD PTR DS:[4909FC]`

`004907DF   .   DFE0               FSTSW AX                         ;  AX=0`



## 机器码与算法分析

### 汇编代码片段

```
004907E1   .  9E                     SAHF  

004907E2   .  0F87 25010000         JA  tk.0049090D           ;  大于跳到标志位置 0  

004907E8   .  8B45 E8               MOV EAX,DWORD PTR SS:[EBP-18]  

004907EB   .  8B55 E0               MOV EDX,DWORD PTR SS:[EBP-20]  

004907EE   .  E8 C53CF7FF           CALL tk.004044B8  

004907F3   .  75 0B                 JNZ SHORT tk.00490800  

\---------------------------中间省略-------------------------------------  

00490904   .  8BC6                  MOV EAX,ESI  

00490906   .  E8 AD29F7FF           CALL tk.004032B8  

0049090B   .  EB 04                 JMP SHORT tk.00490911  

0049090D   >  C645 F7 00            MOV BYTE PTR SS:[EBP-9],0        ;  [EBP-9]标志位  

00490911   >  33C0                  XOR EAX,EAX  

00490913   .  5A                    POP EDX  

00490914   .  59                    POP ECX  

00490915   .  59                    POP ECX  

00490916   .  64:8910              MOV DWORD PTR FS:[EAX],EDX  

00490919   .  EB 0E                 JMP SHORT tk.00490929  

0049091B   .^ E9 382EF7FF           JMP tk.00403758  

00490920   .  C645 F7 00            MOV BYTE PTR SS:[EBP-9],0  

00490924   .  E8 9731F7FF           CALL tk.00403AC0  

00490929   >  33C0                  XOR EAX,EAX  

\---------------------------中间省略-------------------------------------  

00490977   .  BA 02000000           MOV EDX,2  

0049097C   .  E8 4F37F7FF           CALL tk.004040D0  

00490981   .  C3                    RETN  

00490982   .^ E9 8530F7FF           JMP tk.00403A0C  

00490987   .^ EB C1                 JMP SHORT tk.0049094A  

00490989     8A45 F7               MOV AL,BYTE PTR SS:[EBP-9]       ;  [EBP-9]标志位、爆破点  

0049098C     5F                    POP EDI  

0049098D     5E                    POP ESI  

0049098E   .  5B                    POP EBX  

0049098F   .  8BE5                 MOV ESP,EBP  

00490991   .  5D                    POP EBP  

00490992   .  C3                    RETN                             ;  返回到 0048D51A  
```

### 二、算法小结

1. **注册码有两种形式：**
   - `(1)` `*** + *** + ***`
   - `(2)` `*** + ***`
   
   示例：
   - `12345678+123+234`
   - `12345678+123`

2. **用户名最后一位 16 进制数乘位数，再转成 10 进制**
   示例：`wzwgp` -> `70 * 5 = 230` -> `560`

3. **第一种形式的注册码计算逻辑：**
   - 连接：用户名乘位数、位数、第一个加号后的注册码
     - `560 + 5 + 123` --> `5605123`
   - 转成 16 进制：`5605123` --> `558703`
   - 相加：`558703 + 558703` = `AB0E06`
   - `AB0E06 * 95A4EFD` (常数) = `63FD5EB6EAFEE` (`EB6EAFEE r 00063FD5`)
   - `63FD5 + AB0E06 * 4` (常数) = `2B277ED`
   - 连接：`2B277ED + EB6EAFEE` --> `2B277EDEB6EAFEE`
   - 转换：`2B277EDEB6EAFEE(H)` --> `1.9434959767120689400e+17` (浮点数) --> `194349597671206894(D)`
   - 用户名：`wzwgp`
   - 注册码：`194349597671206894+123+234` (第二个加号后面可填任意数或不填)

4. **第二种形式的注册码计算逻辑：**
   - 连接：用户名乘位数、位数
     - `560 + 5` --> `5605`
   - 转成 16 进制：`5605` --> `15E5`
   - 相加：`15E5 + 15E5` = `2BCA`
   - `2BCA * 95A4EFD` (常数) = `1998C86D2A2` (`8C86D2A2 r 00000199`)
   - `199 + 2BCA * 4` (常数) = `B0C1`
   - 连接：`B0C1 + 8C86D2A2` --> `B0C18C86D2A2`
   - 转换：`B0C18C86D2A2(H)` --> `1.9434533282269000000e+14` (浮点数) --> `194345332822690(D)`
   - 用户名：`wzwgp`
   - 注册码：`194345332822690+123` (加号后面可填任意数或不填)

5. **注册成功后 `About` 窗口显示：**
   - 第一种形式注册成功后：
     ```
     Licensed To: 123 Computer(s)
     wzwgp
     ```
   - 第二种形式注册成功后：
     ```
     Licensed To: Computer(s)
     wzwgp
     ```

`4dd466d0e1e0cc123afbe504da2462ca`
