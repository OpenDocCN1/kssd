
# 美达通讯录

> 原文：[`www.pediy.com/kssd/pediy08/pediy8-58.htm`](https://www.pediy.com/kssd/pediy08/pediy8-58.htm)

## 破文信息

**【破文标题】**美达通讯录算法分析  

**【破文作者】**逍遥风\[\[OCN\]\]\[PYG\]  

**【作者邮箱】**`tc-xb@163.COM`  

**【破解工具】**`OD`, `peid`  

**【破解平台】**`winXP`  

**【软件名称】**美达通讯录 1.30  

**【原版下载】**[`www.meidasoft.com/`](http://www.meidasoft.com/)  

**【软件简介】**美达通讯录是一个小型简洁实用的通讯录管理程序，使用简单，界面美观大方。可以管理自己的亲朋好友、同学同事、业务客户等的通讯资料，能直接建立分类，如同学类、朋友类、同事类等任意的分类项目。启动时需要输入密码，使其它人不能看到你的通讯资料，让你的信息更安全。查询栏让你很快找到你的联系人。  

------------------------------------------------------------------------

## 分析过程

`PEID` 检测：`Borland Delphi 6.0 - 7.0` 无壳

根据注册错误的信息，用 `OD` 直接来到如下代码处

```
005EA33E  .|  55           PUSH EBP
005EA33F  .|  68 30A45E00  PUSH Contacts.005EA430
005EA344  .|  64:FF30      PUSH DWORD PTR FS:[EAX]
005EA347  .|  64:8920      MOV DWORD PTR FS:[EAX],ESP
005EA34A  .|  8D55 F8      LEA EDX,DWORD PTR SS:[EBP-8]
005EA34D  .|  8B83 0C030000 MOV EAX,DWORD PTR DS:[EBX+30C]
005EA353  .|  E8 58C8E9FF  CALL Contacts.00486BB0              ;  取注册名位数
005EA358  .|  8B45 F8      MOV EAX,DWORD PTR SS:[EBP-8]        ;  注册名进 EAX
005EA35B  .|  8D55 FC      LEA EDX,DWORD PTR SS:[EBP-4]
005EA35E  .|  E8 D9F5E1FF  CALL Contacts.0040993C
005EA363  .|  837D FC 00   CMP DWORD PTR SS:[EBP-4],0          ;  注册名位数与 O 进行比较
005EA367  .|  75 29        JNZ SHORT Contacts.005EA392          ;  等于 0 就出现提示
005EA369  .|  6A 40        PUSH 40
005EA36B  .|  68 3CA45E00  PUSH Contacts.005EA43C              ;  warning
005EA370  .|  68 44A45E00  PUSH Contacts.005EA444              ;  请输入注册名!
005EA375  .|  8BC3         MOV EAX,EBX
005EA377  .|  E8 0432EAFF  CALL Contacts.0048D580
005EA37C  .|  50           PUSH EAX                              ; |hOwner
005EA37D  .|  E8 26DCE1FF  CALL <JMP.&user32.MessageBoxA>       ; \MessageBoxA
005EA382  .|  8B83 0C030000 MOV EAX,DWORD PTR DS:[EBX+30C]
005EA388  .|  8B10         MOV EDX,DWORD PTR DS:[EAX]
005EA38A  .|  FF92 C4000000 CALL DWORD PTR DS:[EDX+C4]
005EA390  .|  EB 70        JMP SHORT Contacts.005EA402
005EA392  |>  8D55 F0      LEA EDX,DWORD PTR SS:[EBP-10]
005EA395  .|  8B83 10030000 MOV EAX,DWORD PTR DS:[EBX+310]
005EA39B  .|  E8 10C8E9FF  CALL Contacts.00486BB0              ;  取假码位数
005EA3A0  .|  8B45 F0      MOV EAX,DWORD PTR SS:[EBP-10]       ;  假码进 EAX
005EA3A3  .|  8D55 F4      LEA EDX,DWORD PTR SS:[EBP-C]
005EA3A6  .|  E8 91F5E1FF  CALL Contacts.0040993C
005EA3AB  .|  837D F4 00   CMP DWORD PTR SS:[EBP-C],0          ;  假码位数与 0 进行比较
005EA3AF  .|  75 29        JNZ SHORT Contacts.005EA3DA          ;  等于 0 就出现提示
005EA3B1  .|  6A 40        PUSH 40
005EA3B3  .|  68 3CA45E00  PUSH Contacts.005EA43C              ;  warning
005EA3B8  .|  68 54A45E00  PUSH Contacts.005EA454              ;  请输入注册码!
005EA3BD  .|  8BC3         MOV EAX,EBX
005EA3BF  .|  E8 BC31EAFF  CALL Contacts.0048D580
005EA3C4  .|  50           PUSH EAX                              ; |hOwner
005EA3C5  .|  E8 DEDBE1FF  CALL <JMP.&user32.MessageBoxA>       ; \MessageBoxA
005EA3CA  .|  8B83 10030000 MOV EAX,DWORD PTR DS:[EBX+310]
005EA3D0  .|  8B10         MOV EDX,DWORD PTR DS:[EAX]
005EA3D2  .|  FF92 C4000000 CALL DWORD PTR DS:[EDX+C4]
005EA3D8  .|  EB 28        JMP SHORT Contacts.005EA402
005EA3DA  |>  8BC3         MOV EAX,EBX
005EA3DC  .|  E8 1FFDFFFF  CALL Contacts.005EA100              ;  算法 CALL，进
005EA3E1  .|  84C0         TEST AL,AL
005EA3E3  .|  74 13        JE SHORT Contacts.005EA3F8
005EA3E5  .|  B8 6CA45E00  MOV EAX,Contacts.005EA46C           ;  注册成功!
005EA3EA  .|  E8 3133E5FF  CALL Contacts.0043D720
005EA3EF  .|  8BC3         MOV EAX,EBX
005EA3F1  .|  E8 F2FAFFFF  CALL Contacts.005E9EE8
005EA3F6  .|  EB 0A        JMP SHORT Contacts.005EA402
005EA3F8  |>  B8 80A45E00  MOV EAX,Contacts.005EA480           ;  注册码错误!
```

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

进入算法 `CALL`。。。来到这里

```
005EA100  /$ 55           PUSH EBP
005EA101  .| 8BEC         MOV EBP,ESP

省略一些代码。。。。。。

005EA125  .| 8B86 10030000 MOV EAX,DWORD PTR DS:[ESI+310]
005EA12B  .| E8 80CAE9FF  CALL Contacts.00486BB0              ;  取假码位数
005EA130  .| 8B45 F8      MOV EAX,DWORD PTR SS:[EBP-8]        ;  EBP-8=假码
005EA133  .| 8D55 FC      LEA EDX,DWORD PTR SS:[EBP-4]
005EA136  .| E8 01F8E1FF  CALL Contacts.0040993C
005EA13B  .| 8B45 FC      MOV EAX,DWORD PTR SS:[EBP-4]
005EA13E  .| 50           PUSH EAX
005EA13F  .| 8D55 EC      LEA EDX,DWORD PTR SS:[EBP-14]
005EA142  .| 8B86 0C030000 MOV EAX,DWORD PTR DS:[ESI+30C]
005EA148  .| E8 63CAE9FF  CALL Contacts.00486BB0              ;  取注册名位数
005EA14D  .| 8B45 EC      MOV EAX,DWORD PTR SS:[EBP-14]
005EA150  .| 8D55 F0      LEA EDX,DWORD PTR SS:[EBP-10]
005EA153  .| E8 E4F7E1FF  CALL Contacts.0040993C
005EA158  .| 8B55 F0      MOV EDX,DWORD PTR SS:[EBP-10]
005EA15B  .| 8D4D F4      LEA ECX,DWORD PTR SS:[EBP-C]
005EA15E  .| 8BC6         MOV EAX,ESI
005EA160  .| E8 8FFBFFFF  CALL Contacts.005E9CF4              ;  关键 CALL
005EA165  .| 8B55 F4      MOV EDX,DWORD PTR SS:[EBP-C]
005EA168  .| 58           POP EAX
005EA169  .| E8 76ACE1FF  CALL Contacts.00404DE4              ;  比较 CALL
005EA16E  .| 75 50        JNZ SHORT Contacts.005EA1C0         ;  相等就注册成功
```

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

进入关键 `CALL`。来到这里。。。

```
005E9CF4  /$ 55           PUSH EBP
005E9CF5  .| 8BEC         MOV EBP,ESP

省略一些代码

005E9D2D  .| 8B45 FC      MOV EAX,DWORD PTR SS:[EBP-4]
005E9D30  .| E8 63AFE1FF  CALL Contacts.00404C98              ;  取注册名位数
005E9D35  .| 8BF0         MOV ESI,EAX                         ;  注册名位数给 ESI
005E9D37  .| 85F6         TEST ESI,ESI
005E9D39  .| 7E 26        JLE SHORT Contacts.005E9D61
005E9D3B  .| BB 01000000  MOV EBX,1
005E9D40  |> 8D4D EC      /LEA ECX,DWORD PTR SS:[EBP-14]
005E9D43  .| 8B45 FC      |MOV EAX,DWORD PTR SS:[EBP-4]
005E9D46  .| 0FB64418 FF  |MOVZX EAX,BYTE PTR DS:[EAX+EBX-1]  ;  取注册名每一位的 ASCII 值
005E9D4B  .| 33D2         |XOR EDX,EDX
005E9D4D  .| E8 0203E2FF  |CALL Contacts.0040A054
005E9D52  .| 8B55 EC      |MOV EDX,DWORD PTR SS:[EBP-14]
005E9D55  .| 8D45 F8      |LEA EAX,DWORD PTR SS:[EBP-8]
005E9D58  .| E8 43AFE1FF  |CALL Contacts.00404CA0
005E9D5D  .| 43           |INC EBX
005E9D5E  .| 4E           |DEC ESI
005E9D5F  .|. 75 DF       \JNZ SHORT Contacts.005E9D40
005E9D61  |> 8B45 F8      MOV EAX,DWORD PTR SS:[EBP-8]        ;  注册名每一位的 ASCII 值连成一个字符串
005E9D64  .| E8 2FAFE1FF  CALL Contacts.00404C98              ;  取字符串的长度
005E9D69  .| 8BF0         MOV ESI,EAX
005E9D6B  .| 85F6         TEST ESI,ESI
005E9D6D  .| 7E 2C        JLE SHORT Contacts.005E9D9B
005E9D6F  .| BB 01000000  MOV EBX,1
005E9D74  |> 8B45 F8      /MOV EAX,DWORD PTR SS:[EBP-8]       ;  EBP-8=字符串长度
005E9D77  .| E8 1CAFE1FF  |CALL Contacts.00404C98
005E9D7C  .| 2BC3         |SUB EAX,EBX                         ;  EAX=EAX-EBX，EBX=计算次数
005E9D7E  .| 8B55 F8      |MOV EDX,DWORD PTR SS:[EBP-8]
005E9D81  .| 8A1402       |MOV DL,BYTE PTR DS:[EDX+EAX]
005E9D84  .| 8D45 E8      |LEA EAX,DWORD PTR SS:[EBP-18]
005E9D87  .| E8 34AEE1FF  |CALL Contacts.00404BC0
005E9D8C  .| 8B55 E8      |MOV EDX,DWORD PTR SS:[EBP-18]
005E9D8F  .| 8D45 F4      |LEA EAX,DWORD PTR SS:[EBP-C]
005E9D92  .| E8 09AFE1FF  |CALL Contacts.00404CA0
005E9D97  .| 43           |INC EBX                             ;  每计算一次 EBX+1
005E9D98  .| 4E           |DEC ESI                             ;  每计算一次 ESI-1
```



## 汇编代码分析

```assembly
005E9D99   |.^ 75 D9          \JNZ SHORT Contacts.005E9D74             ; 上面这个循环的作用就是把字符串倒序  

005E9D9B   |>  8D45 F8        LEA EAX,DWORD PTR SS:\[EBP-8\]  

005E9D9E   |.  50            PUSH EAX  

005E9D9F   |.  B9 04000000   MOV ECX,4                                  ; 使 ECX=4  

005E9DA4   |.  BA 01000000   MOV EDX,1                                  ; 使 EDX=1  

005E9DA9   |.  8B45 F4       MOV EAX,DWORD PTR SS:\[EBP-C\]             ; EBP-C=倒序后的字符串，设为字符串 A  

005E9DAC   |.  E8 47B1E1FF   CALL Contacts.00404EF8                    ; 取字符串 A 前 4 位  

005E9DB1   |.  8D45 F4       LEA EAX,DWORD PTR SS:\[EBP-C\]  

005E9DB4   |.  50            PUSH EAX  

005E9DB5   |.  B9 04000000   MOV ECX,4                                  ; 使 ECX=4  

005E9DBA   |.  BA 05000000   MOV EDX,5                                  ; 使 EDX=5  

005E9DBF   |.  8B45 F4       MOV EAX,DWORD PTR SS:\[EBP-C\]             ; EBP-C=倒序后的字符串  

005E9DC2   |.  E8 31B1E1FF   CALL Contacts.00404EF8                    ; 取字符串 A 的第 5 位到第八位  

005E9DC7   |.  8B45 F8       MOV EAX,DWORD PTR SS:\[EBP-8\]  

005E9DCA   |.  E8 C9AEE1FF   CALL Contacts.00404C98  

005E9DCF   |.  83F8 04       CMP EAX,4                                  ; 比较所得新的字符串是否为 4 位  

005E9DD2   |.  7D 2F         JGE SHORT Contacts.005E9E03               ; 大于四位就跳走直接进行运算  

005E9DD4   |.  8B45 F8       MOV EAX,DWORD PTR SS:\[EBP-8\]  

005E9DD7   |.  E8 BCAEE1FF   CALL Contacts.00404C98  

005E9DDC   |.  8BD8          MOV EBX,EAX  

005E9DDE   |.  83FB 03       CMP EBX,3                                  ; 比较新字符串是否为 3 位  

005E9DE1   |.  7F 20         JG SHORT Contacts.005E9E03  

005E9DE3   |>  8D4D E4       /LEA ECX,DWORD PTR SS:\[EBP-1C\]  

005E9DE6   |.  8BC3          |MOV EAX,EBX                                ; EBX=新字符串长度  

005E9DE8   |.  C1E0 02       |SHL EAX,2                                  ; EAX=EBX 乘以 2 的平方  

005E9DEB   |.  33D2          |XOR EDX,EDX                                ; EDX 清零  

005E9DED   |.  E8 6202E2FF   |CALL Contacts.0040A054  

005E9DF2   |.  8B55 E4       |MOV EDX,DWORD PTR SS:\[EBP-1C\]  

005E9DF5   |.  8D45 F8       |LEA EAX,DWORD PTR SS:\[EBP-8\]  

005E9DF8   |.  E8 A3AEE1FF   |CALL Contacts.00404CA0  

005E9DFD   |.  43            |INC EBX                                     ; 每计算一次 EBX+1  

005E9DFE   |.  83FB 04       |CMP EBX,4                                     ; 比较 EBX 是否等于 4  

005E9E01   |.^ 75 E0         \JNZ SHORT Contacts.005E9DE3               ; 小于就继续计算  

005E9E03   |>  8B45 F4       MOV EAX,DWORD PTR SS:\[EBP-C\]             ; 字符串 A 与计算结果合并  

005E9E06   |.  E8 8DAEE1FF   CALL Contacts.00404C98  

005E9E0B   |.  83F8 04       CMP EAX,4  

005E9E0E   |.  7D 2F         JGE SHORT <Contacts.输出计算结果>  

005E9E10   |.  8B45 F4       MOV EAX,DWORD PTR SS:\[EBP-C\]  

005E9E13   |.  E8 80AEE1FF   CALL Contacts.00404C98  

005E9E18   |.  8BD8          MOV EBX,EAX  

005E9E1A   |.  83FB 03       CMP EBX,3  

005E9E1D   |.  7F 20         JG SHORT <Contacts.输出计算结果>  

005E9E1F   |>  8D4D E0       /LEA ECX,DWORD PTR SS:\[EBP-20\]  

005E9E22   |.  8BC3          |MOV EAX,EBX                                 ; 使 EAX=EBX  

005E9E24   |.  C1E0 02       |SHL EAX,2                                  ; EAX=EBX 乘以 2 的平方  

005E9E27   |.  33D2          |XOR EDX,EDX  

005E9E29   |.  E8 2602E2FF   |CALL Contacts.0040A054  

005E9E2E   |.  8B55 E0       |MOV EDX,DWORD PTR SS:\[EBP-20\]  

005E9E31   |.  8D45 F4       |LEA EAX,DWORD PTR SS:\[EBP-C\]  

005E9E34   |.  E8 67AEE1FF   |CALL Contacts.00404CA0  

005E9E39   |.  43            |INC EBX                                     ; 每计算一次 EBX+1  

005E9E3A   |.  83FB 04       |CMP EBX,4                                     ; 比较 EBX 是否等于 4  

005E9E3D   |.^ 75 E0         \JNZ SHORT Contacts.005E9E1F               ; 小于就继续计算  

005E9E3F   |>|>  8D45 F0     LEA EAX,DWORD PTR SS:\[EBP-10\]           ; 输出计算结果  

005E9E42   |.  BA CC9E5E00   MOV EDX,Contacts.005E9ECC                ; con62bsar698  

005E9E47   |.  E8 24ACE1FF   CALL Contacts.00404A70                   ; 固定字符串 Con62bsae698  

005E9E4C   |.  8D45 DC       LEA EAX,DWORD PTR SS:\[EBP-24\]  

005E9E4F   |.  50            PUSH EAX  

005E9E50   |.  B9 04000000   MOV ECX,4                                  ; 使 ECX=4  

005E9E55   |.  BA 01000000   MOV EDX,1                                  ; EDX=1  

005E9E5A   |.  8B45 F0       MOV EAX,DWORD PTR SS:\[EBP-10\]  

005E9E5D   |.  E8 96B0E1FF   CALL Contacts.00404EF8                   ; 取固定字符串的前四位  

005E9E62   |.  FF75 DC       PUSH DWORD PTR SS:\[EBP-24\]  

005E9E65   |.  68 E49E5E00   PUSH Contacts.005E9EE4                   ; -  

005E9E6A   |.  FF75 F8       PUSH DWORD PTR SS:\[EBP-8\]  

005E9E6D   |.  8D45 D8       LEA EAX,DWORD PTR SS:\[EBP-28\]  

005E9E70   |.  50            PUSH EAX  

005E9E71   |.  B9 05000000   MOV ECX,5                                  ; 使 ECX=5  

005E9E76   |.  BA 05000000   MOV EDX,5                                  ; 使 EDX=5  

005E9E7B   |.  8B45 F0       MOV EAX,DWORD PTR SS:\[EBP-10\]  

005E9E7E   |.  E8 75B0E1FF   CALL Contacts.00404EF8  

005E9E83   |.  FF75 D8       PUSH DWORD PTR SS:\[EBP-28\]              ; 取固定字符串的 5 到 9 位  

005E9E86   |.  68 E49E5E00   PUSH Contacts.005E9EE4                   ; -  

省略一些代码。。。  

005E9EBC    .  5F            POP EDI  

005E9EBD    .  5E            POP ESI  

005E9EBE    .  5B            POP EBX  

005E9EBF    .  8BE5         MOV ESP,EBP  

005E9EC1    .  5D            POP EBP                                   ; 合并注册码并出栈  

005E9EC2    .  C3            RETN  
```

------------------------------------------------------------------------

## 算法总结

1. 取注册名每一位的 ASCII 值联成一个字符串。
2. 把所得到的字符串倒序。
3. 取字符串的前四位和 5 到 8 位，与固定字符串的 `con62bsar698` 的前四位和 5 到 9 位交叉组成注册码。

### 特殊情况处理

注意：如果注册名每一位的 ASCII 值联成的字符串小于 4 位或小于 8 位，就会进一步处理。

#### 小于 8 位的情况

- `0 * 2^2`
- `1 * 2^2`
- `2 * 2^2`
- `3 * 2^2`

得到一个四位的字符串。

#### 小于 4 位的情况（即输入的注册名只有一位）

1. 注册名的 ASCII 值倒序的结果与 `2 * 2^2`、`3 * 2^2` 所得的两个结果相合并，得到第一个四位字符串。
2. 再重复“小于 8 位”时的处理方法，得到第二个字符串。
3. 最后与固定字符串的 `con62bsar698` 交叉得到注册码。

### 示例

**注册名：** `tcxb`  
**注册码：** `Con6-26872bsar-3647`  

`4dd466d0e1e0cc123afbe504da2462ca`
