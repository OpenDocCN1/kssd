
# 一个 RC4 加密程序（Secure PE V1.6）的分析

> 原文：[`www.pediy.com/kssd/pediy10/82805.html`](https://www.pediy.com/kssd/pediy10/82805.html)

## 一个 RC4 加密程序的分析

快雪时晴

2009-2-25

目标程序 `Secure PE V1.6`

## 引用

[`thinkcore.com/`](http://thinkcore.com/)

`ThinkCore Research`

菜单

`Home` | `Articles` | `Utilities` | `Laboratory` | `Blog` | `About` | `Donate` | `Contact`

## `Secure PE`

`SecurePE` 是一个用户友好的工具，使用密码、强 `RC4` 加密和校验和来保护 `Microsoft Windows?` `PE` 文件，同时保持它们完全可用。

### 描述

`SecurePE` 是一个用 32 位汇编语言编写的用户友好免费软件工具，使用密码、强 `RC4` 加密和校验和来保护 `Microsoft Windows?` `PE` 文件，同时保持它们完全可用。

`SecurePE` 用户友好的界面支持 `Windows Explorer` 的拖放和 shell 操作。因此，您可以轻松地通过鼠标单击来保护文件。

当受保护的文件被执行时，它会显示一个对话框，要求用户输入正确的密码。密码并不存储在文件中，而是通过 `CRC32` 哈希算法进行验证。您必须记住密码，否则您将无法再次执行受保护的程序，除非您从备份中恢复它。使用正确的密码，您可以运行程序，甚至解除文件的保护，使其恢复原始状态。

与其他保护工具不同，您可以将 `SecurePE` 保护的文件复制到另一台计算机上，无论该计算机运行什么操作系统，文件都保持受保护且功能正常，无需重新保护。

最后但同样重要的是，`SecurePE` 有一个校验例程，可以检测受保护文件的 `PE Header` 中的任何篡改。

### 系统需求

- 个人计算机（PC），至少具有 386 CPU。
- `Microsoft Windows` 2008; `Windows Vista`; `Windows 2003`, `Windows XP`; `Windows 2000`; `Windows NT4` (`SP6a`); `Windows ME`; `Windows 9x`。
- 300kb 的可用磁盘空间。

### 下载

`SecurePE` 的最新版本随时可以从该网站下载。

当前版本：`Version 1.6`

下载：`SecurePE v1.6`

`MD5` : `01b7015f580a523e90b34613adf3493d`

`SHA1`: `357ddebb745a7d1434919bb8a266016967a0cb33`

`SecurePE` 不包含广告、广告软件、间谍软件或任何其他恶意软件。

### 更新日志

#### `Version 1.6`

- 添加：支持基于 `x64` 的系统。
- 添加：符合 `Windows Vista UAC`。
- 添加：输入时显示/隐藏密码。
- 改进：一些小的调整。

注意：`SecurePE v1.6` 不再与旧版本兼容，因此您需要解除文件的保护，卸载旧版本，安装 1.6 并再次保护它们。

#### `Version 1.5`

- 第一个稳定版本。

#### `Versions 1.0b to 1.4b`

- 初始 beta 发布。

### 许可

此软件为非商业用途的免费软件，只要不收费，您可以完整复制和分发它。但是，如果您想支持此软件的开发，请慷慨解囊捐款。

如果您计划将此软件用于商业用途，您必须通过电子邮件请求许可。

### 免责声明

此软件按“原样”提供，并否认任何明示或暗示的保证，包括但不限于适销性和特定用途适用性的隐含保证。作者不对使用或误用此程序直接或间接造成的任何损失或损害负责。使用此程序风险自负。

### 反馈

如果您对 `SecurePE` 有任何疑问、建议或错误报告，请随时与我联系。

版权 © 2008 `ThinkCore Research` 保留所有权利 联系方式

## 实验分析

自己简单用 `VC6` 自动创建一个 `helloworld` 的 `WIN32` 程序来实验。

### 初始状态

引用: `Microsoft Visual C++ 6.0`

`oep`：`00001350` `.text`

### 加 `SECUREPE16`（仅加 `PASSWORD`, 暂不加 `CRC32`）

引用: 会把最后一个段 `.rsrc` 更名为 `SEPE`，

并会修改前面各段数据（其所称的 `RC4` 加密？）而且会设置那些原来只有只读和执行的区段属性新增一个可写。

`Unidentified Packer` \*

`oep`：`0040A000` `.SEPE`

### 拿出 `OD1.1+STRONGOD` 调试

一开始是这样的：

```assembly
0040A000 > /EB 02                 JMP SHORT 0.0040A004
0040A002  |C606 E8                MOV BYTE PTR DS:[ESI],0E8
0040A005   0000                  ADD BYTE PTR DS:[EAX],AL
0040A007   0000                  ADD BYTE PTR DS:[EAX],AL
0040A009   5D                    POP EBP                        ; kernel32.7C816FD7
0040A00A   81ED D3314000        SUB EBP,0.004031D3
0040A010   8D9D E2314000        LEA EBX,DWORD PTR SS:[EBP+4031E2]
0040A016   53                    PUSH EBX
0040A017   C3                    RETN
0040A018   8DB5 37324000        LEA ESI,DWORD PTR SS:[EBP+403237]
0040A01E   8BFE                 MOV EDI,ESI                     ; ntdll.7C910738
0040A020   BB E2314000          MOV EBX,0.004031E2
0040A025   B9 C3010000          MOV ECX,1C3
0040A02A   AD                   LODS DWORD PTR DS:[ESI]
0040A02B   2BC3                 SUB EAX,EBX
0040A02D   C1C0 03              ROL EAX,3
0040A030   33C3                 XOR EAX,EBX
0040A032   AB                   STOS DWORD PTR ES:[EDI]
0040A033   43                   INC EBX
0040A034   81FB 0B324000        CMP EBX,0.0040320B
0040A03A   75 05               JNZ SHORT 0.0040A041
0040A03C   BB E2314000          MOV EBX,0.004031E2
0040A041  ^ E2 E7              LOOPD SHORT 0.0040A02A
0040A043   89AD 92334000        MOV DWORD PTR SS:[EBP+403392],EBP
0040A049   89AD C6364000        MOV DWORD PTR SS:[EBP+4036C6],EBP
0040A04F   89AD D9364000        MOV DWORD PTR SS:[EBP+4036D9],EBP
0040A055   8D85 37324000        LEA EAX,DWORD PTR SS:[EBP+403237]
0040A05B   50                   PUSH EAX
0040A05C   C3                   RETN
//EAX 中就是壳自身解码完后要跳转的地址，这里用 RETN 到达。
```

`F7` 慢慢走，程序会自己 `sfx`，然后来到

```assembly
0040A118   55                  PUSH EBP
0040A119   68 00040000         PUSH 400
0040A11E   52                  PUSH EDX                        ; 1msgbox.0040A1C1
//参照
HWND CreateDialogIndirectParam(
    HINSTANCE  hInstance, // handle of application instance
    LPCDLGTEMPLATE  lpTemplate, // address of dialog box template
    HWND  hWndParent, // handle of owner window
    DLGPROC  lpDialogFunc, // address of dialog box procedure   
    LPARAM  lParamInit  // initialization value
   );
这里是 PASSWORD 验证对话框的 lpDialogFunc
0040A11F   6A 00               PUSH 0
0040A121   51                  PUSH ECX                        ; 1msgbox.0040A238
0040A122   50                  PUSH EAX                        ; 1msgbox.00400000
0040A123   FF95 52364000      CALL DWORD PTR SS:[EBP+403652]     ; user32.CreateDialogIndirectParamA
```

`PASSWORD` 验证对话框的 `lpDialogFunc` 过程：



```
0040A1C1    53                 PUSH EBX
0040A1C2    56                 PUSH ESI                       ; 1msgbox.0040A27E
0040A1C3    57                 PUSH EDI
0040A1C4    55                 PUSH EBP
0040A1C5    8BEC               MOV EBP,ESP
0040A1C7    BF 366E0000        MOV EDI,6E36
0040A1CC    817D 18 11010000   CMP DWORD PTR SS:[EBP+18],111
//经查 VC6 自带的头库 WINUSER.H：
//#define WM\_COMMAND                 0x0111
0040A1D3    75 49              JNZ SHORT 1msgbox.0040A21E
0040A1D5    66:817D 1E 0003    CMP WORD PTR SS:[EBP+1E],300
0040A1DB    75 41              JNZ SHORT 1msgbox.0040A21E
0040A1DD    8D8F 48344000      LEA ECX,DWORD PTR DS:[EDI+403448]
//在该行 F2 下断点，当按键时会断下
0040A1E3    51                 PUSH ECX
0040A1E4    51                 PUSH ECX
0040A1E5    6A 1A              PUSH 1A
0040A1E7    6A 0D              PUSH 0D
0040A1E9    FF75 20            PUSH DWORD PTR SS:[EBP+20]
0040A1EC    FF97 66364000      CALL DWORD PTR DS:[EDI+403666]          ; user32.SendMessageA
0040A1F2    5E                 POP ESI                        ; 1msgbox.0040A27E
//执行完后
Stack [0012FB30]=00006E36
ESI=0040A27E (1msgbox.0040A27E)
//
0040A27E  31 00 00 00 00 00 00 00  1.......
即指向我刚才输入的字符“1”
//
该处应该是发送 `GETTEXT` 消息，以获取输入文本
查参考
`#define WM\_GETTEXT                 0x000D`
//
那么就是获取最长 `1A`（26.）字符长的输入文本
0040A1F3    57                 PUSH EDI
0040A1F4    BF 26000000        MOV EDI,26
0040A1F9    E8 F8010000        CALL 1msgbox.0040A3F6
//对输入文本进行计算，哈希算法 CALL
0040A1FE    5F                 POP EDI
0040A1FF    3D 3BFCC4A8        CMP EAX,A8C4FC3B
//比较计算值，下一句不能强制不跳，RC4 不能正确解压出原程序
//注意这里该常数是 `A8C4FC3B`，加密密码不一样，该值也会不一样。
//比如我用 `0000` 作加密密码，值为
//0040A1FF    3D FA01A713         CMP EAX,13A701FA
0040A204    75 11              JNZ SHORT 1msgbox.0040A217
0040A206    FE87 46344000      INC BYTE PTR DS:[EDI+403446]
//如果验证成功，这里 `INC 0-->1`，强制跳到该处修改无效！
0040A20C    6A 00              PUSH 0
0040A20E    FF75 14            PUSH DWORD PTR SS:[EBP+14]
0040A211    FF97 5A364000      CALL DWORD PTR DS:[EDI+40365A]          ; user32.EndDialog
0040A217    5D                 POP EBP
0040A218    5F                 POP EDI
0040A219    5E                 POP ESI                        ; 1msgbox.0040A27E
0040A21A    5B                 POP EBX
0040A21B    33C0               XOR EAX,EAX
0040A21D    C3                 RETN

```

哈希算法 `CALL`:

------------------------------------------------------------------

代码:

```
0040A3F6    FC                 CLD
0040A3F7    33C9               XOR ECX,ECX
0040A3F9    49                 DEC ECX
0040A3FA    8BD1               MOV EDX,ECX
0040A3FC    33C0               XOR EAX,EAX
0040A3FE    33DB               XOR EBX,EBX
0040A400    AC                 LODS BYTE PTR DS:[ESI]
//按字节读入输入 PASSWORD
0040A401    32C1               XOR AL,CL
0040A403    8ACD               MOV CL,CH
0040A405    8AEA               MOV CH,DL
0040A407    8AD6               MOV DL,DH
0040A409    B6 08              MOV DH,8
0040A40B    66:D1EB            SHR BX,1
0040A40E    66:D1D8            RCR AX,1
0040A411    73 09              JNB SHORT 1msgbox.0040A41C
0040A413    66:35 2083         XOR AX,8320
0040A417    66:81F3 B8ED       XOR BX,0EDB8
0040A41C    FECЕ               DEC DH
0040A41E  ^ 75 EB             JNZ SHORT 1msgbox.0040A40B
0040A420    33C8               XOR ECX,EAX
0040A422    33D3               XOR EDX,EBX
0040A424    4F                 DEC EDI
0040A425  ^ 75 D5             JNZ SHORT 1msgbox.0040A3FC
0040A427    F7D2               NOT EDX
0040A429    F7D1               NOT ECX
0040A42B    8BC2               MOV EAX,EDX
0040A42D    C1C0 10            ROL EAX,10
0040A430    66:8BC1            MOV AX,CX
0040A433    C3                 RETN

```

一个散列函数，似乎难以逆推，看来只能暴力求解了。

可以把该 `CALL` 段复制内嵌到 C 程序中。

推测在验证完毕后调用

`0040A211    FF97 5A364000      CALL DWORD PTR DS:[EDI+40365A]          ; user32.EndDialog`

过程中可能激发了某个事件，用于解码原程序。

`OD` 跟踪，注意用 `F8` 会跟踪不到，用 `F7` 走

代码:

```
0040A193    AC                 LODS BYTE PTR DS:[ESI]
//第一次时，DS:[ESI]=[00401001]=A5
//果然！！
0040A194    8D9D 36374000      LEA EBX,DWORD PTR SS:[EBP+403736]
0040A19A    53                 PUSH EBX                       ; 1msgbox.0040A56C
0040A19B    50                 PUSH EAX
0040A19C    E8 B5030000        CALL 1msgbox.0040A556
//代码解密函数
0040A1A1    AA                 STOS BYTE PTR ES:[EDI]
0040A1A2  ^ E2 EF             LOOPD SHORT 1msgbox.0040A193
0040A1A4    33DB               XOR EBX,EBX                      ; 1msgbox.00401350
0040A1A6    64:8F03            POP DWORD PTR FS:[EBX]                ; kernel32.7C816FD7
0040A1A9    58                 POP EAX                        ; kernel32.7C816FD7
0040A1AA    BB 50130000        MOV EBX,1350
0040A1AF    81C3 00004000      ADD EBX,1msgbox.00400000
0040A1B5    53                 PUSH EBX                       ; 1msgbox.00401350
0040A1B6    C3                 RETN
//还记得未加壳前程序 OEP 吗，RVA 就是 00001350，这里就是跳转到原程序执行！

```

至于为什么不能暴力修改跳转

`0040A204    75 11              JNZ SHORT 1msgbox.0040A217`

因为暂时还没找到代码解密函数与输入 `PASSWORD` 之间的联系，无法证实。

休息了几个钟头，继续。

代码:



```
0040A1FF    3D 3BFCC4A8        CMP EAX,A8C4FC3B
0040A204    75 11              JNZ SHORT 1msgbox.0040A217
//强制不跳
0040A206    FE87 46344000      INC BYTE PTR DS:[EDI+403446]
//然后在 DS:[EDI+403446]上下硬件读中断
F9 继续，中断
0040A14F    80BD 46344000 00   CMP BYTE PTR SS:[EBP+403446],0
0040A156  ^ 74 D8              JE SHORT 1msgbox.0040A130
//中断于此，看上一行，SS:[EBP+403446]正是刚才下硬件读中断的地址
0040A158    BE 00104000        MOV ESI,1msgbox.00401000
0040A15D    8BFE               MOV EDI,ESI                         ; 1msgbox.00401000
0040A15F    B9 00100000        MOV ECX,1000
0040A164    60                 PUSHAD
0040A165    8D85 48344000      LEA EAX,DWORD PTR SS:[EBP+403448]
//SS:[EBP+403448]存放的就是我们输入的 PASSWORD 首地址，找到联系了
//Address=0040A27E, (ASCII "1234") 
0040A16B    8D9D 36384000      LEA EBX,DWORD PTR SS:[EBP+403836]
0040A171    50                 PUSH EAX                             ; 1msgbox.0040A66C
0040A172    53                 PUSH EBX                             ; 1msgbox.0040A56C
0040A173    E8 66030000        CALL 1msgbox.0040A4DE
//用输入的 PASSWORD 填充 FF 个空间
0040A178    8D85 36384000      LEA EAX,DWORD PTR SS:[EBP+403836]
//Address=0040A66C, (ASCII //"1234123412341234123412341234123412341234123412341234123412341234123412341234123412341234123412341234123412341//23412341234123412341234123412341234123412341234123412341234123412341234123412341234123412341234123412341234123//4)
//EAX=0040A27E (1msgbox.0040A27E), ASCII "1234"
0040A17E    8D9D 36374000      LEA EBX,DWORD PTR SS:[EBP+403736]
//Address=0040A56C
0040A184    50                 PUSH EAX                             ; 1msgbox.0040A66C
0040A185    53                 PUSH EBX                             ; 1msgbox.0040A56C
0040A186    E8 1B030000        CALL 1msgbox.0040A4A6
//RC4 初始化
0040A18B    E8 69030000        CALL 1msgbox.0040A4F9
0040A190    61                 POPAD
0040A191    33C0               XOR EAX,EAX                          ; 1msgbox.0040A66C
//下面开始还原原程序（昨晚其实就是找到了这里，我自己没再往上看）
0040A193    AC                 LODS BYTE PTR DS:[ESI]
0040A194    8D9D 36374000      LEA EBX,DWORD PTR SS:[EBP+403736]
0040A19A    53                 PUSH EBX                             ; 1msgbox.0040A56C
0040A19B    50                 PUSH EAX                             ; 1msgbox.0040A66C
0040A19C    E8 B5030000        CALL 1msgbox.0040A556
0040A1A1    AA                 STOS BYTE PTR ES:[EDI]
0040A1A2  ^ E2 EF            LOOPD SHORT 1msgbox.0040A193
0040A1A4    33DB               XOR EBX,EBX                          ; 1msgbox.0040A56C
0040A1A6    64:8F03            POP DWORD PTR FS:[EBX]               ; 1msgbox.00401000
0040A1A9    58                 POP EAX                              ; 1msgbox.00401000
0040A1AA    BB 50130000        MOV EBX,1350
0040A1AF    81C3 00004000      ADD EBX,1msgbox.00400000
0040A1B5    53                 PUSH EBX                             ; 1msgbox.0040A56C
0040A1B6    C3                 RETN
```

过程 `0040A173    E8 66030000        CALL 1msgbox.0040A4DE`：

--------------------------------------------------------------

代码:

```
0040A4DE    60                 PUSHAD
0040A4DF    8B7C24 24          MOV EDI,DWORD PTR SS:[ESP+24]
0040A4E3    B9 FF000000        MOV ECX,0FF
0040A4E8    8B7424 28          MOV ESI,DWORD PTR SS:[ESP+28]
0040A4EC    AC                 LODS BYTE PTR DS:[ESI]
0040A4ED    84C0               TEST AL,AL
0040A4EF  ^ 74 F7              JE SHORT 1msgbox.0040A4E8
0040A4F1    AA                 STOS BYTE PTR ES:[EDI]
0040A4F2    49                 DEC ECX
0040A4F3  ^ 79 F7              JNS SHORT 1msgbox.0040A4EC
0040A4F5    61                 POPAD
0040A4F6    C2 0800            RETN 8
```

返回后，用输入的 `PASSWORD` 填充 `FF` 个字节空间

代码:

```
0040A66C  31 32 33 34 31 32 33 34  12341234
0040A674  31 32 33 34 31 32 33 34  12341234
0040A67C  31 32 33 34 31 32 33 34  12341234
0040A684  31 32 33 34 31 32 33 34  12341234
0040A68C  31 32 33 34 31 32 33 34  12341234
0040A694  31 32 33 34 31 32 33 34  12341234
0040A69C  31 32 33 34 31 32 33 34  12341234
0040A6A4  31 32 33 34 31 32 33 34  12341234
0040A6AC  31 32 33 34 31 32 33 34  12341234
0040A6B4  31 32 33 34 31 32 33 34  12341234
0040A6BC  31 32 33 34 31 32 33 34  12341234
0040A6C4  31 32 33 34 31 32 33 34  12341234
0040A6CC  31 32 33 34 31 32 33 34  12341234
0040A6D4  31 32 33 34 31 32 33 34  12341234
0040A6DC  31 32 33 34 31 32 33 34  12341234
0040A6E4  31 32 33 34 31 32 33 34  12341234
0040A6EC  31 32 33 34 31 32 33 34  12341234
0040A6F4  31 32 33 34 31 32 33 34  12341234
0040A6FC  31 32 33 34 31 32 33 34  12341234
0040A704  31 32 33 34 31 32 33 34  12341234
0040A70C  31 32 33 34 31 32 33 34  12341234
0040A714  31 32 33 34 31 32 33 34  12341234
0040A71C  31 32 33 34 31 32 33 34  12341234
0040A724  31 32 33 34 31 32 33 34  12341234
0040A72C  31 32 33 34 31 32 33 34  12341234
0040A734  31 32 33 34 31 32 33 34  12341234
0040A73C  31 32 33 34 31 32 33 34  12341234
0040A744  31 32 33 34 31 32 33 34  12341234
0040A74C  31 32 33 34 31 32 33 34  12341234
0040A754  31 32 33 34 31 32 33 34  12341234
0040A75C  31 32 33 34 31 32 33 34  12341234
0040A764  31 32 33 34 31 32 33 34  12341234
0040A76C  00 00 00 00 00 00 00 00  ........
```

过程 `0040A186    E8 1B030000        CALL 1msgbox.0040A4A6`:

--------------------------------------------------------------

代码:

```
0040A4A6    60                 PUSHAD
0040A4A7    8B6C24 24          MOV EBP,DWORD PTR SS:[ESP+24]        ; 1msgbox.0040A072
0040A4AB    8B7424 28          MOV ESI,DWORD PTR SS:[ESP+28]        ; kernel32.7C816FD7
0040A4AF    B9 FF000000        MOV ECX,0FF
0040A4B4    884C0D 00          MOV BYTE PTR SS:[EBP+ECX],CL
0040A4B8    49                 DEC ECX
0040A4B9  ^ 79 F9              JNS SHORT 1msgbox.0040A4B4
0040A4BB    2BDB               SUB EBX,EBX                           ; 1msgbox.0040A56C
0040A4BD    2BC9               SUB ECX,ECX
0040A4BF    8A440D 00          MOV AL,BYTE PTR SS:[EBP+ECX]
0040A4C3    02040E             ADD AL,BYTE PTR DS:[ESI+ECX]
0040A4C6    02C3               ADD AL,BL
0040A4C8    8AD8               MOV BL,AL
0040A4CA    8A440D 00          MOV AL,BYTE PTR SS:[EBP+ECX]
0040A4CE    86441D 00          XCHG BYTE PTR SS:[EBP+EBX],AL
0040A4D2    88440D 00          MOV BYTE PTR SS:[EBP+ECX],AL
0040A4D6    FEC1               INC CL
0040A4D8  ^ 75 E5              JNZ SHORT 1msgbox.0040A4BF
0040A4DA    61                 POPAD
0040A4DB    C2 0800            RETN 8
```

执行前：`100H` 空间数据

代码:

```
0040A56C  00 01 02 03 04 05 06 07  .
0040A574  08 09 0A 0B 0C 0D 0E 0F  .. ..
0040A57C  10 11 12 13 14 15 16 17  
0040A584  18 19 1A 1B 1C 1D 1E 1F  
0040A58C  20 21 22 23 24 25 26 27  !"#$%&'
0040A594  28 29 2A 2B 2C 2D 2E 2F  ()*+,-./
0040A59C  30 31 32 33 34 35 36 37  01234567
0040A5A4  38 39 3A 3B 3C 3D 3E 3F  89:;<=>?
0040A5AC  40 41 42 43 44 45 46 47  @ABCDEFG
0040A5B4  48 49 4A 4B 4C 4D 4E 4F  HIJKLMNO
0040A5BC  50 51 52 53 54 55 56 57  PQRSTUVW
0040A5C4  58 59 5A 5B 5C 5D 5E 5F  XYZ[\]^_
0040A5CC  60 61 62 63 64 65 66 67  `abcdefg
0040A5D4  68 69 6A 6B 6C 6D 6E 6F  hijklmno
0040A5DC  70 71 72 73 74 75 76 77  pqrstuvw
0040A5E4  78 79 7A 7B 7C 7D 7E 7F  xyz{|}~
0040A5EC  80 81 82 83 84 85 86 87  
0040A5F4  88 89 8A 8B 8C 8D 8E 8F  
0040A5FC  90 91 92 93 94 95 96 97  
0040A604  98 99 9A 9B 9C 9D 9E 9F  
0040A60C  A0 A1 A2 A3 A4 A5 A6 A7  ぅΗ
0040A614  A8 A9 AA AB AC AD AE AF  ī
0040A61C  B0 B1 B2 B3 B4 B5 B6 B7  氨渤吹斗
0040A624  B8 B9 BA BB BC BD BE BF  腹夯冀究
0040A62C  C0 C1 C2 C3 C4 C5 C6 C7  懒旅呐魄
0040A634  C8 C9 CA CB CC CD CE CF  壬仕掏蜗
0040A63C  D0 D1 D2 D3 D4 D5 D6 D7  醒矣哉肿
0040A644  D8 D9 DA DB DC DD DE DF  刭谯茌捱
0040A64C  E0 E1 E2 E3 E4 E5 E6 E7  噌忏溴骁
0040A654  E8 E9 EA EB EC ED EE EF  栝觌祉铒
0040A65C  F0 F1 F2 F3 F4 F5 F6 F7  瘃蝮趱鲼
0040A664  F8 F9 FA FB FC FD FE FF  ?
```

执行后：

代码:



```
0040A56C  31 A0 43 E1 05 3B 4E AF  1?;N
0040A574  62 23 1F A2 01 21 3F 0B  b#?!?
0040A57C  5F 6E 42 07 F4 B3 84 CF  _nB 舫
0040A584  A6 63 8E 25 BD AD DF 61  ?江
0040A58C  82 77 7A F6 AE DA 92 17  z 霎
0040A594  B8 83 D8 37 CB 9D 88 96  ?
0040A59C  47 7F 32 4F F5 FF 12 A8  G?2O?
0040A5A4  18 9B E8 51 BE F3 AC 66  Q 倔
0040A5AC  72 D3 6D 81 9A 13 C4 0F  r?
0040A5B4  68 1D 2C 53 74 85 F7 4B  h,StK
0040A5BC  4A 15 67 5D BC 19 39 57  Jg\]?9W
0040A5C4  10 49 F8 87 41 0D 6A 11  IA.j
0040A5CC  58 45 3A FB 5B DE C5 D6  XE:夼
0040A5D4  B9 50 A7 44 BB 70 7B 86  {
0040A5DC  48 E5 8A D2 76 79 26 A5  Hy&
0040A5E4  CA EA 93 6C 65 BA 7E 69  赎 ei
0040A5EC  A3 95 C9 AB 8F 71 24 C8  色$
0040A5F4  B2 35 22 33 1E 27 2D DD  ?"3'-
0040A5FC  FC 73 52 ED 60 FD 0E 9F  R?
0040A604  EB 08 E9 EE 6F D7 E6 38  ?轭 o 祖 8
0040A60C  64 D4 A9 E2 E0 B6 C0 0C  d 冤忄独.
0040A614  C7 4D 20 91 7C 75 FE 29   u?
0040A61C  9E 00 54 04 FA EC 80 EF  ?T
0040A624  C3 1A A1 D1 3D 3E 97 99  ?⊙=>
0040A62C  D9 03 56 5E A4 2F B7 B0  ?V^?钒
0040A634  F9 02 90 94 1C E3 CE 6B  ?阄 k
0040A63C  C2 09 B1 14 CD 1B 06 F2  ???
0040A644  0A 5A F1 59 28 8C 7D 40  .Z(@
0040A64C  F0 D0 E4 16 E7 8B B5 D5  鹦?嫡
0040A654  34 C6 8D 4C 2E 2A CC 5C  4L.\*
0040A65C  C1 89 78 2B AA B4 9C 98  x+
0040A664  30 DC DB BF 36 55 46 3C  0 苒?UF<
```

上面过程正是完成的 `RC4` 初始化工作，我 `google` 了下 `RC4` 的算法说明：

---------------------------------------------------------------

`RC4` 加密算法是大名鼎鼎的 `RSA` 三人组中的头号人物 `Ron Rivest` 在 1987 年设计的密钥长度可变的流加密算法簇。之所以称其为簇，是由于其核心部分的 `S-box` 长度可为任意，但一般为 256 字节。该算法的速度可以达到 `DES` 加密的 10 倍左右。

`RC4` 算法的原理很简单，包括初始化算法和伪随机子密码生成算法两大部分。假设 `S-box` 长度和密钥长度均为为 `n`。先来看看算法的初始化部分（用类 `C` 伪代码表示）：

代码:

```
for (i=0; i<n; i++)
 s[i]=i;
j=0;
for (i=0; i<n; i++)
{
 j=(j+s[i]+k[i])%256;
 swap(s[i], s[j]);
}
```

在初始化的过程中，密钥的主要功能是将 `S-box` 搅乱，`i` 确保 `S-box` 的每个元素都得到处理，`j` 保证 `S-box` 的搅乱是随机的。而不同的 `S-box` 在经过伪随机子密码生成算法的处理后可以得到不同的子密钥序列，并且，该序列是随机的：

代码:

```
i=j=0;
while (明文未结束)
{
 ++i%=n;
 j=(j+s[i])%n;
 swap(s[i], s[j]);
 sub_k=s((s[i]+s[j])%n);
}
```

得到的子密码 `sub_k` 用以和明文进行 `xor` 运算，得到密文，解密过程也完全相同。

由于 `RC4` 算法加密是采用的 `xor`，所以，一旦子密钥序列出现了重复，密文就有可能被破解。关于如何破解 `xor` 加密，请参看 `Bruce Schneier` 的 `Applied Cryptography` 一书的 1.4 节 `Simple XOR`，在此我就不细说了。那么，`RC4` 算法生成的子密钥序列是否会出现重复呢？经过我的测试，存在部分弱密钥，使得子密钥序列在不到 100 万字节内就发生了完全的重复，如果是部分重复，则可能在不到 10 万字节内就能发生重复，因此，推荐在使用 `RC4` 算法时，必须对加密密钥进行测试，判断其是否为弱密钥。

而且，根据目前的分析结果，没有任何的分析对于密钥长度达到 128 位的 `RC4` 有效，所以，`RC4` 是目前最安全的加密算法之一，大家可以放心使用！

结论：

-----------------------------------------------------------------------------------

1> 没有正确密码是无法爆破的，因为密码是 `RC4` 解密的 `KEY`，爆破程序流程只能导致还原错误。

2> 对于简单的密码可以写个穷举算法，调用

`0040A1F9    E8 F8010000        CALL 1msgbox.0040A3F6`

// 对输入文本进行计算，哈希算法 `CALL`

直到得到该处数值常数 `A8C4FC3B`（不同程序不同密码该数值不一样）为止。

`0040A1FF    3D 3BFCC4A8        CMP EAX,A8C4FC3B`

可以在 `C` 中用内嵌 `ASM` 过程方法实现。

附 `Google` 的一些编码资料：

---------------------------------------------------------------------------------------

`_declspec(naked)`

就是告诉编译器，在编译的时候，不要优化代码，通俗的说就是没代码，完全要自己写

比如

```
#define NAKED  __declspec(naked)

void NAKED code(void)

{
    __asm
    {
         ret
     }
}
```

使用 `_declspec(naked)` 关键字定义函数:

使用 `naked` 关键字必须自己构建 `EBP` 指针 (如果用到了的话);

必须自己使用 `RET` 或 `RET n` 指令返回 (除非你不返回);

---------------------------------------------------------------------------------------

关于 `_declspec(naked)` 编写干净函数[`www.diybl.com/`](http://www.diybl.com/) 2008-6-9

对于用 `_declspec(naked)` 编写干净函数

```
VOID __declspec(naked) MyNakedFunction()

{

strcmp(...);

// __cdecl 函数是调用者清除参数堆栈，对于非内联汇编调用这类函数，编译器将自动平衡堆栈，加入 ADD ESP, 8

}

VOID __declspec(naked) MyNakedFunction()

{

//...

__asm CALL strcmp;

__asm ADD ESP, 8; // 内联汇编需要自己平衡堆栈

}
```

`_declspec(naked)` 是且仅是不产生 `prolog` 和 `epilog` 代码 {保存并恢复使用过的寄存器和分配局部变量、平衡堆栈、返回值}。

附上刚翻译的一些 `MSDN` 可能对 `naked` 函数有点用的资料（我英语很烂，希望没翻译错误）

`Microsoft Specific`

All arguments are widened to 32 bits when they are passed. Return values are also widened to 32 bits and returned in the `EAX` register, except for 8-byte structures, which are returned in the `EDX:EAX` register pair. Larger structures are returned in the `EAX` register as pointers to hidden return structures. Parameters are pushed onto the stack from right to left. Structures that are not `PODs` will not be returned in registers.

The compiler generates `prolog` and `epilog` code to save and restore the `ESI`, `EDI`, `EBX`, and `EBP` registers, if they are used in the function.

`Note`

When a `struct`, `union`, or `class` is returned from a function by value, all definitions of the type need to be the same, else the program may fail at `runtime`.

For information on how to define your own function `prolog` and `epilog` code, see `Naked Function Calls`.

The following calling conventions are supported by the `Visual C/C++` compiler.

`Keyword`   `Stack cleanup`   `Parameter passing`

`__cdecl`    `Caller`          Pushes parameters on the stack, in reverse order (right to left)

`__clrcall`  `n/a`            Load parameters onto `CLR` expression stack in order (left to right).

`__stdcall`  `Callee`         Pushes parameters on the stack, in reverse order (right to left)

`__fastcall` `Callee`         Stored in registers, then pushed on stack

`__thiscall` `Callee`        Pushed on stack; this pointer stored in `ECX`

`微软细节`

所有参数在传递时都被扩展为 32 位。除了 8 字节结构体返回在 `EDX:EAX` 寄存器一对中，其它返回值同样被扩展为 32 位并返回在 `EAX` 寄存器中。大的结构体返回在 `EAX` 寄存器中的是指针来间接返回结构体。参数是自右向左压入堆栈。结构体是非 `POD` (Plain Old Data) 类型将不会返回在寄存器中。

如果 `ESI`、`EDI`、`EBX` 和 `EBP` 寄存器在函数中使用，编译器将产生 `prolog` 和 `epilog` 代码保存并恢复它们。

`注释`

当一个函数用值返回结构体(`struct`)、联合体(`union`)或者类(`class`)时，所有类型的定义必须相同，否则程序可能在运行过程中失败。

关于如何定义你自身函数 `prolog` 和 `epilog` 代码的更多信息，请查阅 `Naked Function Calls`。

`Visual C/C++` 编译器支持以下调用转换：

`关键字`   `堆栈清除` `参数传递`

`__cdecl`   调用者    将参数倒序压入堆栈(自右向左)

`__clrcall` 不适用    将参数顺序加载到 `CLR expression` 堆栈中(自左向右)

`__stdcall` 被调用者  将参数倒序压入堆栈(自右向左)

`__fastcall` 被调用者 保存在寄存器然后压入堆栈

`__thiscall` 被调用者 压入堆栈，指针保存在 `ECX` 寄存器中

※ `__fastcall`：两个以内不大于 4 字节(`DWORD`)的参数将自左向右分别保存在 `ECX`、`EDX` 寄存器内，由于没有将参数压入堆栈，函数返回前无需将参数出栈（这就是它 `FASTCALL` 由来）；其它参数仍按自右向左压入堆栈，函数返回时需要把这些参数出栈。
