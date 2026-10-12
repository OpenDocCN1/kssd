
# VisualToolbar1.5 带壳算法浅析

> 原文：[`www.pediy.com/kssd/pediy09/pediy09-40.htm`](https://www.pediy.com/kssd/pediy09/pediy09-40.htm)

- **【文章标题】**: `VisualToolbar1.5` 带壳算法浅析
- **【文章作者】**: `wyqzm`
- **【作者邮箱】**: 保密
- **【作者 QQ 号】**: 一般不用
- **【软件名称】**: `Visual Toolbar V1.50 For VC and Delphi`
- **【软件大小】**: `463 KB` [主程序]
- **【下载地址】**: `http://www.skycn.com/soft/16024.html`
- **【加壳方式】**: `UPX-Scrambler RC1.x -> ┫nT?L`
- **【保护方式】**: 取位计算, 明码比较
- **【编写语言】**: `VC++`
- **【使用工具】**: `OD` + 计算器
- **【操作平台】**: `WinXP sp2 中文版`
- **【作者声明】**: 只是感兴趣，没有其他目的。失误之处敬请诸位大侠赐教!

--------------------------------------------------------------------------------

**【软件介绍】**

`Visual Toolbar 1.0` 可以自动读入小位图文件 (支持 `16×16`，`24×24`，`32×32`，`48×48`), 自动合并所有小的位图，然后输出您整合以后的位图资源文件。此外，`Visual Toolbar` 允许用户动态调整按钮图标的顺序，支持鼠标托放！合并以后的位图，再次读入的时候，可以自动重新分割成各个合并前的小...

**【详细过程】**

**第一步**, 查壳, 为 `UPX-Scrambler RC1.x -> ┫nT?L`, 呵呵, 加密的压缩壳, 今天就不脱壳了, 载入到 `OD` 提示压缩, `F9` 直接运行试试, 提示注册了, 输入试练码, 用户名: `wyqzm`, 注册码: `151515`, 点注册按钮, 提示: `Incorreect registration code!`, 有提示就更好说了.

**第二步**, `Ctrl+F2` 重来, `F9` 运行, 到注册窗口, 输入试练码, 先不点注册, 在命令行下断 `BP MessageBoxA`, 很容易来到下面.

看看注释的内容, 很明显的是 `MFC` 程序.

```
00407790    6A FF            PUSH -1                  // 首部,在此处下断
00407792    68 60A64900      PUSH VisualTo.0049A660
00407797    64:A1 00000000   MOV EAX,DWORD PTR FS:[0]
0040779D    50               PUSH EAX
0040779E    64:8925 0000000>MOV DWORD PTR FS:[0],ESP
004077A5    83EC 10          SUB ESP,10
004077A8    53               PUSH EBX
004077A9    55               PUSH EBP
004077AA    56               PUSH ESI
004077AB    57               PUSH EDI
004077AC    8BF1             MOV ESI,ECX
004077AE    6A 01            PUSH 1
004077B0    E8 6D1B0900      CALL VisualTo.00499322                   ; JMP 到 MFC42.#6334_?UpdateData@CWnd@@QAEHH@Z
004077B5    8D4C24 14        LEA ECX,DWORD PTR SS:[ESP+14]            ; 取用户名
004077B9    E8 40180900      CALL VisualTo.00498FFE                   ; JMP 到 MFC42.#540_??0CString@@QAE@XZ
004077BE    8D4424 14        LEA EAX,DWORD PTR SS:[ESP+14]
004077C2    33DB             XOR EBX,EBX
004077C4    50               PUSH EAX
004077C5    8D8E 4C0E0000    LEA ECX,DWORD PTR DS:[ESI+E4C]
004077CB    895C24 2C        MOV DWORD PTR SS:[ESP+2C],EBX
004077CF    E8 AA190900      CALL VisualTo.0049917E                   ; 通过 GetWindowTextA 取机器码
004077D4    51               PUSH ECX
004077D5    8DBE 900E0000    LEA EDI,DWORD PTR DS:[ESI+E90]
004077DB    8BCC             MOV ECX,ESP
004077DD    896424 20        MOV DWORD PTR SS:[ESP+20],ESP
004077E1    57               PUSH EDI
004077E2    E8 05180900      CALL VisualTo.00498FEC                   ; JMP 到 MFC42.#535_??0CString@@QAE@ABV0@@Z
004077E7    51               PUSH ECX
004077E8    8D5424 1C        LEA EDX,DWORD PTR SS:[ESP+1C]
004077EC    8BCC             MOV ECX,ESP
004077EE    896424 20        MOV DWORD PTR SS:[ESP+20],ESP
004077F2    52               PUSH EDX
004077F3    C64424 34 01     MOV BYTE PTR SS:[ESP+34],1
004077F8    E8 EF170900      CALL VisualTo.00498FEC                   ; JMP 到 MFC42.#535_??0CString@@QAE@ABV0@@Z
004077FD    8D4424 18        LEA EAX,DWORD PTR SS:[ESP+18]
00407801    885C24 30        MOV BYTE PTR SS:[ESP+30],BL
00407805    50               PUSH EAX
00407806    E8 65A3FFFF      CALL VisualTo.00401B70                   ; 算法 CALL,F7 跟进
0040780B    83C4 0C          ADD ESP,0C
0040780E    8B0F             MOV ECX,DWORD PTR DS:[EDI]
00407810    C64424 28 02     MOV BYTE PTR SS:[ESP+28],2
00407815    8B41 F8          MOV EAX,DWORD PTR DS:[ECX-8]
00407818    83F8 04          CMP EAX,4                              ; 比较用户名的位数是否大于 4
0040781B    7F 35            JG SHORT VisualTo.00407852         // 用户名位数大于 4 则有效,否则出现出错提示
0040781D    8D4C24 18        LEA ECX,DWORD PTR SS:[ESP+18]
00407821    E8 D8170900      CALL VisualTo.00498FFE                   ; JMP 到 MFC42.#540_??0CString@@QAE@XZ
00407826    6A 68            PUSH 68
00407828    8D4C24 1C        LEA ECX,DWORD PTR SS:[ESP+1C]
0040782C    C64424 2C 03     MOV BYTE PTR SS:[ESP+2C],3
00407831    E8 84190900      CALL VisualTo.004991BA                   ; JMP 到 MFC42.#4160_?LoadStringA@CString@@QAEHI@Z
00407836    8B5424 18        MOV EDX,DWORD PTR SS:[ESP+18]
0040783A    53               PUSH EBX
0040783B    53               PUSH EBX
0040783C    52               PUSH EDX
0040783D    8BCE             MOV ECX,ESI
0040783F    E8 D81A0900      CALL VisualTo.0049931C                   ; JMP 到 MFC42.#4224_?MessageBoxA@CWnd@@QAEHPBD0I@Z
00407844    C64424 28 02     MOV BYTE PTR SS:[ESP+28],2
00407849    8D4C24 18        LEA ECX,DWORD PTR SS:[ESP+18]
0040784D    E9 CB000000      JMP VisualTo.0040791D
00407852    8B4C24 10        MOV ECX,DWORD PTR SS:[ESP+10]            ; 取出已算完的注册码
00407856    8B86 940E0000    MOV EAX,DWORD PTR DS:[ESI+E94]          ; 将假码放入 eax
0040785C    51               PUSH ECX                                 ; 真注册码进栈,此处也可以制作内存注册机
0040785D    50               PUSH EAX                                 ; 假注册码进栈
0040785E    FF15 70D84900    CALL DWORD PTR DS:[49D870]               ; 通过函数_mbscmp 比较,这里可以制作内存注册机
00407864    83C4 08          ADD ESP,8
00407867    85C0             TEST EAX,EAX                              ; 比较
00407869    75 78            JNZ SHORT VisualTo.004078E3             ; 爆破点
0040786B    891D F0164D00    MOV DWORD PTR DS:[4D16F0],EBX
00407871    E8 A01A0900      CALL VisualTo.00499316                   ; JMP 到 MFC42.#1168_?AfxGetModuleState@@YGPAVAFX_MODULE_STATE@@XZ
00407876    8B3F             MOV EDI,DWORD PTR DS:[EDI]
00407878    8B68 04          MOV EBP,DWORD PTR DS:[EAX+4]
0040787B    57               PUSH EDI
0040787C    68 D4534C00      PUSH VisualTo.004C53D4                   ; ASCII "USER"
00407881    68 C4534C00      PUSH VisualTo.004C53C4                   ; ASCII "VisualToolbar10"
00407886    8BCD             MOV ECX,EBP
00407888    E8 831A0900      CALL VisualTo.00499310                   ; 将注册用户名写入文件
0040788D    8B5424 10        MOV EDX,DWORD PTR SS:[ESP+10]
00407891    8BCD             MOV ECX,EBP
00407893    52               PUSH EDX
00407894    68 C0534C00      PUSH VisualTo.004C53C0                   ; ASCII "SN"
00407899    68 C4534C00      PUSH VisualTo.004C53C4                   ; ASCII "VisualToolbar10"  //
0040789E    E8 6D1A0900      CALL VisualTo.00499310                   ; JMP 到 MFC42.#6403_?WriteProfileStringA@CWinApp@@QAEHPBD00@Z
004078A3    8D4C24 18        LEA ECX,DWORD PTR SS:[ESP+18]
004078A7    E8 52170900      CALL VisualTo.00498FFE                   ; JMP 到 MFC42.#540_??0CString@@QAE@XZ
004078AC    6A 67            PUSH 67
004078AE    8D4C24 1C        LEA ECX,DWORD PTR SS:[ESP+1C]
004078B2    C64424 2C 04     MOV BYTE PTR SS:[ESP+2C],4
004078B7    E8 FE180900      CALL VisualTo.004991BA                   ; JMP 到 MFC42.#4160_?LoadStringA@CString@@QAEHI@Z
004078BC    8B4424 18        MOV EAX,DWORD PTR SS:[ESP+18]
004078C0    53               PUSH EBX
004078C1    53               PUSH EBX
004078C2    50               PUSH EAX
004078C3    8BCE             MOV ECX,ESI
004078C5    E8 521A0900      CALL VisualTo.0049931C                   ; JMP 到 MFC42.#4224_?MessageBoxA@CWnd@@QAEHPBD0I@Z
004078CA    53               PUSH EBX
004078CB    FF15 14DA4900    CALL DWORD PTR DS:[49DA14]               ; USER32.PostQuitMessage
004078D1    8BCE             MOV ECX,ESI
```



```
004078D3    E8 201A0900     CALL VisualTo.004992F8                 ; JMP 到 MFC42.#4853_?OnOK@CDialog@@MAEXXZ  

004078D8    C64424 28 02    MOV BYTE PTR SS:[ESP+28],2  

004078DD    8D4C24 18       LEA ECX,DWORD PTR SS:[ESP+18]  

004078E1    EB 3A           JMP SHORT VisualTo.0040791D  

004078E3    8D4C24 1C       LEA ECX,DWORD PTR SS:[ESP+1C]  

004078E7    C705 F0164D00 0>MOV DWORD PTR DS:[4D16F0],1  

004078F1    E8 08170900     CALL VisualTo.00498FFE                 ; JMP 到 MFC42.#540_??0CString@@QAE@XZ  

004078F6    6A 66           PUSH 66  

004078F8    8D4C24 20       LEA ECX,DWORD PTR SS:[ESP+20]  

004078FC    C64424 2C 05    MOV BYTE PTR SS:[ESP+2C],5  

00407901    E8 B4180900     CALL VisualTo.004991BA                 ; JMP 到 MFC42.#4160_?LoadStringA@CString@@QAEHI@Z  

00407906    8B4C24 1C       MOV ECX,DWORD PTR SS:[ESP+1C]  

0040790A    53              PUSH EBX  

0040790B    53              PUSH EBX  

0040790C    51              PUSH ECX  

0040790D    8BCE            MOV ECX,ESI  

0040790F    E8 081A0900     CALL VisualTo.0049931C                 ; JMP 到 MFC42.#4224_?MessageBoxA@CWnd@@QAEHPBD0I@Z  

00407914    C64424 28 02    MOV BYTE PTR SS:[ESP+28],2    //返回到这里,往上看  

00407919    8D4C24 1C       LEA ECX,DWORD PTR SS:[ESP+1C]  

0040791D    E8 C4160900     CALL VisualTo.00498FE6                 ; JMP 到 MFC42.#800_??1CString@@QAE@XZ  

00407922    8D4C24 10       LEA ECX,DWORD PTR SS:[ESP+10]  

00407926    885C24 28       MOV BYTE PTR SS:[ESP+28],BL  

0040792A    E8 B7160900     CALL VisualTo.00498FE6                 ; JMP 到 MFC42.#800_??1CString@@QAE@XZ  

0040792F    8D4C24 14       LEA ECX,DWORD PTR SS:[ESP+14]  

00407933    C74424 28 FFFFF>MOV DWORD PTR SS:[ESP+28],-1  

0040793B    E8 A6160900     CALL VisualTo.00498FE6                 ; JMP 到 MFC42.#800_??1CString@@QAE@XZ  

00407940    8B4C24 20       MOV ECX,DWORD PTR SS:[ESP+20]  

00407944    5F              POP EDI  

00407945    5E              POP ESI  

00407946    5D              POP EBP  

00407947    64:890D 0000000>MOV DWORD PTR FS:[0],ECX  

0040794E    5B              POP EBX  

0040794F    83C4 1C         ADD ESP,1C  

00407952    C3              RETN  

00407953    90              NOP  

00407954    90              NOP  

00407955    90              NOP  

00407956    90              NOP  

00407957    90              NOP  

00407958    90              NOP  

00407959    90              NOP  

0040795A    90              NOP  

0040795B    90              NOP  

0040795C    90              NOP  

0040795D    90              NOP  

0040795E    90              NOP  

0040795F    90              NOP  

00407960    56              PUSH ESI  

00407961    8BF1            MOV ESI,ECX  

00407963    E8 84190900     CALL VisualTo.004992EC                 ; JMP 到 MFC42.#4710_?OnInitDialog@CDialog@@UAEHXZ  

00407968    E8 C3A0FFFF     CALL VisualTo.00401A30  

0040796D    6A 00           PUSH 0  

0040796F    8BCE            MOV ECX,ESI  

00407971    8986 8C0E0000   MOV DWORD PTR DS:[ESI+E8C],EAX  

00407977    E8 A6190900     CALL VisualTo.00499322                 ; JMP 到 MFC42.#6334_?UpdateData@CWnd@@QAEHH@Z  

0040797C    6A 20           PUSH 20  

0040797E    6A 20           PUSH 20  

00407980    68 9A000000     PUSH 9A  

00407985    6A 20           PUSH 20  

00407987    6A 20           PUSH 20  

00407989    68 9A000000     PUSH 9A  

0040798E    8D4E 60         LEA ECX,DWORD PTR DS:[ESI+60]  

00407991    E8 7ABEFFFF     CALL VisualTo.00403810  

00407996    6A 20           PUSH 20  

00407998    6A 20           PUSH 20  

0040799A    68 9C000000     PUSH 9C  

0040799F    6A 20           PUSH 20  

004079A1    6A 20           PUSH 20  

004079A3    68 9C000000     PUSH 9C  

004079A8    8D8E A8090000   LEA ECX,DWORD PTR DS:[ESI+9A8]  

004079AE    E8 5DBEFFFF     CALL VisualTo.00403810  

004079B3    6A 20           PUSH 20  

004079B5    6A 20           PUSH 20  

004079B7    68 9B000000     PUSH 9B  

004079BC    6A 20           PUSH 20  

004079BE    6A 20           PUSH 20  

004079C0    68 9B000000     PUSH 9B  

004079C5    8D8E 04050000   LEA ECX,DWORD PTR DS:[ESI+504]  

004079CB    E8 40BEFFFF     CALL VisualTo.00403810  

004079D0    B8 01000000     MOV EAX,1  

004079D5    5E              POP ESI  

004079D6    C3              RETN  

00407953    90              NOP  

//算法 CALL 部分  

00401B70    6A FF           PUSH -1        //首部  

00401B72    68 3F9F4900     PUSH VisualTo.00499F3F  

00401B77    64:A1 00000000  MOV EAX,DWORD PTR FS:[0]  

00401B7D    50              PUSH EAX  

00401B7E    64:8925 0000000>MOV DWORD PTR FS:[0],ESP  

00401B85    83EC 20         SUB ESP,20  

00401B88    53              PUSH EBX  

00401B89    55              PUSH EBP  

00401B8A    56              PUSH ESI  

00401B8B    33F6            XOR ESI,ESI  

00401B8D    57              PUSH EDI  

00401B8E    897424 1C       MOV DWORD PTR SS:[ESP+1C],ESI  

00401B92    8D4C24 20       LEA ECX,DWORD PTR SS:[ESP+20]  

00401B96    C74424 38 02000>MOV DWORD PTR SS:[ESP+38],2  

00401B9E    E8 5B740900     CALL VisualTo.00498FFE                 ; JMP 到 MFC42.#540_??0CString@@QAE@XZ  

00401BA3    8B4424 48       MOV EAX,DWORD PTR SS:[ESP+48]        ; 取用户名  

00401BA7    C64424 38 03    MOV BYTE PTR SS:[ESP+38],3  

00401BAC    C64424 24 56    MOV BYTE PTR SS:[ESP+24],56  

00401BB1    C64424 25 69    MOV BYTE PTR SS:[ESP+25],69  

00401BB6    8B68 F8         MOV EBP,DWORD PTR DS:[EAX-8]  

00401BB9    C64424 26 73    MOV BYTE PTR SS:[ESP+26],73  

00401BBE    83FD 0A         CMP EBP,0A  

00401BC1    C64424 27 75    MOV BYTE PTR SS:[ESP+27],75  

00401BC6    C64424 28 61    MOV BYTE PTR SS:[ESP+28],61  

00401BCB    C64424 29 6C    MOV BYTE PTR SS:[ESP+29],6C  

00401BD0    C64424 2A 54    MOV BYTE PTR SS:[ESP+2A],54  

00401BD5    C64424 2B 42    MOV BYTE PTR SS:[ESP+2B],42  

00401BDA    C64424 2C 31    MOV BYTE PTR SS:[ESP+2C],31  

00401BDF    C64424 2D 30    MOV BYTE PTR SS:[ESP+2D],30  

00401BE4    7C 05           JL SHORT VisualTo.00401BEB  

00401BE6    BD 0A000000     MOV EBP,0A  

00401BEB    55              PUSH EBP  

00401BEC    E8 1F740900     CALL VisualTo.00499010                 ; JMP 到 MFC42.#823_??2@YAPAXI@Z  

00401BF1    83C4 04         ADD ESP,4  

00401BF4    8D4C24 14       LEA ECX,DWORD PTR SS:[ESP+14]  

00401BF8    8BF8            MOV EDI,EAX  

00401BFA    E8 FF730900     CALL VisualTo.00498FFE                 ; JMP 到 MFC42.#540_??0CString@@QAE@XZ  

00401BFF    68 30164D00     PUSH VisualTo.004D1630  

00401C04    8D4C24 14       LEA ECX,DWORD PTR SS:[ESP+14]  

00401C08    C64424 3C 04    MOV BYTE PTR SS:[ESP+3C],4  

00401C0D    E8 F8730900     CALL VisualTo.0049900A                 ; JMP 到 MFC42.#537_??0CString@@QAE@PBD@Z  

00401C12    3BEE            CMP EBP,ESI  

00401C14    C64424 38 05    MOV BYTE PTR SS:[ESP+38],5  

00401C19    7E 53           JLE SHORT VisualTo.00401C6E  

00401C1B    8D4424 24       LEA EAX,DWORD PTR SS:[ESP+24]  

00401C1F    2BC7            SUB EAX,EDI  

00401C21    894424 18       MOV DWORD PTR SS:[ESP+18],EAX  

00401C25    EB 04           JMP SHORT VisualTo.00401C2B  

00401C27    8B4424 18       MOV EAX,DWORD PTR SS:[ESP+18]        ; 算法开始  

00401C2B    8B4C24 48       MOV ECX,DWORD PTR SS:[ESP+48]  
```



## 汇编代码

```
00401C2F    8A140E          MOV DL,BYTE PTR DS:[ESI+ECX]             ; 取注册名的一位,第 1 次取第 1 位,依此类推  
00401C32    8B4C24 44       MOV ECX,DWORD PTR SS:[ESP+44]           ; 将机器码放入 ecx  
00401C36    8A1C0E          MOV BL,BYTE PTR DS:[ESI+ECX]             ; 将机器码相应位的 16 进制值放入 BL  
00401C39    8D0C3E          LEA ECX,DWORD PTR DS:[ESI+EDI]          ;  
00401C3C    8A0408          MOV AL,BYTE PTR DS:[EAX+ECX]             ; 将常数 VisualTB10 相应位放入 AL  
00401C3F    32C3            XOR AL,BL                                ; 机器码的相应位与常量 VisualTB10 的相应位异或  
00401C41    32C2            XOR AL,DL                                ; 将用户名相应每 1 位与前一步结果异或,放入 AL  
00401C43    0FBED0          MOVSX EDX,AL                             ; 把计算的结果放入到 edx,前位不足处符号扩展  
00401C46    8801            MOV BYTE PTR DS:[ECX],AL                 ; 将计算的结果保存在[ecx]  
00401C48    52              PUSH EDX                                 ; 计算结果进栈  
00401C49    8D4424 18       LEA EAX,DWORD PTR SS:[ESP+18]  
00401C4D    68 34504C00     PUSH VisualTo.004C5034                   ; 取格式化操作符  
00401C52    50              PUSH EAX                                 ; 压入函数地址  
00401C53    E8 A0730900     CALL VisualTo.00498FF8                   ; 调用 CString 的 format()方法将结果格式化为十进制  
00401C58    83C4 0C         ADD ESP,0C  
00401C5B    8D4C24 14       LEA ECX,DWORD PTR SS:[ESP+14]            ; 将地址赋给 ecx  
00401C5F    51              PUSH ECX  
00401C60    8D4C24 14       LEA ECX,DWORD PTR SS:[ESP+14]  
00401C64    E8 9B730900     CALL VisualTo.00499004                   ; JMP 到 MFC42.#939_??YCString@@QAEABV0@ABV0@@Z  
00401C69    46              INC ESI                                  ; ESI 加 1  
00401C6A    3BF5            CMP ESI,EBP                              ; 与注册名的位数比较  
00401C6C    7C B9           JL SHORT VisualTo.00401C27               ; 小于则跳回 0401C27 处继续计算  
00401C6E    57              PUSH EDI  
00401C6F    E8 6C730900     CALL VisualTo.00498FE0                   ; JMP 到 MFC42.#825_??3@YAXPAX@Z  
00401C74    8B7424 44       MOV ESI,DWORD PTR SS:[ESP+44]  
00401C78    83C4 04         ADD ESP,4  
00401C7B    8D5424 10       LEA EDX,DWORD PTR SS:[ESP+10]  
00401C7F    8BCE            MOV ECX,ESI  
00401C81    52              PUSH EDX  
00401C82    E8 65730900     CALL VisualTo.00498FEC                   ; JMP 到 MFC42.#535_??0CString@@QAE@ABV0@@Z  
00401C87    BB 01000000     MOV EBX,1  
00401C8C    895C24 1C       MOV DWORD PTR SS:[ESP+1C],EBX  
00401C90    8D4C24 10       LEA ECX,DWORD PTR SS:[ESP+10]  
00401C94    C64424 38 04    MOV BYTE PTR SS:[ESP+38],4  
00401C99    E8 48730900     CALL VisualTo.00498FE6                   ; JMP 到 MFC42.#800_??1CString@@QAE@XZ  
00401C9E    8D4C24 14       LEA ECX,DWORD PTR SS:[ESP+14]  
00401CA2    C64424 38 03    MOV BYTE PTR SS:[ESP+38],3  
00401CA7    E8 3A730900     CALL VisualTo.00498FE6                   ; JMP 到 MFC42.#800_??1CString@@QAE@XZ  
00401CAC    8D4C24 20       LEA ECX,DWORD PTR SS:[ESP+20]  
00401CB0    C64424 38 02    MOV BYTE PTR SS:[ESP+38],2  
00401CB5    E8 2C730900     CALL VisualTo.00498FE6                   ; JMP 到 MFC42.#800_??1CString@@QAE@XZ  
00401CBA    8D4C24 44       LEA ECX,DWORD PTR SS:[ESP+44]  
00401CBE    885C24 38       MOV BYTE PTR SS:[ESP+38],BL  
00401CC2    E8 1F730900     CALL VisualTo.00498FE6                   ; JMP 到 MFC42.#800_??1CString@@QAE@XZ  
00401CC7    8D4C24 48       LEA ECX,DWORD PTR SS:[ESP+48]  
00401CCB    C64424 38 00    MOV BYTE PTR SS:[ESP+38],0  
00401CD0    E8 11730900     CALL VisualTo.00498FE6                   ; JMP 到 MFC42.#800_??1CString@@QAE@XZ  
00401CD5    8B4C24 30       MOV ECX,DWORD PTR SS:[ESP+30]  
00401CD9    8BC6            MOV EAX,ESI  
00401CDB    5F              POP EDI  
00401CDC    5E              POP ESI  
00401CDD    5D              POP EBP  
00401CDE    5B              POP EBX  
00401CDF    64:890D 00000000 MOV DWORD PTR FS:[0],ECX  
00401CE6    83C4 2C         ADD ESP,2C  
00401CE9    C3              RETN                                     ; CALL 结束,返回  
```

## 说明

没时间写算法注册机了，偷懒留下内存注册机吧！

- 中断地址:`40785E`
- 中断次数:`1`
- 第一字节:`FF`
- 指令长度:`6`

选择内存方式(寄存器):`ECX`

我是一只小菜鸟，路过的大侠不要笑话我啊！

--------------------------------------------------------------------------------

## 经验总结

1. `UPX`的壳主要是作为压缩之用，在内存中完全解码之后，其实跟没加壳的程序是一样的
2. 关键点比较容易找到，但注册码的生成要注意，在比较的时候，注册码是直接显示的，所以注册码的计算肯定在前面已经完成了，于是向上找很容易找到那个算法 `CALL`，让我们再次熟悉了算法流程
3. 此程序与注册名、机器码及常量字符 `"VisualTB10"`相关。
4. 注册算法还是比较典型的，可惜作者最后还是用了明码比较。

--------------------------------------------------------------------------------

## 版权声明

本文原创于看雪技术论坛，转载请注明作者并保持文章的完整，谢谢！

`2007 年 02 月 05 日 16:30:11`

`4dd466d0e1e0cc123afbe504da2462ca`
