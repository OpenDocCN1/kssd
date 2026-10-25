
# Windows 系统 RunAs 命令的 DIY 改造【原创】

> 原文：[`www.pediy.com/kssd/pediy08/pediy8-45.htm`](https://www.pediy.com/kssd/pediy08/pediy8-45.htm)

## 破文信息

*   **破文标题**：Windows 系统 RunAs 命令的 DIY 改造
*   **破文作者**：FishSeeWater
*   **作者主页**：`htp://shuihua.hostrocket.com`
*   **作者邮箱**：`yujianshui99@163.com`
*   **所属组织**：BCG
*   **软件名称**：`RunAs.EXE`
*   **下载地址**：Microsoft 的
*   **破解工具**：`OllyDbg(cao_cong 汉化版)` + `C32Asm`
*   **保护方式**：无
*   **软件限制**：不接受密码参数
*   **破解难度**：简单

## 软件介绍

Microsoft 系统自带的命令，不用我多罗嗦了吧：)

## DIY 声明

最近由于公司领导要求“计算机资料保密”，我决定采用 `WIN2000` 的系统特性加上 `NTFS` 权限完成。用 `Win2003` 架了台服务器，然后为每位工程师建了 `User` 帐户，但是发现好几个 `CAD` 软件在 `User` 用户下没法运行，但通过右键的“运行方式”以 `"Administrator"` 运行就没事儿，调整组权限，整了好几天也没能搞定，微软出于安全考虑 `RunAs` 命令不能直接跟密码参数。

每次都要单独输入密码，这样的话那我岂不要~~\*&@#$$@：(。在网上找 `RunAs` 命令的程序实现方式，准备写个程序调用 `CAD` 来运行，可是用 `LogonUser()`；`CreateProcessAsUse()`；`CreateProcessWithLogonW()` 这几个函数死活不行，`LogonUser()` 总是返加 `1314` 错误，郁闷。无奈之下，还是打 `RunAs` 的主意吧。让它支持密码参数不就 OK 了：)

## 破解分析

好!开始：

首先预习一下基础知识，控制台程序获取运行参数是用 `GetCommandLine(VOID)` 函数。

打开 `OD` 加载目标程序，参数为 `"/user:administrator cmd.exe"`。

然后下断点 `BPX GetCommandLine` 回车。

`F9` 运行，程序断在 `01001572 CALL DWORD PTR DS:[<&KERNEL32.GetCommandLineW>]`。

瞧瞧它是怎样 `RunAs` 的：)

`F8` 一步一步走：

```asm
01001555   |.  50   PUSH EAX                              ; /pArgc
01001556   |.  895D>MOV DWORD PTR SS:[EBP-1C],EBX         ; |
01001559   |.  895D>MOV DWORD PTR SS:[EBP-20],EBX         ; |
0100155C   |.  895D>MOV DWORD PTR SS:[EBP-C],EBX          ; |
0100155F   |.  895D>MOV DWORD PTR SS:[EBP-18],EBX         ; |
01001562   |.  895D>MOV DWORD PTR SS:[EBP-38],EBX         ; |
01001565   |.  8975>MOV DWORD PTR SS:[EBP-4],ESI          ; |
01001568   |.  895D>MOV DWORD PTR SS:[EBP-14],EBX         ; |
0100156B   |.  C745>MOV DWORD PTR SS:[EBP-7C],44          ; |

//得到参数
01001572   |.  FF15>CALL DWORD PTR DS:[<&KERNEL32.GetCommandLineW>] ; [GetCommandLineW
01001578   |.  50   PUSH EAX                              ; |CmdLine

//得到参数个数
01001579   |.  FF15>CALL DWORD PTR DS:[<&SHELL32.CommandLineToArg>] ; \CommandLineToArgvW
0100157F   |.  8BC8 MOV ECX,EAX
01001581   |.  3BCB CMP ECX,EBX
01001583   |.  894D>MOV DWORD PTR SS:[EBP+8],ECX
01001586   |.  0F85>JNZ runas.0100163E
0100158C   |.  895D>MOV DWORD PTR SS:[EBP-4],EBX

.......

.......

.......

//比较参数个数，不等于 3 就跳走，显示 RunAs 的帮助(自身的程序名也算一个参数)
0100163E  |>  \8B45>MOV EAX,DWORD PTR SS:[EBP-14]
01001641   |.  83F8>CMP EAX,5
01001644   |.  74 0>JE SHORT runas.01001650
01001646   |.  83F8>CMP EAX,4
01001649   |.  74 0>JE SHORT runas.01001650
0100164B   |.  83F8>CMP EAX,3
0100164E   |.  75 7>JNZ SHORT runas.010016CF
01001650   |>  8D50>LEA EDX,DWORD PTR DS:[EAX-2]

//以下是判断并取出第二个参数"administraor" (第一个参数是 RunAs.exe)
01001653   |.  8975>MOV DWORD PTR SS:[EBP-8],ESI
01001656   |.  3BD6 CMP EDX,ESI
01001658   |.  7C 7>JL SHORT runas.010016CA
0100165A   |.  8D79>LEA EDI,DWORD PTR DS:[ECX+4]
0100165D   |>  8B07 MOV EAX,DWORD PTR DS:[EDI]
0100165F   |.  66:8>CMP WORD PTR DS:[EAX],2F
01001663   |.  75 6>JNZ SHORT runas.010016CF
01001665   |.  0FB7>MOVZX ECX,WORD PTR DS:[EAX+2]
01001669   |.  83F9>CMP ECX,4E                      ;  Switch (cases 45..75)
0100166C   |.  7F 1>JG SHORT runas.0100167E
0100166E   |.  0F84>JE runas.01001BA4
01001674   |.  83F9>CMP ECX,45
01001677   |.  75 5>JNZ SHORT runas.010016CF
01001679   |>  8975>MOV DWORD PTR SS:[EBP-1C],ESI         ;  Cases 45 ('E'),65 ('e') of switch 01001669
0100167C   |.  EB 4>JMP SHORT runas.010016BF
0100167E   |>  83E9>SUB ECX,50
01001681   |.  74 3>JE SHORT runas.010016BC
01001683   |.  83E9>SUB ECX,5
01001686   |.  74 1>JE SHORT runas.0100169F
01001688   |.  83E9>SUB ECX,10
0100168B   |.^ 74 E>JE SHORT runas.01001679
0100168D   |.  83E9>SUB ECX,9
01001690   |.  0F84>JE runas.01001BA4
01001696   |.  49   DEC ECX
01001697   |.  49   DEC ECX
01001698   |.  74 2>JE SHORT runas.010016BC
0100169A   |.  83E9>SUB ECX,5
0100169D   |.  75 3>JNZ SHORT runas.010016CF
0100169F   |>  83C0>ADD EAX,4                     ;  Cases 55 ('U'),75 ('u') of switch 01001669
010016A2   |>  66:8>/MOV CX,WORD PTR DS:[EAX]

.......

.......

.......

010016CD   |.  /75 0>JNZ SHORT runas.010016DC
010016CF   |>  |E8 8>CALL runas.0100125B           ;  Default case of switch 01001669
010016D4   |>  |895D>MOV DWORD PTR SS:[EBP-4],EBX
010016D7   |.  |E9 5>JMP runas.01001B38

//加载提示资源串"键入密码 administrator："
010016DC   |>  \8B3D>MOV EDI,DWORD PTR DS:[<&USER32.LoadStringW>]  ;  USER32.LoadStringW
010016E2   |.  8D85>LEA EAX,DWORD PTR SS:[EBP-630]
010016E8   |.  68 F>PUSH 1F4                        ; /Count = 1F4 (500.)
010016ED   |.  50   PUSH EAX                        ; |Buffer
010016EE   |.  68 7>PUSH 1B76                       ; |RsrcID = STRING "Enter password for"
010016F3   |.  FF35>PUSH DWORD PTR DS:[1003020]       ; |hInst = 01000000
010016F9   |.  FFD7 CALL EDI                         ; \LoadStringW
010016FB   |.  8B35>MOV ESI,DWORD PTR DS:[<&USER32.CharToOemW>]  ;  USER32.CharToOemW
01001701   |.  8D85>LEA EAX,DWORD PTR SS:[EBP-8EC]
01001707   |.  50   PUSH EAX                        ; /pDest
01001708   |.  8D85>LEA EAX,DWORD PTR SS:[EBP-630]         ; |
0100170E   |.  50   PUSH EAX                        ; |pSrc
0100170F   |.  FFD6 CALL ESI                         ; \CharToOemW
01001711   |.  FF75>PUSH DWORD PTR SS:[EBP-C]           ; /String
01001714   |.  8B1D>MOV EBX,DWORD PTR DS:[<&KERNEL32.lstrlenW>]  ; |kernel32.lstrlenW
0100171A   |.  FFD3 CALL EBX                         ; \lstrlenW
0100171C   |.  3D F>CMP EAX,1F4
01001721   |.  76 0>JBE SHORT runas.0100172E
01001723   |.  8B45>MOV EAX,DWORD PTR SS:[EBP-C]
01001726   |.  66:8>AND WORD PTR DS:[EAX+3E6],0
0100172E   |>  8D85>LEA EAX,DWORD PTR SS:[EBP-D9C]
01001734   |.  50   PUSH EAX
01001735   |.  FF75>PUSH DWORD PTR SS:[EBP-C]
01001738   |.  FFD6 CALL ESI
0100173A   |.  8D85>LEA EAX,DWORD PTR SS:[EBP-D9C]
01001740   |.  50   PUSH EAX                        ; /<%s>
01001741   |.  8D85>LEA EAX,DWORD PTR SS:[EBP-8EC]         ; |
01001747   |.  50   PUSH EAX                        ; |<%s>
01001748   |.  68 8>PUSH runas.0100118C             ; |format = "%s %s:"
0100174D   |.  FF15>CALL DWORD PTR DS:[<&MSVCRT.printf>]      ; \printf
01001753   |.  83C4>ADD ESP,0C
01001756   |.  8D85>LEA EAX,DWORD PTR SS:[EBP-248]

//这个 104 是干嘛的？难道是最大长度？请高手指点：)
0100175C   |.  68 0>PUSH 104                        ; /Arg2 = 00000104

//压入变量地址
01001761   |.  50   PUSH EAX                        ; |Arg1

//下面是调用 ReadConsoleW 函数读取键盘输入的密码
01001762  >|.  E8 4>CALL runas.010011B0             ; \读入密码
01001767   |.  85C0 TEST EAX,EAX
01001769   |.  74 4>JE SHORT runas.010017AD
0100176B   |.  8D85>LEA EAX,DWORD PTR SS:[EBP-630]

........

........

........

0100179B   |.  68 3>PUSH runas.01001134             ; |format = "%s"
010017A0   |.  FF15>CALL DWORD PTR DS:[<&MSVCRT.printf>]      ; \printf
010017A6   |.  59   POP ECX
010017A7   |.  59   POP ECX
010017A8   |.  E9 4>JMP runas.010019ED

//处理输入的密码
010017AD   |>  8D85>LEA EAX,DWORD PTR SS:[EBP-248]
010017B3   |.  50   PUSH EAX                        ; /Translation
010017B4   |.  8D85>LEA EAX,DWORD PTR SS:[EBP-248]         ; |
```



# 汇编代码分析

## 1. 密码复制与保存

```
010017BA  |.  50   PUSH EAX                                   ; |OemString
010017BB  |.  FF15>CALL DWORD PTR DS:[<&USER32.OemToCharA>]   ; \OemToCharA
010017C1  |.  8D85>LEA EAX,DWORD PTR SS:[EBP-248]

// 将密码复制后保存

010017C7  |.  50   PUSH EAX                                    ; /<%S>
010017C8  |.  8D85>LEA EAX,DWORD PTR SS:[EBP-1198]             ; |
010017CE  |.  68 8>PUSH runas.01001184                          ; |Format = "%S"
010017D3  |.  50   PUSH EAX                                    ; |s
010017D4  |.  FF15>CALL DWORD PTR DS:[<&USER32.wsprintfW>]     ; \wsprintfW
010017DA  |.  83C4>ADD ESP,0C
010017DD  |.  8D85>LEA EAX,DWORD PTR SS:[EBP-E0]
010017E3  |.  6A 3>PUSH 32
010017E5  |.  50   PUSH EAX
010017E6  |.  68 7>PUSH 1B77
010017EB  |.  FF35>PUSH DWORD PTR DS:[1003020]                  ; runas.01000000
010017F1  |.  FFD7 CALL EDI
010017F3  |.  8B45>MOV EAX,DWORD PTR SS:[EBP-14]
010017F6  |.  8B4D>MOV ECX,DWORD PTR SS:[EBP+8]

// 得到命令行参数的第三个参数长度。(DS:[ECX+EAX*4-4]为第三个参数)

010017F9  |.  FF74>PUSH DWORD PTR DS:[ECX+EAX*4-4]
010017FD  |.  FFD3 CALL EBX                                    ; kernel32.lstrlenW
010017FF  |.  8BF8 MOV EDI,EAX
01001801  |.  8D85>LEA EAX,DWORD PTR SS:[EBP-E0]
01001807  |.  50   PUSH EAX
01001808  |.  FFD3 CALL EBX
0100180A  |.  FF75>PUSH DWORD PTR SS:[EBP-C]

........
........
........

01001A0B  |.  C745>MOV DWORD PTR SS:[EBP-24],400
01001A12  |.  EB 0>JMP SHORT runas.01001A17
01001A14  |>  8B5D>MOV EBX,DWORD PTR SS:[EBP-38]
01001A17  |>  8D45>LEA EAX,DWORD PTR SS:[EBP-34]

// 调用 CreateProcessWithLogonW 函数以 "administrator" 身份创建进程

01001A1A  |.  50   PUSH EAX                                    ; /Arg11
01001A1B  |.  8D45>LEA EAX,DWORD PTR SS:[EBP-7C]               ; |
01001A1E  |.  50   PUSH EAX                                    ; |Arg10
01001A1F  |.  8B45>MOV EAX,DWORD PTR SS:[EBP-14]               ; |
01001A22  |.  53   PUSH EBX                                    ; |Arg9
01001A23  |.  FF75>PUSH DWORD PTR SS:[EBP-20]                  ; |Arg8
01001A26  |.  FF75>PUSH DWORD PTR SS:[EBP-24]                  ; |Arg7
01001A29  |.  FF74>PUSH DWORD PTR DS:[EDI+EAX*4-4]             ; |Arg6
01001A2D  |.  8D85>LEA EAX,DWORD PTR SS:[EBP-1198]             ; |
01001A33  |.  6A 0>PUSH 0                                      ; |Arg5 = 00000000
01001A35  |.  FF75>PUSH DWORD PTR SS:[EBP-10]                  ; |Arg4
01001A38  |.  50   PUSH EAX                                    ; |Arg3
01001A39  |.  FF75>PUSH DWORD PTR SS:[EBP-18]                  ; |Arg2
01001A3C  |.  FF75>PUSH DWORD PTR SS:[EBP-C]                   ; |Arg1
01001A3F  |.  FF15>CALL DWORD PTR DS:[<&ADVAPI32.CreateProcessWi>; \CreateProcessWithLogonW
01001A45  |.  85C0 TEST EAX,EAX
01001A47  |.  0F85>JNZ runas.01001B36
01001A4D  |.  33FF XOR EDI,EDI
01001A4F  |.  C745>MOV DWORD PTR SS:[EBP-10],7B
```

## 2. 实现思路与修改步骤

`[CODE]`

过程基本分析完毕，要想达到我们的目的，只要在 `ReadConsoleW` 函数之后，将我们的已有密码添上就 OK 了。

### 修改思路

1.  先将程序中提示输入密码和读取密码的函数 `NOP` 掉！从 `10016e2` 处开始一直到 `1001766`。
2.  我们有现成的密码，也不用他来处理，所以嘛处理过程也 `NOP`！从 `1007AD` 处开始到 `1007D4`（哈哈，这下有足够的空间来写我们的代码了）。
3.  由于我们要用参数型式添入密码，而程序中有检测参数个数的语句，所以在 `CommandLineToArgvW` 函数后，将参数个数减 1。

好！开工：

### 具体修改操作

#### 处理步骤 1 和 2

-   修改 `1001586` 处：
    `01001586 JNZ runas.0100163E` 为 `01001586 JNZ runas.010016F2`
-   在 `010016EF` 处添加语句：
    `010016EF JMP SHORT runas.01001767` //正常运行时跳过下面处理参数的两句
-   在 `010016F2` 处添加语句：
    `010016F2 DEC DWORD PTR SS:[EBP-14]` //将参数个数减 1
    `010016F5 JMP runas.0100163E` //跳回去

#### 处理参数完毕，再处理密码

-   修改 `01001767` 处：
    `01001767 TEST EAX,EAX` 为 `01001767 XOR EAX,EAX` //标志位为零
-   从 `010017AF` 处开始添加语句：

```
010017AF  MOV EAX,DWORD PTR SS:[EBP-14]
010017B2  INC EAX      //将参数个数加 1
010017B3  MOV ECX,DWORD PTR SS:[EBP+8]
010017B6  NOP
010017B7  NOP
010017B8  MOV EAX,DWORD PTR DS:[ECX+EAX*4-4]
010017BC  PUSH EAX                     ; /String2
010017BD  LEA EAX,DWORD PTR SS:[EBP-1198]      ; |
010017C3  NOP                          ; |
010017C4  PUSH EAX                     ; |String1
010017C5  CALL DWORD PTR DS:[<&KERNEL32.lstrcpyW>]     ; \lstrcpyW
```

OK!!!处理完成!!!运行试试!真爽：)

再写个外壳调用就 OK 了：)

收工

## 3. 结尾信息

-------------------------------------------------

本文写于 `2006-1-25 09:04:55` By `FishSeeWater`

`4dd466d0e1e0cc123afbe504da2462ca`
