
# [原创]一个小游戏工具的注册算法分析

- 软件介绍：国产软件, Windows游戏工具, 简体中文, 317 KB
- 软件下载：`9ffK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3&6G2N6r3q4T1k6r3y4Q4x3X3g2$3K9i4m8Q4x3X3g2K6K9h3&6S2i4K6u0W2j5$3!0E0i4K6u0r3b7$3&6e0L8$3k6@1i4K6u0r3k6Y4u0W2k6h3y4W2L8r3I4@1L8$3!0D9i4K6u0W2P5X3W2H3`
- 保护方式：注册码 + 功能限制
- 破解工具：Win2000, PEiD, Ollydbg, W32DSM
- 破解目的：研究算法分析
- 作者声明：初学Crack，只是感兴趣，没有其他目的。失误之处敬请诸位大侠赐教！

---

## 破解过程

侦测：用 PEiD 查壳，无壳，Microsoft Visual C++ 6.0。

OllyDbg 打开软件，F9 运行，帮助(H) -> 注册(R) 输入：

注册用户名：`blackeyes`

注册码：`123456789`

点击 OK，没错误提示。

用 W32DSM 反汇编，发现 `UserName`、`RegisterCode`，以及 `RegQueryValueExA()`。

Ctrl+F2 重来，下断点 `bpx RegQueryValueExA`，F9 运行，拦截下来，注意看右边堆栈中的提示，F9 几次后看到 `RegQueryValueExA` 在读 `UserName` 与 `RegisterCode`，Ctrl+F9 返回程序 `0044157D` 处。

```
00441560  |.  57            PUSH EDI
00441561  |.  50            PUSH EAX                                 ; /pBufSize
00441562  |.  8D45 FC       LEA EAX,DWORD PTR SS:[EBP-4]             ; |
00441565  |.  50            PUSH EAX                                 ; |Buffer
00441566  |.  8D45 F8       LEA EAX,DWORD PTR SS:[EBP-8]             ; |
00441569  |.  50            PUSH EAX                                 ; |pValueType
0044156A  |.  6A 00         PUSH 0                                   ; |Reserved = NULL
0044156C  |.  FF75 0C       PUSH DWORD PTR SS:[EBP+C]                ; |ValueName
0044156F  |.  C745 08 04000>MOV DWORD PTR SS:[EBP+8],4               ; |
00441576  |.  56            PUSH ESI                                 ; |hKey
00441577  |.  FF15 0C604400 CALL DWORD PTR DS:[<&ADVAPI32.RegQueryVa>; \RegQueryValueExA
0044157D  |.  56            PUSH ESI                                 ; /hKey = 00000070      <==== 返回这
0044157E  |.  8BF8          MOV EDI,EAX                              ; |
00441580  |.  FF15 04604400 CALL DWORD PTR DS:[<&ADVAPI32.RegCloseKe>; \RegCloseKey
00441586  |.  85FF          TEST EDI,EDI
00441588  |.  5F            POP EDI
00441589  |.^ 75 CD         JNZ SHORT Freecell.00441558
0044158B  |.  8B45 FC       MOV EAX,DWORD PTR SS:[EBP-4]
0044158E  |.  EB 15         JMP SHORT Freecell.004415A5
00441590  |>  FFB1 90000000 PUSH DWORD PTR DS:[ECX+90]               ; /IniFileName
00441596  |.  FF75 10       PUSH DWORD PTR SS:[EBP+10]               ; |Default
00441599  |.  FF75 0C       PUSH DWORD PTR SS:[EBP+C]                ; |Key
0044159C  |.  FF75 08       PUSH DWORD PTR SS:[EBP+8]                ; |Section
0044159F  |.  FF15 3C624400 CALL DWORD PTR DS:[<&KERNEL32.GetPrivate>; \GetPrivateProfileIntA
004415A5  |>  5E            POP ESI
004415A6  |.  C9            LEAVE
004415A7  \.  C2 0C00       RETN 0C                                  ;  // 返回00406113
```

再 F8 单步几下，返回 `00406113`，再 F8 单步跟，很快就发现关键 `CALL` 及注册码比较的地方。

```
00405FCC   .  50            PUSH EAX
00405FCD   .  B9 B0994500   MOV ECX,Freecell.004599B0
00405FD2   .  E8 F9170300   CALL Freecell.004377D0
00405FD7   .  BB 01000000   MOV EBX,1                                ;  EBX=1
00405FDC   .  8BCE          MOV ECX,ESI
00405FDE   .  53            PUSH EBX
...
004060D2   .  0F95C1        SETNE CL
004060D5   .  888E C0000000 MOV BYTE PTR DS:[ESI+C0],CL
004060DB   .  E8 B0FCFFFF   CALL Freecell.00405D90
004060E0   .  68 CC414500   PUSH Freecell.004541CC                   ;  ASCII "UnknownUser"
004060E5   .  68 B0414500   PUSH Freecell.004541B0                   ;  ASCII "UserName"
004060EA   .  8D5424 18     LEA EDX,DWORD PTR SS:[ESP+18]
004060EE   .  68 58414500   PUSH Freecell.00454158                   ;  ASCII "Options"
004060F3   .  52            PUSH EDX
004060F4   .  8BCE          MOV ECX,ESI
004060F6   .  8BF8          MOV EDI,EAX
004060F8   .  E8 ADB40300   CALL Freecell.004415AA
004060FD   .  6A 00         PUSH 0
004060FF   .  68 BC414500   PUSH Freecell.004541BC                   ;  ASCII "RegisterCode"
00406104   .  68 58414500   PUSH Freecell.00454158                   ;  ASCII "Options"
00406109   .  8BCE          MOV ECX,ESI
0040610B   .  C74424 2C 000>MOV DWORD PTR SS:[ESP+2C],0
00406113   .  E8 26B40300   CALL Freecell.0044153E                   ;  EAX = RegCode
00406118   .  8BE8          MOV EBP,EAX                              ;  EBP = RegCode
0040611A   .  51            PUSH ECX
0040611B   .  8D4424 14     LEA EAX,DWORD PTR SS:[ESP+14]
0040611F   .  8BCC          MOV ECX,ESP
00406121   .  896424 18     MOV DWORD PTR SS:[ESP+18],ESP
00406125   .  50            PUSH EAX
00406126   .  E8 E2F40200   CALL Freecell.0043560D
0040612B   .  57            PUSH EDI
0040612C   .  E8 AFFCFFFF   CALL Freecell.00405DE0                   ; // 关键CALL, EAX=00405DE0(),  F7跟进
00406131   .  83C4 08       ADD ESP,8
00406134   .  3BC5          CMP EAX,EBP                              ; EAX==EBP?
00406136   .  75 08         JNZ SHORT Freecell.00406140
00406138   .  889E C1000000 MOV BYTE PTR DS:[ESI+C1],BL              ; BL=1, 已注册
0040613E   .  EB 07         JMP SHORT Freecell.00406147
00406140   >  C686 C1000000>MOV BYTE PTR DS:[ESI+C1],0               ; 0 = 未注册
00406147   >  68 E0020000   PUSH 2E0
0040614C   .  E8 6DFA0200   CALL Freecell.00435BBE
```

详细的注册码计算，入口参数为系统信息码和注册用户名，返回真的注册码。注册码计算与系统信息码无关！

```
00405DE0  /$  8B4424 08     MOV EAX,DWORD PTR SS:[ESP+8]             ;  username
00405DE4  |.  56            PUSH ESI
00405DE5  |.  57            PUSH EDI
00405DE6  |.  33FF          XOR EDI,EDI
00405DE8  |.  8B48 F8       MOV ECX,DWORD PTR DS:[EAX-8]             ;  len=strlen(username);
00405DEB  |.  BE 31D40000   MOV ESI,0D431                            ;  num=0x0D431;
00405DF0  |.  85C9          TEST ECX,ECX
00405DF2  |.  7E 34         JLE SHORT Freecell.00405E28              ;  for(i=0;i<len;i++){
00405DF4  |>  0FBE0407      /MOVSX EAX,BYTE PTR DS:[EDI+EAX]         ;  ch = username[i];
00405DF8  |.  35 35211414   |XOR EAX,14142135                        ;  EAX = ch^0x14142135;
00405DFD  |.  50            |PUSH EAX
00405DFE  |.  E8 E1AB0000   |CALL Freecell.004109E4                  ;  sub1(EAX);
00405E03  |.  83C4 04       |ADD ESP,4
00405E06  |.  E8 E6AB0000   |CALL Freecell.004109F1                  ;  EAX=sub2();
00405E0B  |.  8BCE          |MOV ECX,ESI
00405E0D  |.  C1E9 10       |SHR ECX,10
00405E10  |.  C1E6 10       |SHL ESI,10
00405E13  |.  0BCE          |OR ECX,ESI                              ;  ECX=(num<<16) | (num>>16);
00405E15  |.  03C1          |ADD EAX,ECX                             ;  EAX=EAX+ECX;
00405E17  |.  35 28181827   |XOR EAX,27181828                        ;  EAX^=0x27181828;
00405E1C  |.  47            |INC EDI
00405E1D  |.  8BF0          |MOV ESI,EAX                             ;  num=EAX
00405E1F  |.  8B4424 10     |MOV EAX,DWORD PTR SS:[ESP+10]
00405E23  |.  3B78 F8       |CMP EDI,DWORD PTR DS:[EAX-8]
00405E26  |.^ 7C CC         \JL SHORT Freecell.00405DF4              ;  }
00405E28  |>  68 15CD5B07   PUSH 75BCD15
00405E2D  |.  E8 B2AB0000   CALL Freecell.004109E4                   ;  sub1(0x75BCD15);
00405E32  |.  83C4 04       ADD ESP,4
00405E35  |.  BF 64000000   MOV EDI,64                               ;  for(i=0;i<100;i++) {
00405E3A  |>  E8 B2AB0000   /CALL Freecell.004109F1                  ;  EAX=sub2();
00405E3F  |.  8BD6          |MOV EDX,ESI                             ;  EDX=(num<<16) | (num>>16);
00405E41  |.  C1EA 10       |SHR EDX,10
00405E44  |.  C1E6 10       |SHL ESI,10
00405E47  |.  0BD6          |OR EDX,ESI
00405E49  |.  03C2          |ADD EAX,EDX                             ;  EAX=EAX+EDX;
00405E4B  |.  35 08053217   |XOR EAX,17320508                        ;  EAX^=0x17320508;
00405E50  |.  4F            |DEC EDI
00405E51  |.  8BF0          |MOV ESI,EAX                             ;  num = EAX;
00405E53  |.^ 75 E5         \JNZ SHORT Freecell.00405E3A             ;  }
00405E55  |.  8D4C24 10     LEA ECX,DWORD PTR SS:[ESP+10]
00405E59  |.  E8 3AFA0200   CALL Freecell.00435898
00405E5E  |.  8BC6          MOV EAX,ESI                              ;  return num;
```
