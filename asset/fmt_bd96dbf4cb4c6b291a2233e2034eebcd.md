
# ASPROTECT 2.0

> 原文：[`www.pediy.com/kssd/pediy06/pediy6725.htm`](https://www.pediy.com/kssd/pediy06/pediy6725.htm)

**ASPROTECT 2.0 脱壳示例**

- **【目 标】**：Win98’s notepad
- **【工 具】**：`Olydbg1.1` (diy 版)、`LORDPE`、`ImportREC1.6F`
- **【任 务】**：简单的脱一下 asprotect 的壳
- **【操作平台】**：Windows Xp sp2
- **【作 者】**：loveboom\[DFCG\]\[FCG\]\[US\]

**【简要说明】**：有一段时间没写文章了，这篇文章本来是想在奥运会那时写出来，因为公事比较多，有空的时候心情不好等原因，所以一直放到现在才写。

**【实例下载】**：点击此处下载 或 [`bbs.pediy.com/showthread.php?s=&threadid=6320`](http://bbs.pediy.com/showthread.php?s=&threadid=6320)

## 详细过程

**【详细过程】**：

设置忽略全部异常，去掉调试标志。

### 1. 载入与断点设置

载入后到这里：

```
00401000 >  68 01D040>PUSH 0040D001                     ; EP 壳的入口
00401005    E8 010000>CALL 0040100B
```

载入后，直接 `g LoadLibraryA` 运行到 `LoadLibraryA` 这个 API 处。

```
7C801D77 >  8BFF      MOV EDI,EDI                      ; LoadLibraryA
7C801D79    55        PUSH EBP
```

到了后，`ALT+F9` 执行到返回：

```
009884B7    8985 4D29>MOV DWORD PTR SS:[EBP+44294D],EAX        ; 返回到这里
009884BD    C785 5129>MOV DWORD PTR SS:[EBP+442951],0
……
009885C1    61        POPAD
009885C2    75 08     JNZ SHORT 009885CC                    ; 看到这里，跟过 Aspack 的朋友就是知道这里是什么了
009885C4    B8 010000>MOV EAX,1
009885C9    C2 0C00   RETN 0C
009885CC    68 000000>PUSH 0                             ; 如果解压完壳代码这里会 push 一个返回的地址
009885D1    C3        RETN
```

返回到 `009884B7` 处后，右键查找全部字符串，然后在字符串窗中查找 `250`，这样就会看到这些东西：

![](img/04676c5091e93626f41c79dcdf4cb5b8.jpg)

双击 `10` 那里到了 cpu 窗口处：

```
0096CD66  /75 0A     JNZ SHORT 0096CD72
0096CD68  |68 C8D096>PUSH 96D0C8                       ; 双击到这里,这里向上找到代码开始处
0096CD6D  |E8 BE6DFF>CALL 00963B30
```

向上找到这里：

```
0096CC70  /EB 01     JMP SHORT 0096CC73
0096CC72  |90        NOP
0096CC73  \8B43 08   MOV EAX,DWORD PTR DS:[EBX+8]       ; 这里开始对 IAT 进行处理
0096CC76   8B30      MOV ESI,DWORD PTR DS:[EAX]
0096CC78   8343 08 0>ADD DWORD PTR DS:[EBX+8],4
0096CC7C   8B43 08   MOV EAX,DWORD PTR DS:[EBX+8]
0096CC7F   8A00      MOV AL,BYTE PTR DS:[EAX]
0096CC81   884424 07 MOV BYTE PTR SS:[ESP+7],AL
0096CC85   FF43 08   INC DWORD PTR DS:[EBX+8]
0096CC88   85F6      TEST ESI,ESI
0096CC8A   75 1A     JNZ SHORT 0096CCA6                   ; 这里比较输入表是否处理完毕，如果没有就跳下去
0096CC8C   EB 01     JMP SHORT 0096CC8F
……
0096CCC9   FF43 08   INC DWORD PTR DS:[EBX+8]
0096CCCC   84C0      TEST AL,AL                           ; 这里开始对 AL 的值的不同而进行相关的处理
0096CCCE   75 20     JNZ SHORT 0096CCF0
……
0096CD59   8B4424 14 MOV EAX,DWORD PTR SS:[ESP+14]
0096CD5D   E8 DEECFF>CALL 0096BA40                       ; GetProcAddress
0096CD62   8BE8      MOV EBP,EAX
0096CD64   85ED      TEST EBP,EBP
0096CD66   75 0A     JNZ SHORT 0096CD72                   ; 判断获取 API 是否成功
0096CD68   68 C8D096>PUSH 96D0C8                       ; 双击到这里,这里向上找到代码开始处
0096CD6D   E8 BE6DFF>CALL 00963B30
0096CD72   837C24 20>CMP DWORD PTR SS:[ESP+20],0
……
0096CDF7   E8 8469FF>CALL 00963780                       ; 当 AL 为 2 时对 IAT 的处理，跟进去
0096CDFC  ^ E9 72FEFF>JMP 0096CC73                       ; 处理完跳回去
……
```

### 2. 跟进分析

跟进看看：

```
00963756    8BC0      MOV EAX,EAX
00963758    55        PUSH EBP
00963759    8BEC      MOV EBP,ESP
0096375B    53        PUSH EBX
0096375C    8BD8      MOV EBX,EAX
0096375E    8BC2      MOV EAX,EDX
00963760    8BD1      MOV EDX,ECX
00963762    E8 79FFFF>CALL 009636E0
00963767    C603 E9   MOV BYTE PTR DS:[EBX],0E9            ; 先在这里也下个断，因为这里就是程序的 OEP
0096376A    8D53 01   LEA EDX,DWORD PTR DS:[EBX+1]         ; 这里原程跳就是壳抽程序代码的开始地方
0096376D    8902      MOV DWORD PTR DS:[EDX],EAX
0096376F    8B45 08   MOV EAX,DWORD PTR SS:[EBP+8]
00963772    8910      MOV DWORD PTR DS:[EAX],EDX
00963774    B8 050000>MOV EAX,5
00963779    5B        POP EBX
0096377A    5D        POP EBP
0096377B    C2 0400   RETN 4
0096377E    8BC0      MOV EAX,EAX
00963780    53        PUSH EBX                           ; 进到这里
00963781    8BD8      MOV EBX,EAX
00963783    8BC3      MOV EAX,EBX
00963785    E8 56FFFF>CALL 009636E0
0096378A    C603 E8   MOV BYTE PTR DS:[EBX],0E8            ; 这里把 IAT 的改成 call xxxxxx 的样式，所以我们这里要进去处理
0096378D    43        INC EBX
0096378E    8903      MOV DWORD PTR DS:[EBX],EAX
00963790    5B        POP EBX
00963791    C3        RETN
```

### 3. 修复代码

现在这里我们来修复一下：

先自己申请两块空间，当然你也可以直接找空闲的地方来写代码，我分别申请了 `00B90000` 和 `00BA0000` 这个块内存空间，`00B90000` 这块是用来写 patch 代码，`00BA0000` 是用来保存临时要存放的数据。

`00BA0000` 用来保存 DLL 的基址，`00BA0010` 用来保存要存放 IAT 的地址。

搞清这些东西后，我们开始写代码：

```
00963781    8BD8      MOV EBX,EAX
00963783    8BC3      MOV EAX,EBX
00963785    E8 56FFFF>CALL 009636E0
0096378A    E8 71C822>CALL 00B90000                       ; 调用我们改的代码处
0096378F    90        NOP
00963790    5B        POP EBX
00963791    C3        RETN
```

`00B90000` 用来写我们自己的修复代码：

```
00B90000    51        PUSH ECX                           ; 保护现场
00B90001    52        PUSH EDX
00B90002    8B5424 28 MOV EDX,DWORD PTR SS:[ESP+28]      ; 取出基址到 EDX 中
00B90006    3B15 0000>CMP EDX,DWORD PTR DS:[BA0000]      ; 比较基址是否相同
00B9000C    74 0D     JE SHORT 00B9001B
00B9000E    8915 0000>MOV DWORD PTR DS:[BA0000],EDX      ; 如果不同就写入新的基址
00B90014    8305 1000>ADD DWORD PTR DS:[BA0010],4        ; 并把填入 IAT 的地址再加上 4
00B9001B    8B0D 1000>MOV ECX,DWORD PTR DS:[BA0010]      ; 如果是第一次我们要手工写一下保存 IAT 的地址,我选择的是 40C000
00B90021    8929      MOV DWORD PTR DS:[ECX],EBP         ; 写入正确的函数
00B90023    66:C703 F>MOV WORD PTR DS:[EBX],15FF         ; 这里要看程序的情况而定，如果是 DELPHI 之类的，这果就可能是 FF25 了，因为是 C 的，所以这里是 ff15
00B90028    890E      MOV DWORD PTR DS:[ESI],ECX         ; 把存放 IAT 的地址放到程序里
00B9002A    8305 1000>ADD DWORD PTR DS:[BA0010],4        ; 保存 IAT 的地址＋4
00B90031    5A        POP EDX                             ; 还原现场
00B90032    59        POP ECX
00B90033    C3        RETN                               ; 返回到壳那边继续
```

### 4. AL=1 时的处理与修复

继续到 `AL=1` 时的处理：

```
0096CE07 ^\E9 67FEFF>JMP 0096CC73
0096CE0C    3C 01     CMP AL,1                           ; 当 AL＝1 的处理
0096CE0E    0F85 B200>JNZ 0096CEC6
……
0096CE7E    A1 B8A697>MOV EAX,DWORD PTR DS:[97A6B8]
0096CE83    8B00      MOV EAX,DWORD PTR DS:[EAX]
0096CE85    FFD0      CALL EAX                           ; GetProcAddress
0096CE87    8BE8      MOV EBP,EAX
0096CE89    85ED      TEST EBP,EBP
0096CE8B    75 0A     JNZ SHORT 0096CE97                 ; 如果获取成功就跳
0096CE8D    68 D8D096>PUSH 96D0D8                       ; ASCII "11
"
0096CE92    E8 996CFF>CALL 00963B30
0096CE97    8B0424    MOV EAX,DWORD PTR SS:[ESP]
0096CE9A    50        PUSH EAX
0096CE9B    68 08BC96>PUSH 96BC08
0096CEA0    8D4C24 20 LEA ECX,DWORD PTR SS:[ESP+20]
0096CEA4    8BD5      MOV EDX,EBP
0096CEA6    8BC3      MOV EAX,EBX
0096CEA8    E8 BFF4FF>CALL 0096C36C
0096CEAD    8B5424 0C MOV EDX,DWORD PTR SS:[ESP+C]       ; 这里壳又要对 IAT 破坏处理，我们这里又要自己修复一下
0096CEB1    8902      MOV DWORD PTR DS:[EDX],EAX         ; 这里写上我们自己的代码
0096CEB3    8B4424 0C MOV EAX,DWORD PTR SS:[ESP+C]
0096CEB7    8906      MOV DWORD PTR DS:[ESI],EAX
0096CEB9    0FB74424 >MOVZX EAX,WORD PTR SS:[ESP+4]
0096CEBE    0143 08   ADD DWORD PTR DS:[EBX+8],EAX
0096CEC1  ^ E9 ADFDFF>JMP 0096CC73                       ; 处理完就跳回去
```

当 `AL=1` 时的修复代码：

```
0096CEA8    E8 BFF4FF>CALL 0096C36C
0096CEAD    E8 8E3122>CALL 00B90040
0096CEB2    90        NOP
0096CEB3    90        NOP
0096CEB4    90        NOP
0096CEB5    90        NOP
0096CEB6    90        NOP
0096CEB7    90        NOP
0096CEB8    90        NOP
0096CEB9    0FB74424 >MOVZX EAX,WORD PTR SS:[ESP+4]
0096CEBE    0143 08   ADD DWORD PTR DS:[EBX+8],EAX
0096CEC1  ^ E9 ADFDFF>JMP 0096CC73
```

`00B90040` 处的修复代码：
