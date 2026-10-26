
保存修改代码，重新运行`22`，并在第一个需要修复的`CALL`

```
004651BC   .  E8 172FFBFF   CALL Doit_ASP.004180D8
```

下断点，运行停到这里`004651BC`。

`CALL`的修复，这些`call`都是`delphi`的内部接口，虽然`212r`和`22`的`delphi`版本不同但是想来区别不会太大。提取代码进行查找试试。

`212r`中的`CALL`

```
0043F5F6   .  E8 2993FDFF   CALL ASPack.00418924

00418924  /$  53            PUSH EBX
00418925  |.  3B50 10       CMP EDX,DWORD PTR DS:[EAX+10]
00418928  |.  74 0C         JE SHORT ASPack.00418936
0041892A  |.  8950 10       MOV DWORD PTR DS:[EAX+10],EDX
0041892D  |.  66:BB FFFF    MOV BX,0FFFF
00418931  |.  E8 4EA3FEFF   CALL ASPack.00402C84
00418936  |>  5B            POP EBX                                  ;  0012FE50
00418937  \.  C3            RETN
```

复制一段二进制，注意把把常数,绝对跳转等换成`??`

```
53 3B 50 10 74 0C 89 50 10 66 BB

53 3B 50 ?? 74 0C 89 50 ?? 66 BB
```

在`212r`中搜索特征码，检查是否有特殊性，只找到一处，在`22`中搜索

找到`CALL`

```
004180D8  /$  53            PUSH EBX
004180D9  |.  3B50 18       CMP EDX,DWORD PTR DS:[EAX+18]
004180DC  |.  74 0C         JE SHORT Doit_ASP.004180EA
004180DE  |.  8950 18       MOV DWORD PTR DS:[EAX+18],EDX
004180E1  |.  66:BB FDFF    MOV BX,0FFFD
004180E5  |.  E8 C2AFFEFF   CALL Doit_ASP.004030AC
004180EA  |>  5B            POP EBX                                  ;  0012FE1C
004180EB  \.  C3            RETN
```

`212r`中的`00418924`对应`22`中的`004180D8`

修复代码

```
004651BC   .  E8 172FFBFF   CALL Doit_ASP.004180D8
```

`F8`走过这个检查是否正确，`OK`

第二个`CALL`

`IN 212r`

```
00406360  /$  53            PUSH EBX
00406361  |.  56            PUSH ESI                                 ;  ASPack.0043E920
00406362  |.  8BF2          MOV ESI,EDX
00406364  |.  8BD8          MOV EBX,EAX
00406366  |.  8BC6          MOV EAX,ESI                              ;  ASPack.0043E920
00406368  |.  8BD3          MOV EDX,EBX
0040636A  |.  E8 C5D2FFFF   CALL ASPack.00403634
0040636F  |.  5E            POP ESI                                  ;  0012FE50
00406370  |.  5B            POP EBX                                  ;  0012FE50
00406371  \.  C3            RETN
```

提取特征

```
53 56 8B F2 8B D8 8B C6 8B D3
```

`IN 22`

```
00408CC0  /$  53            PUSH EBX
00408CC1  |.  56            PUSH ESI
00408CC2  |.  8BF2          MOV ESI,EDX
00408CC4  |.  8BD8          MOV EBX,EAX
00408CC6  |.  8BC6          MOV EAX,ESI
00408CC8  |.  8BD3          MOV EDX,EBX
00408CCA  |.  E8 41B0FFFF   CALL Doit_ASP.00403D10
00408CCF  |.  5E            POP ESI                                  ;  0012FE1C
00408CD0  |.  5B            POP EBX                                  ;  0012FE1C
00408CD1  \.  C3            RETN
```

修复代码

```
004651C9   .  E8 F23AFAFF   CALL Doit_ASP.00408CC0
```

`F8`走过这个检查是否正确，`OK`

第三个`CALL`

`IN 212r`

```
00413120  /$  55            PUSH EBP
00413121  |.  8BEC          MOV EBP,ESP
00413123  |.  6A 00         PUSH 0
00413125  |.  53            PUSH EBX
00413126  |.  56            PUSH ESI                                 ;  ASPack.0043E920
00413127  |.  57            PUSH EDI
00413128  |.  8BF2          MOV ESI,EDX
0041312A  |.  8BD8          MOV EBX,EAX
0041312C  |.  33C0          XOR EAX,EAX
0041312E  |.  55            PUSH EBP
```

提取特征码

```
55 8B EC 6A 00 53 56 57 8B F2 8B D8 33 C0 55
```

`IN 22`

```
0042CB94  /$  55            PUSH EBP
0042CB95  |.  8BEC          MOV EBP,ESP
0042CB97  |.  6A 00         PUSH 0
0042CB99  |.  53            PUSH EBX
0042CB9A  |.  56            PUSH ESI
0042CB9B  |.  57            PUSH EDI                                 ;  Doit_ASP.0046446C
0042CB9C  |.  8BF2          MOV ESI,EDX
0042CB9E  |.  8BD8          MOV EBX,EAX
0042CBA0  |.  33C0          XOR EAX,EAX
0042CBA2  |.  55            PUSH EBP
```

修复代码

```
004651DA      E8 B579FCFF   CALL Doit_ASP.0042CB94
```

`F8`走过这个检查是否正确，蹦！程序出错了。 重新运行程序，执行到`004651DA`，重新修复这个`CALL`。

换个方法定位`CALL`，在`212r`中搜索 `CALL 00413120` ，观察附近代码辅助定位。

找到这里

```
004404D1  |.  8BC6          MOV EAX,ESI                              ;  ASPack.0043E920
004404D3  |.  BA 14054400   MOV EDX,ASPack.00440514                  ;  ASCII ".data"
004404D8  |.  E8 432CFDFF   CALL ASPack.00413120
004404DD  |>  33C0          XOR EAX,EAX
004404DF  |.  5A            POP EDX                                  ;  0012FE50
004404E0  |.  59            POP ECX                                  ;  0012FE50
004404E1  |.  59            POP ECX                                  ;  0012FE50
004404E2  |.  64:8910       MOV DWORD PTR FS:[EAX],EDX
```

可爱的字串啊

在`22`中搜索字串`.data`

找到

```
00466100  |.  BA 40614600   MOV EDX,Doit_ASP.00466140                ;  ASCII ".data"
00466105  |.  E8 7A4DFCFF   CALL Doit_ASP.0042AE84
```

修复代码

```
004651DA   .  E8 A55CFCFF   CALL Doit_ASP.0042AE84
```

`F8`走过这个检查是否正确，`OK`。

第四个`CALL`

`IN 212r`

```
0041350C  /$  33D2          XOR EDX,EDX
0041350E  |.  E8 39FBFFFF   CALL ASPack.0041304C
00413513  \.  C3            RETN
```

好像没啥特点，观察前后代码，

前面面的代码

```
004134F9      8D40 00       LEA EAX,DWORD PTR DS:[EAX]
004134FC   .  F640 34 40    TEST BYTE PTR DS:[EAX+34],40
00413500   .  0F95C1        SETNE CL
00413503   .  8A50 37       MOV DL,BYTE PTR DS:[EAX+37]
00413506   .  E8 65FFFFFF   CALL ASPack.00413470
0041350B   .  C3            RETN
```

看上去不错

提取特征码

```
F6 40 34 40 0F 95 C1 8A 50 37
```

替换常数

```
F6 40 ?? 40 0F 95 C1 8A 50 ??
```

`IN 22`

找到

```
0042B30C   .  F640 40 40    TEST BYTE PTR DS:[EAX+40],40
0042B310   .  0F95C1        SETNE CL
0042B313   .  8A50 47       MOV DL,BYTE PTR DS:[EAX+47]
0042B316   .  E8 65FFFFFF   CALL Doit_ASP.0042B280
0042B31B   .  C3            RETN

0042B31C  /$  33D2          XOR EDX,EDX
0042B31E  |.  E8 49FAFFFF   CALL Doit_ASP.0042AD6C
0042B323  \.  C3            RETN
```

修复代码

```
004651E8   .  E8 2F61FCFF   CALL Doit_ASP.0042B31C
```

`F8`走过这个检查是否正确，`OK`。

其他`CALL`的寻找参考前面的方法，全部搞定。

没发现因局部变量错误产生的异常，暂且不修改，运行不出错就可以了。

修复好的代码如下

```
004651A8   . /E9 01000000   JMP Doit_ASP.004651AE
004651AD     |90            NOP
004651AE   > \8B45 FC       MOV EAX,DWORD PTR SS:[EBP-4]
004651B1   .  8B80 04030000 MOV EAX,DWORD PTR DS:[EAX+304]
004651B7   .  8B40 58       MOV EAX,DWORD PTR DS:[EAX+58]
004651BA   .  33D2          XOR EDX,EDX                              ;  Doit_ASP.00420CA0
004651BC   .  E8 172FFBFF   CALL Doit_ASP.004180D8
004651C1   .  8D55 B0       LEA EDX,DWORD PTR SS:[EBP-50]
004651C4   .  A1 24C94600   MOV EAX,DWORD PTR DS:[46C924]
004651C9   .  E8 F23AFAFF   CALL Doit_ASP.00408CC0
004651CE   .  8B55 B0       MOV EDX,DWORD PTR SS:[EBP-50]
004651D1   .  8B45 FC       MOV EAX,DWORD PTR SS:[EBP-4]
004651D4   .  8B80 04030000 MOV EAX,DWORD PTR DS:[EAX+304]
004651DA   .  E8 A55CFCFF   CALL Doit_ASP.0042AE84
004651DF   .  8B45 FC       MOV EAX,DWORD PTR SS:[EBP-4]
004651E2   .  8B80 90030000 MOV EAX,DWORD PTR DS:[EAX+390]
004651E8   .  E8 2F61FCFF   CALL Doit_ASP.0042B31C
004651ED   .  B2 01         MOV DL,1
004651EF   .  8B45 FC       MOV EAX,DWORD PTR SS:[EBP-4]
004651F2   .  8B80 A0030000 MOV EAX,DWORD PTR DS:[EAX+3A0]
004651F8   .  E8 B35BFCFF   CALL Doit_ASP.0042ADB0
004651FD   .  B2 01         MOV DL,1
004651FF   .  8B45 FC       MOV EAX,DWORD PTR SS:[EBP-4]
00465202   .  8B80 9C030000 MOV EAX,DWORD PTR DS:[EAX+39C]
00465208   .  E8 A35BFCFF   CALL Doit_ASP.0042ADB0
0046520D   .  B2 01         MOV DL,1
0046520F   .  8B45 FC       MOV EAX,DWORD PTR SS:[EBP-4]
00465212   .  8B80 10040000 MOV EAX,DWORD PTR DS:[EAX+410]
00465218   .  E8 935BFCFF   CALL Doit_ASP.0042ADB0
0046521D   .  8B45 FC       MOV EAX,DWORD PTR SS:[EBP-4]
00465220   .  8B80 08040000 MOV EAX,DWORD PTR DS:[EAX+408]
00465226   .  C640 24 01    MOV BYTE PTR DS:[EAX+24],1
0046522A   .  EB 04         JMP SHORT Doit_ASP.00465230
```
