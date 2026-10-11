
# 珠联网络棋牌游戏(共享版)V2.94 (RSA 算法 N 和 E 比较小)

> 原文：[`www.pediy.com/kssd/pediy07/pediy7-138.htm`](https://www.pediy.com/kssd/pediy07/pediy7-138.htm)

机械码:

`912846851261`

输入注册码:

`111122223333`

怎么到达这里我就不在重复了,很多文章都写过.

```
00407368   .  E8 77180100   CALL zlsrv.00418BE4
0040736D   .  8BF8          MOV EDI,EAX
0040736F   .  83C9 FF       OR ECX,FFFFFFFF
00407372   .  33C0          XOR EAX,EAX
00407374   .  8D5424 34     LEA EDX,DWORD PTR SS:[ESP+34]
00407378   .  F2:AE         REPNE SCAS BYTE PTR ES:[EDI]
0040737A   .  F7D1          NOT ECX
0040737C   .  2BF9          SUB EDI,ECX
0040737E   .  8BC1          MOV EAX,ECX
00407380   .  8BF7          MOV ESI,EDI
00407382   .  8BFA          MOV EDI,EDX
00407384   .  C1E9 02       SHR ECX,2
00407387   .  F3:A5         REP MOVS DWORD PTR ES:[EDI],DWORD PTR DS>;  拷贝输入的注册码
00407389   .  8BC8          MOV ECX,EAX
0040738B   .  83E1 03       AND ECX,3
0040738E   .  F3:A4         REP MOVS BYTE PTR ES:[EDI],BYTE PTR DS:[>;  最后加入空格
00407390   .  8D4C24 14     LEA ECX,DWORD PTR SS:[ESP+14]
00407394   .  51            PUSH ECX
00407395   .  E8 36140000   CALL zlsrv.004087D0                          ;  机械码种子生成
0040739A   .  8B4C24 18     MOV ECX,DWORD PTR SS:[ESP+18]                ;  ECX=6E6D0F00
0040739E   .  8BF0          MOV ESI,EAX                                  ;  ESI=EAX=$5B
004073A0   .  8BC1          MOV EAX,ECX                                  ;  EAX=ECX
004073A2   .  33D2          XOR EDX,EDX
004073A4   .  BF 731F0000   MOV EDI,1F73
004073A9   .  F7F7          DIV EDI                                      ;  EDX=$4ED
004073AB   .  B8 53F01E41   MOV EAX,411EF053                              ;  EAX=411EF053
004073B0   .  895424 20     MOV DWORD PTR SS:[ESP+20],EDX                ;  EDX 保存
004073B4   .  F7E1          MUL ECX                                      ;  EAX=EAX*ECX
004073B6   .  8BCA          MOV ECX,EDX                                    ;  EDX 保存高 8 位
004073B8   .  33D2          XOR EDX,EDX                                    ;  ECX=EDX
004073BA   .  C1E9 0B       SHR ECX,0B                                     ;  ECX=ECX SHR $B
004073BD   .  8BC1          MOV EAX,ECX                                    ;  EAX=ECX
004073BF   .  F7F7          DIV EDI                                        ;  EDX=EAX MOD EDI
004073C1   .  B8 53F01E41   MOV EAX,411EF053
004073C6   .  895424 24     MOV DWORD PTR SS:[ESP+24],EDX                ;  EDX=$124D 保存
004073CA   .  F7E1          MUL ECX                                        ;  EAX=EAX*ECX   EDX 保存高 8 位
004073CC   .  C1EA 0B       SHR EDX,0B                                     ;  EDX=EDX SHR $B
004073CF   .  895424 18     MOV DWORD PTR SS:[ESP+18],EDX                ;  EDX=$1C
004073D3   .  895424 28     MOV DWORD PTR SS:[ESP+28],EDX
004073D7   .  8D5424 40     LEA EDX,DWORD PTR SS:[ESP+40]                ;利用种子生成机械码
004073DB   .  52            PUSH EDX
004073DC   .  E8 C83D0000   CALL zlsrv.0040B1A9                          ;  EAX=3333 输入注册码的后 4 位
004073E1   .  8BF8          MOV EDI,EAX
004073E3   .  8D4424 40     LEA EAX,DWORD PTR SS:[ESP+40]
004073E7   .  50            PUSH EAX
004073E8   .  C64424 48 00  MOV BYTE PTR SS:[ESP+48],0
004073ED   .  E8 B73D0000   CALL zlsrv.0040B1A9                          ;  EAX=$8AE=2222
004073F2   .  8D4C24 40     LEA ECX,DWORD PTR SS:[ESP+40]
004073F6   .  894424 1C     MOV DWORD PTR SS:[ESP+1C],EAX                ;  EAX 保存
004073FA   .  51            PUSH ECX
004073FB   .  C64424 48 00  MOV BYTE PTR SS:[ESP+48],0
00407400   .  E8 A43D0000   CALL zlsrv.0040B1A9                          ;  EAX=$457=1111
00407405   .  56            PUSH ESI
00407406   .  57            PUSH EDI
00407407   .  894424 30     MOV DWORD PTR SS:[ESP+30],EAX
0040740B   .  E8 C0060000   CALL zlsrv.00407AD0                          ;  RSA(3333)=12B4
00407410   .  8B5424 28     MOV EDX,DWORD PTR SS:[ESP+28]
00407414   .  56            PUSH ESI
00407415   .  52            PUSH EDX
00407416   .  8BF8          MOV EDI,EAX
00407418   .  E8 B3060000   CALL zlsrv.00407AD0                          ;  RSA(2222)=$304
0040741D   .  894424 30     MOV DWORD PTR SS:[ESP+30],EAX
00407421   .  8B4424 38     MOV EAX,DWORD PTR SS:[ESP+38]
00407425   .  56            PUSH ESI
00407426   .  50            PUSH EAX
00407427   .  E8 A4060000   CALL zlsrv.00407AD0                          ;  RSA(1111)=$288
0040742C   .  8B4C24 44     MOV ECX,DWORD PTR SS:[ESP+44]
00407430   .  83C4 28       ADD ESP,28
00407433   .  3BF9          CMP EDI,ECX                                    ;  比较 1261 和$12B4
00407435      75 72         JNZ SHORT zlsrv.004074A9
00407437   .  8B4C24 10     MOV ECX,DWORD PTR SS:[ESP+10]
0040743B   .  8B5424 20     MOV EDX,DWORD PTR SS:[ESP+20]
0040743F   .  3BCA          CMP ECX,EDX
00407441      75 66         JNZ SHORT zlsrv.004074A9                      ;  比较 4685 和$304
00407443   .  3B4424 24     CMP EAX,DWORD PTR SS:[ESP+24]
00407447      75 60         JNZ SHORT zlsrv.004074A9                      ;  比较 28 和$288
00407449   .  E8 3E790100   CALL zlsrv.0041ED8C
0040744E   .  8B13          MOV EDX,DWORD PTR DS:[EBX]
00407450   .  8B0D A0E19000 MOV ECX,DWORD PTR DS:[90E1A0]
00407456   .  8B40 04       MOV EAX,DWORD PTR DS:[EAX+4]
00407459   .  52            PUSH EDX                                      ; /Arg3
0040745A   .  8B15 A4E19000 MOV EDX,DWORD PTR DS:[90E1A4]                 ; |
00407460   .  51            PUSH ECX                                      ; |Arg2 => 00E06EE8 ASCII "CODE"
00407461   .  52            PUSH EDX                                      ; |Arg1 => 00E06E98 ASCII "Zlnet"
00407462   .  8BC8          MOV ECX,EAX                                   ; |
00407464   .  E8 2D710100   CALL zlsrv.0041E596                           ; \\zlsrv.0041E596
00407469   .  8D4424 28     LEA EAX,DWORD PTR SS:[ESP+28]
0040746D   .  6A 00          PUSH 0
0040746F   .  8D4C24 2C     LEA ECX,DWORD PTR SS:[ESP+2C]
00407473   .  50            PUSH EAX
00407474   .  51            PUSH ECX
00407475   .  8BCD          MOV ECX,EBP
00407477   .  C64424 3C 00  MOV BYTE PTR SS:[ESP+3C],0
0040747C   .  C64424 34 D7  MOV BYTE PTR SS:[ESP+34],0D7
00407481   .  C64424 35 A2  MOV BYTE PTR SS:[ESP+35],0A2
00407486   .  C64424 36 B2  MOV BYTE PTR SS:[ESP+36],0B2
0040748B   .  C64424 37 E1  MOV BYTE PTR SS:[ESP+37],0E1
00407490   .  C64424 38 B3  MOV BYTE PTR SS:[ESP+38],0B3
00407495   .  C64424 39 C9  MOV BYTE PTR SS:[ESP+39],0C9
0040749A   .  C64424 3A B9  MOV BYTE PTR SS:[ESP+3A],0B9
0040749F   .  C64424 3B A6  MOV BYTE PTR SS:[ESP+3B],0A6
004074A4   .  E8 933B0100   CALL zlsrv.0041B03C                           ;注册成功提示
004074A9   >  5F            POP EDI
004074AA   .  5E            POP ESI
004074AB   .  5D            POP EBP
004074AC   .  5B            POP EBX
004074AD   .  83C4 74       ADD ESP,74
004074B0   .  C3            RETN
```

---------------------------------------------------------------------------------------------

`X ^ Y % Z`

RSA 算法的实现部分:

```
00407AD0  /$  56            PUSH ESI
00407AD1  |.  8B7424 0C     MOV ESI,DWORD PTR SS:[ESP+C]
00407AD5  |.  56            PUSH ESI
00407AD6  |.  E8 C5FFFFFF   CALL zlsrv.00407AA0          ;算法在下面
00407ADB  |.  83C4 04       ADD ESP,4
00407ADE  |.  B8 01000000   MOV EAX,1
00407AE3  |.  85F6          TEST ESI,ESI
00407AE5  |.  7E 19         JLE SHORT zlsrv.00407B00
00407AE7  |.  8BCE          MOV ECX,ESI                 ;这个 ECX 里面的数据就是 E
00407AE9  |.  8B7424 08     MOV ESI,DWORD PTR SS:[ESP+8]
00407AED  |.  57            PUSH EDI
00407AEE  |>  0FAFC6        /IMUL EAX,ESI
00407AF1  |.  33D2          |XOR EDX,EDX
```



# 00407AF3

```
00407AF3  |.  BF 731F0000   |MOV EDI,1F73        ;这个数据就是 N  
00407AF8  |.  F7F7          |DIV EDI  
00407AFA  |.  49           |DEC ECX  
00407AFB  |.  8BC2         |MOV EAX,EDX  
00407AFD  |.^ 75 EF        \JNZ SHORT zlsrv.00407AEE  
00407AFF  |.  5F           POP EDI  
00407B00  |>  5E           POP ESI  
00407B01  \.  C3           RETN                   

00407AA0  /$  53           PUSH EBX  
00407AA1  |.  56           PUSH ESI  
00407AA2  |.  57           PUSH EDI  
00407AA3  |.  8B7C24 10    MOV EDI,DWORD PTR SS:[ESP+10]  
00407AA7  |.  33F6         XOR ESI,ESI  
00407AA9  |.  33C9         XOR ECX,ECX  
00407AAB  |>  03CF         ADD ECX,EDI  
00407AAD  |.  BB C01E0000  MOV EBX,1EC0  
00407AB2  |.  8BC1         MOV EAX,ECX  
00407AB4  |.  46           INC ESI  
00407AB5  |.  99           CDQ  
00407AB6  |.  F7FB         IDIV EBX  
00407AB8  |.  83FA 01      CMP EDX,1  
00407ABB  |.^ 75 EE        JNZ SHORT zlsrv.00407AAB  
00407ABD  |.  8BC6         MOV EAX,ESI  
00407ABF  |.  5F           POP EDI  
00407AC0  |.  5E           POP ESI  
00407AC1  |.  5B           POP EBX  
00407AC2  \.  C3           RETN  
```

# 机械码种子的生成

```
004087D0  /$  81EC 0C010000 SUB ESP,10C  
004087D6  |.  53            PUSH EBX  
004087D7  |.  56            PUSH ESI  
004087D8  |.  57            PUSH EDI  
004087D9  |.  8D4424 4C     LEA EAX,DWORD PTR SS:[ESP+4C]  
004087DD  |.  68 C8000000   PUSH 0C8                            ; /BufSize = C8 (200.)  
004087E2  |.  50            PUSH EAX                            ; |Buffer  
004087E3  |.  E8 04BB0000   CALL <JMP.&WSOCK32.#57>            ; \gethostname  
004087E8  |.  B9 10000000   MOV ECX,10                         ;种子生成的第一部分  
004087ED  |.  33C0          XOR EAX,EAX  
004087EF  |.  8D7C24 0C     LEA EDI,DWORD PTR SS:[ESP+C]  
004087F3  |.  68 5C010000   PUSH 15C  
004087F8  |.  F3:AB         REP STOS DWORD PTR ES:[EDI]  
004087FA  |.  E8 DF2B0000   CALL zlsrv.0040B3DE  
004087FF  |.  8BD8          MOV EBX,EAX  
00408801  |.  8D7C24 50     LEA EDI,DWORD PTR SS:[ESP+50]  
00408805  |.  83C9 FF       OR ECX,FFFFFFFF  
00408808  |.  33C0          XOR EAX,EAX  
0040880A  |.  83C4 04       ADD ESP,4  
0040880D  |.  8D5424 16     LEA EDX,DWORD PTR SS:[ESP+16]  
00408811  |.  F2:AE         REPNE SCAS BYTE PTR ES:[EDI]  
00408813  |.  F7D1          NOT ECX  
00408815  |.  2BF9          SUB EDI,ECX  
00408817  |.  8BC1          MOV EAX,ECX  
00408819  |.  8BF7          MOV ESI,EDI  
0040881B  |.  8BFA          MOV EDI,EDX  
0040881D  |.  BA 0F000000   MOV EDX,0F  
00408822  |.  C1E9 02       SHR ECX,2  
00408825  |.  F3:A5         REP MOVS DWORD PTR ES:[EDI],DWORD PTR DS>  
00408827  |.  8BC8          MOV ECX,EAX  
00408829  |.  33C0          XOR EAX,EAX  
0040882B  |.  83E1 03       AND ECX,3  
0040882E  |.  F3:A4         REP MOVS BYTE PTR ES:[EDI],BYTE PTR DS:[>  
00408830  |.  8D7C24 4C     LEA EDI,DWORD PTR SS:[ESP+4C]  
00408834  |.  83C9 FF       OR ECX,FFFFFFFF  
00408837  |.  F2:AE         REPNE SCAS BYTE PTR ES:[EDI]  
00408839  |.  F7D1          NOT ECX  
0040883B  |.  49            DEC ECX  
0040883C  |.  8D7C24 4C     LEA EDI,DWORD PTR SS:[ESP+4C]  
00408840  |.  2BD1          SUB EDX,ECX  
00408842  |.  83C9 FF       OR ECX,FFFFFFFF  
00408845  |.  F2:AE         REPNE SCAS BYTE PTR ES:[EDI]  
00408847  |.  F7D1          NOT ECX  
00408849  |.  49            DEC ECX  
0040884A  |.  B8 20202020   MOV EAX,20202020  
0040884F  |.  8D7C0C 16     LEA EDI,DWORD PTR SS:[ESP+ECX+16]  
00408853  |.  8BCA          MOV ECX,EDX  
00408855  |.  C1E9 02       SHR ECX,2  
00408858  |.  F3:AB         REP STOS DWORD PTR ES:[EDI]  
0040885A  |.  8BCA          MOV ECX,EDX  
0040885C  |.  83E1 03       AND ECX,3  
0040885F  |.  F3:AA         REP STOS BYTE PTR ES:[EDI]  
00408861  |.  8D4424 0C     LEA EAX,DWORD PTR SS:[ESP+C]  
00408865  |.  C64424 0C 32  MOV BYTE PTR SS:[ESP+C],32  
0040886A  |.  50            PUSH EAX  
0040886B  |.  895C24 14     MOV DWORD PTR SS:[ESP+14],EBX  
0040886F  |.  C64424 40 00  MOV BYTE PTR SS:[ESP+40],0  
00408874  |.  66:C74424 18 >MOV WORD PTR SS:[ESP+18],15C  
0040887B  |.  E8 50180000   CALL <JMP.&NETAPI32.Netbios>      ;种子生成的第二部分  
00408880  |.  8D4C24 0C     LEA ECX,DWORD PTR SS:[ESP+C]  
00408884  |.  C64424 0C 33  MOV BYTE PTR SS:[ESP+C],33  
00408889  |.  51            PUSH ECX  
0040888A  |.  E8 41180000   CALL <JMP.&NETAPI32.Netbios>  
0040888F  |.  8A4424 0D     MOV AL,BYTE PTR SS:[ESP+D]  
00408893  |.  84C0          TEST AL,AL  
00408895  |.  75 24         JNZ SHORT zlsrv.004088BB  
00408897  |.  8B8424 1C0100>MOV EAX,DWORD PTR SS:[ESP+11C]  
0040889E  |.  8A13          MOV DL,BYTE PTR DS:[EBX]  
004088A0  |.  BE 5B000000   MOV ESI,5B                       ;E=$5B  
004088A5  |.  8810          MOV BYTE PTR DS:[EAX],DL  
004088A7  |.  8A4B 02       MOV CL,BYTE PTR DS:[EBX+2]  
004088AA  |.  8848 01       MOV BYTE PTR DS:[EAX+1],CL        ;保存种子 1  
004088AD  |.  8A53 04       MOV DL,BYTE PTR DS:[EBX+4]  
004088B0  |.  8850 02       MOV BYTE PTR DS:[EAX+2],DL        ;保存种子 2  
004088B3  |.  8A4B 05       MOV CL,BYTE PTR DS:[EBX+5]  
004088B6  |.  8848 03       MOV BYTE PTR DS:[EAX+3],CL        ;保存种子 3  
004088B9  |.  EB 24         JMP SHORT zlsrv.004088DF  
004088BB  |>  8B9424 1C0100>MOV EDX,DWORD PTR SS:[ESP+11C]  
004088C2  |.  6A 0A         PUSH 0A                           ; /pFileSystemNameSize = 0000000A  
004088C4  |.  6A 00         PUSH 0                            ; |pFileSystemNameBuffer = NULL  
004088C6  |.  6A 00         PUSH 0                            ; |pFileSystemFlags = NULL  
004088C8  |.  6A 00         PUSH 0                            ; |pMaxFilenameLength = NULL  
004088CA  |.  52            PUSH EDX                          ; |pVolumeSerialNumber  
004088CB  |.  6A 0C         PUSH 0C                           ; |MaxVolumeNameSize = C(12.)  
004088CD  |.  6A 00         PUSH 0                            ; |VolumeNameBuffer = NULL  
004088CF  |.  68 38B34200   PUSH zlsrv.0042B338               ; |RootPathName = "c:\\"  
004088D4  |.  FF15 58314200 CALL DWORD PTR DS:[<&KERNEL32.GetVolumeI>; \GetVolumeInformationA  
004088DA  |.  BE 5F000000   MOV ESI,5F                        ;E=$5F  
004088DF  |>  53            PUSH EBX                           ;如果已经生成种子了  
004088E0  |.  E8 B12A0000   CALL zlsrv.0040B396               ;就不用硬盘码做种子了  
004088E5  |.  83C4 04       ADD ESP,4  
004088E8  |.  8BC6          MOV EAX,ESI  
004088EA  |.  5F            POP EDI  
004088EB  |.  5E            POP ESI  
004088EC  |.  5B            POP EBX  
004088ED  |.  81C4 0C010000 ADD ESP,10C  
004088F3  \.  C3            RETN  
```

# 算法总结

机械码其实被分成 3 部分进行处理的.

```
91 28   4685 1261
   ^^   ^^^^ ^^^^
   1111 2222 3333
```

注册码:

```
831223855529
```

## RSA 算法介绍

```
P = 1st large prime number
Q =  2nd large prime number  (sizes of P and Q should not differ too much!)
E = Public  Exponent (a random number  which must fulfil:
              GCD(E,  (P-1)*(Q-1))==1)
N = Public Modulus, the product of P and Q: N=P*Q
D = Private Exponent: D=E^(-1) mod ((P-1)*(Q-1))

C = M^E mod N.
M=C^D mod N.
```

=======================================================================================================

由于这次遇到的 `N` 和 `E` 比较小可以用工具很容易分解所以写注册机不是很麻烦!

`E=$5B` 或者 `E=$5F`  `N=$1F73`;

对 `N` 进行分解求出

`P=$53` `Q=$61`

`D=$1E13` 或者 `D=$71F`

`3333^$5B MOD $1F73 =4788($12B4)`

用 `4788` 跟 `1261` 比较是否相等

求逆:

`1261^$1E13 MOD $1F73=5529` 正确注册码

所以注册机有 2 种办法写: 1.穷举 2.RSA 求逆



`function SF(Mint,E,N:Dword):Dword; //软件里面的 RSA 算法`

`var`

`int,MMod,a:Dword;`

`begin`

`int:=1;`

`for a:=1 to E do`

`begin`

`int:=int*Mint;`

`Mmod:=int mod N;`

`int:=MMod;`

`end;`

`Sf:=MMOd;`

`end;`

`//穷举注册机`

`procedure TForm1.Button1Click(Sender: TObject);`

`var`

`int,a,mn1,mn2,mn3:Dword;`

`sn1,sn2,sn3:string;`

`begin`

`mn1:=strtoint(copy(edit1.Text,3,2));`

`for a:=1 to 8051 do //这里之所以到 8051 是因为 N 为$1F73 余数不可能大于他`

`begin`

`int:=sf(a,$5B,$1F73);`

`if int=mn1 then`

`sn1:=inttostr(a);`

`end;`

`mn2:=strtoint(copy(edit1.Text,5,4));`

`for a:=1 to 8051 do`

`begin`

`int:=sf(a,$5B,$1F73);`

`if int=mn2 then`

`sn2:=inttostr(a);`

`end;`

`mn3:=strtoint(copy(edit1.Text,9,4));`

`for a:=1 to 8051 do`

`begin`

`int:=sf(a,$5B,$1F73);`

`if int=mn3 then`

`sn3:=inttostr(a);`

`end;`

`SN.Text:=sn1+sn2+sn3;`

`END;`

`//RSA 求逆的注册机也比较简单就可以写出来,利用软件里面的算法就可以,而且效率比这个高多了!这里我就不写了!`

不过这个软件即使注册成功也还是 10 人限制版本的!所以才有的这篇算法分析! 

`4dd466d0e1e0cc123afbe504da2462ca`
