
# 给 OD 添加工具菜单

> 原文：[`www.pediy.com/kssd/pediy10/58098.html`](https://www.pediy.com/kssd/pediy10/58098.html)

给 OD 添加工具菜单

大家好，第一次写这种文章，写的不好，请见谅，有什么不明白的地方，请提出，谢谢！

OD 是我们常用的一个软件调试，在这里我就不做多的介绍了，进入正题

参考文章：API 手册，打造自己喜欢的 Ollydbg、看雪论坛精华 7 中的文章

修改流程->增加空间->添加函数->找插入菜单的位置->读取配置列表->插入菜单->处理工具菜单事件

## 第一步，增加空间

肯定需要有空间了，OD 本身是不够的，我们用 `ZeroAdd`，为 OD 添加一个区段，大小为 `2000`（HEX），名字 `patch`，把我们所要写的代码写到这个区段中

## 第二步，添加函数

我们这次用到是 `lstrlenA`，这个函数取数据，打开 `LoaderPE`->`PE 编辑`->修改的 OD->目录->单击“输入表”后面的"..."按钮，打开输入表对话，在 `OLE32.dll` 这行单击右键选择“添加导入表”，在打开的“添加导入函数”窗口中 DLL 处输入：`Kernel32.dll`,在 API 处，输入“`lstrlenA`”，点确定->关闭添加导入表对话框->保存->保存－>关闭 Loadpe，到时函数添加完成

## 第三步，找 OD 启动时的消息流

让他根本我们的配置加载工具菜单。方法：大家都知道，OD 在启动后会加载插件，就让 OD 添加插件菜单后，添加我们的工具菜单吧。用另一个 OD 打开我们要修改的 OD，插入菜单的 API 是 `InsertMenu`，

在 OD 命令行输入 `bp InsertMenuA,F9` 运行,OD 断下，Alt＋F9 返回,来到这里

```
00496B06  |.  51            PUSH ECX                                  ; /pNewItem
00496B07  |.  8B4424 0C     MOV EAX,DWORD PTR SS:[ESP+C]             ; |
00496B0B  |.  50            PUSH EAX                                  ; |NewItemID
00496B0C  |.  68 10040000   PUSH 410                                  ; |Flags = MF_BYPOSITION|MF_ENABLED|MF_STRING|MF_POPUP
00496B11  |.  6A 03         PUSH 3                                    ; |ItemID = 3
00496B13  |.  8B15 7C3B4D00 MOV EDX,DWORD PTR DS:[4D3B7C]             ; |
00496B19  |.  52            PUSH EDX                                  ; |/hWnd => 0014056A (class='Aytcgjb')
00496B1A  |.  E8 49890100   CALL <JMP.&USER32.GetMenu>                ; |\\GetMenu
00496B1F  |.  50            PUSH EAX                                  ; |hMenu
00496B20  |.  E8 9D890100   CALL <JMP.&USER32.InsertMenuA>           ; \\InsertMenuA
00496B25  |.  8B0D 7C3B4D00 MOV ECX,DWORD PTR DS:[4D3B7C]
00496B2B  |.  51            PUSH ECX                                  ; /hWnd => 0014056A (class='Aytcgjb')
00496B2C  |.  E8 A1880100   CALL <JMP.&USER32.DrawMenuBar>           ; \\DrawMenuBar
```

断续返回

```
00438EE6   .  833D B8544D00>CMP DWORD PTR DS:[4D54B8],0
00438EED   .  0F84 27010000 JE ollydbgb.0043901A
00438EF3   .  C705 C8544D00>MOV DWORD PTR DS:[4D54C8],1
00438EFD   .  F605 CC544D00>TEST BYTE PTR DS:[4D54CC],2
00438F04   .  74 05         JE SHORT ollydbgb.00438F0B
00438F06   .  E8 E5220200   CALL ollydbgb._Createlistwindow
00438F0B   >  F605 CC544D00>TEST BYTE PTR DS:[4D54CC],4
```

此时，OD 已经添加了插件菜单，该我们动手了，

## 第四步，插入工具菜单

`566000`-`5667FFE` 是我们添加的空间

`00438EE6  CMP DWORD PTR DS:[4D54B8],0` 改成 `JMP 005661FE` 跳到我们的空间去

字符串`"Tools"`,ollydbg.ini 中的工具列表区节的区节名,OD 添加如下

```
005661E6    54              PUSH ESP
005661E7    6F              OUTS DX,DWORD PTR ES:[EDI]               ; I/O 命令
005661E8    6F              OUTS DX,DWORD PTR ES:[EDI]               ; I/O 命令
005661E9    6C              INS BYTE PTR ES:[EDI],DX                 ; I/O 命令
005661EA    73 00           JNB SHORT 复件_oll.005661EC
```

字符串`"工具(&T)"`,为 OD 添加的菜单名称

```
005661D5    B9 A4BEDF28     MOV ECX,28DFBEA4
005661DA    26:54           PUSH ESP                                 ; 多余的前缀
005661DC    2900            SUB DWORD PTR DS:[EAX],EAX
```

DIY 一定要注意的问题，一、堆栈平台问题，二、跨平台问题，本人首次修改，都遇到了

流程->读取工具列表名称->添加到一个菜单中->插入菜单

用到函数：

创建一个菜单,如果成功返回创建菜单的句柄，没有参数

`HMENU CreateMenu(VOID);`

取 INI 的配置文件,如果 `lpKeyName` 为 `NULL`，则读取这个区节中所有工具名称

```
DWORD GetPrivateProfileString(
  LPCTSTR lpAppName,        // section name
  LPCTSTR lpKeyName,        // key name
  LPCTSTR lpDefault,        // default string
  LPTSTR lpReturnedString,  // destination buffer
  DWORD nSize,              // size of destination buffer
  LPCTSTR lpFileName        // initialization file name
);
```

把读取到的工具名称，追到了一个菜单中,我们要用他，把读取的工具名称添加到菜单句柄

```
BOOL AppendMenu(
  HMENU hMenu,         // handle to menu
  UINT uFlags,         // menu-item options
  UINT_PTR uIDNewItem, // identifier, menu, or submenu
  LPCTSTR lpNewItem    // menu-item content
);
```

把创建的菜单添加到 OD 的菜单中

```
BOOL InsertMenu(
  HMENU hMenu,          // handle to menu
  UINT uPosition,       // item that new item precedes
  UINT uFlags,          // options
  UINT_PTR uIDNewItem,  // identifier, menu, or submenu
  LPCTSTR lpNewItem     // menu item content
);
```

重画菜单栏

```
BOOL DrawMenuBar(
  HWND hWnd  // handle to window
);
```

堆栈平衡问题：做完以后遇到堆栈平台问题，`popad` 出栈时，堆栈中参数少了一个，所以使用使用两次 `PUSHAD`，大家以后再 DIY 时，注意一下。

我们开始了

```
005661FE    9C              PUSHFD      寄存器之栈
005661FF    60              PUSHAD
00566200    60              PUSHAD
00566201    68 A4534D00     PUSH 复件_oll.004D53A4                     ; INI 的文件路径
00566206    68 00020000     PUSH 200                                  ; 200 个字节的位置
0056620B    68 00775600     PUSH 复件_oll.00567700                     ; 存放取出字符串的地址
00566210    68 F4615600     PUSH 复件_oll.005661F4
00566215    6A 00           PUSH 0
00566217    68 E6615600     PUSH 复件_oll.005661E6                     ; ASCII "Tools"
0056621C    E8 698EF4FF     CALL <JMP.&KERNEL32.GetPrivateProfileStr>
```

读取 ini 中的工具名列表

```
00566221    BF 00775600     MOV EDI,复件_oll.00567700
00566226    803F 00         CMP BYTE PTR DS:[EDI],0
00566229  ^ 0F84 49FFFFFF   JE 复件_oll.00566178
```

这里列表是否为空，如果为空，则直接跳回返回处

```
0056622F    E8 5C91F4FF     CALL <JMP.&USER32.CreateMenu>
00566234    8BD8            MOV EBX,EAX
00566236    75 05           JNZ SHORT 复件_oll.0056623D
00566238  ^ E9 44FFFFFF     JMP 复件_oll.00566181
```

创建一个菜单句柄，如果失败，跳到返回处

```
0056623D    BE F10A0000     MOV ESI,0AF1          0AF1 是第一个菜单的 ID＝2801
00566242    57              PUSH EDI              EDI 的地址中存放要添加的工具名称
00566243    56              PUSH ESI              菜单的 ID
00566244    6A 00           PUSH 0                指定添加方式
00566246    53              PUSH EBX              追到菜单的句柄
00566247    E8 0291F4FF     CALL <JMP.&USER32.AppendMenuA>
```

添加一个菜单

```
0056624C    57              PUSH EDI
0056624D    E9 38010000     JMP 复件_oll.0056638A      lstrlenA，跨平台问题移动下面
00566252    03F8            ADD EDI,EAX               地址＋长度+1＝下一个工具名的位置
00566254    47              INC EDI
00566255    46              INC ESI          菜单 ID 也要加 1
00566256    803F 00         CMP BYTE PTR DS:[EDI],0
00566259  ^ 75 E7           JNZ SHORT 复件_oll.00566242
```

如果读完后，继续

```
0056625B    68 D5615600     PUSH 复件_oll.005661D5
00566260    53              PUSH EBX
00566261    68 10040000     PUSH 410
00566266    6A 05           PUSH 5
00566268    8B15 7C3B4D00   MOV EDX,DWORD PTR DS:[4D3B7C]
0056626E    52              PUSH EDX
0056626F    E8 F491F4FF     CALL <JMP.&USER32.GetMenu>
00566274    50              PUSH EAX
00566275    E8 4892F4FF     CALL <JMP.&USER32.InsertMenuA>
```

插入菜单,参加 OD 插入插件菜单的方法

```
0056627A    8B0D 7C3B4D00   MOV ECX,DWORD PTR DS:[4D3B7C]
00566280    E8 4D91F4FF     CALL <JMP.&USER32.DrawMenuBar>
```

重画菜单栏

```
00566285  ^ E9 EFFEFFFF     JMP 复件_oll.00566179
```

跳到返回处

返回处代码

```
00566178    58              POP EAX
00566179    58              POP EAX
0056617A    58              POP EAX
0056617B    58              POP EAX
0056617C    58              POP EAX
0056617D    58              POP EAX
0056617E    58              POP EAX
0056617F    58              POP EAX
00566180    61              POPAD
00566181    9D              POPFD
00566182    833D B8544D00 0>CMP DWORD PTR DS:[4D54B8],0
00566189  - E9 5F2DEDFF     JMP 复件_oll.00438EED
```

字符串`"kernel32.dll"`

```
00566372    6B65 72 6E      IMUL ESP,DWORD PTR SS:[EBP+72],6E
00566376    65:6C           INS BYTE PTR ES:[EDI],DX                 ; I/O 命令
00566378    3332            XOR ESI,DWORD PTR DS:[EDX]
0056637A    2E:             PREFIX CS:                               ; 多余的前缀
0056637B    64:6C           INS BYTE PTR ES:[EDI],DX                 ; I/O 命令
0056637D    6C              INS BYTE PTR ES:[EDI],DX                 ; I/O 命令
```

字符串`"lstrlenA"`
