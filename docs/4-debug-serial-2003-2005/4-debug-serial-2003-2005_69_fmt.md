
# File splitters

> 原文：[`www.pediy.com/kssd/pediy08/pediy8-205.htm`](https://www.pediy.com/kssd/pediy08/pediy8-205.htm)

【破文标题】File splitters 一个 VB 程序的算法分析（初级）  

【破文作者】山村小子  

【作者邮箱】无  

【作者主页】暂无  

【破解工具】PEiD, W32DASM, UC32, OD  

【破解平台】Windows 2K&XP  

【软件名称】Easy File Splitter  

【软件大小】1.32M  

【原版下载】[`www.filesplitter.net/download.htm`](http://www.filesplitter.net/download.htm)  

【保护方式】序列号  

【软件简介】Easy File Splitter is a piece of beautiful software for splitting large file to smaller files by bytes, kilo bytes, mega bytes, lines. Join splitted files to the original file. Use Easy File Splitter to split files into pieces so that you can move them between systems on floppy disks, or email them out sharing with your friends over the Internet.  

【破解声明】我是一只小菜鸟，偶得一点心得，愿与大家分享：）  

---

1、直接用 `OD` 载入，输入用户名：`lhl8730` 假码：`1234-2222-1111`

2、下万能断点，按确定，取消断点，再按 `ALT+F9` 返回到程序领空，到这。。。

```
0040CAFA   .  50             PUSH EAX
0040CAFB   .  56             PUSH ESI
0040CAFC   .  8B16          MOV EDX,DWORD PTR DS:[ESI]
0040CAFE   .  FF92 A0000000 CALL DWORD PTR DS:[EDX+A0]
0040CB04   .  3BC3          CMP EAX,EBX                          ;  断在这，往下一直按 F8
0040CB06   .  DBE2          FCLEX
0040CB08   .  7D 12         JGE SHORT 1.0040CB1C
```

。。。。。。。中间省略了一部分代码。。。。。

```
0040CD12   .  FF15 48104000 CALL DWORD PTR DS:[<&msvbvm60.__vbaFreeO>;  msvbvm60.__vbaFreeObjList
0040CD18   .  8D45 AC       LEA EAX,DWORD PTR SS:[EBP-54]
0040CD1B   .  8D4D BC       LEA ECX,DWORD PTR SS:[EBP-44]
0040CD1E   .  50             PUSH EAX
0040CD1F   .  51             PUSH ECX
0040CD20   .  6A 02         PUSH 2
0040CD22   .  FFD3          CALL EBX
0040CD24   .  83C4 2C       ADD ESP,2C
0040CD27   .  E8 A4D60000   CALL 1.0041A3D0                       ;  算法 CALL，跟进
0040CD2C   .  66:85C0       TEST AX,AX
0040CD2F   .  0F84 B4000000 JE 1.0040CDE9                         ;  暴破点
0040CD35   .  B9 04000280   MOV ECX,80020004
0040CD3A   .  B8 0A000000   MOV EAX,0A
0040CD3F   .  894D 94       MOV DWORD PTR SS:[EBP-6C],ECX
0040CD42   .  894D A4       MOV DWORD PTR SS:[EBP-5C],ECX
0040CD45   .  894D B4       MOV DWORD PTR SS:[EBP-4C],ECX
0040CD48   .  8D95 7CFFFFFF LEA EDX,DWORD PTR SS:[EBP-84]
0040CD4E   .  8D4D BC       LEA ECX,DWORD PTR SS:[EBP-44]
0040CD51   .  8945 8C       MOV DWORD PTR SS:[EBP-74],EAX
0040CD54   .  8945 9C       MOV DWORD PTR SS:[EBP-64],EAX
0040CD57   .  8945 AC       MOV DWORD PTR SS:[EBP-54],EAX
0040CD5A   .  C745 84 04884>MOV DWORD PTR SS:[EBP-7C],1.00408804    ;  UNICODE "Registration Succeeded!"
0040CD61   .  C785 7CFFFFFF>MOV DWORD PTR SS:[EBP-84],8
0040CD6B   .  FF15 20124000 CALL DWORD PTR DS:[<&msvbvm60.__vbaVarDu>;  msvbvm60.__vbaVarDup
```

。。。。。跟进 CALL 1.0041A3D0 到这

```
0041A3D0   $  55             PUSH EBP
0041A3D1   .  8BEC          MOV EBP,ESP
0041A3D3   .  83EC 14       SUB ESP,14
0041A3D6   .  68 F61A4000   PUSH <JMP.&msvbvm60.__vbaExceptHandler>  ;  SE 处理程序安装
0041A3DB   .  64:A1 0000000>MOV EAX,DWORD PTR FS:[0]
0041A3E1   .  50             PUSH EAX
0041A3E2   .  64:8925 00000>MOV DWORD PTR FS:[0],ESP
0041A3E9   .  81EC 84000000 SUB ESP,84
0041A3EF   .  53             PUSH EBX
0041A3F0   .  56             PUSH ESI
0041A3F1   .  57             PUSH EDI
0041A3F2   .  8965 EC       MOV DWORD PTR SS:[EBP-14],ESP
0041A3F5   .  C745 F0 281A4>MOV DWORD PTR SS:[EBP-10],1.00401A28
0041A3FC   .  33C0          XOR EAX,EAX
0041A3FE   .  8945 F4       MOV DWORD PTR SS:[EBP-C],EAX
0041A401   .  8945 F8       MOV DWORD PTR SS:[EBP-8],EAX
0041A404   .  8945 DC       MOV DWORD PTR SS:[EBP-24],EAX
0041A407   .  8945 CC       MOV DWORD PTR SS:[EBP-34],EAX
0041A40A   .  8945 BC       MOV DWORD PTR SS:[EBP-44],EAX
0041A40D   .  8945 B4       MOV DWORD PTR SS:[EBP-4C],EAX
0041A410   .  8945 A4       MOV DWORD PTR SS:[EBP-5C],EAX
0041A413   .  8945 94       MOV DWORD PTR SS:[EBP-6C],EAX
0041A416   .  8945 84       MOV DWORD PTR SS:[EBP-7C],EAX
0041A419   .  6A 01         PUSH 1
0041A41B   .  FF15 A4104000 CALL DWORD PTR DS:[<&msvbvm60.__vbaOnErr>;  msvbvm60.__vbaOnError
0041A421   .  E8 BA020000   CALL 1.0041A6E0
0041A426   .  8BD0          MOV EDX,EAX
0041A428   .  8D4D DC       LEA ECX,DWORD PTR SS:[EBP-24]
0041A42B   .  8B1D 50124000 MOV EBX,DWORD PTR DS:[<&msvbvm60.__vbaSt>;  msvbvm60.__vbaStrMove
0041A431   .  FFD3          CALL EBX                              ;  <&msvbvm60.__vbaStrMove>
0041A433   .  E8 E8030000   CALL 1.0041A820
0041A438   .  8BD0          MOV EDX,EAX
0041A43A   .  8D4D BC       LEA ECX,DWORD PTR SS:[EBP-44]
0041A43D   .  FFD3          CALL EBX
0041A43F   .  8D45 DC       LEA EAX,DWORD PTR SS:[EBP-24]
0041A442   .  8945 8C       MOV DWORD PTR SS:[EBP-74],EAX
0041A445   .  C745 84 08400>MOV DWORD PTR SS:[EBP-7C],4008
0041A44C   .  8D4D 84       LEA ECX,DWORD PTR SS:[EBP-7C]
0041A44F   .  51             PUSH ECX
0041A450   .  8D55 A4       LEA EDX,DWORD PTR SS:[EBP-5C]
0041A453   .  52             PUSH EDX
0041A454   .  8B3D CC104000 MOV EDI,DWORD PTR DS:[<&msvbvm60.rtcTrim>;  msvbvm60.rtcTrimVar
0041A45A   .  FFD7          CALL EDI                              ;  <&msvbvm60.rtcTrimVar>
0041A45C   .  8D45 A4       LEA EAX,DWORD PTR SS:[EBP-5C]
0041A45F   .  50             PUSH EAX
0041A460   .  FF15 30104000 CALL DWORD PTR DS:[<&msvbvm60.__vbaStrVa>;  msvbvm60.__vbaStrVarMove
0041A466   .  8BD0          MOV EDX,EAX
0041A468   .  8D4D DC       LEA ECX,DWORD PTR SS:[EBP-24]
0041A46B   .  FFD3          CALL EBX
0041A46D   .  8D4D A4       LEA ECX,DWORD PTR SS:[EBP-5C]
0041A470   .  8B35 24104000 MOV ESI,DWORD PTR DS:[<&msvbvm60.__vbaFr>;  msvbvm60.__vbaFreeVar
0041A476   .  FFD6          CALL ESI                              ;  <&msvbvm60.__vbaFreeVar>
0041A478   .  8D4D BC       LEA ECX,DWORD PTR SS:[EBP-44]
0041A47B   .  894D 8C       MOV DWORD PTR SS:[EBP-74],ECX
0041A47E   .  C745 84 08400>MOV DWORD PTR SS:[EBP-7C],4008
0041A485   .  8D55 84       LEA EDX,DWORD PTR SS:[EBP-7C]
0041A488   .  52             PUSH EDX
0041A489   .  8D45 A4       LEA EAX,DWORD PTR SS:[EBP-5C]
0041A48C   .  50             PUSH EAX
0041A48D   .  FFD7          CALL EDI
0041A48F   .  8D4D A4       LEA ECX,DWORD PTR SS:[EBP-5C]
0041A492   .  51             PUSH ECX
0041A493   .  FF15 30104000 CALL DWORD PTR DS:[<&msvbvm60.__vbaStrVa>;  msvbvm60.__vbaStrVarMove
0041A499   .  8BD0          MOV EDX,EAX
0041A49B   .  8D4D BC       LEA ECX,DWORD PTR SS:[EBP-44]
0041A49E   .  FFD3          CALL EBX
0041A4A0   .  8D4D A4       LEA ECX,DWORD PTR SS:[EBP-5C]
0041A4A3   .  FFD6          CALL ESI
0041A4A5   .  8B55 BC       MOV EDX,DWORD PTR SS:[EBP-44]
0041A4A8   .  52             PUSH EDX
0041A4A9   .  8B3D 2C104000 MOV EDI,DWORD PTR DS:[<&msvbvm60.__vbaLe>;  msvbvm60.__vbaLenBstr
0041A4AF   .  FFD7          CALL EDI                              ;  <&msvbvm60.__vbaLenBstr>
0041A4B1   .  33DB          XOR EBX,EBX
0041A4B3   .  83F8 05       CMP EAX,5                             ;  比较注册码长度是否为 5
0041A4B6   .  0F9DC3        SETGE BL
0041A4B9   .  8B45 DC       MOV EAX,DWORD PTR SS:[EBP-24]
0041A4BC   .  50             PUSH EAX
0041A4BD   .  FFD7          CALL EDI
0041A4BF   .  33C9          XOR ECX,ECX
0041A4C1   .  83F8 03       CMP EAX,3                             ;  比较用户名长度是否为 3
0041A4C4   .  0F9DC1        SETGE CL
0041A4C7   .  85D9          TEST ECX,EBX
0041A4C9   .  75 17         JNZ SHORT 1.0041A4E2                  ;  都不是跳走
0041A4CB   .  C745 C8 00000>MOV DWORD PTR SS:[EBP-38],0
```



# 汇编代码分析

## 代码片段

```
0041A4D2   .  FF15 98104000 CALL DWORD PTR DS:[<&msvbvm60.__vbaExitP>;  msvbvm60.__vbaExitProc  

0041A4D8   .  68 C2A64100   PUSH 1.0041A6C2  

0041A4DD   .  E9 C6010000   JMP 1.0041A6A8  

0041A4E2   >  C745 8C 0C9A4>MOV DWORD PTR SS:[EBP-74],1.00409A0C  

0041A4E9   .  C745 84 08000>MOV DWORD PTR SS:[EBP-7C],8  

0041A4F0   .  8D55 84       LEA EDX,DWORD PTR SS:[EBP-7C]  

0041A4F3   .  8D4D A4       LEA ECX,DWORD PTR SS:[EBP-5C]  

0041A4F6   .  FF15 20124000 CALL DWORD PTR DS:[<&msvbvm60.__vbaVarDu>;  msvbvm60.__vbaVarDup  

0041A4FC   .  6A 00         PUSH 0  

0041A4FE   .  6A FF         PUSH -1  

0041A500   .  8D55 A4       LEA EDX,DWORD PTR SS:[EBP-5C]  

0041A503   .  52            PUSH EDX  

0041A504   .  8B45 BC       MOV EAX,DWORD PTR SS:[EBP-44]  

0041A507   .  50            PUSH EAX  

0041A508   .  8D4D 94       LEA ECX,DWORD PTR SS:[EBP-6C]  

0041A50B   .  51            PUSH ECX  

0041A50C   .  FF15 5C114000 CALL DWORD PTR DS:[<&msvbvm60.rtcSplit>] ;  msvbvm60.rtcSplit  

0041A512   .  8D55 94       LEA EDX,DWORD PTR SS:[EBP-6C]            ;  这一部分是看有几个“-”  

0041A515   .  8D4D CC       LEA ECX,DWORD PTR SS:[EBP-34]  

0041A518   .  FF15 1C104000 CALL DWORD PTR DS:[<&msvbvm60.__vbaVarMo>;  msvbvm60.__vbaVarMove  

0041A51E   .  8D4D A4       LEA ECX,DWORD PTR SS:[EBP-5C]  

0041A521   .  FFD6          CALL ESI  

0041A523   .  8D55 CC       LEA EDX,DWORD PTR SS:[EBP-34]  

0041A526   .  52            PUSH EDX  

0041A527   .  8B1D D8104000 MOV EBX,DWORD PTR DS:[<&msvbvm60.__vbaRe>;  msvbvm60.__vbaRefVarAry  

0041A52D   .  FFD3          CALL EBX                            ;  <&msvbvm60.__vbaRefVarAry>  

0041A52F   .  8B00          MOV EAX,DWORD PTR DS:[EAX]  

0041A531   .  50            PUSH EAX  

0041A532   .  6A 01         PUSH 1  

0041A534   .  FF15 30114000 CALL DWORD PTR DS:[<&msvbvm60.__vbaLboun>;  msvbvm60.__vbaLbound  

0041A53A   .  8BF8          MOV EDI,EAX  

0041A53C   .  8D4D CC       LEA ECX,DWORD PTR SS:[EBP-34]  

0041A53F   .  51            PUSH ECX  

0041A540   .  FFD3          CALL EBX  

0041A542   .  8B10          MOV EDX,DWORD PTR DS:[EAX]  

0041A544   .  52            PUSH EDX  

0041A545   .  6A 01         PUSH 1  

0041A547   .  FF15 90114000 CALL DWORD PTR DS:[<&msvbvm60.__vbaUboun>;  msvbvm60.__vbaUbound  

0041A54D   .  2BC7          SUB EAX,EDI  

0041A54F   .  0F80 82010000 JO 1.0041A6D7  

0041A555   .  83C0 01       ADD EAX,1  

0041A558   .  0F80 79010000 JO 1.0041A6D7  

0041A55E   .  83F8 03       CMP EAX,3  

0041A561   .  74 17         JE SHORT 1.0041A57A                ;  注册码是否由三部分组成，不是就 over.  

0041A563   .  C745 C8 00000>MOV DWORD PTR SS:[EBP-38],0  

0041A56A   .  FF15 98104000 CALL DWORD PTR DS:[<&msvbvm60.__vbaExitP>;  msvbvm60.__vbaExitProc  

0041A570   .  68 C2A64100   PUSH 1.0041A6C2  

0041A575     E9 2E010000   JMP 1.0041A6A8  

0041A57A   >  8D45 CC       LEA EAX,DWORD PTR SS:[EBP-34]  

0041A57D   .  50            PUSH EAX  

0041A57E   .  FFD3          CALL EBX  

0041A580   .  8B08          MOV ECX,DWORD PTR DS:[EAX]  

0041A582   .  51            PUSH ECX  

0041A583   .  6A 01         PUSH 1  

0041A585   .  FF15 30114000 CALL DWORD PTR DS:[<&msvbvm60.__vbaLboun>;  msvbvm60.__vbaLbound  

0041A58B   .  8945 8C       MOV DWORD PTR SS:[EBP-74],EAX  

0041A58E   .  B9 03000000   MOV ECX,3  

0041A593   .  894D 84       MOV DWORD PTR SS:[EBP-7C],ECX  

0041A596   .  83EC 10       SUB ESP,10  

0041A599   .  8BD4          MOV EDX,ESP  

0041A59B   .  890A          MOV DWORD PTR DS:[EDX],ECX  

0041A59D   .  8B4D 88       MOV ECX,DWORD PTR SS:[EBP-78]  

0041A5A0   .  894A 04       MOV DWORD PTR DS:[EDX+4],ECX  

0041A5A3   .  8942 08       MOV DWORD PTR DS:[EDX+8],EAX  

0041A5A6   .  8B45 90       MOV EAX,DWORD PTR SS:[EBP-70]  

0041A5A9   .  8942 0C       MOV DWORD PTR DS:[EDX+C],EAX  

0041A5AC   .  6A 01         PUSH 1  

0041A5AE   .  8D4D CC       LEA ECX,DWORD PTR SS:[EBP-34]  

0041A5B1   .  51            PUSH ECX  

0041A5B2   .  8D55 A4       LEA EDX,DWORD PTR SS:[EBP-5C]  

0041A5B5   .  52            PUSH EDX  

0041A5B6   .  8B3D C0104000 MOV EDI,DWORD PTR DS:[<&msvbvm60.__vbaVa>;  msvbvm60.__vbaVarIndexLoad  

0041A5BC   .  FFD7          CALL EDI                            ;  <&msvbvm60.__vbaVarIndexLoad>  

0041A5BE   .  83C4 1C       ADD ESP,1C  

0041A5C1   .  50            PUSH EAX  

0041A5C2   .  FF15 7C124000 CALL DWORD PTR DS:[<&msvbvm60.__vbaI4Err>;  msvbvm60.__vbaI4ErrVar  

0041A5C8   .  8945 C4       MOV DWORD PTR SS:[EBP-3C],EAX        ;  得假码的第一部分十六进制值，我这是 4d2  

0041A5CB   .  8D4D A4       LEA ECX,DWORD PTR SS:[EBP-5C]  

0041A5CE   .  FFD6          CALL ESI  

0041A5D0   .  8D45 CC       LEA EAX,DWORD PTR SS:[EBP-34]  

0041A5D3   .  50            PUSH EAX  

0041A5D4   .  FFD3          CALL EBX  

0041A5D6   .  8B08          MOV ECX,DWORD PTR DS:[EAX]  

0041A5D8   .  51            PUSH ECX  

0041A5D9   .  6A 01         PUSH 1  

0041A5DB   .  FF15 90114000 CALL DWORD PTR DS:[<&msvbvm60.__vbaUboun>;  msvbvm60.__vbaUbound  

0041A5E1   .  8945 8C       MOV DWORD PTR SS:[EBP-74],EAX  

0041A5E4   .  B9 03000000   MOV ECX,3  

0041A5E9   .  894D 84       MOV DWORD PTR SS:[EBP-7C],ECX  

0041A5EC   .  83EC 10       SUB ESP,10  

0041A5EF   .  8BD4          MOV EDX,ESP  

0041A5F1   .  890A          MOV DWORD PTR DS:[EDX],ECX  

0041A5F3   .  8B4D 88       MOV ECX,DWORD PTR SS:[EBP-78]  

0041A5F6   .  894A 04       MOV DWORD PTR DS:[EDX+4],ECX  

0041A5F9   .  8942 08       MOV DWORD PTR DS:[EDX+8],EAX  

0041A5FC   .  8B45 90       MOV EAX,DWORD PTR SS:[EBP-70]  

0041A5FF   .  8942 0C       MOV DWORD PTR DS:[EDX+C],EAX  

0041A602   .  6A 01         PUSH 1  

0041A604   .  8D4D CC       LEA ECX,DWORD PTR SS:[EBP-34]  

0041A607   .  51            PUSH ECX  

0041A608   .  8D55 A4       LEA EDX,DWORD PTR SS:[EBP-5C]  

0041A60B   .  52            PUSH EDX  

0041A60C   .  FFD7          CALL EDI  

0041A60E   .  83C4 1C       ADD ESP,1C  

0041A611   .  50            PUSH EAX  

0041A612   .  FF15 7C124000 CALL DWORD PTR DS:[<&msvbvm60.__vbaI4Err>;  msvbvm60.__vbaI4ErrVar  

0041A618   .  8BF8          MOV EDI,EAX                      ;  得假码的第三部分十六进制值，我这是 457  

0041A61A   .  8D4D A4       LEA ECX,DWORD PTR SS:[EBP-5C]  

0041A61D   .  FFD6          CALL ESI  

0041A61F   .  8B45 BC       MOV EAX,DWORD PTR SS:[EBP-44]  

0041A622   .  50            PUSH EAX  

0041A623   .  57            PUSH EDI  

0041A624   .  8B4D C4       MOV ECX,DWORD PTR SS:[EBP-3C]  

0041A627   .  51            PUSH ECX  

0041A628   .  8B55 DC       MOV EDX,DWORD PTR SS:[EBP-24]  

0041A62B   .  52            PUSH EDX  

0041A62C   .  E8 BFFAFFFF   CALL 1.0041A0F0                   ;  关键 CALL,跟进  

0041A631 >  .  8BD0          MOV EDX,EAX                      ;  可在这里做内存注册机。  

0041A633   .  8D4D B4       LEA ECX,DWORD PTR SS:[EBP-4C]  

0041A636   .  FF15 50124000 CALL DWORD PTR DS:[<&msvbvm60.__vbaStrMo>;  msvbvm60.__vbaStrMove  

0041A63C   .  50            PUSH EAX  

0041A63D   .  FF15 0C114000 CALL DWORD PTR DS:[<&msvbvm60.__vbaStrCm>;  msvbvm60.__vbaStrCmp  

0041A643   .  8BF0          MOV ESI,EAX  

0041A645   .  F7DE          NEG ESI  

0041A647   .  1BF6          SBB ESI,ESI  

0041A649   .  46            INC ESI  

0041A64A   .  F7DE          NEG ESI  

0041A64C   .  8D4D B4       LEA ECX,DWORD PTR SS:[EBP-4C]  

0041A64F   .  FF15 80124000 CALL DWORD PTR DS:[<&msvbvm60.__vbaFreeS>;  msvbvm60.__vbaFreeStr  

0041A655   .  66:85F6       TEST SI,SI  

0041A658   .  74 19         JE SHORT 1.0041A673                ;  

0041A65A   .  83C8 FF       OR EAX,FFFFFFFF  

0041A65D   .  8945 C8       MOV DWORD PTR SS:[EBP-38],EAX  

0041A660   .  66:A3 44C0410>MOV WORD PTR DS:[41C044],AX  
```



```
0041A666   .  FF15 98104000 CALL DWORD PTR DS:[<&msvbvm60.__vbaExitP>;  msvbvm60.__vbaExitProc  

0041A66C   .  68 C2A64100   PUSH 1.0041A6C2  

0041A671   .  EB 35         JMP SHORT 1.0041A6A8  

0041A673   >  33C0          XOR EAX,EAX  

0041A675   .  66:A3 44C0410>MOV WORD PTR DS:[41C044],AX  

0041A67B   .  8945 C8       MOV DWORD PTR SS:[EBP-38],EAX  

0041A67E   .  FF15 98104000 CALL DWORD PTR DS:[<&msvbvm60.__vbaExitP>;  msvbvm60.__vbaExitProc  

0041A684   .  68 C2A64100   PUSH 1.0041A6C2  

0041A689   .  EB 1D         JMP SHORT 1.0041A6A8  

0041A68B   .  8D4D B4       LEA ECX,DWORD PTR SS:[EBP-4C]  

0041A68E   .  FF15 80124000 CALL DWORD PTR DS:[<&msvbvm60.__vbaFreeS>;  msvbvm60.__vbaFreeStr  

0041A694   .  8D45 94       LEA EAX,DWORD PTR SS:[EBP-6C]  

0041A697   .  50            PUSH EAX  

0041A698   .  8D4D A4       LEA ECX,DWORD PTR SS:[EBP-5C]  

0041A69B   .  51            PUSH ECX  

0041A69C   .  6A 02        PUSH 2  

0041A69E   .  FF15 3C104000 CALL DWORD PTR DS:[<&msvbvm60.__vbaFreeV>;  msvbvm60.__vbaFreeVarList  

0041A6A4   .  83C4 0C       ADD ESP,0C  

0041A6A7   .  C3            RETN  

0041A6A8   >  8D4D DC       LEA ECX,DWORD PTR SS:[EBP-24]  

0041A6AB   .  8B35 80124000 MOV ESI,DWORD PTR DS:[<&msvbvm60.__vbaFr>;  msvbvm60.__vbaFreeStr  

0041A6B1   .  FFD6          CALL ESI                          ;  <&msvbvm60.__vbaFreeStr>  

0041A6B3   .  8D4D CC       LEA ECX,DWORD PTR SS:[EBP-34]  

0041A6B6   .  FF15 24104000 CALL DWORD PTR DS:[<&msvbvm60.__vbaFreeV>;  msvbvm60.__vbaFreeVar  

0041A6BC   .  8D4D BC       LEA ECX,DWORD PTR SS:[EBP-44]  

0041A6BF   .  FFD6          CALL ESI  

0041A6C1   .  C3            RETN  

0041A6C2   .  66:8B45 C8    MOV AX,WORD PTR SS:[EBP-38]  

0041A6C6   .  8B4D E4       MOV ECX,DWORD PTR SS:[EBP-1C]  

0041A6C9   .  64:890D 00000>MOV DWORD PTR FS:[0],ECX  

0041A6D0   .  5F            POP EDI  

0041A6D1   .  5E            POP ESI  

0041A6D2   .  5B            POP EBX  

0041A6D3   .  8BE5          MOV ESP,EBP  

0041A6D5   .  5D            POP EBP  

0041A6D6   .  C3            RETN  
```

跟进关键 `CALL 1.0041A0F0` 到这

```
0041A0F0   $  55            PUSH EBP  

0041A0F1   .  8BEC          MOV EBP,ESP  

0041A0F3   .  83EC 0C       SUB ESP,0C  

0041A0F6   .  68 F61A4000   PUSH <JMP.&msvbvm60.__vbaExceptHandler>  ;  SE 处理程序安装  

0041A0FB   .  64:A1 0000000>MOV EAX,DWORD PTR FS:[0]  

0041A101   .  50            PUSH EAX  

0041A102   .  64:8925 00000>MOV DWORD PTR FS:[0],ESP  

0041A109   .  83EC 4C       SUB ESP,4C  

0041A10C   .  53            PUSH EBX  

0041A10D   .  56            PUSH ESI  

0041A10E   .  57            PUSH EDI  

0041A10F   .  8965 F4       MOV DWORD PTR SS:[EBP-C],ESP  

0041A112   .  C745 F8 081A4>MOV DWORD PTR SS:[EBP-8],1.00401A08  

0041A119   .  8B55 08       MOV EDX,DWORD PTR SS:[EBP+8]  

0041A11C   .  33C0          XOR EAX,EAX  

0041A11E   .  8D4D E0       LEA ECX,DWORD PTR SS:[EBP-20]  

0041A121   .  8945 E8       MOV DWORD PTR SS:[EBP-18],EAX  

0041A124   .  8945 E4       MOV DWORD PTR SS:[EBP-1C],EAX  

0041A127   .  8945 E0       MOV DWORD PTR SS:[EBP-20],EAX  

0041A12A   .  8945 DC       MOV DWORD PTR SS:[EBP-24],EAX  

0041A12D   .  8945 D8       MOV DWORD PTR SS:[EBP-28],EAX  

0041A130   .  8945 D4       MOV DWORD PTR SS:[EBP-2C],EAX  

0041A133   .  8945 D0       MOV DWORD PTR SS:[EBP-30],EAX  

0041A136   .  8945 CC       MOV DWORD PTR SS:[EBP-34],EAX  

0041A139   .  8945 C8       MOV DWORD PTR SS:[EBP-38],EAX  

0041A13C   .  8945 B8       MOV DWORD PTR SS:[EBP-48],EAX  

0041A13F   .  8945 A8       MOV DWORD PTR SS:[EBP-58],EAX  

0041A142   .  FF15 DC114000 CALL DWORD PTR DS:[<&msvbvm60.__vbaStrCo>;  msvbvm60.__vbaStrCopy  

0041A148   .  8B45 10       MOV EAX,DWORD PTR SS:[EBP+10]  

0041A14B   .  8B7D 0C       MOV EDI,DWORD PTR SS:[EBP+C]  

0041A14E   .  8B4D E0       MOV ECX,DWORD PTR SS:[EBP-20]  

0041A151   .  50            PUSH EAX  

0041A152   .  57            PUSH EDI  

0041A153   .  51            PUSH ECX  

0041A154   .  E8 A7FCFFFF   CALL 1.00419E00                ;  算法 CALL，跟进  

0041A159   .  8945 C0       MOV DWORD PTR SS:[EBP-40],EAX  

0041A15C   .  8D55 B8       LEA EDX,DWORD PTR SS:[EBP-48]  

0041A15F   .  8D45 A8       LEA EAX,DWORD PTR SS:[EBP-58]  

0041A162   .  52            PUSH EDX  

0041A163   .  50            PUSH EAX  

0041A164   .  C745 B8 08000>MOV DWORD PTR SS:[EBP-48],8  

0041A16B   .  FF15 CC104000 CALL DWORD PTR DS:[<&msvbvm60.rtcTrimVar>;  msvbvm60.rtcTrimVar  

0041A171   .  8D4D A8       LEA ECX,DWORD PTR SS:[EBP-58]  

0041A174   .  51            PUSH ECX  

0041A175   .  FF15 30104000 CALL DWORD PTR DS:[<&msvbvm60.__vbaStrVa>;  msvbvm60.__vbaStrVarMove  

0041A17B   .  8B35 50124000 MOV ESI,DWORD PTR DS:[<&msvbvm60.__vbaSt>;  msvbvm60.__vbaStrMove  

0041A181   .  8BD0          MOV EDX,EAX  

0041A183   .  8D4D E4       LEA ECX,DWORD PTR SS:[EBP-1C]  

0041A186   .  FFD6          CALL ESI                      ;  <&msvbvm60.__vbaStrMove>  

0041A188   .  8D55 A8       LEA EDX,DWORD PTR SS:[EBP-58]  

0041A18B   .  8D45 B8       LEA EAX,DWORD PTR SS:[EBP-48]  

0041A18E   .  52            PUSH EDX  

0041A18F   .  50            PUSH EAX  

0041A190   .  6A 02        PUSH 2  

0041A192   .  FF15 3C104000 CALL DWORD PTR DS:[<&msvbvm60.__vbaFreeV>;  msvbvm60.__vbaFreeVarList  

0041A198   .  8B1D 18104000 MOV EBX,DWORD PTR DS:[<&msvbvm60.__vbaSt>;  msvbvm60.__vbaStrI4  

0041A19E   .  83C4 0C       ADD ESP,0C  

0041A1A1   .  57            PUSH EDI  

0041A1A2   .  FFD3          CALL EBX                      ;  <&msvbvm60.__vbaStrI4>  

0041A1A4   .  8BD0          MOV EDX,EAX  

0041A1A6   .  8D4D DC       LEA ECX,DWORD PTR SS:[EBP-24]  

0041A1A9   .  FFD6          CALL ESI  

0041A1AB   .  8B3D 60104000 MOV EDI,DWORD PTR DS:[<&msvbvm60.__vbaSt>;  msvbvm60.__vbaStrCat  

0041A1B1   .  50            PUSH EAX  

0041A1B2   .  68 0C9A4000   PUSH 1.00409A0C  

0041A1B7   .  FFD7          CALL EDI                      ;  <&msvbvm60.__vbaStrCat>  

0041A1B9   .  8BD0          MOV EDX,EAX  

0041A1BB   .  8D4D D8       LEA ECX,DWORD PTR SS:[EBP-28]  

0041A1BE   .  FFD6          CALL ESI  

0041A1C0   .  8B4D E4       MOV ECX,DWORD PTR SS:[EBP-1C]  

0041A1C3   .  50            PUSH EAX  

0041A1C4   .  51            PUSH ECX  

0041A1C5   .  E8 D6000000   CALL 1.0041A2A0               ;  重要的关键 CALL3，跟进  

0041A1CA   .  50            PUSH EAX                    ;  把字符串 S 的所有字符的 ASC 码加起来为 c43  

0041A1CB   .  FFD3          CALL EBX  

0041A1CD   .  8BD0          MOV EDX,EAX                 ;  转为十进值为 3139  

0041A1CF   .  8D4D D4       LEA ECX,DWORD PTR SS:[EBP-2C]  

0041A1D2   .  FFD6          CALL ESI  

0041A1D4   .  50            PUSH EAX  

0041A1D5   .  FFD7          CALL EDI  

0041A1D7   .  8BD0          MOV EDX,EAX  

0041A1D9   .  8D4D D0       LEA ECX,DWORD PTR SS:[EBP-30]  

0041A1DC   .  FFD6          CALL ESI  

0041A1DE   .  50            PUSH EAX  

0041A1DF   .  68 0C9A4000   PUSH 1.00409A0C  

0041A1E4   .  FFD7          CALL EDI  

0041A1E6   .  8BD0          MOV EDX,EAX  

0041A1E8   .  8D4D CC       LEA ECX,DWORD PTR SS:[EBP-34]  

0041A1EB   .  FFD6          CALL ESI  

0041A1ED   .  8B55 10       MOV EDX,DWORD PTR SS:[EBP+10]  

0041A1F0   .  50            PUSH EAX  

0041A1F1   .  52            PUSH EDX  

0041A1F2   .  FFD3          CALL EBX  

0041A1F4   .  8BD0          MOV EDX,EAX  

0041A1F6   .  8D4D C8       LEA ECX,DWORD PTR SS:[EBP-38]  

0041A1F9   .  FFD6          CALL ESI  

0041A1FB   .  50            PUSH EAX  

0041A1FC   .  FFD7          CALL EDI  

0041A1FE   .  8BD0          MOV EDX,EAX                 ;  把第一部分与 3139 及第三部分连起来即为注册码。
```



```
0041A200   .  8D4D E8       LEA ECX,DWORD PTR SS:[EBP-18]  
0041A203   .  FFD6          CALL ESI  
0041A205   .  8D45 C8       LEA EAX,DWORD PTR SS:[EBP-38]  
0041A208   .  8D4D CC       LEA ECX,DWORD PTR SS:[EBP-34]  
0041A20B   .  50           PUSH EAX  
0041A20C   .  8D55 D0       LEA EDX,DWORD PTR SS:[EBP-30]  
0041A20F   .  51           PUSH ECX  
0041A210   .  8D45 D4       LEA EAX,DWORD PTR SS:[EBP-2C]  
0041A213   .  52           PUSH EDX  
0041A214   .  8D4D D8       LEA ECX,DWORD PTR SS:[EBP-28]  
0041A217   .  50           PUSH EAX  
0041A218   .  8D55 DC       LEA EDX,DWORD PTR SS:[EBP-24]  
0041A21B   .  51           PUSH ECX  
0041A21C   .  52           PUSH EDX  
0041A21D   .  6A 06        PUSH 6  
0041A21F   .  FF15 E8114000 CALL DWORD PTR DS:[<&msvbvm60.__vbaFreeS>;  msvbvm60.__vbaFreeStrList  
0041A225   .  83C4 1C       ADD ESP,1C  
0041A228   .  68 83A24100   PUSH 1.0041A283  
0041A22D   .  EB 43        JMP SHORT 1.0041A272  
0041A22F   .  F645 FC 04   TEST BYTE PTR SS:[EBP-4],4  
0041A233   .  74 09        JE SHORT 1.0041A23E  
0041A235   .  8D4D E8       LEA ECX,DWORD PTR SS:[EBP-18]  
0041A238   .  FF15 80124000 CALL DWORD PTR DS:[<&msvbvm60.__vbaFreeS>;  msvbvm60.__vbaFreeStr  
0041A23E   >  8D45 C8       LEA EAX,DWORD PTR SS:[EBP-38]  
0041A241   .  8D4D CC       LEA ECX,DWORD PTR SS:[EBP-34]  
0041A244   .  50           PUSH EAX  
0041A245   .  8D55 D0       LEA EDX,DWORD PTR SS:[EBP-30]  
0041A248   .  51           PUSH ECX  
0041A249   .  8D45 D4       LEA EAX,DWORD PTR SS:[EBP-2C]  
0041A24C   .  52           PUSH EDX  
0041A24D   .  8D4D D8       LEA ECX,DWORD PTR SS:[EBP-28]  
0041A250   .  50           PUSH EAX  
0041A251   .  8D55 DC       LEA EDX,DWORD PTR SS:[EBP-24]  
0041A254   .  51           PUSH ECX  
0041A255   .  52           PUSH EDX  
0041A256   .  6A 06        PUSH 6  
0041A258   .  FF15 E8114000 CALL DWORD PTR DS:[<&msvbvm60.__vbaFreeS>;  msvbvm60.__vbaFreeStrList  
0041A25E   .  8D45 A8       LEA EAX,DWORD PTR SS:[EBP-58]  
0041A261   .  8D4D B8       LEA ECX,DWORD PTR SS:[EBP-48]  
0041A264   .  50           PUSH EAX  
0041A265   .  51           PUSH ECX  
0041A266   .  6A 02        PUSH 2  
0041A268   .  FF15 3C104000 CALL DWORD PTR DS:[<&msvbvm60.__vbaFreeV>;  msvbvm60.__vbaFreeVarList  
0041A26E   .  83C4 28       ADD ESP,28  
0041A271   .  C3           RETN  
0041A272   >  8B35 80124000 MOV ESI,DWORD PTR DS:[<&msvbvm60.__vbaFr>;  msvbvm60.__vbaFreeStr  
0041A278   .  8D4D E4       LEA ECX,DWORD PTR SS:[EBP-1C]  
0041A27B   .  FFD6          CALL ESI                              ;  <&msvbvm60.__vbaFreeStr>  
0041A27D   .  8D4D E0       LEA ECX,DWORD PTR SS:[EBP-20]  
0041A280   .  FFD6          CALL ESI  
0041A282   .  C3           RETN  
0041A283   .  8B4D EC       MOV ECX,DWORD PTR SS:[EBP-14]  
0041A286   .  8B45 E8       MOV EAX,DWORD PTR SS:[EBP-18]  
0041A289   .  5F           POP EDI  
0041A28A   .  5E           POP ESI  
0041A28B   .  64:890D 00000>MOV DWORD PTR FS:[0],ECX  
0041A292   .  5B           POP EBX  
0041A293   .  8BE5          MOV ESP,EBP  
0041A295   .  5D           POP EBP  
0041A296   .  C2 0C00       RETN 0C  
```

跟进关键 `CALL` `CALL` `1.00419E00` 到这

```
00419E00   $  55           PUSH EBP  
00419E01   .  8BEC         MOV EBP,ESP  
00419E03   .  83EC 0C      SUB ESP,0C  
```

```
00419E99   .  52           PUSH EDX  
00419E9A   .  C745 8C 7C7C4>MOV DWORD PTR SS:[EBP-74],1.00407C7C  
00419EA1   .  C745 84 08800>MOV DWORD PTR SS:[EBP-7C],8008  
00419EA8   .  FF15 10114000 CALL DWORD PTR DS:[<&msvbvm60.__vbaVarTs>;  msvbvm60.__vbaVarTstEq  
00419EAE   .  66:85C0       TEST AX,AX                             ;  用户名长度是否为 0  
00419EB1   .  0F85 9D010000 JNZ 1.0041A054  
00419EB7   .  8D45 A4       LEA EAX,DWORD PTR SS:[EBP-5C]  
00419EBA   .  BB 01000000   MOV EBX,1  
00419EBF   .  50           PUSH EAX  
00419EC0   .  8D4D CC       LEA ECX,DWORD PTR SS:[EBP-34]  
00419EC3   .  53           PUSH EBX  
00419EC4   .  8D55 94       LEA EDX,DWORD PTR SS:[EBP-6C]  
00419EC7   .  BF 02000000   MOV EDI,2  
00419ECC   .  51           PUSH ECX  
00419ECD   .  52           PUSH EDX  
00419ECE   .  895D AC       MOV DWORD PTR SS:[EBP-54],EBX  
00419ED1   .  897D A4       MOV DWORD PTR SS:[EBP-5C],EDI  
00419ED4   .  FF15 F0104000 CALL DWORD PTR DS:[<&msvbvm60.rtcMidChar>;  msvbvm60.rtcMidCharVar  
00419EDA   .  8D45 94       LEA EAX,DWORD PTR SS:[EBP-6C]             ;  取用户名第一位数 l  
00419EDD   .  8D4D BC       LEA ECX,DWORD PTR SS:[EBP-44]  
00419EE0   .  50           PUSH EAX  
00419EE1   .  51           PUSH ECX  
00419EE2   .  FF15 98114000 CALL DWORD PTR DS:[<&msvbvm60.__vbaStrVa>;  msvbvm60.__vbaStrVarVal  
00419EE8   .  50           PUSH EAX  
00419EE9   .  FF15 4C104000 CALL DWORD PTR DS:[<&msvbvm60.rtcAnsiVal>;  msvbvm60.rtcAnsiValueBstr  
00419EEF   .  8B55 10       MOV EDX,DWORD PTR SS:[EBP+10]  
00419EF2   .  8B4D 0C       MOV ECX,DWORD PTR SS:[EBP+C]  
00419EF5   .  52           PUSH EDX                             ;  用户名第一位数转十六进制值。我这是 6c。  
00419EF6   .  51           PUSH ECX  
00419EF7   .  0FBFD0        MOVSX EDX,AX  
00419EFA   .  52           PUSH EDX  
00419EFB   .  E8 80F6FFFF   CALL 1.00419580                       ;  第一次调用重要的算法 CALL1  跟进  
00419F00   .  50           PUSH EAX  
00419F01   .  FF15 18104000 CALL DWORD PTR DS:[<&msvbvm60.__vbaStrI4>;  msvbvm60.__vbaStrI4  
00419F07   .  8B35 50124000 MOV ESI,DWORD PTR DS:[<&msvbvm60.__vbaSt>;  sn1=3cd 转十进制为 973  
00419F0D   .  8BD0          MOV EDX,EAX  
00419F0F   .  8D4D C4       LEA ECX,DWORD PTR SS:[EBP-3C]  
00419F12   .  FFD6          CALL ESI                              ;  <&msvbvm60.__vbaStrMove>  
00419F14   .  8D4D BC       LEA ECX,DWORD PTR SS:[EBP-44]  
00419F17   .  FF15 80124000 CALL DWORD PTR DS:[<&msvbvm60.__vbaFreeS>;  msvbvm60.__vbaFreeStr  
00419F1D   .  8D45 94       LEA EAX,DWORD PTR SS:[EBP-6C]  
00419F20   .  8D4D A4       LEA ECX,DWORD PTR SS:[EBP-5C]  
00419F23   .  50           PUSH EAX  
00419F24   .  51           PUSH ECX  
00419F25   .  57           PUSH EDI  
00419F26   .  FF15 3C104000 CALL DWORD PTR DS:[<&msvbvm60.__vbaFreeV>;  msvbvm60.__vbaFreeVarList  
00419F2C   .  83C4 0C       ADD ESP,0C  
00419F2F   .  8D55 84       LEA EDX,DWORD PTR SS:[EBP-7C]  
00419F32   .  8D45 CC       LEA EAX,DWORD PTR SS:[EBP-34]  
00419F35   .  8D4D A4       LEA ECX,DWORD PTR SS:[EBP-5C]  
00419F38   .  52           PUSH EDX  
00419F39   .  50           PUSH EAX  
00419F3A   .  51           PUSH ECX  
00419F3B   .  895D 8C       MOV DWORD PTR SS:[EBP-74],EBX  
00419F3E   .  897D 84       MOV DWORD PTR SS:[EBP-7C],EDI  
00419F41   .  89BD 7CFFFFFF MOV DWORD PTR SS:[EBP-84],EDI  
00419F47   .  89BD 74FFFFFF MOV DWORD PTR SS:[EBP-8C],EDI  
00419F4D   .  FF15 78104000 CALL DWORD PTR DS:[<&msvbvm60.__vbaLenVa>;  msvbvm60.__vbaLenVar  
00419F53   .  50           PUSH EAX  
00419F54   .  8D95 74FFFFFF LEA EDX,DWORD PTR SS:[EBP-8C]  
00419F5A   .  8D85 50FFFFFF LEA EAX,DWORD PTR SS:[EBP-B0]  
00419F60   .  52           PUSH EDX  
00419F61   .  8D8D 60FFFFFF LEA ECX,DWORD PTR SS:[EBP-A0]  
00419F67   .  50           PUSH EAX  
00419F68   .  8D55 DC       LEA EDX,DWORD PTR SS:[EBP-24]  
00419F6B   .  51           PUSH ECX  
00419F6C   .  52           PUSH EDX  
00419F6D   .  FF15 94104000 CALL DWORD PTR DS:[<&msvbvm60.__vbaVarFo>;  msvbvm60.__vbaVarForInit  
00419F73   .  8B1D 60104000 MOV EBX,DWORD PTR DS:[<&msvbvm60.__vbaSt>;  msvbvm60.__vbaStrCat  
00419F79   >  85C0          TEST EAX,EAX  
00419F7B   .  0F84 C3000000 JE 1.0041A044  
00419F81   .  8D45 A4       LEA EAX,DWORD PTR SS:[EBP-5C]  
```



```
00419F84   .  8D4D DC       LEA ECX,DWORD PTR SS:[EBP-24]  

00419F87   .  50            PUSH EAX  

00419F88   .  51            PUSH ECX  

00419F89   .  C745 AC 01000>MOV DWORD PTR SS:[EBP-54],1  

00419F90   .  897D A4       MOV DWORD PTR SS:[EBP-5C],EDI  

00419F93   .  FF15 08124000 CALL DWORD PTR DS:[<&msvbvm60.__vbaI4Var>;  msvbvm60.__vbaI4Var  

00419F99   .  50            PUSH EAX  

00419F9A   .  8D55 CC       LEA EDX,DWORD PTR SS:[EBP-34]  

00419F9D   .  8D45 94       LEA EAX,DWORD PTR SS:[EBP-6C]  

00419FA0   .  52            PUSH EDX  

00419FA1   .  50            PUSH EAX  

00419FA2   .  FF15 F0104000 CALL DWORD PTR DS:[<&msvbvm60.rtcMidChar>;  msvbvm60.rtcMidCharVar  

00419FA8   .  8D4D 94       LEA ECX,DWORD PTR SS:[EBP-6C]            ;  取用户名第二个数，后面重复  

00419FAB   .  8D55 BC       LEA EDX,DWORD PTR SS:[EBP-44]  

00419FAE   .  51            PUSH ECX  

00419FAF   .  52            PUSH EDX  

00419FB0   .  FF15 98114000 CALL DWORD PTR DS:[<&msvbvm60.__vbaStrVa>;  msvbvm60.__vbaStrVarVal  

00419FB6   .  50            PUSH EAX  

00419FB7   .  FF15 4C104000 CALL DWORD PTR DS:[<&msvbvm60.rtcAnsiVal>;  msvbvm60.rtcAnsiValueBstr  

00419FBD   .  8BF8          MOV EDI,EAX  

00419FBF   .  8B45 C4       MOV EAX,DWORD PTR SS:[EBP-3C]  

00419FC2   .  50            PUSH EAX  

00419FC3   .  68 049A4000   PUSH 1.00409A04  

00419FC8   .  FFD3          CALL EBX  

00419FCA   .  8BD0          MOV EDX,EAX                             ;  973 与“+”相连我这为 973+  

00419FCC   .  8D4D B8       LEA ECX,DWORD PTR SS:[EBP-48]  

00419FCF   .  FFD6          CALL ESI  

00419FD1   .  8B4D 10       MOV ECX,DWORD PTR SS:[EBP+10]  

00419FD4   .  8B55 0C       MOV EDX,DWORD PTR SS:[EBP+C]  

00419FD7   .  50            PUSH EAX  

00419FD8   .  51            PUSH ECX  

00419FD9   .  0FBFC7        MOVSX EAX,DI  

00419FDC   .  52            PUSH EDX  

00419FDD   .  50            PUSH EAX  

00419FDE   .  E8 9DF5FFFF   CALL 1.00419580                         ;  第二次调用重要的算法 CALL1  

00419FE3   .  50            PUSH EAX                                ;  结果为 SN2=11c  

00419FE4   .  FF15 18104000 CALL DWORD PTR DS:[<&msvbvm60.__vbaStrI4>;  msvbvm60.__vbaStrI4  

00419FEA   .  8BD0          MOV EDX,EAX                             ;  转十进制为 284  

00419FEC   .  8D4D B4       LEA ECX,DWORD PTR SS:[EBP-4C]  

00419FEF   .  FFD6          CALL ESI  

00419FF1   .  50            PUSH EAX  

00419FF2   .  FFD3          CALL EBX  

00419FF4   .  8BD0          MOV EDX,EAX  

00419FF6   .  8D4D C4       LEA ECX,DWORD PTR SS:[EBP-3C]  

00419FF9   .  FFD6          CALL ESI  

00419FFB   .  8D4D B4       LEA ECX,DWORD PTR SS:[EBP-4C]  

00419FFE   .  8D55 B8       LEA EDX,DWORD PTR SS:[EBP-48]            ;  与前面的相连为 973+284  

0041A001   .  51            PUSH ECX  

0041A002   .  8D45 BC       LEA EAX,DWORD PTR SS:[EBP-44]  

0041A005   .  52            PUSH EDX  

0041A006   .  50            PUSH EAX  

0041A007   .  6A 03         PUSH 3  

0041A009   .  FF15 E8114000 CALL DWORD PTR DS:[<&msvbvm60.__vbaFreeS>;  msvbvm60.__vbaFreeStrList  

0041A00F   .  8D4D 94       LEA ECX,DWORD PTR SS:[EBP-6C]  

0041A012   .  8D55 A4       LEA EDX,DWORD PTR SS:[EBP-5C]  

0041A015   .  51            PUSH ECX  

0041A016   .  52            PUSH EDX  

0041A017   .  6A 02         PUSH 2  

0041A019   .  FF15 3C104000 CALL DWORD PTR DS:[<&msvbvm60.__vbaFreeV>;  msvbvm60.__vbaFreeVarList  

0041A01F   .  83C4 1C       ADD ESP,1C  

0041A022   .  8D85 50FFFFFF LEA EAX,DWORD PTR SS:[EBP-B0]  

0041A028   .  8D8D 60FFFFFF LEA ECX,DWORD PTR SS:[EBP-A0]  

0041A02E   .  8D55 DC       LEA EDX,DWORD PTR SS:[EBP-24]  

0041A031   .  50            PUSH EAX  

0041A032   .  51            PUSH ECX  

0041A033   .  52            PUSH EDX  

0041A034   .  FF15 74124000 CALL DWORD PTR DS:[<&msvbvm60.__vbaVarFo>;  msvbvm60.__vbaVarForNext  

0041A03A   .  BF 02000000   MOV EDI,2  

0041A03F   .^ E9 35FFFFFF   JMP 1.00419F79                          ;  反回去循环直到把用户名算完  

0041A044   >  8D45 C4       LEA EAX,DWORD PTR SS:[EBP-3C]          ;  最终得字符串 SN  

0041A047   .  50            PUSH EAX  

0041A048   .  E8 43F7FFFF   CALL 1.00419790                        ;  重要的算法 CALL2 跟进  

0041A04D   .  8BD0          MOV EDX,EAX                             ;  把 SN 转为字符串记为 S="OTczKzI4NCs5NzMrMTU1KzQ1MSs2MTkrNzQw"  

0041A04F   .  8D4D C8       LEA ECX,DWORD PTR SS:[EBP-38]  

0041A052   .  FFD6          CALL ESI  

0041A054   >  68 CCA04100   PUSH 1.0041A0CC  

0041A059   .  EB 37         JMP SHORT 1.0041A092  

0041A05B   .  F645 FC 04    TEST BYTE PTR SS:[EBP-4],4  

0041A05F   .  74 09        JE SHORT 1.0041A06A  

0041A061   .  8D4D C8       LEA ECX,DWORD PTR SS:[EBP-38]  

0041A064   .  FF15 80124000 CALL DWORD PTR DS:[<&msvbvm60.__vbaFreeS>;  msvbvm60.__vbaFreeStr  

0041A06A   >  8D4D B4       LEA ECX,DWORD PTR SS:[EBP-4C]  

0041A06D   .  8D55 B8       LEA EDX,DWORD PTR SS:[EBP-48]  

0041A070   .  51            PUSH ECX  

0041A071   .  8D45 BC       LEA EAX,DWORD PTR SS:[EBP-44]  

0041A074   .  52            PUSH EDX  

0041A075   .  50            PUSH EAX  

0041A076   .  6A 03         PUSH 3  

0041A078   .  FF15 E8114000 CALL DWORD PTR DS:[<&msvbvm60.__vbaFreeS>;  msvbvm60.__vbaFreeStrList  

0041A07E   .  8D4D 94       LEA ECX,DWORD PTR SS:[EBP-6C]  

0041A081   .  8D55 A4       LEA EDX,DWORD PTR SS:[EBP-5C]  

0041A084   .  51            PUSH ECX  

0041A085   .  52            PUSH EDX  

0041A086   .  6A 02         PUSH 2  

0041A088   .  FF15 3C104000 CALL DWORD PTR DS:[<&msvbvm60.__vbaFreeV>;  msvbvm60.__vbaFreeVarList  

0041A08E   .  83C4 1C       ADD ESP,1C  

0041A091   .  C3            RETN  

。。。。。。跟进重要的算法 CALL1 1.00419580 到这。。。。。。  

00419580   $  55            PUSH EBP  

00419581   .  8BEC          MOV EBP,ESP  

00419583   .  83EC 14       SUB ESP,14  

。。。。。。  

004195F1   >  85FF          TEST EDI,EDI  

004195F3   .  0F8E 10010000 JLE 1.00419709                          ;   算法部分开始  

004195F9   >  DB45 0C       FILD DWORD PTR SS:[EBP+C]  

004195FC   .  DD9D 78FFFFFF FSTP QWORD PTR SS:[EBP-88]  

00419602   .  DD85 78FFFFFF FLD QWORD PTR SS:[EBP-88]  

00419608   .  833D 00C04100>CMP DWORD PTR DS:[41C000],0  

0041960F   .  75 08         JNZ SHORT 1.00419619  

00419611   .  DC35 C8194000 FDIV QWORD PTR DS:[4019C8]              ;  得第一部分的值/2  

00419617   .  EB 11         JMP SHORT 1.0041962A  

00419619   >  FF35 CC194000 PUSH DWORD PTR DS:[4019CC]  

0041961F   .  FF35 C8194000 PUSH DWORD PTR DS:[4019C8]  

00419625   .  E8 EA84FEFF   CALL <JMP.&msvbvm60._adj_fdiv_m64>  

0041962A   >  DFE0          FSTSW AX  

0041962C   .  A8 0D         TEST AL,0D  

0041962E   .  0F85 4C010000 JNZ 1.00419780  

00419634   .  FF15 DC104000 CALL DWORD PTR DS:[<&msvbvm60.__vbaFpR8>>;  msvbvm60.__vbaFpR8  

0041963A   .  8BC7          MOV EAX,EDI  

0041963C   .  99            CDQ  

0041963D   .  2BC2          SUB EAX,EDX  

0041963F   .  D1F8          SAR EAX,1                                 ;  右移一位相当于得第一部分的值\ 2  

00419641   .  8985 74FFFFFF MOV DWORD PTR SS:[EBP-8C],EAX  

00419647   .  DB85 74FFFFFF FILD DWORD PTR SS:[EBP-8C]  

0041964D   .  DD9D 6CFFFFFF FSTP QWORD PTR SS:[EBP-94]  

00419653   .  DC9D 6CFFFFFF FCOMP QWORD PTR SS:[EBP-94]  

00419659   .  DFE0          FSTSW AX  

0041965B   .  F6C4 40       TEST AH,40  

0041965E   .  74 5B         JE SHORT 1.004196BB                       ;  得第一部分的值/2 = 第一部分的值\ 2 就不跳走。第一次是 1234 不跳走。  

00419660   .  8BC6          MOV EAX,ESI                               ;  即第一部分的值是偶数不跳走。如果是奇数就跳走，第二次来到这时是 617 要跳走。  

00419662   .  0FAFC6        IMUL EAX,ESI                               ;  用户名第一个数的平方为我这是 6c*6c=2d90  

00419665   .  0F80 1A010000 JO 1.00419785  

0041966B   .  99            CDQ  

0041966C   .  F7FB          IDIV EBX                                  ;  2d90 除第三部分的十六进制值 457, 取余数我这是 22a
```



`0041966E   .   8BF2`             `MOV ESI,EDX`

`00419670   .   DB45 0C`          `FILD DWORD PTR SS:[EBP+C]`

`00419673   .   DD9D 64FFFFFF`    `FSTP QWORD PTR SS:[EBP-9C]`

`00419679   .   DD85 64FFFFFF`    `FLD QWORD PTR SS:[EBP-9C]`

`0041967F   .   833D 00C04100>`   `CMP DWORD PTR DS:[41C000],0`

`00419686   .   75 08`           `JNZ SHORT 1.00419690`

`00419688   .   DC35 C8194000`    `FDIV QWORD PTR DS:[4019C8]`                       `; 1234/2=617`

`0041968E   .   EB 11`           `JMP SHORT 1.004196A1`

`00419690   >   FF35 CC194000`    `PUSH DWORD PTR DS:[4019CC]`

`00419696   .   FF35 C8194000`    `PUSH DWORD PTR DS:[4019C8]`

`0041969C   .   E8 7384FEFF`      `CALL <JMP.&msvbvm60._adj_fdiv_m64>`

`004196A1   >   DFE0`             `FSTSW AX`

`004196A3   .   A8 0D`            `TEST AL,0D`

`004196A5   .   0F85 D5000000`    `JNZ 1.00419780`

`004196AB   .   FF15 38124000`    `CALL DWORD PTR DS:[<&msvbvm60.__vbaFpI4>>; msvbvm60.__vbaFpI4`

`004196B1   .   8BF8`             `MOV EDI,EAX`

`004196B3   .   897D 0C`          `MOV DWORD PTR SS:[EBP+C],EDI`

`004196B6   .^  E9 3EFFFFFF`      `JMP 1.004195F9`                                  `; 跳回去重复运算`

`004196BB   >   8975 A8`          `MOV DWORD PTR SS:[EBP-58],ESI`

`004196BE   .   B8 03000000`      `MOV EAX,3`

`004196C3   .   8945 A0`          `MOV DWORD PTR SS:[EBP-60],EAX`

`004196C6   .   895D 98`          `MOV DWORD PTR SS:[EBP-68],EBX`

`004196C9   .   8945 90`          `MOV DWORD PTR SS:[EBP-70],EAX`

`004196CC   .   8D45 A0`          `LEA EAX,DWORD PTR SS:[EBP-60]`

`004196CF   .   50`               `PUSH EAX`                                       `; 运算结果 22a`

`004196D0   .   8D4D D0`          `LEA ECX,DWORD PTR SS:[EBP-30]`

`004196D3   .   51`               `PUSH ECX`                                       `; 数字 1`

`004196D4   .   8D55 C0`          `LEA EDX,DWORD PTR SS:[EBP-40]`

`004196D7   .   52`               `PUSH EDX`

`004196D8   .   FF15 54114000`    `CALL DWORD PTR DS:[<&msvbvm60.__vbaVarMu>; msvbvm60.__vbaVarMul`

`004196DE   .   50`               `PUSH EAX`                                       `; 22a*1=22a`

`004196DF   .   8D45 90`          `LEA EAX,DWORD PTR SS:[EBP-70]`

`004196E2   .   50`               `PUSH EAX`                                       `; 第三部分的十六进制值 457`

`004196E3   .   8D4D B0`          `LEA ECX,DWORD PTR SS:[EBP-50]`

`004196E6   .   51`               `PUSH ECX`

`004196E7   .   FF15 2C124000`    `CALL DWORD PTR DS:[<&msvbvm60.__vbaVarMo>; msvbvm60.__vbaVarMod`

`004196ED   .   8BD0`             `MOV EDX,EAX`                                    `; 22a mod 457=22a，，，循环的最终结果记为 SN1=3cd`

`004196EF   .   8D4D D0`          `LEA ECX,DWORD PTR SS:[EBP-30]`                  `; 用户名第二个数来运算时记结果为 SN2，依次类推，我这还有 SN3，SN4，SN5，SN6，SN7。`

`004196F2   .   FF15 1C104000`    `CALL DWORD PTR DS:[<&msvbvm60.__vbaVarMo>; msvbvm60.__vbaVarMove`

`004196F8   .   83EF 01`          `SUB EDI,1`                                      `; 617 的十六进制-1=618（偶数）,即 269-1=288`

`004196FB   .   0F80 84000000`    `JO 1.00419785`

`00419701   .   897D 0C`          `MOV DWORD PTR SS:[EBP+C],EDI`

`00419704   .^  E9 E8FEFFFF`      `JMP 1.004195F1`                                  `; 返回去循环。直到 EDI=0 时结束。`

`00419709   >   8D55 D0`          `LEA EDX,DWORD PTR SS:[EBP-30]`

`0041970C   .   52`               `PUSH EDX`                                       `; SND=3cd`

。。。。。。

。。。。。。跟进重要的算法 `CALL2 1.00419790` 到这。。。

`00419790   $   55`               `PUSH EBP`

`00419791   .   8BEC`             `MOV EBP,ESP`

。。。。。。

`00419864   .   897D 80`          `MOV DWORD PTR SS:[EBP-80],EDI`

`00419867   .   FF15 E8104000`    `CALL DWORD PTR DS:[<&msvbvm60.rtcMidChar>; msvbvm60.rtcMidCharBstr`

`0041986D   .   8BD0`             `MOV EDX,EAX`                                    `; 取 SN 的第一位数 9`

`0041986F   .   8D4D A8`          `LEA ECX,DWORD PTR SS:[EBP-58]`

`00419872   .   FFD6`             `CALL ESI`

`00419874   .   50`               `PUSH EAX`

`00419875   .   FF15 4C104000`    `CALL DWORD PTR DS:[<&msvbvm60.rtcAnsiVal>; msvbvm60.rtcAnsiValueBstr`

`0041987B   .   8D95 20FFFFFF`    `LEA EDX,DWORD PTR SS:[EBP-E0]`

`00419881   .   8D4D D0`          `LEA ECX,DWORD PTR SS:[EBP-30]`

`00419884   .   66:8985 28FFF>`   `MOV WORD PTR SS:[EBP-D8],AX`

`0041988B   .   89BD 20FFFFFF`    `MOV DWORD PTR SS:[EBP-E0],EDI`                  `; 得 9 的 ASC 码十六进制为 39`

`00419891   .   FF15 1C104000`    `CALL DWORD PTR DS:[<&msvbvm60.__vbaVarMo>; msvbvm60.__vbaVarMove`

`00419897   .   8D4D A8`          `LEA ECX,DWORD PTR SS:[EBP-58]`

`0041989A   .   FF15 80124000`    `CALL DWORD PTR DS:[<&msvbvm60.__vbaFreeS>; msvbvm60.__vbaFreeStr`

`004198A0   .   8D4D 80`          `LEA ECX,DWORD PTR SS:[EBP-80]`

`004198A3   .   FF15 24104000`    `CALL DWORD PTR DS:[<&msvbvm60.__vbaFreeV>; msvbvm60.__vbaFreeVar`

`004198A9   .   66:8BCB`          `MOV CX,BX`

`004198AC   .   8D45 80`          `LEA EAX,DWORD PTR SS:[EBP-80]`

`004198AF   .   66:83C1 01`       `ADD CX,1`

`004198B3   .   50`               `PUSH EAX`

`004198B4   .   8B45 08`          `MOV EAX,DWORD PTR SS:[EBP+8]`

`004198B7   .   C745 88 01000>`   `MOV DWORD PTR SS:[EBP-78],1`

`004198BE   .   0F80 64040000`    `JO 1.00419D28`

`004198C4   .   0FBFD1`           `MOVSX EDX,CX`

`004198C7   .   8B08`             `MOV ECX,DWORD PTR DS:[EAX]`

`004198C9   .   52`               `PUSH EDX`

`004198CA   .   51`               `PUSH ECX`

`004198CB   .   897D 80`          `MOV DWORD PTR SS:[EBP-80],EDI`

`004198CE   .   FF15 E8104000`    `CALL DWORD PTR DS:[<&msvbvm60.rtcMidChar>; msvbvm60.rtcMidCharBstr`

`004198D4   .   8BD0`             `MOV EDX,EAX`                                    `; 取 SN 的第二位数 7`

`004198D6   .   8D4D A8`          `LEA ECX,DWORD PTR SS:[EBP-58]`

`004198D9   .   FFD6`             `CALL ESI`

`004198DB   .   50`               `PUSH EAX`

`004198DC   .   6A 00`            `PUSH 0`

`004198DE   .   FF15 A4114000`    `CALL DWORD PTR DS:[<&msvbvm60.rtcBstrFro>; msvbvm60.rtcBstrFromAnsi`

`004198E4   .   8BD0`             `MOV EDX,EAX`                                    `; 得 7 的 ASC 码十六进制为 37`

`004198E6   .   8D4D A4`          `LEA ECX,DWORD PTR SS:[EBP-5C]`

`004198E9   .   FFD6`             `CALL ESI`

`004198EB   .   50`               `PUSH EAX`

`004198EC   .   FF15 60104000`    `CALL DWORD PTR DS:[<&msvbvm60.__vbaStrCa>; msvbvm60.__vbaStrCat`

`004198F2   .   8BD0`             `MOV EDX,EAX`

`004198F4   .   8D4D A0`          `LEA ECX,DWORD PTR SS:[EBP-60]`

`004198F7   .   FFD6`             `CALL ESI`

`004198F9   .   50`               `PUSH EAX`

`004198FA   .   FF15 4C104000`    `CALL DWORD PTR DS:[<&msvbvm60.rtcAnsiVal>; msvbvm60.rtcAnsiValueBstr`

`00419900   .   8D95 20FFFFFF`    `LEA EDX,DWORD PTR SS:[EBP-E0]`

`00419906   .   8D4D BC`          `LEA ECX,DWORD PTR SS:[EBP-44]`

`00419909   .   66:8985 28FFF>`   `MOV WORD PTR SS:[EBP-D8],AX`

`00419910   .   89BD 20FFFFFF`    `MOV DWORD PTR SS:[EBP-E0],EDI`

`00419916   .   FF15 1C104000`    `CALL DWORD PTR DS:[<&msvbvm60.__vbaVarMo>; msvbvm60.__vbaVarMove`

`0041991C   .   8D55 A0`          `LEA EDX,DWORD PTR SS:[EBP-60]`

`0041991F   .   8D45 A4`          `LEA EAX,DWORD PTR SS:[EBP-5C]`

`00419922   .   52`               `PUSH EDX`

`00419923   .   8D4D A8`          `LEA ECX,DWORD PTR SS:[EBP-58]`

`00419926   .   50`               `PUSH EAX`

`00419927   .   51`               `PUSH ECX`

`00419928   .   6A 03`            `PUSH 3`

`0041992A   .   FF15 E8114000`    `CALL DWORD PTR DS:[<&msvbvm60.__vbaFreeS>; msvbvm60.__vbaFreeStrList`

`00419930   .   83C4 10`          `ADD ESP,10`

`00419933   .   8D4D 80`          `LEA ECX,DWORD PTR SS:[EBP-80]`

`00419936   .   FF15 24104000`    `CALL DWORD PTR DS:[<&msvbvm60.__vbaFreeV>; msvbvm60.__vbaFreeVar`

`0041993C   .   66:8BC3`          `MOV AX,BX`

`0041993F   .   8D55 80`          `LEA EDX,DWORD PTR SS:[EBP-80]`

`00419942   .   66:03C7`          `ADD AX,DI`

`00419945   .   52`               `PUSH EDX`

`00419946   .   8B55 08`          `MOV EDX,DWORD PTR SS:[EBP+8]`

`00419949   .   C745 88 01000>`   `MOV DWORD PTR SS:[EBP-78],1`

`00419950   .   0F80 D2030000`    `JO 1.00419D28`

`00419956   .   0FBFC8`           `MOVSX ECX,AX`

`00419959   .   8B02`             `MOV EAX,DWORD PTR DS:[EDX]`

`0041995B   .   51`               `PUSH ECX`

`0041995C   .   897D 80`          `MOV DWORD PTR SS:[EBP-80],EDI`

`0041995F   .   50`               `PUSH EAX`

`00419960   .   FF15 E8104000`    `CALL DWORD PTR DS:[<&msvbvm60.rtcMidChar>; msvbvm60.rtcMidCharBstr`

`00419966   .   8BD0`             `MOV EDX,EAX`                                    `; 取 SN 的第三位数 3`



```
00419968   .  8D4D A8       LEA ECX,DWORD PTR SS:[EBP-58]  

0041996B   .  FFD6         CALL ESI  

0041996D   .  50          PUSH EAX  

0041996E   .  6A 00       PUSH 0  

00419970   .  FF15 A4114000 CALL DWORD PTR DS:[<&msvbvm60.rtcBstrFro>;  msvbvm60.rtcBstrFromAnsi  

00419976   .  8BD0        MOV EDX,EAX                         ;  得 3 的 ASC 码十六进制为 33  

00419978   .  8D4D A4     LEA ECX,DWORD PTR SS:[EBP-5C]  

0041997B   .  FFD6       CALL ESI  

0041997D   .  50         PUSH EAX  

0041997E   .  FF15 60104000 CALL DWORD PTR DS:[<&msvbvm60.\_\_vbaStrCa>;  msvbvm60.\_\_vbaStrCat  

00419984   .  8BD0       MOV EDX,EAX  

00419986   .  8D4D A0    LEA ECX,DWORD PTR SS:[EBP-60]  

00419989   .  FFD6       CALL ESI  

0041998B   .  50         PUSH EAX  

0041998C   .  FF15 4C104000 CALL DWORD PTR DS:[<&msvbvm60.rtcAnsiVal>;  msvbvm60.rtcAnsiValueBstr  

00419992   .  8D4D A0    LEA ECX,DWORD PTR SS:[EBP-60]  

00419995   .  8945 B4    MOV DWORD PTR SS:[EBP-4C],EAX  

00419998   .  8D55 A4    LEA EDX,DWORD PTR SS:[EBP-5C]  

0041999B   .  51         PUSH ECX  

0041999C   .  8D45 A8    LEA EAX,DWORD PTR SS:[EBP-58]  

0041999F   .  52         PUSH EDX  

004199A0   .  50         PUSH EAX  

004199A1   .  6A 03      PUSH 3                                ;  得到的三个数，开始运算，注意有四次运算。  

004199A3   .  FF15 E8114000 CALL DWORD PTR DS:[<&msvbvm60.\_\_vbaFreeS>;  msvbvm60.\_\_vbaFreeStrList  

004199A9   .  83C4 10    ADD ESP,10  

004199AC   .  8D4D 80    LEA ECX,DWORD PTR SS:[EBP-80]  

004199AF   .  FF15 24104000 CALL DWORD PTR DS:[<&msvbvm60.\_\_vbaFreeV>;  msvbvm60.\_\_vbaFreeVar  

004199B5   .  8D4D D0    LEA ECX,DWORD PTR SS:[EBP-30]  

004199B8   .  8D95 30FFFFFF LEA EDX,DWORD PTR SS:[EBP-D0]  

004199BE   .  51         PUSH ECX                              ;  这里的值为 39  

004199BF   .  8D45 80    LEA EAX,DWORD PTR SS:[EBP-80]  

004199C2   .  52         PUSH EDX                               ;  这里的值为 4  

004199C3   .  50         PUSH EAX  

004199C4   .  C785 38FFFFFF>MOV DWORD PTR SS:[EBP-C8],4  

004199CE   .  89BD 30FFFFFF MOV DWORD PTR SS:[EBP-D0],EDI  

004199D4   .  FF15 78114000 CALL DWORD PTR DS:[<&msvbvm60.\_\_vbaVarDi>;  msvbvm60.\_\_vbaVarDiv  

004199DA   .  8D8D 70FFFFFF LEA ECX,DWORD PTR SS:[EBP-90]         ;  39/4  

004199E0   .  50         PUSH EAX  

004199E1   .  51         PUSH ECX  

004199E2   .  FF15 CC114000 CALL DWORD PTR DS:[<&msvbvm60.\_\_vbaVarIn>;  msvbvm60.\_\_vbaVarInt  

004199E8   .  50         PUSH EAX                              ;  39/4 取整数部分  

004199E9   .  FF15 A0114000 CALL DWORD PTR DS:[<&msvbvm60.\_\_vbaI2Var>;  msvbvm60.\_\_vbaI2Var  

004199EF   .  8945 E8    MOV DWORD PTR SS:[EBP-18],EAX         ;  39/4 取整数部分相当于 39\\4=E 为第一个结果  

004199F2   .  B8 10000000 MOV EAX,10  

004199F7   .  8985 28FFFFFF MOV DWORD PTR SS:[EBP-D8],EAX  

004199FD   .  8985 18FFFFFF MOV DWORD PTR SS:[EBP-E8],EAX  

00419A03   .  8D55 D0    LEA EDX,DWORD PTR SS:[EBP-30]  

00419A06   .  8D85 30FFFFFF LEA EAX,DWORD PTR SS:[EBP-D0]  

00419A0C   .  52         PUSH EDX  

00419A0D   .  8D4D 80    LEA ECX,DWORD PTR SS:[EBP-80]  

00419A10   .  50         PUSH EAX                              ;  下面计算我就简单写一下  

00419A11   .  51         PUSH ECX  

00419A12   .  C785 38FFFFFF>MOV DWORD PTR SS:[EBP-C8],3  

00419A1C   .  89BD 30FFFFFF MOV DWORD PTR SS:[EBP-D0],EDI  

00419A22   .  89BD 20FFFFFF MOV DWORD PTR SS:[EBP-E0],EDI  

00419A28   .  89BD 10FFFFFF MOV DWORD PTR SS:[EBP-F0],EDI  

00419A2E   .  FF15 4C114000 CALL DWORD PTR DS:[<&msvbvm60.\_\_vbaVarAn>;  msvbvm60.\_\_vbaVarAnd  

00419A34   .  50         PUSH EAX  

00419A35   .  8D95 20FFFFFF LEA EDX,DWORD PTR SS:[EBP-E0]  

00419A3B   .  8D85 70FFFFFF LEA EAX,DWORD PTR SS:[EBP-90]  

00419A41   .  52         PUSH EDX  

00419A42   .  50         PUSH EAX  

00419A43   .  FF15 54114000 CALL DWORD PTR DS:[<&msvbvm60.\_\_vbaVarMu>;  msvbvm60.\_\_vbaVarMul  

00419A49   .  8D4D BC    LEA ECX,DWORD PTR SS:[EBP-44]  

00419A4C   .  50         PUSH EAX                              ;  以上是运算过程是(39 and 3)\*10  

00419A4D   .  8D95 10FFFFFF LEA EDX,DWORD PTR SS:[EBP-F0]  

00419A53   .  51         PUSH ECX  

00419A54   .  8D85 60FFFFFF LEA EAX,DWORD PTR SS:[EBP-A0]  

00419A5A   .  52         PUSH EDX  

00419A5B   .  50         PUSH EAX  

00419A5C   .  FF15 78114000 CALL DWORD PTR DS:[<&msvbvm60.\_\_vbaVarDi>;  msvbvm60.\_\_vbaVarDiv  

00419A62   .  8D8D 50FFFFFF LEA ECX,DWORD PTR SS:[EBP-B0]  

00419A68   .  50         PUSH EAX  

00419A69   .  51         PUSH ECX  

00419A6A   .  FF15 CC114000 CALL DWORD PTR DS:[<&msvbvm60.\_\_vbaVarIn>;  msvbvm60.\_\_vbaVarInt  

00419A70   .  8D95 40FFFFFF LEA EDX,DWORD PTR SS:[EBP-C0]         ;  以上是运算过程是 37\\10  

00419A76   .  50         PUSH EAX  

00419A77   .  52         PUSH EDX  

00419A78   .  FF15 18124000 CALL DWORD PTR DS:[<&msvbvm60.\_\_vbaVarAd>;  msvbvm60.\_\_vbaVarAdd  

00419A7E   .  50         PUSH EAX                              ;  以上是运算过程是(39 and 3)\*10 +37\\10=13  

00419A7F   .  FF15 A0114000 CALL DWORD PTR DS:[<&msvbvm60.\_\_vbaI2Var>;  msvbvm60.\_\_vbaI2Var  

00419A85   .  8D8D 40FFFFFF LEA ECX,DWORD PTR SS:[EBP-C0]         ;  第二个结果为 13  

00419A8B   .  8945 E0    MOV DWORD PTR SS:[EBP-20],EAX  

00419A8E   .  FF15 24104000 CALL DWORD PTR DS:[<&msvbvm60.\_\_vbaFreeV>;  msvbvm60.\_\_vbaFreeVar  

00419A94   .  8B45 08    MOV EAX,DWORD PTR SS:[EBP+8]  

00419A97   .  8B08       MOV ECX,DWORD PTR DS:[EAX]  

00419A99   .  51         PUSH ECX  

00419A9A   .  FF15 2C104000 CALL DWORD PTR DS:[<&msvbvm60.\_\_vbaLenBs>;  msvbvm60.\_\_vbaLenBstr  

00419AA0   .  66:8BD3    MOV DX,BX  

00419AA3   .  66:83C2 01 ADD DX,1  

00419AA7   .  0F80 7B020000 JO 1.00419D28  

00419AAD   .  0FBFCA     MOVSX ECX,DX  

00419AB0   .  3BC1       CMP EAX,ECX  

00419AB2   .  0F8C D5000000 JL 1.00419B8D  

00419AB8   .  0FBF55 B4  MOVSX EDX,WORD PTR SS:[EBP-4C]  

00419ABC   .  8995 FCFEFFFF MOV DWORD PTR SS:[EBP-104],EDX  

00419AC2   .  C785 38FFFFFF>MOV DWORD PTR SS:[EBP-C8],0F  

00419ACC   .  DB85 FCFEFFFF FILD DWORD PTR SS:[EBP-104]  

00419AD2   .  89BD 30FFFFFF MOV DWORD PTR SS:[EBP-D0],EDI  

00419AD8   .  C785 28FFFFFF>MOV DWORD PTR SS:[EBP-D8],4  

00419AE2   .  89BD 20FFFFFF MOV DWORD PTR SS:[EBP-E0],EDI  

00419AE8   .  DD9D F4FEFFFF FSTP QWORD PTR SS:[EBP-10C]  

00419AEE   .  DD85 F4FEFFFF FLD QWORD PTR SS:[EBP-10C]  

00419AF4   .  833D 00C04100>CMP DWORD PTR DS:[41C000],0  

00419AFB   .  75 08      JNZ SHORT 1.00419B05  

00419AFD   .  DC35 D0194000 FDIV QWORD PTR DS:[4019D0]  

00419B03   .  EB 11      JMP SHORT 1.00419B16  

00419B05   >  FF35 D4194000 PUSH DWORD PTR DS:[4019D4]  

00419B0B   .  FF35 D0194000 PUSH DWORD PTR DS:[4019D0]  

00419B11   .  E8 FE7FFEFF  CALL <JMP.&msvbvm60.\_adj\_fdiv\_m64>  

00419B16   >  DFE0       FSTSW AX  

00419B18   .  A8 0D      TEST AL,0D  

00419B1A   .  0F85 03020000 JNZ 1.00419D23  

00419B20   .  FF15 70124000 CALL DWORD PTR DS:[<&msvbvm60.\_\_vbaFPInt>;  msvbvm60.\_\_vbaFPInt  

00419B26   .  DD9D 18FFFFFF FSTP QWORD PTR SS:[EBP-E8]  

00419B2C   .  8D45 BC    LEA EAX,DWORD PTR SS:[EBP-44]  

00419B2F   .  8D8D 30FFFFFF LEA ECX,DWORD PTR SS:[EBP-D0]  

00419B35   .  50         PUSH EAX  

00419B36   .  8D55 80    LEA EDX,DWORD PTR SS:[EBP-80]  

00419B39   .  51         PUSH ECX  

00419B3A   .  52         PUSH EDX  

00419B3B   .  C785 10FFFFFF>MOV DWORD PTR SS:[EBP-F0],5  

00419B45   .  FF15 4C114000 CALL DWORD PTR DS:[<&msvbvm60.\_\_vbaVarAn>;  msvbvm60.\_\_vbaVarAnd  

00419B4B   .  50         PUSH EAX                              ;  37 and 0F=7  

00419B4C   .  8D85 20FFFFFF LEA EAX,DWORD PTR SS:[EBP-E0]  
```



```
00419B52   .  8D8D 70FFFFFF LEA ECX,DWORD PTR SS:[EBP-90]  
00419B58   .  50            PUSH EAX  
00419B59   .  51            PUSH ECX  
00419B5A   .  FF15 54114000 CALL DWORD PTR DS:[<&msvbvm60.__vbaVarMu>;  msvbvm60.__vbaVarMul  
00419B60   .  50            PUSH EAX                                      ;  7*4=1c  
00419B61   .  8D95 10FFFFFF LEA EDX,DWORD PTR SS:[EBP-F0]  
00419B67   .  8D85 60FFFFFF LEA EAX,DWORD PTR SS:[EBP-A0]  
00419B6D   .  52            PUSH EDX  
00419B6E   .  50            PUSH EAX  
00419B6F   .  FF15 18124000 CALL DWORD PTR DS:[<&msvbvm60.__vbaVarAd>;  msvbvm60.__vbaVarAdd  
00419B75   .  50            PUSH EAX                                      ;  以上是运算过程是（37 and 0F）*4+0=1c  
00419B76   .  FF15 A0114000 CALL DWORD PTR DS:[<&msvbvm60.__vbaI2Var>;  msvbvm60.__vbaI2Var  
00419B7C   .  8D8D 60FFFFFF LEA ECX,DWORD PTR SS:[EBP-A0]                ;  得第三个结果为 1c  
00419B82   .  8945 CC        MOV DWORD PTR SS:[EBP-34],EAX  
00419B85   .  FF15 24104000 CALL DWORD PTR DS:[<&msvbvm60.__vbaFreeV>;  msvbvm60.__vbaFreeVar  
00419B8B   .  EB 07          JMP SHORT 1.00419B94  
00419B8D   >  C745 CC FFFFF>MOV DWORD PTR SS:[EBP-34],-1  
00419B94   >  8B4D 08        MOV ECX,DWORD PTR SS:[EBP+8]  
00419B97   .  8B11          MOV EDX,DWORD PTR DS:[ECX]  
00419B99   .  52            PUSH EDX  
00419B9A   .  FF15 2C104000 CALL DWORD PTR DS:[<&msvbvm60.__vbaLenBs>;  msvbvm60.__vbaLenBstr  
00419BA0   .  66:8BCB       MOV CX,BX  
00419BA3   .  66:03CF       ADD CX,DI  
00419BA6   .  0F80 7C010000 JO 1.00419D28  
00419BAC   .  0FBFD1        MOVSX EDX,CX  
00419BAF   .  3BC2          CMP EAX,EDX  
00419BB1   .  7C 0B         JL SHORT 1.00419BBE  
00419BB3   .  8B45 B4       MOV EAX,DWORD PTR SS:[EBP-4C]  
00419BB6   .  83E0 3F       AND EAX,3F                                   ;  33 and 3f=33  
00419BB9   .  8945 B8       MOV DWORD PTR SS:[EBP-48],EAX                ;  得第四个结果为 33  
00419BBC   .  EB 07         JMP SHORT 1.00419BC5  
00419BBE   >  C745 B8 FFFFF>MOV DWORD PTR SS:[EBP-48],-1  
00419BC5   >  8B45 AC       MOV EAX,DWORD PTR SS:[EBP-54]  
00419BC8   .  8D4D E8       LEA ECX,DWORD PTR SS:[EBP-18]  
00419BCB   .  50            PUSH EAX  
00419BCC   .  51            PUSH ECX  
00419BCD   .  E8 5E010000    CALL 1.00419D30                              ;  第三个重要的算法 CALL，跟进  
00419BD2   .  8BD0          MOV EDX,EAX                                  ;  这一个 CALL 是取得第一个结果 E 对应的字符“O”字。  
00419BD4   .  8D4D A8       LEA ECX,DWORD PTR SS:[EBP-58]  
00419BD7   .  FFD6          CALL ESI  
00419BD9   .  50            PUSH EAX  
00419BDA   .  FF15 60104000 CALL DWORD PTR DS:[<&msvbvm60.__vbaStrCa>;  msvbvm60.__vbaStrCat  
00419BE0   .  8BD0          MOV EDX,EAX  
00419BE2   .  8D4D A4       LEA ECX,DWORD PTR SS:[EBP-5C]  
00419BE5   .  FFD6          CALL ESI  
00419BE7   .  8D55 E0       LEA EDX,DWORD PTR SS:[EBP-20]  
00419BEA   .  50            PUSH EAX  
00419BEB   .  52            PUSH EDX  
00419BEC   >  E8 3F010000    CALL 1.00419D30                              ;  取得第二个结果 13 对应的字符“T”字。  
00419BF1   .  8BD0          MOV EDX,EAX  
00419BF3   .  8D4D A0       LEA ECX,DWORD PTR SS:[EBP-60]  
00419BF6   .  FFD6          CALL ESI  
00419BF8   .  50            PUSH EAX  
00419BF9   .  FF15 60104000 CALL DWORD PTR DS:[<&msvbvm60.__vbaStrCa>;  msvbvm60.__vbaStrCat  
00419BFF   .  8BD0          MOV EDX,EAX                                  ;  把得的字符连起来  
00419C01   .  8D4D 9C       LEA ECX,DWORD PTR SS:[EBP-64]  
00419C04   .  FFD6          CALL ESI  
00419C06   .  50            PUSH EAX  
00419C07   .  8D45 CC       LEA EAX,DWORD PTR SS:[EBP-34]  
00419C0A   .  50            PUSH EAX  
00419C0B   .  E8 20010000    CALL 1.00419D30                              ;  取得第三个结果 1c 对应的字符“c”字。  
00419C10   .  8BD0          MOV EDX,EAX  
00419C12   .  8D4D 98       LEA ECX,DWORD PTR SS:[EBP-68]  
00419C15   .  FFD6          CALL ESI  
00419C17   .  50            PUSH EAX  
00419C18   .  FF15 60104000 CALL DWORD PTR DS:[<&msvbvm60.__vbaStrCa>;  msvbvm60.__vbaStrCat  
00419C1E   .  8BD0          MOV EDX,EAX  
00419C20   .  8D4D 94       LEA ECX,DWORD PTR SS:[EBP-6C]  
00419C23   .  FFD6          CALL ESI  
00419C25   .  8D4D B8       LEA ECX,DWORD PTR SS:[EBP-48]  
00419C28   .  50            PUSH EAX  
00419C29   .  51            PUSH ECX  
00419C2A   .  E8 01010000    CALL 1.00419D30                              ;  取得第四个结果 33 对应的字符“z”字。  
00419C2F   .  8BD0          MOV EDX,EAX  
00419C31   .  8D4D 90       LEA ECX,DWORD PTR SS:[EBP-70]  
00419C34   .  FFD6          CALL ESI  
00419C36   .  50            PUSH EAX  
00419C37   .  FF15 60104000 CALL DWORD PTR DS:[<&msvbvm60.__vbaStrCa>;  msvbvm60.__vbaStrCat  
00419C3D   .  8BD0          MOV EDX,EAX                                  ;  连起来为 OTcz  
00419C3F   .  8D4D AC       LEA ECX,DWORD PTR SS:[EBP-54]  
00419C42   .  FFD6          CALL ESI  
00419C44   .  8D55 90       LEA EDX,DWORD PTR SS:[EBP-70]  
00419C47   .  8D45 94       LEA EAX,DWORD PTR SS:[EBP-6C]  
00419C4A   .  52            PUSH EDX  
00419C4B   .  8D4D 98       LEA ECX,DWORD PTR SS:[EBP-68]  
00419C4E   .  50            PUSH EAX  
00419C4F   .  8D55 9C       LEA EDX,DWORD PTR SS:[EBP-64]  
00419C52   .  51            PUSH ECX  
00419C53   .  8D45 A0       LEA EAX,DWORD PTR SS:[EBP-60]  
00419C56   .  52            PUSH EDX  
00419C57   .  8D4D A4       LEA ECX,DWORD PTR SS:[EBP-5C]  
00419C5A   .  50            PUSH EAX  
00419C5B   .  8D55 A8       LEA EDX,DWORD PTR SS:[EBP-58]  
00419C5E   .  51            PUSH ECX  
00419C5F   .  52            PUSH EDX  
00419C60   .  6A 07          PUSH 7  
00419C62   .  FF15 E8114000 CALL DWORD PTR DS:[<&msvbvm60.__vbaFreeS>;  msvbvm60.__vbaFreeStrList  
00419C68   .  B8 03000000    MOV EAX,3  
00419C6D   .  83C4 20       ADD ESP,20  
00419C70   .  66:03C3       ADD AX,BX  
00419C73   .  0F80 AF000000 JO 1.00419D28  
00419C79   .  8BD8          MOV EBX,EAX  
00419C7B   .  ^ E9 C2FBFFFF   JMP 1.00419842                              ;  返回去重复计算，第次取三位数，直到取完为止。  
00419C80   >  8B55 AC       MOV EDX,DWORD PTR SS:[EBP-54]  
00419C83   .  8D4D B0       LEA ECX,DWORD PTR SS:[EBP-50]                ;  得最终字符串记为 S="OTczKzI4NCs5NzMrMTU1KzQ1MSs2MTkrNzQw"  
00419C86   .  FF15 DC114000 CALL DWORD PTR DS:[<&msvbvm60.__vbaStrCo>;  msvbvm60.__vbaStrCopy  
00419C8C   .  9B            WAIT  
00419C8D   .  68 0D9D4100    PUSH 1.00419D0D  
```

。。。。。。跟进重要的算法 `CALL3 1.00419D30` 。。。。。。到这。。。

```
00419D30   $  55             PUSH EBP  
```

。。。。。。

```
00419D67   .  66:3BC1       CMP AX,CX  
00419D6A   .  7C 42         JL SHORT 1.00419DAE  
00419D6C   .  66:05 0100    ADD AX,1                                   ;  第一个结果加 1 即 E+1=F  
00419D70   .  8D4D D8       LEA ECX,DWORD PTR SS:[EBP-28]  
00419D73   .  70 7E         JO SHORT 1.00419DF3  
00419D75   .  0FBFD0        MOVSX EDX,AX  
00419D78   .  51            PUSH ECX  
00419D79   .  52            PUSH EDX  
00419D7A   .  68 EC794000    PUSH 1.004079EC                              ;  UNICODE "ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789+/"  
00419D7F   .  C745 E0 01000>MOV DWORD PTR SS:[EBP-20],1                   ;  固定字符串  
00419D86   .  C745 D8 02000>MOV DWORD PTR SS:[EBP-28],2  
00419D8D   .  FF15 E8104000 CALL DWORD PTR DS:[<&msvbvm60.rtcMidChar>;  msvbvm60.rtcMidCharBstr  
00419D93   .  8BD0          MOV EDX,EAX  
00419D95   .  8D4D E8       LEA ECX,DWORD PTR SS:[EBP-18]  
00419D98   .  FF15 50124000 CALL DWORD PTR DS:[<&msvbvm60.__vbaStrMo>;  msvbvm60.__vbaStrMove  
00419D9E   .  8D4D D8       LEA ECX,DWORD PTR SS:[EBP-28]  
00419DA1   .  FF15 24104000 CALL DWORD PTR DS:[<&msvbvm60.__vbaFreeV>;  msvbvm60.__vbaFreeVar  
00419DA7   .  68 DD9D4100    PUSH 1.00419DDD  
00419DAC   .  EB 2E         JMP SHORT 1.00419DDC                          ;  这一过程是取得固定字符串的第（F=15）位数即“O”字。  
```



## 汇编代码段 1

```
00419DAE   >  BA 7C7C4000    MOV EDX,1.00407C7C  

00419DB3   .  8D4D E8        LEA ECX,DWORD PTR SS:\[EBP-18\]  

00419DB6   .  FF15 DC114000  CALL DWORD PTR DS:\[<&msvbvm60.\_\_vbaStrCo>;  msvbvm60.\_\_vbaStrCopy  

00419DBC   .  68 DD9D4100    PUSH 1.00419DDD  

00419DC1   .  EB 19          JMP SHORT 1.00419DDC  

00419DC3   .  F645 FC 04     TEST BYTE PTR SS:\[EBP-4\],4  

00419DC7   .  74 09          JE SHORT 1.00419DD2  

00419DC9   .  8D4D E8        LEA ECX,DWORD PTR SS:\[EBP-18\]  

00419DCC   .  FF15 80124000  CALL DWORD PTR DS:\[<&msvbvm60.\_\_vbaFreeS>;  msvbvm60.\_\_vbaFreeStr  

00419DD2   >  8D4D D8        LEA ECX,DWORD PTR SS:\[EBP-28\]  

00419DD5   .  FF15 24104000  CALL DWORD PTR DS:\[<&msvbvm60.\_\_vbaFreeV>;  msvbvm60.\_\_vbaFreeVar  

00419DDB   .  C3             RETN  
```

`CALL4` `1.0041A2A0`

## 汇编代码段 2

```
0041A2A0   $  55             PUSH EBP  

0041A2A1   .  8BEC           MOV EBP,ESP  

0041A2A3   .  83EC 08        SUB ESP,8  

0041A2A6   .  68 F61A4000    PUSH <JMP.&msvbvm60.\_\_vbaExceptHandler>  ;  SE 处理程序安装  

0041A2AB   .  64:A1 0000000>MOV EAX,DWORD PTR FS:\[0\]  

0041A2B1   .  50             PUSH EAX  

0041A2B2   .  64:8925 00000>MOV DWORD PTR FS:\[0\],ESP  

0041A2B9   .  83EC 60        SUB ESP,60  

0041A2BC   .  53             PUSH EBX  

0041A2BD   .  56             PUSH ESI  

0041A2BE   .  57             PUSH EDI  

0041A2BF   .  8965 F8        MOV DWORD PTR SS:\[EBP-8\],ESP  

0041A2C2   .  C745 FC 181A4>MOV DWORD PTR SS:\[EBP-4\],1.00401A18  

0041A2C9   .  8B55 08        MOV EDX,DWORD PTR SS:\[EBP+8\]  

0041A2CC   .  33FF           XOR EDI,EDI  

0041A2CE   .  8D4D E8        LEA ECX,DWORD PTR SS:\[EBP-18\]  

0041A2D1   .  897D E8        MOV DWORD PTR SS:\[EBP-18\],EDI  

0041A2D4   .  897D DC        MOV DWORD PTR SS:\[EBP-24\],EDI  

0041A2D7   .  897D CC        MOV DWORD PTR SS:\[EBP-34\],EDI  

0041A2DA   .  897D BC        MOV DWORD PTR SS:\[EBP-44\],EDI  

0041A2DD   .  897D AC        MOV DWORD PTR SS:\[EBP-54\],EDI  

0041A2E0   .  FF15 DC114000  CALL DWORD PTR DS:\[<&msvbvm60.\_\_vbaStrCo>;  msvbvm60.\_\_vbaStrCopy  

0041A2E6   .  8B45 E8        MOV EAX,DWORD PTR SS:\[EBP-18\]  

0041A2E9   .  50             PUSH EAX  

0041A2EA   .  FF15 2C104000  CALL DWORD PTR DS:\[<&msvbvm60.\_\_vbaLenBs>;  msvbvm60.\_\_vbaLenBstr  

0041A2F0   .  8BC8           MOV ECX,EAX                          ;  得字符串 S 的长度  

0041A2F2   .  FF15 20114000  CALL DWORD PTR DS:\[<&msvbvm60.\_\_vbaI2I4>>;  msvbvm60.\_\_vbaI2I4  

0041A2F8   .  BB 01000000    MOV EBX,1  

0041A2FD   .  8945 94        MOV DWORD PTR SS:\[EBP-6C\],EAX  

0041A300   .  8BF3           MOV ESI,EBX  

0041A302   >  66:3B75 94     CMP SI,WORD PTR SS:\[EBP-6C\]  

0041A306   .  7F 77          JG SHORT 1.0041A37F  

0041A308   .  8D4D E8        LEA ECX,DWORD PTR SS:\[EBP-18\]  

0041A30B   .  8D55 CC        LEA EDX,DWORD PTR SS:\[EBP-34\]  

0041A30E   .  0FBFC6         MOVSX EAX,SI  

0041A311   .  894D B4        MOV DWORD PTR SS:\[EBP-4C\],ECX  

0041A314   .  52             PUSH EDX  

0041A315   .  8D4D AC        LEA ECX,DWORD PTR SS:\[EBP-54\]  

0041A318   .  50             PUSH EAX  

0041A319   .  8D55 BC        LEA EDX,DWORD PTR SS:\[EBP-44\]  

0041A31C   .  51             PUSH ECX  

0041A31D   .  52             PUSH EDX  

0041A31E   .  C745 D4 01000>MOV DWORD PTR SS:\[EBP-2C\],1  

0041A325   .  C745 CC 02000>MOV DWORD PTR SS:\[EBP-34\],2  

0041A32C   .  C745 AC 08400>MOV DWORD PTR SS:\[EBP-54\],4008  

0041A333   .  FF15 F0104000  CALL DWORD PTR DS:\[<&msvbvm60.rtcMidChar>;  msvbvm60.rtcMidCharVar  

0041A339   .  8D45 BC        LEA EAX,DWORD PTR SS:\[EBP-44\]        ;  得第一个字符“O”  

0041A33C   .  8D4D DC        LEA ECX,DWORD PTR SS:\[EBP-24\]  

0041A33F   .  50             PUSH EAX  

0041A340   .  51             PUSH ECX  

0041A341   .  FF15 98114000  CALL DWORD PTR DS:\[<&msvbvm60.\_\_vbaStrVa>;  msvbvm60.\_\_vbaStrVarVal  

0041A347   .  50             PUSH EAX  

0041A348   .  FF15 4C104000  CALL DWORD PTR DS:\[<&msvbvm60.rtcAnsiVal>;  msvbvm60.rtcAnsiValueBstr  

0041A34E   .  0FBFD0         MOVSX EDX,AX                           ;  转十六进制为 4F  

0041A351   .  03D7           ADD EDX,EDI                            ;  累加的值在 EDX 中，最终结果为 c43  

0041A353   .  8D4D DC        LEA ECX,DWORD PTR SS:\[EBP-24\]  

0041A356   .  70 6E          JO SHORT 1.0041A3C6  

0041A358   .  8BFA           MOV EDI,EDX  

0041A35A   .  FF15 80124000  CALL DWORD PTR DS:\[<&msvbvm60.\_\_vbaFreeS>;  msvbvm60.\_\_vbaFreeStr  

0041A360   .  8D45 BC        LEA EAX,DWORD PTR SS:\[EBP-44\]  

0041A363   .  8D4D CC        LEA ECX,DWORD PTR SS:\[EBP-34\]  

0041A366   .  50             PUSH EAX  

0041A367   .  51             PUSH ECX  

0041A368   .  6A 02          PUSH 2  

0041A36A   .  FF15 3C104000  CALL DWORD PTR DS:\[<&msvbvm60.\_\_vbaFreeV>;  msvbvm60.\_\_vbaFreeVarList  

0041A370   .  66:8BD3        MOV DX,BX  

0041A373   .  83C4 0C        ADD ESP,0C  

0041A376   .  66:03D6        ADD DX,SI  

0041A379   .  70 4B          JO SHORT 1.0041A3C6  

0041A37B   .  8BF2           MOV ESI,EDX  

0041A37D   .^ EB 83          JMP SHORT 1.0041A302                   ;  返回去循环，直到加完所字符的十六进制值为止。  

0041A37F   >  897D E0        MOV DWORD PTR SS:\[EBP-20\],EDI  

0041A382   .  68 B0A34100    PUSH 1.0041A3B0  

0041A387   .  EB 1D          JMP SHORT 1.0041A3A6  

0041A389   .  8D4D DC        LEA ECX,DWORD PTR SS:\[EBP-24\]  
```

------------------------------------------------------------------------

## 算法总结

*   用户名：`lhl8730`
*   假码：`1234-2222-1111`

1.  注册码由三部分组成，取用户名每一个数的十六进制值，取假码的第一部分十六进制值，取假码的第三部分十六进制值，进入关键 `CALL1` 运算，得几组数字再用 `+` 相连。我这得到的是 `SN=973+284+973+155+451+619+740`
2.  每次取 `SN` 的三位数进入关键 `CALL2` 运算，重复过算直止取完 `SN` 的每一个数。每次的结果进入关键 `CALL3` 运算得相应的字符，最后把所有的字符连起来。我这得字符串 `S=OTczKzI4NCs5NzMrMTU1KzQ1MSs2MTkrNzQw`
3.  关键 `CALL4` 是把每个字符的十六进制加起来。我这的结果为 `c43`，转为十进制为 `3139`。
4.  真的注册码为：`1234-3139-1111`
5.  注册机源码：（写的比较乱，让大家见笑了，还有一个问题是注册码第三部分的值不能超过 4 位，否则有溢出错误，希高手来指点一下。）

```vb
Private Sub Command1_Click()

Dim k As Double

Dim l As Double

Dim m As Double

Dim X1 As Long

Dim X2 As Long

Dim X3 As Long

t = Val(Text2.Text)

If t Mod 2 = 1 Then       '注册码第一部分是奇数运行。

n = Len(Text1.Text)

For i = 1 To n

k = Val(Text2.Text)

l = Val(Text4.Text)

h = Mid$(Text1.Text, i, 1)

m = Asc(h)

x = x & lhl1(k, l, m) & "+"

Next i

Else

n = Len(Text1.Text)

For i = 1 To n

k = Val(Text2.Text)

l = Val(Text4.Text)

h = Mid$(Text1.Text, i, 1)

m = Asc(h)

x = x & lhl0(k, l, m) & "+"

Next i

End If

u = Len(x) \ 3           '关键 CALL2，3 的算法

For i1 = 1 To u

X1 = Asc(Mid(x, 3 * (i1 - 1) + 1, 1))

X2 = Asc(Mid(x, 3 * (i1 - 1) + 2, 1))

X3 = Asc(Mid(x, 3 * (i1 - 1) + 3, 1))

Sn = Sn + lhl3(X1, X2, X3)

Next i1

u2 = Len(Sn)

For i2 = 1 To u2               '关键 CALL4 的算法

SN2 = SN2 + Asc(Mid(Sn, i2, 1))

Next i2

Text5.Text = Text2.Text & "-" & SN2 & "-" & Text4.Text

End Sub                       ’下面是调用的三个函数。

Function lhl1(a As Double, b As Double, d As Double) As Long     '注册码第一部分是奇数时，关键 CALL1 的算法。

Do

If a Mod 2 = 1 Then

If c = 0 Then

d = (d * 1) Mod b

Else

d = (d * c) Mod b

End If

a = a - 1

Else

If c = 0 Then

c = (d * d) Mod b

Else

c = (c * c) Mod b

End If

a = a \ 2

End If

Loop Until a = 0

lhl1 = d

End Function

Function lhl0(a As Double, b As Double, d As Double) As Long     '注册码第一部分是偶数时，关键 CALL1 的算法。

e = 1

Do

If a Mod 2 = 1 Then

If c = 0 Then

d = (d * 1) Mod b

Else

d = (e * c) Mod b

e = d

End If

a = a - 1

Else

If c = 0 Then

c = (d * d) Mod b

Else

c = (c * c) Mod b

End If

a = a \ 2

End If

Loop Until a = 0

lhl0 = d

End Function

Function lhl3(a As Long, b As Long, c As Long) As Variant      '关键 CALL2，3 的算法

x = "ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789+/"

a1 = (a \ &H4) + 1

a2 = (((a) And &H3) * &H10 + (b \ &H10)) + 1

a3 = (b And ("&H" & "0F")) * &H4 + 1
```



```
a4 = ((c) And (&H3F)) + 1  

b1 = Mid$(x, a1, 1)  

b2 = Mid$(x, a2, 1)  

b3 = Mid$(x, a3, 1)  

b4 = Mid$(x, a4, 1)  

lhl3 = b1 & b2 & b3 & b4  

End Function  
```

------------------------------------------------------------------------

【版权声明】本破文纯属技术交流, 转载请注明作者并保持文章的完整, 谢谢! 

`4dd466d0e1e0cc123afbe504da2462ca`
