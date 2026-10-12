
# Hide Window Hotkey V2.5 注册算法分析(适合新手)

> 原文：[`www.pediy.com/kssd/pediy11/110572.html`](https://www.pediy.com/kssd/pediy11/110572.html)

**【文章标题】**: `Hide Window Hotkey` V2.5 注册算法分析

**【文章作者】**: `TyroneKing`

**【作者邮箱】**: `tyroneking@126.com`

**【软件名称】**: `Hide Window Hotkey`

**【下载地址】**: `HWHSetup.rar`

**【KeyGen】**: `KeyGen.rar`

**【加壳方式】**: `ASPack 2.12 -> Alexey Solodovnikov`

**【保护方式】**: 注册名+序列号

**【编写语言】**: `Borland Delphi 6.0 - 7.0`

**【使用工具】**: `OD + PEiD + DeDe + MD5 计算器 + AspackDie + IDA (可以不用，加强分析)`

**【操作平台】**: `WIN XP + SP3、 VC++6.0`

**【软件介绍】**: 隐藏运行的窗口和应用程序软件。

**【作者声明】**: 只是为了学习，没有其他目的。失误之处敬请诸位大侠赐教!

## 【分析过程】

1. 用 `PEiD` 查壳，为 `ASPack 2.12 -> Alexey Solodovnikov`，用 `ESP` 定律很容易就可以脱壳了，不过这里还是用 `AspackDie`，不对如何脱壳详说（我也是新手，脱一些简单的行，强壳，还得继续努力呀，：））。

2. 用 `AspackDie` 脱壳，用 `PEiD` 查为 `Borland Delphi 6.0 - 7.0` 编写，`KANAL` 算法识别插件分析，软件用了 `MD5(地址 004B549A)` 算法，用，直接运行脱壳后程序没啥反应，应该是自检验了。

3. 用 `OD` 加载下 `bp GetFileSize` 断点，程序中断在 `GetFileSize` 函数中，按 `Alt+F9` 返回程序领空 `004030A1`

引用:
```asm
00403097  |.  6A 00           PUSH 0                          ; /pFileSizeHigh = NULL
00403099  |.  8B03           MOV EAX,DWORD PTR DS:[EBX]       ; |
0040309B  |.  50             PUSH EAX                          ; |hFile
0040309C  |.  E8 FBE1FFFF    CALL <JMP.&kernel32.GetFileSize> ; \GetFileSize
004030A1  |.  8BF0           MOV ESI,EAX                       ;  EAX=文件大小
……
004030C8  \.  C3             RETN
```

4. 单步运行过 `004030C8` 后，跳出该函数，运行到 `004C4B8B`

引用:
```asm
004C4B86  |.  E8 79DDF3FF    CALL unpacked.00402904           ;  取文件大小
004C4B8B  |.  3D 20A10700    CMP EAX,7A120                    ;  EAX 为文件大小
004C4B90  |.  7E 15          JLE SHORT unpacked.004C4BA7      ;  <=7A120 则跳，原文件大小 6E000,现 EAX=1A2000 没跳
004C4B92  |.  8D85 ACFEFFFF LEA EAX,DWORD PTR SS:[EBP-154]    ;  将 4C4B90 跳为 JMP，或将 7A120 改为比 1A2000 即可
004C4B98  |.  E8 3FE3F3FF    CALL unpacked.00402EDC
004C4B9D  |.  E8 62DDF3FF    CALL unpacked.00402904
004C4BA2  |.  E8 49F9F3FF    CALL unpacked.004044F0           ;  该函数中调用 ExitProcess，退出程序
004C4BA7  |>  8D85 ACFEFFFF LEA EAX,DWORD PTR SS:[EBP-154]
```

5. 将 `4C4B90` 改为 `JMP SHORT unpacked.004C4BA7`，保存成 `Crack.exe` 文件，运行 `Crack.exe`，程序正常运行（注意要先前面调试的 `OD`，因为这个软件只允许一个进程运行）。

6. 在 `Help->Register` 中打开注册窗口，输入 `Name` 和 `Code`，提示错误，退出程序，分别用 `OD` 和 `DeDe` 加载，下面开始算法分析。

7. 在 `DeDe` 中查找注册窗口的 `OK` 按钮事件

![](img/0bae0d0eb75bce721dd557c0b21a6e30.jpg)

在 `OD` 中 `004B609C` 下断点。

8. `OD` 中运行程序，（在第 2 点中提到 `004B549A`（`MD5` 算法位置）附近 `004B5499` 下断点，为了判断程序是否用 `MD5` 加密）打开注册窗口 `Name` 输入 `TYRONEKING`，`Code` 输入 `0123456789`，点击 `OK`，`OD` 在 `004B609C` 中断。

引用:
```asm
004B609C  /.  55             PUSH EBP                           ;  OKButton
……
004B60BE  |.  8D55 FC        LEA EDX,DWORD PTR SS:[EBP-4]
004B60C1  |.  8B83 2C030000 MOV EAX,DWORD PTR DS:[EBX+32C]     ;  TfrmRegister.eName
004B60C7  |.  E8 5061FAFF    CALL <Crack.TControl.GetText(TControl)> ;  这里函数名是作了标签的，具体是哪个函数参考 DeDe
004B60CC  |.  8B45 FC        MOV EAX,DWORD PTR SS:[EBP-4]      ;  Tips:知道函数名后进入函数，按 Shift+;（即输入冒号）给该函数作标签
004B60CF  |.  E8 ECE7F4FF    CALL <Crack.@LStrLen(String)>
004B60D4  |.  83F8 08        CMP EAX,8
004B60D7  |.  7D 3E          JGE SHORT Crack.004B6117          ;  Name 的长度<8 则提示错误
004B60D9  |.  6A 10          PUSH 10
004B60DB  |.  8D55 F8        LEA EDX,DWORD PTR SS:[EBP-8]
004B60DE  |.  A1 30E74C00    MOV EAX,DWORD PTR DS:[4CE730]
004B60E3  |.  E8 4005F5FF    CALL <Crack.LoadResString(PResStringRec)>
004B60E8  |.  8B45 F8        MOV EAX,DWORD PTR SS:[EBP-8]
004B60EB  |.  E8 D0E9F4FF    CALL <Crack.@LStrToPChar(String)>
004B60F0  |.  50             PUSH EAX
004B60F1  |.  8D55 F4        LEA EDX,DWORD PTR SS:[EBP-C]
004B60F4  |.  A1 C4E64C00    MOV EAX,DWORD PTR DS:[4CE6C4]
004B60F9  |.  E8 2A05F5FF    CALL <Crack.LoadResString(PResStringRec)> ;  在资源文件中提取“提示字符串”，避免通过字符串查找破解
004B60FE  |.  8B45 F4        MOV EAX,DWORD PTR SS:[EBP-C]
004B6101  |.  E8 BAE9F4FF    CALL <Crack.@LStrToPChar(String)>
004B6106  |.  50             PUSH EAX                          ; |Text
004B6107  |.  E8 5010F5FF    CALL <JMP.&user32.GetActiveWindow> ; |\[GetActiveWindow
004B610C  |.  50             PUSH EAX                          ; |hOwner
004B610D  |.  E8 7A12F5FF    CALL <JMP.&user32.MessageBoxA>    ; \MessageBoxA
004B6112  |.  E9 4F010000    JMP Crack.004B6266
004B6117  |>  8D55 F0        LEA EDX,DWORD PTR SS:[EBP-10]
004B611A  |.  8B83 2C030000 MOV EAX,DWORD PTR DS:[EBX+32C]     ;  (初始 cpu 选择)
004B6120  |.  E8 F760FAFF    CALL <Crack.TControl.GetText(TControl)>
004B6125  |.  8B55 F0        MOV EDX,DWORD PTR SS:[EBP-10]     ;  取 Name
004B6128  |.  B8 DC624B00    MOV EAX,Crack.004B62DC            ;  空格 20H
004B612D  |.  E8 D2EAF4FF    CALL <Crack.@LStrPos>             ;  Name 是否包含空格
004B6132  |.  85C0           TEST EAX,EAX
004B6134  |.  7E 3E          JLE SHORT Crack.004B6174          ;  不包含则跳
……
004B6174  |>  8D55 E4        LEA EDX,DWORD PTR SS:[EBP-1C]
004B6177  |.  8B83 2C030000 MOV EAX,DWORD PTR DS:[EBX+32C]
004B617D  |.  E8 9A60FAFF    CALL <Crack.TControl.GetText(TControl)>
004B6182  |.  8B45 E4        MOV EAX,DWORD PTR SS:[EBP-1C]
004B6185  |.  E8 36E7F4FF    CALL <Crack.@LStrLen(String)>
004B618A  |.  8BF0           MOV ESI,EAX
004B618C  |.  83EE 02        SUB ESI,2
004B618F  |.  7C 36          JL SHORT Crack.004B61C7          ;  Name 的长度小于 2,则错误
004B6191  |.  46             INC ESI                           ; /------------------------
004B6192  |.  BF 02000000    MOV EDI,2
004B6197  |>  8D55 E0        /LEA EDX,DWORD PTR SS:[EBP-20]
004B619A  |.  8B83 2C030000 |MOV EAX,DWORD PTR DS:[EBX+32C]
004B61A0  |.  E8 7760FAFF    |CALL <Crack.TControl.GetText(TControl)> ; 取 Name
004B61A5  |.  8B45 E0        |MOV EAX,DWORD PTR SS:[EBP-20]
004B61A8  |.  8A4438 FF      |MOV AL,BYTE PTR DS:[EAX+EDI-1]  ; AL=Name[EDI-1]
004B61AC  |.  50             |PUSH EAX
004B61AD  |.  8D55 DC        |LEA EDX,DWORD PTR SS:[EBP-24]
004B61B0  |.  8B83 2C030000 |MOV EAX,DWORD PTR DS:[EBX+32C]
004B61B6  |.  E8 6160FAFF    |CALL <Crack.TControl.GetText(TControl)>
004B61BB  |.  8B45 DC        |MOV EAX,DWORD PTR SS:[EBP-24]
004B61BE  |.  5A             |POP EDX
004B61BF  |.  3A10           |CMP DL,BYTE PTR DS:[EAX]        ; Name[EDI-1]与 Name[0]比较
004B61C1  |.  75 04          |JNZ SHORT Crack.004B61C7
004B61C3  |.  47             |INC EDI                           ; 相等则 EDI+1
004B61C4  |.  4E             |DEC ESI
004B61C5  |.^ 75 D0          \JNZ SHORT Crack.004B6197
004B61C7  |>  8D55 D8        LEA EDX,DWORD PTR SS:[EBP-28]
004B61CA  |.  8B83 2C030000 MOV EAX,DWORD PTR DS:[EBX+32C]
004B61D0  |.  E8 4760FAFF    CALL <Crack.TControl.GetText(TControl)>
004B61D5  |.  8B45 D8        MOV EAX,DWORD PTR SS:[EBP-28]
004B61D8  |.  E8 E3E6F4FF    CALL <Crack.@LStrLen(String)>
004B61DD  |.  3BF8           CMP EDI,EAX
004B61DF  |.  7E 3B          JLE SHORT Crack.004B621C          ; EDI>EAX 提示出错
004B61E1  |.  6A 10          PUSH 10                           ; \\若用户名为单一字符，出提示错误
……
004B621C  |>  8D55 CC        LEA EDX,DWORD PTR SS:[EBP-34]
004B621F  |.  8B83 2C030000 MOV EAX,DWORD PTR DS:[EBX+32C]
004B6225  |.  E8 F25FFAFF    CALL <Crack.TControl.GetText(TControl)> ; 取 Name
004B622A  |.  8B55 CC        MOV EDX,DWORD PTR SS:[EBP-34]
004B622D  |.  8BC3           MOV EAX,EBX
004B622F  |.  E8 CC020000    CALL Crack.004B6500               ; 计算 Code
004B6234  |.  8D55 C8        LEA EDX,DWORD PTR SS:[EBP-38]
004B6237  |.  8B83 30030000 MOV EAX,DWORD PTR DS:[EBX+330]
004B623D  |.  E8 DA5FFAFF    CALL <Crack.TControl.GetText(TControl)> ; 取假 Code
004B6242  |.  8B55 C8        MOV EDX,DWORD PTR SS:[EBP-38]
004B6245  |.  8D83 38030000 LEA EAX,DWORD PTR DS:[EBX+338]     ; 保假 Code
```



`004B624B  |.  E8 04E4F4FF   CALL <Crack.@LStrAsg>`

9、单步跟进 `004B622F  |.  E8 CC020000   CALL Crack.004B6500` ; 计算 Code

### 进一步分析

引用:

`004B652D  |.  837D F8 00    CMP DWORD PTR SS:[EBP-8],0`

`004B6531  |.  0F84 FB000000 JE Crack.004B6632` ; Name 为空?

`004B6537  |.  8D45 F4       LEA EAX,DWORD PTR SS:[EBP-C]` ; /-----------

`004B653A  |.  8B55 F8       MOV EDX,DWORD PTR SS:[EBP-8]`

`004B653D  |.  E8 56E1F4FF   CALL <Crack.@LStrLAsg(void;void;void;v>`

`004B6542  |.  8B45 F4       MOV EAX,DWORD PTR SS:[EBP-C]`

`004B6545  |.  E8 76E3F4FF   CALL <Crack.@LStrLen(String)>` ; 取 Name 的长度

`004B654A  |.  8BF8          MOV EDI,EAX`

`004B654C  |.  8B45 F4       MOV EAX,DWORD PTR SS:[EBP-C]` ; Name

`004B654F  |.  8A00          MOV AL,BYTE PTR DS:[EAX]` ; AL=Name[0]

`004B6551  |.  8845 F3       MOV BYTE PTR SS:[EBP-D],AL`

`004B6554  |.  8BF7          MOV ESI,EDI` ; ESI=Name 的长度

`004B6556  |.  4E            DEC ESI`

`004B6557  |.  85F6          TEST ESI,ESI`

`004B6559  |.  7E 22         JLE SHORT Crack.004B657D`

`004B655B  |.  BB 01000000   MOV EBX,1`

`004B6560  |>  8D45 F4       /LEA EAX,DWORD PTR SS:[EBP-C]`

`004B6563  |.  E8 B0E5F4FF   |CALL <Crack.InternalUniqueString>` ; UniqueString is used only in cases where an application casts a string to a PChar or PWidthChar and then modifies the contents of the string.

`004B6568  |.  8B55 F4       |MOV EDX,DWORD PTR SS:[EBP-C]`

`004B656B  |.  8A541A FF     |MOV DL,BYTE PTR DS:[EDX+EBX-1]` ; DL=Name[EBX-1]

`004B656F  |.  8B4D F4       |MOV ECX,DWORD PTR SS:[EBP-C]`

`004B6572  |.  0A1419        |OR DL,BYTE PTR DS:[ECX+EBX]` ; DL=Name[EBX-1] OR Name[EBX]

`004B6575  |.  885418 FF     |MOV BYTE PTR DS:[EAX+EBX-1],DL` ; 结果存在[EBP-C]的 EBX-1 中

`004B6579  |.  43            |INC EBX`

`004B657A  |.  4E            |DEC ESI`

`004B657B  |.^ 75 E3         \JNZ SHORT Crack.004B6560` ; 处理完 Len-1 位了?

`004B657D  |>  8D45 F4       LEA EAX,DWORD PTR SS:[EBP-C]`

`004B6580  |.  E8 93E5F4FF   CALL <Crack.InternalUniqueString>`

`004B6585  |.  8B55 F4       MOV EDX,DWORD PTR SS:[EBP-C]`

`004B6588  |.  8A543A FF     MOV DL,BYTE PTR DS:[EDX+EDI-1]` ; DL=Name 的最后一位

`004B658C  |.  0A55 F3       OR DL,BYTE PTR SS:[EBP-D]` ; 与 004B6551 对应，即 Name[0]

`004B658F  |.  885438 FF     MOV BYTE PTR DS:[EAX+EDI-1],DL` ; DL=Name[Len-1] OR Name[0]

`004B6593  |.  8D4D EC       LEA ECX,DWORD PTR SS:[EBP-14]` ; \\--对 Name 的第 i 位与第 i+1 位（最后一位与第 0 位）进行 OR 运算

`004B6596  |.  8B55 F4       MOV EDX,DWORD PTR SS:[EBP-C]` ; 记为 NameOR，这里="][_OOOKOOW"

`004B6599  |.  8B45 FC       MOV EAX,DWORD PTR SS:[EBP-4]`

`004B659C  |.  E8 C3000000   CALL Crack.004B6664` ; 第二层运算

10、单步跟进 `004B659C  |.  E8 C3000000   CALL Crack.004B6664` ; 第二层运算

### 进一步分析

引用:

`004B6694  |.  8B45 FC       MOV EAX,DWORD PTR SS:[EBP-4]` ; NameOR

`004B6697  |.  E8 24E2F4FF   CALL <Crack.@LStrLen(String)>`

`004B669C  |.  8B55 FC       MOV EDX,DWORD PTR SS:[EBP-4]` ; /---EAX=长度,EDX=NameOR

`004B669F  |.  8A5402 FF     MOV DL,BYTE PTR DS:[EDX+EAX-1]` ; DL=NameOR[Len-1]

`004B66A3  |.  8855 FB       MOV BYTE PTR SS:[EBP-5],DL` ; NameOR 的最后一位

`004B66A6  |.  8BD8          MOV EBX,EAX`

`004B66A8  |.  83FB 02       CMP EBX,2`

`004B66AB  |.  7C 19         JL SHORT Crack.004B66C6`

`004B66AD  |>  8D45 FC       /LEA EAX,DWORD PTR SS:[EBP-4]`

`004B66B0  |.  E8 63E4F4FF   |CALL <Crack.InternalUniqueString>`

`004B66B5  |.  8B55 FC       |MOV EDX,DWORD PTR SS:[EBP-4]` ; NameOR="][_OOOKOOW"

`004B66B8  |.  8A541A FE     |MOV DL,BYTE PTR DS:[EDX+EBX-2]` ; DL=NameOR[EBX-2]

`004B66BC  |.  885418 FF     |MOV BYTE PTR DS:[EAX+EBX-1],DL` ; NameOR[EBX-1]=DL=NameOR[EBX-2],即后移一位

`004B66C0  |.  4B            |DEC EBX`

`004B66C1  |.  83FB 01       |CMP EBX,1`

`004B66C4  |.^ 75 E7         \JNZ SHORT Crack.004B66AD` ; 移了 Len-1 次？

`004B66C6  |>  8D45 FC       LEA EAX,DWORD PTR SS:[EBP-4]`

`004B66C9  |.  E8 4AE4F4FF   CALL <Crack.InternalUniqueString>`

`004B66CE  |.  8A55 FB       MOV DL,BYTE PTR SS:[EBP-5]` ; NameOR 的最后一位

`004B66D1  |.  8810          MOV BYTE PTR DS:[EAX],DL` ; NameOR[0]=Name[Len-1]

`004B66D3  |.  8D55 E4       LEA EDX,DWORD PTR SS:[EBP-1C]` ; \\--以上算法完成对 NameOR 进行循环右移一位

`004B66D6  |.  8B45 FC       MOV EAX,DWORD PTR SS:[EBP-4]` ; 运算后 NameOR="W][\_OOOKOO"

`004B66D9  |.  E8 1AF6FFFF   CALL <Crack.MD5?>` ; 用 F8 步过，时为在前面下的 004B5499（MD5） 中断，初步判断该函数为 MD5 加密

`004B66DE  |.  8D45 E4       LEA EAX,DWORD PTR SS:[EBP-1C]`

`004B66E1  |.  8D55 F4       LEA EDX,DWORD PTR SS:[EBP-C]`

`004B66E4  |.  E8 9BF5FFFF   CALL <Crack.HEX2Str>` ; 将 HEX 转成 STR

`004B66E9  |.  8B55 F4       MOV EDX,DWORD PTR SS:[EBP-C]` ; EDX＝ 1599C9A545CE5B7CBD605583D82E45B5

`004B66EC  |.  8D45 FC       LEA EAX,DWORD PTR SS:[EBP-4]` ; NameOR 的 MD5 值 1599C9A545CE5B7C60B7CD3D35D8CCFA 前 16 位一致

`004B66EF  |.  E8 A4DFF4FF   CALL <Crack.@LStrLAsg(void;void;void>;` ; 这里先不分析为什么后 16 位不一致，猜想作者可能对 MD5 算法变形了吧

`004B66F4  |.  8BC7          MOV EAX,EDI`

`004B66F6  |.  E8 05DFF4FF   CALL <Crack.@LStrClr(void;void)>`

`004B66FB  |.  8B45 FC       MOV EAX,DWORD PTR SS:[EBP-4]`

`004B66FE  |.  E8 BDE1F4FF   CALL <Crack.@LStrLen(String)>`

`004B6703  |.  8BF0          MOV ESI,EAX` ; NameORMD5 的长度

`004B6705  |.  85F6          TEST ESI,ESI`

`004B6707  |.  7E 3D         JLE SHORT Crack.004B6746`

`004B6709  |.  BB 01000000   MOV EBX,1` ; /-------------------

`004B670E  |>  8B45 FC       /MOV EAX,DWORD PTR SS:[EBP-4]`

`004B6711  |.  8A4418 FF     |MOV AL,BYTE PTR DS:[EAX+EBX-1]` ; 对 NameORMD5[EBX-1]位

`004B6715  |.  3C 30         |CMP AL,30`

`004B6717  |.  72 29         |JB SHORT Crack.004B6742` ; <'0'

`004B6719  |.  3C 39         |CMP AL,39`

`004B671B  |.  77 25         |JA SHORT Crack.004B6742` ; >'9'

`004B671D  |.  8D45 E0       |LEA EAX,DWORD PTR SS:[EBP-20]` ; >='0' AND <='9'

`004B6720  |.  8B55 FC       |MOV EDX,DWORD PTR SS:[EBP-4]`

`004B6723  |.  8A541A FF     |MOV DL,BYTE PTR DS:[EDX+EBX-1]`

`004B6727  |.  E8 BCE0F4FF   |CALL <Crack.StrFromChar>`

`004B672C  |.  8B55 E0       |MOV EDX,DWORD PTR SS:[EBP-20]`

`004B672F  |.  8BC7          |MOV EAX,EDI`

`004B6731  |.  E8 92E1F4FF   |CALL <Crack.@LStrCat>`

`004B6736  |.  8B07          |MOV EAX,DWORD PTR DS:[EDI]`

`004B6738  |.  E8 83E1F4FF   |CALL <Crack.@LStrLen(String)>`

`004B673D  |.  83F8 0A       |CMP EAX,0A`

`004B6740  |.  7D 04         |JGE SHORT Crack.004B6746` ; >=10,取前 10 位数字部分

`004B6742  |>  43            |INC EBX`

`004B6743  |.  4E            |DEC ESI`

`004B6744  |.^ 75 C8         \JNZ SHORT Crack.004B670E`

`004B6746  |>  33C0          XOR EAX,EAX` ; \\以上算法为取 MD5 值的前 10 位数字

……

`004B676B  \.  C3            RETN`

11、在 `004B676B` 返回后回到 `004B65A1`

引用:

`004B659C  |.  E8 C3000000   CALL Crack.004B6664` ; 第二层运算

`004B65A1  |.  8B55 EC       MOV EDX,DWORD PTR SS:[EBP-14]` ; 运算结果="1599954557"

`004B65A4  |.  8D45 F4       LEA EAX,DWORD PTR SS:[EBP-C]`

`004B65A7  |.  E8 ECE0F4FF   CALL <Crack.@LStrLAsg(void;void;void>`

`004B65AC  |.  8B45 F4       MOV EAX,DWORD PTR SS:[EBP-C]`

`004B65AF  |.  E8 0CE3F4FF   CALL <Crack.@LStrLen(String)>`

`004B65B4  |.  8BF8          MOV EDI,EAX` ; /-------------------

`004B65B6  |.  8B45 F4       MOV EAX,DWORD PTR SS:[EBP-C]`

`004B65B9  |.  8A4438 FF     MOV AL,BYTE PTR DS:[EAX+EDI-1]`

`004B65BD  |.  8845 F3       MOV BYTE PTR SS:[EBP-D],AL`



## 12、返回到 004B6234

```
004B65C0  .|. 8BDF          MOV EBX,EDI

004B65C2  .|. 83FB 02       CMP EBX,2

004B65C5  .|. 7C 19         JL SHORT Crack.004B65E0

004B65C7  .|> 8D45 F4       /LEA EAX,DWORD PTR SS:[EBP-C]

004B65CA  .|. E8 49E5F4FF   |CALL <Crack.InternalUniqueString>

004B65CF  .|. B55 F4        |MOV EDX,DWORD PTR SS:[EBP-C]

004B65D2  .|. 8A541A FE     |MOV DL,BYTE PTR DS:[EDX+EBX-2]

004B65D6  .|. 885418 FF     |MOV BYTE PTR DS:[EAX+EBX-1],DL      ;  NameORMD5[i-1]=NameORMD5[i-2],右移一位

004B65DA  .|. 4B            |DEC EBX

004B65DB  .|. 83FB 01       |CMP EBX,1

004B65DE  .|.^ 75 E7        \JNZ SHORT Crack.004B65C7            ;  处理 Len-1 次

004B65E0  .|> 8D45 F4       LEA EAX,DWORD PTR SS:[EBP-C]

004B65E3  .|. E8 30E5F4FF   CALL <Crack.InternalUniqueString>

004B65E8  .|. 8A55 F3       MOV DL,BYTE PTR SS:[EBP-D]

004B65EB  .|. 8810          MOV BYTE PTR DS:[EAX],DL             ;  ---循环右移一位

004B65ED  .|. 8B45 F4       MOV EAX,DWORD PTR SS:[EBP-C]         ;  KeyFind="7159995455"

004B65F0  .|. E8 CBE2F4FF   CALL <Crack.@LStrLen(String)>

004B65F5  .|. 8BF0          MOV ESI,EAX

004B65F7  .|. 85F6          TEST ESI,ESI

004B65F9  .|. 7E 27         JLE SHORT Crack.004B6622

004B65FB  .|. BB 01000000   MOV EBX,1

004B6600  .|> 8D45 F4       /LEA EAX,DWORD PTR SS:[EBP-C]

004B6603  .|. E8 10E5F4FF   |CALL <Crack.InternalUniqueString>

004B6608  .|. 8B55 F4       |MOV EDX,DWORD PTR SS:[EBP-C]

004B660B  .|. 0FB6541A FF   |MOVZX EDX,BYTE PTR DS:[EDX+EBX-1]   ;  取第[EBX-1]位

004B6610  .|. 8B0D 00E44C00 |MOV ECX,DWORD PTR DS:[4CE400]       ;  KeyStr="5497312680"

004B6616  .|. 8A5411 D0     |MOV DL,BYTE PTR DS:[ECX+EDX-30]     ;  取固定串中[EBX-1]对应数位数的字符

004B661A  .|. 885418 FF     |MOV BYTE PTR DS:[EAX+EBX-1],DL      ;  Code[EBX-1]=KeyStr[KeyFind[EBX-1]-30]

004B661E  .|. 43            |INC EBX

004B661F  .|. 4E            |DEC ESI

004B6620  .|.^ 75 DE        \JNZ SHORT Crack.004B6600

004B6622  .|> 8B45 FC       MOV EAX,DWORD PTR SS:[EBP-4]

004B6625  .|. 05 34030000   ADD EAX,334                           ;  注册码保存地址

004B662A  .|. 8B55 F4       MOV EDX,DWORD PTR SS:[EBP-C]          ;  最终注册码="6410001311"

004B662D  .|. E8 22E0F4FF   CALL <Crack.@LStrAsg>

004B6632  .|> 33C0          XOR EAX,EAX

...

004B6654  .\. C3            RETN

引用: 004B622F  .|. E8 CC020000   CALL Crack.004B6500                  ;  计算 Code
004B6234  .|. 8D55 C8       LEA EDX,DWORD PTR SS:[EBP-38]
004B6237  .|. 8B83 30030000 MOV EAX,DWORD PTR DS:[EBX+330]
004B623D  .|. E8 DA5FFAFF   CALL <Crack.TControl.GetText(TControl)>  ;  取假 Code
004B6242  .|. 8B55 C8       MOV EDX,DWORD PTR SS:[EBP-38]
004B6245  .|. 8D83 38030000 LEA EAX,DWORD PTR DS:[EBX+338]         ;  保存假 Code,
004B624B  .|. E8 04E4F4FF   CALL <Crack.@LStrAsg>
004B6250  .|. 6A 00         PUSH 0                                  ;  lParam = 0
004B6252  .|. 6A 00         PUSH 0                                  ;  wParam = 0
004B6254  .|. 68 06140000   PUSH 1406                               ;  Message = MSG(1406)
004B6259  .|. 8BC3          MOV EAX,EBX
004B625B  .|. E8 A4C7FAFF   CALL <Crack.QForms.TCustomForm.GetClientHa>
004B6260  .|. 50            PUSH EAX                                  ; |hWnd
004B6261  .|. E8 4E11F5FF   CALL <JMP.&user32.PostMessageA>         ; 通过提交消息来处判断注册码是否正确,在这里下 hr EBX+338 硬件访问断点，这样在消息处理访问到假码就可以中断下来。
```

## 13、在 004B6261 下 hr EBX+338 硬件访问断点后，按 F9 运行，在 004B6373 中断

```
004B636D  .|. 8B83 38030000 MOV EAX,DWORD PTR DS:[EBX+338]         ;  假码
004B6373  .|. E8 48E7F4FF   CALL <Crack.@LStrToPChar(String)>
004B6378  .|. 50            PUSH EAX
004B6379  .|. 8B83 34030000 MOV EAX,DWORD PTR DS:[EBX+334]         ;  真码
004B637F  .|. E8 3CE7F4FF   CALL <Crack.@LStrToPChar(String)>
004B6384  .|. 5A            POP EDX
004B6385  .|. E8 9E31F5FF   CALL <Crack.StrComp(PChar;PChar)>       ;  关键比较
004B638A  .|. 85C0          TEST EAX,EAX
004B638C  .|. 75 70         JNZ SHORT Crack.004B63FE               ;  关键跳

...

004B639E  .|. BA 44644B00   MOV EDX,Crack.004B6444                 ;  ASCII "Software\Hide Window Hotkey"

...

004B63BB  .|. BA 68644B00   MOV EDX,Crack.004B6468                 ;  ASCII "Name"

...

004B63D8  .|. BA 78644B00   MOV EDX,Crack.004B6478                 ;  ASCII "Code"
004B63DD  .|. 8BC6          MOV EAX,ESI
004B63DF  .|. E8 7C47F8FF   CALL <Crack.TRegistry.WriteString>     ;  写注册表

...

004B63F2  .|. C783 4C020000>MOV DWORD PTR DS:[EBX+24C],1           ;  注册标志？
004B63FC  .|. EB 16         JMP SHORT Crack.004B6414
004B63FE  .|> 6A 00         PUSH 0                                  ;  错误提示
004B6400  .|. 6A 00         PUSH 0
004B6402  .|. 68 07140000   PUSH 1407                               ;  MSG=1407
004B6407  .|. 8BC3          MOV EAX,EBX
004B6409  .|. E8 F6C5FAFF   CALL <Crack.QForms.TCustomForm.GetClientHa>
004B640E  .|. 50            PUSH EAX                                  ; |hWnd
004B640F  .|. E8 A00FF5FF   CALL <JMP.&user32.PostMessageA>         ;  PostMessageA
```

## 14、注册算法分析

以上对程序的注册算法已基本分析完成，但要写出注册机，还是分析作者对 `MD5` 算法做了什么手脚，导致该程序的 `MD5` 加密结果与正常的 `MD5` 加密有所不同，经分析（我是通过一个正常的 `MD5calculator.`，反编译后与该程序的 `MD5` 算法进行比较，最终发现差别之处，这里不详细说明）：

```
引用: 004B5B13  .|. 68 91D386EB   PUSH EB86D391                ;  MD5 常量
004B5B18  .|. 8BC6          MOV EAX,ESI
004B5B1A  .|. 8B4D 00       MOV ECX,DWORD PTR SS:[EBP]
004B5B1D  .|. 8B17          MOV EDX,DWORD PTR DS:[EDI]
004B5B1F  .|. E8 28F8FFFF   CALL <Crack.II>
004B5B24  .|. 8B0424        MOV EAX,DWORD PTR SS:[ESP]         ;  中间计算一样，最后结果相加
004B5B27  .|. 8B13          MOV EDX,DWORD PTR DS:[EBX]
004B5B29  .|. 0110          ADD DWORD PTR DS:[EAX],EDX         ;     [0]=67452301+3E847614=A5C99915
004B5B2B  .|. 8B0424        MOV EAX,DWORD PTR SS:[ESP]         ;  正常[0]=67452301+3E847614=A5C99915
004B5B2E  .|. 8B16          MOV EDX,DWORD PTR DS:[ESI]
004B5B30  .|. 0150 04       ADD DWORD PTR DS:[EAX+4],EDX       ;     [1]=EFCDAB89+8C8E22BC=7C5BCE45
004B5B33  .|. 8B0424        MOV EAX,DWORD PTR SS:[ESP]         ;  正常[1]=EFCDAB89+8C8E22BC=7C5BCE45
004B5B36  .|. 8B17          MOV EDX,DWORD PTR DS:[EDI]
004B5B38  .|. 0150 0C       ADD DWORD PTR DS:[EAX+C],EDX       ;     [3]=10325476+A512DA62=B5452ED8
004B5B3B  .|. 8B0424        MOV EAX,DWORD PTR SS:[ESP]         ;  正常[3]=10325476+EA9A83BF=FACCD835
004B5B3E  .|. 8B55 00       MOV EDX,DWORD PTR SS:[EBP]
004B5B41  .|. 0150 08       ADD DWORD PTR DS:[EAX+8],EDX       ;     [2]=98BADCFE+EA9A83BF=835560BD
004B5B44  .|. 8D4424 18     LEA EAX,DWORD PTR SS:[ESP+18]      ;  正常[2]=98BADCFE+A512DA62=3DCDB760
```

对应为 `MD5` 算法里的 `MD5Update` => `MD5Transform` 函数中进行 4 轮变化后结果相加时作了修改。

正常 `MD5` 算法：

```
  state[0] += a;
  state[1] += b;
  state[2] += c;
  state[3] += d;
```

该程序 `MD5` 算法：

```
  state[0] += a;
  state[1] += b;
  state[2] += d;
  state[3] += c;
```

以上是对该程序的所有注册算法进行分析。

## 【算法总结】

1. 用户名长度不能小于 8，不能包含空格，不能为单一字符。
2. 对用户名的第 `i` 位与第 `i+1` 位（最后一位与第 `0` 位）进行 OR 运算。
3. 对 2 的结果循环右移一字节。
4. 对 3 的结果进行变形 `MD5` 运算（该结果与正常 `MD5` 值的前 16 字节一致，后 16 字节不同）。
5. 取 4 的结果前 10 位数字部分。
6. 对 5 的结果循环右移一字节。
7. 根据 6 的结果对应的数字取固定字符串“5497312680”对应位，即为注册码。

这里附上用 `VC++6.0` 编写的注册机的原码，注册机是经《加密与解密》第三版 6.1.1 `MD5` 算法的 `MD5calculator` 修改而成的。

下面是部分原码：

```



```c
MD5_CTX context;
  long dtLength;
  int i;
  int len;
  int j;
  TCHAR *szKeyStr="5497312680";
  TCHAR sTemp=' ';

  TCHAR szName[MAXINPUTLEN]={0};
  TCHAR szHash[MAXINPUTLEN]={0};
  TCHAR szBuffer[MAXINPUTLEN]={0};

  dtLength=GetDlgItemText(hWnd, IDC_TXT0, szName, sizeof(szName)/sizeof(TCHAR)+1);   
  len=strlen(szName);
  if (len<=8)//长度判断
  {
    SetDlgItemText(hWnd, IDC_TXT1, "Name 必须大于 8 位！");
    return FALSE;
  }
  for(i=0;i < len; i++)//是否包含 空格
    if(szName[i]==' ')
    {
    SetDlgItemText(hWnd, IDC_TXT1, "Name 不能包含空格！");
    return FALSE;
    }       
  sTemp=szName[0];
  i=1;
  while(i<len)
  {
    if(sTemp==szName[i])
      i++;
    else
      break;
  }
  if(i>=len)
  {
    SetDlgItemText(hWnd, IDC_TXT1, "Name 不能为单一字符！");
    return FALSE;
  }

  for(i=0;i < len -1; i++)
    szName[i]=szName[i] | szName[i+1];
  szName[len-1]=szName[len-1] | sTemp;//各字节进行 OR 运算
   
  sTemp=szName[len-1];
  for(i=len-1; i > 0 ;i--)
    szName[i]=szName[i-1];
  szName[0]=sTemp;//循环右移一字节

  MD5Init(&context);
  MD5Update(&context, szName, dtLength);//该函数里的 MD5Transform 函数变形了
  MD5Final(szHash, &context);
  for(i=0; i < 16; i++)   // 将 szHash[]中的 16 进制转换成字符形式显示
  {
    wsprintf(&szBuffer[i*2], "%02X", *(byte*)(szHash+i));
  }//变形 MD5
   
  i=0;
  len=0;
  while(len<10 && i<32)
  {
    if(szBuffer[i]>='0' && szBuffer[i]<='9')
    {
      szName[len]=szBuffer[i];
      len+=1;
    }
    i++;
  }//取前 10 位数字
  szName[len]='\0';
  if(len<1)
  {
    SetDlgItemText(hWnd, IDC_TXT1,"注册码为空？！");        
    return TRUE;
  }

  sTemp=szName[len-1];
  for(i=len-1; i > 0 ;i--)
    szName[i]=szName[i-1];
  szName[0]=sTemp;//循环右移一字节

  for(i=0; i < len ;i++)
  {
    j=szName[i]-'0';
    szName[i]=szKeyStr[j];//根据 szName[i]值对取应 szKeyStr 值
  }

  SetDlgItemText(hWnd, IDC_TXT1,szName);        
  return TRUE;  
```

## 总结

1. 软件虽然经过多次运算，但注册码还是在内存中出现，不分析算法也可以轻易写出内存注册机。

2. 不知作者是有意还是无意（为什么只变形后 `16` 位，而不 `32` 位一起变形呢？）将 `MD5` 变形了，这点会加大算法分析者的难度。

3. 将注册码计算、注册码判断、错误提示分开，多少会加大破解的难度（至少对于新手，本人一开始为了查找消息处理函数就花了大量时间）。

4. 没有很好地加解密提示字符串（或不提示），将提示串放在资源文件也能简单防止通过字符串查找破解。

5. 注册码与机器无关，只要有一对正确用户名和注册码就可以任意机器注册。

第一次写破文，不足之处还望大侠多多指教！谢谢！

---

## 版权声明

本文原创于看雪技术论坛，转载请注明作者并保持文章的完整，谢谢！

2010 年 04 月 09 日
