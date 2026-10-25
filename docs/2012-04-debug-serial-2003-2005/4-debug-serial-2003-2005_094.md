
# DB Commander 2000 PRO (简称 DBC_2000) 笔记

> 原文：[`www.pediy.com/kssd/pediy06/pediy6119.htm`](https://www.pediy.com/kssd/pediy06/pediy6119.htm)

### 前言

这是我用 `ME` 以外的操作系统破的第一个软件！这个软件对于架设<传奇>私服的朋友是再熟悉不过了！以前这个软件下载的地方都有破解补丁放在一起的，可是我找了好久都没有找到，看来还是自己动手的好！

- **软件名称**：`DB Commander 2000 PRO` (简称 `DBC_2000`)
- **下载页面**：不记得了！
- **软件大小**：`6812K`
- **应用平台**：`WIN9X/WINNT/WIN2K/WINXP`
- **软件简介**：全是鸟文。不知道有什么用，就是知道架设<传奇>私服一定要有的东西！
- **软件限制**：全是鸟文，看不懂！就知道没有注册的话，在启动软件的时候会有一个提醒注册的窗口！
- **文章作者**：辉仔 `Yock[DFCG][YCG]`
- **作者声明**：本人发表这篇文章只是为了学习！！！请不用于商业用途或是将本文方法制作的注册机任意传播，读者看了文章后所做的事情与我无关，我也不会负责，请读者看了文章后三思而后行！最后希望大家在经济基础好的时候，支持共享软件！
- **破解工具**：`Ollydbg`

### 过程

用 `Ollydbg` 加载主程序 `DB_Commander.exe`

根据字符参考知道出错的地方一共有三处，分别对这三个地方下断！

输入用户名 `Yock` 和注册码 `987-654-321` 后，按注册，拦下：

```
00541398   . 53             PUSH EBX
00541399   . 56             PUSH ESI
0054139A   . 8BF2           MOV ESI,EDX
0054139C   . 8BD8           MOV EBX,EAX
0054139E   . BA E4135400    MOV EDX,TKDB_Com.005413E4                ;  ASCII "\\Software\\T&T Solutions\\DB Commander 2000 PRO"
005413A3   . B8 1C145400    MOV EAX,TKDB_Com.0054141C                ;  ASCII "Tsapanidis"
005413A8   . E8 5F2FECFF    CALL TKDB_Com.0040430C

             //这里好像是验证注册表是否有"Tsapanidis"这个值!
             //但是我注册表里面没有这个值!
             //这里可能是一个暗桩吧!

005413AD   . 85C0           TEST EAX,EAX
005413AF   . 7E 0C          JLE SHORT TKDB_Com.005413BD

             //这里不能跳,以跳下去就挂了!
             //我这里是跳的,所以我把这里 NOP 掉了!

005413B1   . 8BD6           MOV EDX,ESI
005413B3   . 8BC3           MOV EAX,EBX
005413B5   . E8 76FCFFFF    CALL TKDB_Com.00541030

             //这里是关键,根进去!

005413BA   . 5E             POP ESI
005413BB   . 5B             POP EBX
005413BC   . C3             RETN
005413BD   > E8 5AB4ECFF    CALL TKDB_Com.0040C81C
005413C2   . 6A 00          PUSH 0                                    ; /Arg1 = 00000000
005413C4   . 66:8B0D 281454>MOV CX,WORD PTR DS:[541428]               ; |
005413CB   . 33D2           XOR EDX,EDX                               ; |
005413CD   . B8 34145400    MOV EAX,TKDB_Com.00541434                 ; |ASCII "Invalid registration number entered !"

             //这里就是出错的地方了!

005413D2   . E8 3569F0FF    CALL TKDB_Com.00447D0C                    ; \TKDB_Com.00447D0C
005413D7   . 5E             POP ESI
005413D8   . 5B             POP EBX
005413D9   . C3             RETN

-----------------------------------------------------------------

上面 5413b5 的 CALL 来到这里!

00541030  /$ 55            PUSH EBP
00541031  |. 8BEC          MOV EBP,ESP
00541033  |. 6A 00         PUSH 0
00541035  |. 6A 00         PUSH 0
00541037  |. 6A 00         PUSH 0
00541039  |. 53            PUSH EBX
0054103A  |. 33C0          XOR EAX,EAX
0054103C  |. 55            PUSH EBP
0054103D  |. 68 20125400   PUSH TKDB_Com.00541220
00541042  |. 64:FF30       PUSH DWORD PTR FS:[EAX]
00541045  |. 64:8920       MOV DWORD PTR FS:[EAX],ESP
00541048  |. A1 AC215700   MOV EAX,DWORD PTR DS:[5721AC]
0054104D  |. FF00          INC DWORD PTR DS:[EAX]
0054104F  |. 8D55 FC       LEA EDX,DWORD PTR SS:[EBP-4]
00541052  |. A1 383B5700   MOV EAX,DWORD PTR DS:[573B38]
00541057  |. 8B80 EC010000  MOV EAX,DWORD PTR DS:[EAX+1EC]
0054105D  |. E8 EA8CF0FF   CALL TKDB_Com.00449D4C
00541062  |. 8B45 FC       MOV EAX,DWORD PTR SS:[EBP-4]
00541065  |. 50            PUSH EAX
00541066  |. 8D55 F8       LEA EDX,DWORD PTR SS:[EBP-8]
00541069  |. A1 383B5700   MOV EAX,DWORD PTR DS:[573B38]
0054106E  |. 8B80 E0010000  MOV EAX,DWORD PTR DS:[EAX+1E0]
00541074  |. E8 8B3DEEFF   CALL TKDB_Com.00424E04
00541079  |. 8B45 F8       MOV EAX,DWORD PTR SS:[EBP-8]

             //这里是用户名!

0054107C  |. 5A            POP EDX

             //这里是输入的假注册码!

0054107D  |. E8 8EF6FFFF   CALL TKDB_Com.00540710

             //关键跟进去!

00541082  |. 84C0          TEST AL,AL
00541084  |. 0F84 C1000000  JE TKDB_Com.0054114B

             //这里不跳的话,就把正确的注册信息保存起来!
             //跳走的话就出错咯!

0054108A  |. A1 C81C5700   MOV EAX,DWORD PTR DS:[571CC8]
0054108F  |. C600 01       MOV BYTE PTR DS:[EAX],1
00541092  |. A1 B0215700   MOV EAX,DWORD PTR DS:[5721B0]
00541097  |. C600 00       MOV BYTE PTR DS:[EAX],0
0054109A  |. B9 34125400   MOV ECX,TKDB_Com.00541234                 ;  ASCII "\\Software\\T&T Solutions\\DB Commander 2000 PRO"
0054109F  |. B2 01         MOV DL,1
005410A1  |. A1 C8BC4900   MOV EAX,DWORD PTR DS:[49BCC8]
005410A6  |. E8 6DB6F5FF   CALL TKDB_Com.0049C718
005410AB  |. 8BD8          MOV EBX,EAX
005410AD  |. BA 02000080   MOV EDX,80000002
005410B2  |. 8BC3          MOV EAX,EBX
005410B4  |. E8 A3ADF5FF   CALL TKDB_Com.0049BE5C
005410B9  |. BA 34125400   MOV EDX,TKDB_Com.00541234                 ;  ASCII "\\Software\\T&T Solutions\\DB Commander 2000 PRO"
005410BE  |. B1 01         MOV CL,1
005410C0  |. 8BC3          MOV EAX,EBX
005410C2  |. E8 F1AEF5FF   CALL TKDB_Com.0049BFB8
005410C7  |. 84C0          TEST AL,AL
005410C9  |. 74 5B         JE SHORT TKDB_Com.00541126
005410CB  |. 8D55 F8       LEA EDX,DWORD PTR SS:[EBP-8]
005410CE  |. A1 383B5700   MOV EAX,DWORD PTR DS:[573B38]
005410D3  |. 8B80 E0010000  MOV EAX,DWORD PTR DS:[EAX+1E0]
005410D9  |. E8 263DEEFF   CALL TKDB_Com.00424E04
005410DE  |. 8B45 F8       MOV EAX,DWORD PTR SS:[EBP-8]
005410E1  |. 50            PUSH EAX
005410E2  |. B9 6C125400   MOV ECX,TKDB_Com.0054126C                 ;  ASCII "Name"
005410E7  |. BA 7C125400   MOV EDX,TKDB_Com.0054127C                 ;  ASCII "5.0.0.0"
005410EC  |. 8BC3          MOV EAX,EBX
005410EE  |. E8 39B7F5FF   CALL TKDB_Com.0049C82C
005410F3  |. 8D55 F4       LEA EDX,DWORD PTR SS:[EBP-C]
005410F6  |. A1 383B5700   MOV EAX,DWORD PTR DS:[573B38]
005410FB  |. 8B80 EC010000  MOV EAX,DWORD PTR DS:[EAX+1EC]
00541101  |. E8 468CF0FF   CALL TKDB_Com.00449D4C
00541106  |. 8B45 F4       MOV EAX,DWORD PTR SS:[EBP-C]
00541109  |. 8D55 FC       LEA EDX,DWORD PTR SS:[EBP-4]
0054110C  |. E8 53F8FFFF   CALL TKDB_Com.00540964
00541111  |. 8B45 FC       MOV EAX,DWORD PTR SS:[EBP-4]
00541114  |. 50            PUSH EAX
00541115  |. BA 7C125400   MOV EDX,TKDB_Com.0054127C                 ;  ASCII "5.0.0.0"
0054111A  |. B9 8C125400   MOV ECX,TKDB_Com.0054128C                 ;  ASCII "RegNumber"
0054111F  |. 8BC3          MOV EAX,EBX
00541121  |. E8 06B7F5FF   CALL TKDB_Com.0049C82C
00541126  |> 8BC3          MOV EAX,EBX
00541128  |. E8 C320ECFF   CALL TKDB_Com.004031F0
0054112D  |. 6A 00         PUSH 0                                    ; /BeepType = MB_OK
0054112F  |. E8 4C64ECFF   CALL <JMP.&user32.MessageBeep>           ; \MessageBeep
00541134  |. 6A 00         PUSH 0                                    ; /Arg1 = 00000000
00541136  |. 66:8B0D 981254>MOV CX,WORD PTR DS:[541298]             ; |
0054113D  |. B2 02         MOV DL,2                                  ; |
0054113F  |. B8 A4125400   MOV EAX,TKDB_Com.005412A4                 ; |ASCII "Congratulations, upgrade was succesful.
You now have the registered version of DB Commander 2000 PRO !"
00541144  |. E8 C36BF0FF   CALL TKDB_Com.00447D0C                    ; \TKDB_Com.00447D0C
00541149  |. EB 49         JMP SHORT TKDB_Com.00541194
0054114B  |> 6A 40         PUSH 40                                   ; /BeepType = MB_ICONASTERISK
0054114D  |. E8 2E64ECFF   CALL <JMP.&user32.MessageBeep>           ; \MessageBeep
00541152  |. A1 AC215700   MOV EAX,DWORD PTR DS:[5721AC]
00541157  |. 8338 03       CMP DWORD PTR DS:[EAX],3
0054115A  |. 7C 23         JL SHORT TKDB_Com.0054117F
```



#### 注册成功路径

```
0054115C  .| 6A 00                    PUSH 0                                   ; /Arg1 = 00000000  
0054115E  .| 66:8B0D 981254>        MOV CX,WORD PTR DS:[541298]              ; |  
00541165  .| 33D2                   XOR EDX,EDX                              ; |  
00541167  .| B8 14135400            MOV EAX,TKDB_Com.00541314                ; |ASCII "Invalid registration number entered !  
00541167  .|                        Goodbye..."  
```

//这里是注册成功的地方! 

```
0054116C  .| E8 9B6BF0FF            CALL TKDB_Com.00447D0C                   ; \TKDB_Com.00447D0C  
00541171  .| A1 D41F5700            MOV EAX,DWORD PTR DS:[571FD4]  
00541176  .| 8B00                   MOV EAX,DWORD PTR DS:[EAX]  
00541178  .| E8 634FEFFF            CALL TKDB_Com.004360E0  
0054117D  .| EB 15                  JMP SHORT TKDB_Com.00541194  
```

#### 错误处理路径

```
0054117F  |> 6A 00                  PUSH 0                                   ; /Arg1 = 00000000  
00541181  .| 66:8B0D 981254>       MOV CX,WORD PTR DS:[541298]              ; |  
00541188  .| 33D2                  XOR EDX,EDX                              ; |  
0054118A  .| B8 50135400           MOV EAX,TKDB_Com.00541350                ; |ASCII "Invalid registration number entered !"  
```

//这里是另外一个出错的地方了! 

```
0054118F  .| E8 786BF0FF           CALL TKDB_Com.00447D0C                   ; \TKDB_Com.00447D0C  
00541194  |> A1 C81C5700           MOV EAX,DWORD PTR DS:[571CC8]  
00541199  .| 8038 00               CMP BYTE PTR DS:[EAX],0  
0054119C  .| 74 52                 JE SHORT TKDB_Com.005411F0  
```

-----------------------------------------------------------------
上面 54107d 的 `CALL` 来到这里!

#### 主验证逻辑函数 `TKDB_Com.004041D8` (或相关子程序入口)

```
00540710  /$ 55                    PUSH EBP  
00540711  .| 8BEC                  MOV EBP,ESP  
00540713  .| 83C4 F0               ADD ESP,-10  
00540716  .| 53                    PUSH EBX  
00540717  .| 56                    PUSH ESI  
00540718  .| 57                    PUSH EDI  
00540719  .| 33C9                  XOR ECX,ECX  
0054071B  .| 894D F0               MOV DWORD PTR SS:[EBP-10],ECX  
0054071E  .| 894D F4               MOV DWORD PTR SS:[EBP-C],ECX  
00540721  .| 8955 F8               MOV DWORD PTR SS:[EBP-8],EDX  
00540724  .| 8945 FC               MOV DWORD PTR SS:[EBP-4],EAX  
00540727  .| 8B45 FC               MOV EAX,DWORD PTR SS:[EBP-4]  
0054072A  .| E8 A93AECFF           CALL TKDB_Com.004041D8  
0054072F  .| 8B45 F8               MOV EAX,DWORD PTR SS:[EBP-8]  
00540732  .| E8 A13AECFF           CALL TKDB_Com.004041D8  
00540737  .| 33C0                  XOR EAX,EAX  
00540739  .| 55                    PUSH EBP  
0054073A  .| 68 4C085400           PUSH TKDB_Com.0054084C  
0054073F  .| 64:FF30               PUSH DWORD PTR FS:[EAX]  
00540742  .| 64:8920               MOV DWORD PTR FS:[EAX],ESP  
00540745  .| 33FF                  XOR EDI,EDI  
00540747  .| 8B45 FC               MOV EAX,DWORD PTR SS:[EBP-4]  
```

//输入的用户名! 

```
0054074A  .| E8 D538ECFF           CALL TKDB_Com.00404024  
```

//取用户名的位数! 

```
0054074F  .| 8BF0                  MOV ESI,EAX  
00540751  .| 85F6                  TEST ESI,ESI  
```

//是否有用户名! 

```
00540753  .| 7E 34                 JLE SHORT TKDB_Com.00540789  
```

//没有就跳走咯! 

```
00540755  .| BB 01000000           MOV EBX,1  
```

//赋值 1,`EBX` 作为计数器! 

```
0054075A  |> 8B45 FC               /MOV EAX,DWORD PTR SS:[EBP-4]  
0054075D  .| 8A4418 FF             |MOV AL,BYTE PTR DS:[EAX+EBX-1]  
00540761  .| 8BD0                  |MOV EDX,EAX  
00540763  .| 80C2 D0               |ADD DL,0D0  
00540766  .| 80EA 0A               |SUB DL,0A  
00540769  .| 72 10                 |JB SHORT TKDB_Com.0054077B  
```

//比较当前用户名是否是数字,是的话就跳下去直接累加! 

```
0054076B  .| 80C2 F9               |ADD DL,0F9  
0054076E  .| 80EA 1A               |SUB DL,1A  
00540771  .| 72 08                 |JB SHORT TKDB_Com.0054077B  
```

//比较当前用户名是否是大写字母,是的话就跳下取直接累加! 

```
00540773  .| 80C2 FA               |ADD DL,0FA  
00540776  .| 80EA 1A               |SUB DL,1A  
00540779  .| 73 0A                 |JNB SHORT TKDB_Com.00540785  
```

//比较当前用户名是否是小写字母,是的话就不跳!  
//如果不是小写字母的话跳走不累加了! 

```
0054077B  |> 8B55 FC               |MOV EDX,DWORD PTR SS:[EBP-4]  
0054077E  .| 25 FF000000           |AND EAX,0FF  
00540783  .| 03F8                  |ADD EDI,EAX  
00540785  |> 43                    |INC EBX  
00540786  .| 4E                    |DEC ESI  
00540787  .| 75 D1                 \JNZ SHORT TKDB_Com.0054075A  
```

//从 0054075A 到这里的循环是把用户名的值累加起来  
//累加的结果保存再 `EDI` 里面!  
//注意:要是用户名有非法字符和空格的话,就不作计算!  
//也就是说,只计算用户名的`"A-Z"`和`"a-z"`和`"0-9"`之间的数! 

```
00540789  |> 8BC7                  MOV EAX,EDI  
```

//累加值=`EDI`=`EAX` 

```
0054078B  .| C1E0 02               SHL EAX,2  
```

//逻辑左移 

```
0054078E  .| 8BD7                  MOV EDX,EDI  
```

//`EDI`=`EDX`=累加值 

```
00540790  .| 4A                    DEC EDX  
```

//减一 

```
00540791  .| 8BCA                  MOV ECX,EDX  
```

//`EDX`=`ECX`=累加值减一 

```
00540793  .| 99                    CDQ  
00540794  .| F7F9                  IDIV ECX  
```

//整数除法 

```
00540796  .| 8BD7                  MOV EDX,EDI  
```

//累加值=`EDI`=`EDX` 

```
00540798  .| 85D2                  TEST EDX,EDX  
```

//这里我理解是--是否有累加值,有的话就跳! 

```
0054079A  .| 79 03                 JNS SHORT TKDB_Com.0054079F  
```

//我这里是跳下去的! 

```
0054079C  .| 83C2 03               ADD EDX,3  
0054079F  |> C1FA 02               SAR EDX,2  
```

//算术右移 

```
005407A2  .| 0FAFD7                IMUL EDX,EDI  
```

//`EDX`=`EDX*EDI` 

```
005407A5  .| 03C2                  ADD EAX,EDX  
```

//`EAX`=`EAX+EDX` 

```
005407A7  .| 05 391B0000           ADD EAX,1B39  
```

//累加 `1B39` 

```
005407AC  .| 8BF8                  MOV EDI,EAX  
```

//`EAX`=`EDI` 

```
005407AE  .| 81C7 00E1F505        ADD EDI,5F5E100  
```

//累加 `5F5E100`  
//这里的 `EDI` 的十进制就是真的注册码了!  
//注册码就是这么练成的!  
//对于我来说,我有学到了一些宝贵的东西了,你呢??? 

```
005407B4  .| 8D45 F4               LEA EAX,DWORD PTR SS:[EBP-C]  
005407B7  .| E8 EC35ECFF           CALL TKDB_Com.00403DA8  
005407BC  .| 8B45 F8               MOV EAX,DWORD PTR SS:[EBP-8]  
```

//这里是输入的假注册码! 

```
005407BF  .| E8 6038ECFF           CALL TKDB_Com.00404024  
```

//取假注册码的位数! 

```
005407C4  .| 8BF0                  MOV ESI,EAX  
005407C6  .| 85F6                  TEST ESI,ESI  
```

//有没有输入? 

```
005407C8  .| 7E 3C                 JLE SHORT TKDB_Com.00540806  
```

//没有输入注册码就跳走咯! 

```
005407CA  .| BB 01000000           MOV EBX,1  
```

//赋值 1,`EBX` 作为计数器! 

```
005407CF  |> 8B45 F8               /MOV EAX,DWORD PTR SS:[EBP-8]  
005407D2  .| 8A4418 FF             |MOV AL,BYTE PTR DS:[EAX+EBX-1]  
005407D6  .| 04 D0                 |ADD AL,0D0  
005407D8  .| 2C 0A                 |SUB AL,0A  
005407DA  .| 72 0C                 |JB SHORT TKDB_Com.005407E8  
005407DC  .| 04 F9                 |ADD AL,0F9  
005407DE  .| 2C 1A                 |SUB AL,1A  
005407E0  .| 72 06                 |JB SHORT TKDB_Com.005407E8  
005407E2  .| 04 FA                 |ADD AL,0FA  
005407E4  .| 2C 1A                 |SUB AL,1A  
005407E6  .| 73 1A                 |JNB SHORT TKDB_Com.00540802  
```

```
005407E8  |> 8D45 F0               |LEA EAX,DWORD PTR SS:[EBP-10]  
005407EB  .| 8B55 F8               |MOV EDX,DWORD PTR SS:[EBP-8]  
005407EE  .| 8A541A FF             |MOV DL,BYTE PTR DS:[EDX+EBX-1]  
005407F2  .| E8 5537ECFF           |CALL TKDB_Com.00403F4C  
005407F7  .| 8B55 F0               |MOV EDX,DWORD PTR SS:[EBP-10]  
005407FA  .| 8D45 F4               |LEA EAX,DWORD PTR SS:[EBP-C]  
005407FD  .| E8 2A38ECFF           |CALL TKDB_Com.0040402C  
```

```
00540802  |> 43                    |INC EBX  
00540803  .| 4E                    |DEC ESI  
00540804  .| 75 C9                 \JNZ SHORT TKDB_Com.005407CF  
```

//从 005407CF 到这里的循环是把注册码的`"-"`去掉的!  
//举个例子,我输入的注册码是`"987-654-321"`  
//经过这里之后就变成`"987654321"`了! 

```
00540806  |> 8D55 F0               LEA EDX,DWORD PTR SS:[EBP-10]  
00540809  .| 8BC7                  MOV EAX,EDI  
0054080B  .| E8 F47FECFF           CALL TKDB_Com.00408804  
00540810  .| 8B45 F0               MOV EAX,DWORD PTR SS:[EBP-10]  
```

//这里就是真的注册码了! 

```
00540813  .| 8B55 F4               MOV EDX,DWORD PTR SS:[EBP-C]  
```

//这里是我输入的假注册码! 

```
00540816  .| E8 1939ECFF           CALL TKDB_Com.00404134  
0054081B  .| 0F94C3                SETE BL  
0054081E  .| BA 64085400           MOV EDX,TKDB_Com.00540864                ;  ASCII "Beverly Saunders Site"  
```

//这个就是万能注册名了! 

```
00540823  .| 8B45 FC               MOV EAX,DWORD PTR SS:[EBP-4]  
```

//这里是输入的用户名!  



`//可见这个软件有一个万能注册名的!`

```
00540826  |. E8 E13AECFF    CALL TKDB_Com.0040430C  
0054082B  |. 85C0           TEST EAX,EAX  
0054082D  |. 7E 02          JLE SHORT TKDB_Com.00540831  
0054082F  |. 33DB           XOR EBX,EBX  
00540831  |> 33C0           XOR EAX,EAX  
00540833  |. 5A             POP EDX  
00540834  |. 59             POP ECX  
00540835  |. 59             POP ECX  
00540836  |. 64:8910        MOV DWORD PTR FS:[EAX],EDX  
00540839  |. 68 53085400    PUSH TKDB_Com.00540853  
0054083E  |> 8D45 F0        LEA EAX,DWORD PTR SS:[EBP-10]  
00540841  |. BA 04000000    MOV EDX,4  
00540846  |. E8 8135ECFF    CALL TKDB_Com.00403DCC  
0054084B  \\. C3            RETN  
0054084C   .^E9 6B2FECFF    JMP TKDB_Com.004037BC  
00540851   .^EB EB          JMP SHORT TKDB_Com.0054083E  
00540853   . 8BC3           MOV EAX,EBX  
00540855   . 5F             POP EDI  
00540856   . 5E             POP ESI  
00540857   . 5B             POP EBX  
00540858   . 8BE5           MOV ESP,EBP  
0054085A   . 5D             POP EBP  
0054085B   . C3             RETN 
```

---

**总结**

又来到我最怕的地方了——总结!

说起来，这个软件又有一个地方我搞不懂的就是为什么要检查注册表的：

`\[\\Software\\T&T Solutions\\DB Commander 2000 PRO\]`

`Tsapanidis`

我的机子里面没有这个项目，所以要把：

`005413AF   . 7E 0C          JLE SHORT TKDB_Com.005413BD`

`NOP` 掉才可以注册成功! 如果大虾们愿意研究的话，可以告诉我一个究竟吗?

把用户名的字符的 `ASCII` 码累加以来! (非法字符和空格除外，只累加这些值. 包括 `"A-Z"`/`"a-z"`/`"0-9"`)

最后把累加值经过下面的运算得出得结果的十进制就是真的注册码了!

`累加值=EAX=EDI`

```
MOV EAX,EDI  
SHL EAX,2  
MOV EDX,EDI  
DEC EDX  
MOV ECX,EDX  
CDQ  
IDIV ECX  
SAR EDX,2  
IMUL EDX,EDI  
ADD EAX,EDX  
ADD EAX,1B39  
MOV EDI,EAX  
ADD EDI,5F5E100  
```

最后注册码的结果=`EDI` 的十进制值!

注册信息保存在:

`\[HKEY_LOCAL_MACHINE\SOFTWARE\T&T Solutions\DB Commander 2000 PRO\5.0.0.0\]`

`"RegNumber"="005F69C6B"`

`//保存方式是注册码的十六进制!`

一组可用的注册码:

`Yock`
`100-047-979`

---

最后在这里真心感谢你花了那么多时间看这篇文章! 谢谢了...

`2003.6.18 于清远`

`4dd466d0e1e0cc123afbe504da2462ca`
