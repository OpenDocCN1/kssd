
# Flash Cam 1.79

> 原文：[`www.pediy.com/kssd/pediy05/pediy50102.htm`](https://www.pediy.com/kssd/pediy05/pediy50102.htm)

Flash Cam 1.79 注册算法分析与注册机制作以及爆破方法

工具：`ollydbg` 1.06

运行 `Flash Cam`，进入注册画面，然后运行 `ollydbg`，单击 `[file]` 选单中的 `[Attach]`，找到 `FlashCam` 进程。

确认后，进入 `ollydbg` 主画面，按 `Alt+E` 出现 `[Executable Modules]`，选中 `user32.dll`，再按 `Ctrl+N`。

出现 `[Names in user32]`，找到 `showwindow` 后，按 `F2` 下断点，按 `Alt+C` 回到 `[CPU-main thread]`，按 `F9` 运行。

在 `FlashCam` 注册画面中输入 `RegCode`，确认后，程序在所设断点处被拦截，清除断点后，反复按 `Ctrl+F9`，回到程序领空。然后一步一步按 `F8`，注意观察屏幕左上角和右下角，如果出现了你输入的 `RegCode` 或者其他可疑的字符串，则放慢脚步，在当前执行行的前面几行设断点，保证重新注册时程序能在出错窗口显示前被拦截。好了，再次注册更踪，这样，程序在出错窗口显示前便被拦截，然后按 `F7` 单步更踪，不要放过任何可疑的 `Call`（呵呵，这是对菜鸟而言啦，我可是直接就到了这里哦），最后就来到了这里：

```
00516B5B  50                 PUSH EAX
00516B5C  B9 20000000        MOV ECX,20
00516B61  BA F19D0000        MOV EDX,9DF1
00516B66  B8 1A000000        MOV EAX,1A
00516B6B  E8 70FCFFFF        CALL FlashCam.005167E0     //注册码计算过程
00516B70  8B45 E8            MOV EAX,DWORD PTR SS:[EBP-18]  //真正的注册码
00516B73  8B55 FC            MOV EDX,DWORD PTR SS:[EBP-4]   //你输入的假的注册码
00516B76  E8 B5D6EEFF        CALL FlashCam.00404230     //比较过程
00516B7B  75 04              JNZ SHORT FlashCam.00516B81    //不相等则 over

//////////////////////////////////////////////////////////////////////////////////////
////////--------00516B6B    CALL FlashCam.005167E0-------///////////////////////////////
//////////////////////////////////////////////////////////////////////////////////////

005167E0  55                 PUSH EBP
005167E1  8BEC               MOV EBP,ESP
005167E3  83C4 E8            ADD ESP,-18
005167E6  53                 PUSH EBX
005167E7  56                 PUSH ESI
005167E8  894D F8            MOV DWORD PTR SS:[EBP-8],ECX
005167EB  8BF2               MOV ESI,EDX
005167ED  8945 FC            MOV DWORD PTR SS:[EBP-4],EAX
005167F0  FF75 10            PUSH DWORD PTR SS:[EBP+10]
005167F3  FF75 0C            PUSH DWORD PTR SS:[EBP+C]
005167F6  8D4D F2            LEA ECX,DWORD PTR SS:[EBP-E]
005167F9  8D55 F4            LEA EDX,DWORD PTR SS:[EBP-C]
005167FC  8D45 F6            LEA EAX,DWORD PTR SS:[EBP-A]
005167FF  E8 AC4BEFFF        CALL FlashCam.0040B3B0
00516804  0FB755 F4          MOVZX EDX,WORD PTR SS:[EBP-C]  //变量 Month
00516808  8BDA               MOV EBX,EDX
0051680A  0FB745 F6          MOVZX EAX,WORD PTR SS:[EBP-A]  //变量 Year
0051680E  03D8               ADD EBX,EAX                     //ebx=Month+Year
00516810  69C8 A6030000      IMUL ECX,EAX,3A6                //ecx=Year*0x3a6
00516816  03D9               ADD EBX,ECX                     //ebx=ebx+ecx
00516818  69CA 4C010000      IMUL ECX,EDX,14C                //ecx=Month*0x14c
0051681E  03D9               ADD EBX,ECX                     //ebx=ebx+ecx
00516820  0FAFD0             IMUL EDX,EAX                    //edx=Month*Year
00516823  03DA               ADD EBX,EDX                     //ebx=ebx+edx
00516825  0FB745 F4          MOVZX EAX,WORD PTR SS:[EBP-C]  //Month
00516829  0FB755 F6          MOVZX EDX,WORD PTR SS:[EBP-A]  //Year
0051682D  F7EA               IMUL EDX                        //eax=Month*Year
0051682F  3345 FC            XOR EAX,DWORD PTR SS:[EBP-4]   //eax=eax ^ 0x01a
00516832  03D8               ADD EBX,EAX                     //ebx=ebx+eax
00516834  E8 BF19FFFF        CALL FlashCam.005081F8          //分析见下
00516839  B9 E7030000        MOV ECX,3E7                     //ecx=0x3e7
0051683E  99                 CDQ
0051683F  F7F9               IDIV ECX
00516841  8BC2               MOV EAX,EDX                     //eax=eax % ecx
00516843  03D8               ADD EBX,EAX                     //ebx=ebx+eax
00516845  0FB745 F4          MOVZX EAX,WORD PTR SS:[EBP-C]   //eax=Month
00516849  85C0               TEST EAX,EAX
0051684B  76 06              JBE SHORT FlashCam.00516853     //循环
0051684D  0FAFDE             IMUL EBX,ESI                    //ebx=ebx*0x9df1
00516850  48                 DEC EAX                         //循环 Month 次
00516851  75 FA              JNZ SHORT FlashCam.0051684D
00516853  0FB745 F6          MOVZX EAX,WORD PTR SS:[EBP-A]   //eax=Year
00516857  85C0               TEST EAX,EAX
00516859  76 06              JBE SHORT FlashCam.00516861     //循环
0051685B  035D F8            ADD EBX,DWORD PTR SS:[EBP-8]    //ebx=ebx+0x20
0051685E  48                 DEC EAX                         //循环 Year 次
0051685F  75 FA              JNZ SHORT FlashCam.0051685B
00516861  8BC3               MOV EAX,EBX                     //eax=ebx
00516863  99                 CDQ
00516864  33C2               XOR EAX,EDX                     //eax=eax ^ 0xffffffff
00516866  2BC2               SUB EAX,EDX                     //eax=eax-0xffffffff
00516868  8BD8               MOV EBX,EAX
0051686A  8BC3               MOV EAX,EBX
0051686C  B9 FFC99A3B        MOV ECX,3B9AC9FF                //ecx=0x3b9ac9ff
00516871  33D2               XOR EDX,EDX
00516873  F7F1               DIV ECX                         //edx=eax % ecx
00516875  8BDA               MOV EBX,EDX                     //edx->ebx
00516877  895D E8            MOV DWORD PTR SS:[EBP-18],EBX
0051687A  33C0               XOR EAX,EAX
0051687C  8945 EC            MOV DWORD PTR SS:[EBP-14],EAX
0051687F  DF6D E8            FILD QWORD PTR SS:[EBP-18]      //ebx 转换为十进制,呵呵，这就是
                                //我们苦苦追寻的东西了。当然
                                //还不完美，因为现在得到的仅仅是
                                //去掉“-”连接符号后的 RegCode。
                                //后面的工作就不需要我讲了吧，
                                //哎，为了菜鸟，我就再多说一句吧，
                                //分别在此串的第 4 位和第 7 位添上
                                //“-”，就是 RegCode 了。哈哈，简单吧。

//////////////////////////////////////////////////////////////////////////////////////
////////--------00516834    CALL FlashCam.005081F8-------///////////////////////////////
//////////////////////////////////////////////////////////////////////////////////////

005081F8  55                 PUSH EBP
、、
00508251  6A 00              PUSH 0
00508253  6A 00              PUSH 0
00508255  6A 00              PUSH 0
00508257  E8 CCF8EFFF        CALL FlashCam.00407B28          ; JMP to kernel32.GetVolumeInformationA
0050825C  8B03               MOV EAX,DWORD PTR DS:[EBX]      //卷序号->eax
0050825E  99                 CDQ
0050825F  33C2               XOR EAX,EDX                     //eax ^ 0xffffffff ->eax
00508261  2BC2               SUB EAX,EDX                     //eax - 0xffffffff ->eax
、、
0050828B  C3                 RETN
```

## 破解方法

1. 爆破

修改 `00516b7b` 处 `7504` 为 `9090`（注意：由于此软件已加壳，所以必须脱壳后才能修改代码）

2. 注册机

前面的分析，基本上就是标准的 `c` 程序了。呵呵，为了便于阅读，我还是再整理一遍吧。（`c builder 6` 通过）

```c
DWORD nBL;
LPTSTR lpB;
LPCTSTR lpRPN;
LPTSTR lpVNB;
DWORD nVNS;
LPDWORD lpVSN;
LPDWORD lpMCL;
LPDWORD lpFSF;
LPTSTR lpFSNB;
DWORD nFSNS;
int i;
AnsiString RegCode;
Word Year, Month, Day;
unsigned long eax, ebx, ecx, edx, esi, lpVSN;
char szHome[MAX_PATH];

BitBtn1->Enabled = False;
TDateTime dtPresent = Now();
DecodeDate(dtPresent, Year, Month, Day);

ebx = Year + Month;
ecx = Year * 0x3a6;
ebx = ebx + ecx;
ecx = Month * 0x14c;
ebx = ebx + ecx;
edx = Year * Month;
ebx = ebx + edx;
eax = edx ^ 0x1a;
ebx = eax + ebx;

GetCurrentDirectory(MAX_PATH, szHome);
szHome[3] = '\0';
GetVolumeInformationA(szHome, 0, 0, &lpVSN, 0, 0, 0, 0);

eax = lpVSN;
eax = eax ^ 0xffffffff;
eax = eax - 0xffffffff;
eax = eax % 0x3e7;
```
