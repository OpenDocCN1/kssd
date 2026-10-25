
# Phatnotes

> 原文：[`www.pediy.com/kssd/pediy08/pediy8-215.htm`](https://www.pediy.com/kssd/pediy08/pediy8-215.htm)

转轮里的秘密，跟踪一非经典算法

Phatnotes 是一款功能强大的信息管理软件，同时拥有 Pocket PC 端、PalmOS 端以及 PC 端版本。内置强大的文本搜索、文档安全保护和 email 收发功能(同时允许你快速查看 email 地址和联络信息)。特点如下：

1. 可以按照主题，类别，创建/修改时间对信息进行归类
2. 数据兼容性良好，可以随意导出/导入文本信息，或在不同程序间进行复制、粘贴操作
3. 程序、数据占用存储空间小
4. 支持对数据进行密码保护
5. 简洁直观的用户操作界面
6. 可以在记事编辑器中选择 Send mail 按钮来直接进行邮件发送
7. 支持用户自定义浏览方式/参数
8. 强大的文档过滤、搜索功能
9. 支持多记事数据库同步
10. 支持 Microsoft Outlook，导入或到处 Outlook 记事和邮件信息
11. 内置记事类别管理功能，允许用户按记事的修改日期、类别、颜色或优先级进行分类
12. 支持记事间的超链接跳转
13. 支持文件附件(一个记事文件最多支持 32 个附件链接)
14. 提供目录树的数据浏览模式
15. 支持拼写检查
16. 支持文本格式设定
17. 支持 PhatPad 直接调用
18. 可以与 Palm OS 用户(装有 PhatNotes)的数据进行同步。此为 Phatnotes 的专业版，在同步更新功能方面做了改进。支持在 Pocket PC 与 PC(或 PalmOS)间对多个数据库中的记事，记事间的链接，文件附件和数据库层次等信息进行同步。

软件主页：

`http://www.phatware.com/`

最近总是心情浮躁，看到软件爆破了之。很是不爽。定下决心，潜心看看这个软件的算法，以解郁闷。

其实这个软件有 2 种注册方式，对应不同的版本，其中一种注册码是：4000610710001 而另外一种注册码格式是 xxxxx-xxxxx-xxxxx-xxxxx-xxxxx 这就是我为什么要跟踪他的原因了。（是在跟踪中无意看到算法要临时调用 dll 才发现的）

在 OD 中 `bp LoadLibraryA` 来到核心：

```
0046DF70  /$  53            PUSH    EBX
0046DF71  |.  56            PUSH    ESI
0046DF72  |.  68 D8105500   PUSH    005510D8                ; /FileName = "PltInstall.dll"
0046DF77  |.  FF15 60525000 CALL    [<&KERNEL32.LoadLibraryA>] ; \LoadLibraryA
0046DF7D  |.  8BF0          MOV     ESI, EAX
0046DF7F  |.  32DB          XOR     BL, BL
0046DF81  |.  85F6          TEST    ESI, ESI
0046DF83  |.  74 5A         JE      SHORT 0046DFDF
0046DF85  |.  68 C4105500   PUSH    005510C4                ; /ProcNameOrOrdinal = "PW_IsCodeCorrect"
0046DF8A  |.  56            PUSH    ESI                    ; |hModule
0046DF8B  |.  FF15 64525000 CALL    [<&KERNEL32.GetProcAddress>]  ; \GetProcAddress
0046DF91  |.  85C0          TEST    EAX, EAX
0046DF93  |.  74 43         JE      SHORT 0046DFD8
0046DF95  |.  8A4C24 14     MOV     CL, [ESP+14]
0046DF99  |.  84C9          TEST    CL, CL
0046DF9B  |.  74 2B         JE      SHORT 0046DFC8
0046DF9D  |.  8B4C24 10     MOV     ECX, [ESP+10]
0046DFA1  |.  85C9          TEST    ECX, ECX
0046DFA3  |.  74 23         JE      SHORT 0046DFC8
0046DFA5  |.  8039 00       CMP     BYTE PTR [ECX], 0
0046DFA8  |.  74 1E         JE      SHORT 0046DFC8
0046DFAA  |.  51            PUSH    ECX
0046DFAB  |.  8B4C24 10     MOV     ECX, [ESP+10]
0046DFAF  |.  68 01000010   PUSH    10000001
0046DFB4  |.  51            PUSH    ECX
0046DFB5  |.  FFD0          CALL    EAX                    ;  PltInsta.PW_IsCodeCorrect   调用 dll 的这个函数
0046DFB7  |.  85C0          TEST    EAX, EAX
0046DFB9  |.  56            PUSH    ESI                    ; /hLibModule
0046DFBA  |.  0F95C3        SETNE   BL                    ; |
0046DFBD  |.  FF15 68525000 CALL    [<&KERNEL32.FreeLibrary>]     ; \FreeLibrary
0046DFC3  |.  8AC3          MOV     AL, BL
0046DFC5  |.  5E            POP     ESI
0046DFC6  |.  5B            POP     EBX
0046DFC7  |.  C3            RETN
0046DFC8  |>  8B5424 0C     MOV     EDX, [ESP+C]
0046DFCC  |.  6A 00         PUSH    0
0046DFCE  |.  6A 01         PUSH    1
0046DFD0  |.  52            PUSH    EDX
0046DFD1  |.  FFD0          CALL    EAX
0046DFD3  |.  85C0          TEST    EAX, EAX
0046DFD5  |.  0F95C3        SETNE   BL
0046DFD8  |>  56            PUSH    ESI                    ; /hLibModule
0046DFD9  |.  FF15 68525000 CALL    [<&KERNEL32.FreeLibrary>]     ; \FreeLibrary
0046DFDF  |>  8AC3          MOV     AL, BL
0046DFE1  |.  5E            POP     ESI
0046DFE2  |.  5B            POP     EBX
0046DFE3  \.  C3            RETN

01302900   >SUB      ESP, 114                    ; 进入算法核心
01302906   >PUSH     EBX
01302907   >PUSH     EBP
01302908   >MOV      EBP, [ESP+120]
0130290F   >LEA      EAX, [ESP+1D]
01302913   >PUSH     ESI
01302914   >MOV      [ESP+10], ECX
01302918   >PUSH     EDI
01302919   >MOV      [ESP+10], EAX
0130291D   >MOV      EDI, EBP
0130291F   >OR       ECX, FFFFFFFF
01302922   >XOR      EAX, EAX
01302924   >XOR      EDX, EDX
01302926   >XOR      ESI, ESI
01302928   >LEA      EBX, [ESP+20]
0130292C   >REPNE    SCAS BYTE PTR ES:[EDI]
0130292E   >NOT      ECX
01302930   >DEC      ECX
01302931   >JE       SHORT 01302997
01302933   >MOV      AL, [EDX+EBP]
01302936   >CMP      AL, 2D                   ; 是 - 就去掉
01302938   >JE       SHORT 01302986
0130293A   >INC      ESI
0130293B   >CMP      ESI, 5                   ; 每组五位
0130293E   >JB       SHORT 01302947
01302940   >MOV      [EBX], AL                ; 取每组的第五位
01302942   >INC      EBX
01302943   >XOR      ESI, ESI
01302945   >JMP      SHORT 01302986
01302947   >CMP      AL, [13098A8]            ; 如果是数字必须在 2-9 之间  内存=2
0130294D   >JL       SHORT 01302957
0130294F   >CMP      AL, [13098AB]            ; 内存=9
01302955   >JLE      SHORT 0130296B
01302957   >CMP      AL, [13098A4]            ; 如果是字符在 A-Z 之间 DS:[013098A6]=4F ('O')
0130295D   >JL       013029E6
01302963   >CMP      AL, [13098A5]            ; DS:[013098A5]=5A ('Z')
01302969   >JG       SHORT 013029E6
0130296B   >CMP      AL, [13098A7]            ; 注册码中不能有 I  DS:[013098A7]=49 ('I')
01302971   >JE       SHORT 013029E6
01302973   >CMP      AL, [13098A6]            ; 注册码中不能有 O  DS:[013098A6]=4F ('O')
01302979   >JE       SHORT 013029E6
0130297B   >MOV      ECX, [ESP+10]
0130297F   >MOV      [ECX], AL
01302981   >INC      ECX
01302982   >MOV      [ESP+10], ECX
01302986   >MOV      EDI, EBP
01302988   >OR       ECX, FFFFFFFF
0130298B   >XOR      EAX, EAX
0130298D   >INC      EDX
0130298E   >REPNE    SCAS BYTE PTR ES:[EDI]
01302990   >NOT      ECX
01302992   >DEC      ECX
01302993   >CMP      EDX, ECX
01302995  ^>JB       SHORT 01302933
01302997   >MOV      ECX, [ESP+10]
0130299B   >LEA      EDI, [ESP+20]
0130299F   >XOR      EAX, EAX
013029A1   >MOV      BYTE PTR [ECX], 0
013029A4   >OR       ECX, FFFFFFFF
013029A7   >REPNE    SCAS BYTE PTR ES:[EDI]
013029A9   >NOT      ECX
013029AB   >DEC      ECX
013029AC   >CMP      ECX, 19                 ; 去掉-后的注册码长度
013029AF   >JNZ      SHORT 013029E6
013029B1   >PUSH     105
013029B6   >CALL     013036DD
013029BB   >MOV      ESI, EAX
013029BD   >ADD      ESP, 4
013029C0   >TEST     ESI, ESI
013029C2   >MOV      [ESP+10], ESI
013029C6   >JE       SHORT 013029E6
013029C8   >PUSH     105
013029CD   >CALL     013036DD
013029D2   >ADD      ESP, 4
013029D5   >MOV      [ESP+18], EAX
013029D9   >TEST     EAX, EAX
013029DB   >JNZ      SHORT 013029F5
013029DD   >PUSH     ESI
013029DE   >CALL     013036D2
013029E3   >ADD      ESP, 4
013029E6   >POP      EDI
013029E7   >POP      ESI
013029E8   >POP      EBP
013029E9   >XOR      EAX, EAX
013029EB   >POP      EBX
013029EC   >ADD      ESP, 114
013029F2   >RETN     4
013029F5   >XOR      EDI, EDI                 ; 到这里
013029F7   >XOR      EBX, EBX
013029F9   >XOR      ESI, ESI
013029FB   >MOV      DL, [ESP+EBX+20]         ; 经过上面的重排，取每组的第五位计算
013029FF   >MOV      ECX, [ESP+14]
01302A03   >PUSH     EDX
01302A04   >CALL     01302B30                 ; 转轮指针
01302A09   >AND      EAX, 1F
01302A0C   >MOV      ECX, ESI
01302A0E   >SHL      EAX, CL
01302A10   >ADD      ESI, 5
01302A13   >OR       EDI, EAX
01302A15   >INC      EBX
01302A16   >CMP      ESI, 19
01302A19  ^>JB       SHORT 013029FB
```



### 01302A1B
```asm
01302A1B  >MOV     ESI, [ESP+14]                     ; 经过上面分析，每组的第五位越靠近 A 越好  
01302A1F  >MOV     EAX, 1  
01302A24  >LEA     EDX, [EDI-3E8]  
01302A2A  >LEA     ECX, [ESI+75]  
01302A2D  >CMP     [ECX], EDX                        ; 判断上面计算的值的范围  
01302A2F  >JA      SHORT 01302A3A  
01302A31  >INC     EAX  
01302A32  >ADD     ECX, 25  
01302A35  >CMP     EAX, 33  
01302A38 ^>JB      SHORT 01302A2D  
01302A3A  >DEC     EAX  
01302A3B  >ADD     ESI, EAX                          ; 这个范围将用于下面取表的指针  
01302A3D  >LEA     ECX, [EAX+EAX*8]  
01302A40  >MOV     EDX, [ESI+ECX*4+50]  
01302A44  >LEA     EBX, [ESI+ECX*4]  
01302A47  >SUB     EDI, EDX                          ; 计算的值与表中的值相减  
01302A49  >SUB     EDI, 3E8  
01302A4F  >MOV     EBP, EDI  
01302A51  >CMP     EBP, 1                            ; 上面函数计算的值必须符合第一个条件。  
01302A54 ^>JB      SHORT 013029E6  
01302A56  >CMP     EBP, 7C830                        ; 第一个条件是值在 0x1--0x7C830 之间  
01302A5C ^>JA      SHORT 013029E6  
01302A5E  >MOV     EDX, [ESP+10]  
01302A62  >MOV     ECX, 41                           ; 转轮长度  
01302A67  >XOR     EAX, EAX  
01302A69  >MOV     EDI, EDX  
01302A6B  >REP     STOS DWORD PTR ES:[EDI]           ; 初始化转轮的内存地址  
01302A6D  >MOV     EDI, [ESP+18]  
01302A71  >MOV     ECX, 41  
01302A76  >REP     STOS DWORD PTR ES:[EDI]  
01302A78  >MOV     EDI, EDX  
01302A7A  >XOR     ESI, ESI  
01302A7C  >MOV     DL, [EBX+ESI+54]  
01302A80  >MOV     ECX, [ESP+14]  
01302A84  >PUSH    EDI  
01302A85  >PUSH    EDX  
01302A86  >CALL    01302C50                          ; 开始制造转轮  
01302A8B  >ADD     EDI, 8  
01302A8E  >INC     ESI  
01302A8F  >CMP     ESI, 20  
01302A92 ^>JB      SHORT 01302A7C  
01302A94  >TEST    EBP, EBP                          ; 如果是 0 就直接到注册码验证部分  
01302A96  >JBE     SHORT 01302ABC  
01302A98  >MOV     EBX, EBP                          ; 如果值不等于 0 就开始转动转轮  
01302A9A  >MOV     ESI, [ESP+18]  
01302A9E  >MOV     EDI, [ESP+10]  
01302AA2  >MOV     ECX, [ESP+14]  
01302AA6  >PUSH    102  
01302AAB  >PUSH    ESI  
01302AAC  >PUSH    EDI  
01302AAD  >CALL    01302CC0  
01302AB2  >MOV     ECX, 41  
01302AB7  >DEC     EBX                                 ; 转轮转动的次数就是上面计算的值  
01302AB8  >REP     MOVS DWORD PTR ES:[EDI], DWORD PTR [ESI]       ; 写入新的值  
01302ABA ^>JNZ     SHORT 01302A9A  
01302ABC  >MOV     EBP, [ESP+10]                     ; 第二个条件计算  
01302AC0  >MOV     DWORD PTR [ESP+1C], 1  
01302AC8  >MOV     ESI, EBP                          ; 转轮起始地址  
01302ACA  >XOR     EDI, EDI  
01302ACC  >XOR     BL, BL  
01302ACE  >XOR     EAX, EAX  
01302AD0  >CMP     BYTE PTR [ESI+EAX], 1  
01302AD4  >SETE    CL  
01302AD7  >OR      CL, BL  
01302AD9  >SHL     CL, 1  
01302ADB  >INC     EAX  
01302ADC  >MOV     BL, CL  
01302ADE  >CMP     EAX, 5                            ; 表中每 5 位一组  
01302AE1 ^>JB      SHORT 01302AD0  
01302AE3  >MOV     DL, [ESP+EDI+25]                  ; 每组剩下的四位组成新的串，共 20 位  
01302AE7  >MOV     ECX, [ESP+14]                     ; 0012E7E9  53 33 53 53 34 43 51 55 45 51 51 45 4D 43 51 35  S3SS4CQUEQQEMCQ5  
01302AEB  >PUSH    EDX  
01302AEC  >CALL    01302B30                          ; 用注册试验码变换，变换函数使用转轮指针函数  
01302AF1  >AND     EBX, 1F  
01302AF4  >CMP     EAX, EBX                          ; 注册码变相比较  
01302AF6  >JNZ     SHORT 01302B07  
01302AF8  >ADD     ESI, 5  
01302AFB  >INC     EDI  
01302AFC  >CMP     EDI, 14  
01302AFF ^>JB      SHORT 01302ACC  
01302B01  >MOV     ESI, [ESP+1C]                     ; 成功标志  
01302B05  >JMP     SHORT 01302B09  
01302B07  >XOR     ESI, ESI                          ; 失败标志  
01302B09  >PUSH    EBP  
01302B0A  >CALL    013036D2  
01302B0F  >MOV     EAX, [ESP+1C]  
01302B13  >PUSH    EAX  
01302B14  >CALL    013036D2  
01302B19  >ADD     ESP, 8  
01302B1C  >MOV     EAX, ESI  
01302B1E  >POP     EDI  
01302B1F  >POP     ESI  
01302B20  >POP     EBP  
01302B21  >POP     EBX  
01302B22  >ADD     ESP, 114  
01302B28  >RETN    4  
```

### 01302B30
```asm
01302B30   8A4C24 04      MOV     CL, [ESP+4]               ; 重点是这几个函数的值将作为指针  
01302B34   8A15 A4983001  MOV     DL, [13098A4]             ; A 到 Z 之间值处理  
01302B3A   33C0           XOR     EAX, EAX  
01302B3C   3ACA           CMP     CL, DL  
01302B3E   7C 13          JL      SHORT 01302B53  
01302B40   3A0D A5983001  CMP     CL, [13098A5]  
01302B46   7F 0B          JG      SHORT 01302B53  
01302B48   0FBED2         MOVSX   EDX, DL  
01302B4B   0FBEC1         MOVSX   EAX, CL  
01302B4E   2BC2           SUB     EAX, EDX                    ; 值-A  
01302B50   C2 0400        RETN    4  
01302B53   3A0D A8983001  CMP     CL, [13098A8]               ; 数字 2 的处理  
01302B59   75 0F          JNZ     SHORT 01302B6A  
01302B5B   0FBE05 A6983001 MOVSX  EAX, BYTE PTR [13098A6]     ; DS:[010598A6]=4F ('O')  
01302B62   0FBECA         MOVSX   ECX, DL  
01302B65   2BC1           SUB     EAX, ECX                    ; DS:[010598A6]=4F ('O')-41('A')=E  
01302B67   C2 0400        RETN    4  
01302B6A   3A0D A9983001  CMP     CL, [13098A9]               ; 数字 3 的处理  
01302B70   75 0F          JNZ     SHORT 01302B81  
01302B72   0FBE05 A7983001 MOVSX  EAX, BYTE PTR [13098A7]     ; DS:[010598A7]=49 ('I')  
01302B79   0FBED2         MOVSX   EDX, DL  
01302B7C   2BC2           SUB     EAX, EDX                    ; DS:[010598A7]=49 ('I')-41('A')=8  
01302B7E   C2 0400        RETN    4  
01302B81   8A15 AA983001  MOV     DL, [13098AA]               ; 数字 4 到 9 的处理  DS:[013098AA]=34 ('4')  
01302B87   3ACA           CMP     CL, DL  
01302B89   7C 13          JL      SHORT 01302B9E  
01302B8B   3A0D AB983001  CMP     CL, [13098AB]               ; DS:[013098AB]=39 ('9')  
01302B91   7F 0B          JG      SHORT 01302B9E  
01302B93   0FBED2         MOVSX   EDX, DL  
01302B96   0FBEC1         MOVSX   EAX, CL  
01302B99   2BC2           SUB     EAX, EDX                    ; 4~9-4=0~5  
01302B9B   83C0 1A        ADD     EAX, 1A  
01302B9E   C2 0400        RETN    4  
```

### 转轮的一部分
```
01322E90  00 01 00 00 01 01 01 01 01 00 00 01 00 00 01 00  ........  
01322EA0  01 00 00 01 01 01 01 01 01 00 00 00 00 01 01 01  ......  
01322EB0  00 00 00 00 01 00 01 00 01 00 00 01 00 00 01 00  ...........  
01322EC0  00 00 01 01 00 00 00 00 00 00 01 00 01 00 01 01  ..........  
01322ED0  00 00 00 00 00 01 00 01 00 00 00 01 01 01 01 00  ..........  
01322EE0  00 00 01 00 01 00 00 01 00 01 00 00 01 01 00 00  ..........  
```

好了，知道了算法流程，怎么得到可用的注册码呢？

第一要满足第一个条件：每组的第五位计算的值保证在 `0x1`--`0x7C830` 之间，这个比较好办，通过分析发现越靠近 `A` 的字符指针返回的值越小。

越到后面组的第五位值会越大。根据这个原则可以取很多值。

第二个要通过第一个条件做指针通过转轮后变换的值来逆出剩余的注册码：

1. 修改 `01302AF6` 处代码：
   ```asm
   >JNZ     SHORT 01302B07
   ```
   为 `nop`

2. 在代码 `01302AF4` 处下 条件记录 断点，设置记录的参数为 `EBX` 且始终记录这个参数。
   ```asm
   >CMP     EAX, EBX
   ```

3. 在 `01302B01` 处下个中断，运行程序到中断的地方，可以得到一张表，如下：
   ```asm
   >MOV     ESI, [ESP+1C]
   ```

```
01302AF4  COND: 00000012  
01302AF4  COND: 0000001C  
01302AF4  COND: 00000012  
01302AF4  COND: 00000012  
01302AF4  COND: 0000001E  
01302AF4  COND: 00000002  
01302AF4  COND: 00000010  
01302AF4  COND: 00000014  
01302AF4  COND: 00000004  
01302AF4  COND: 00000010  
01302AF4  COND: 00000010  
01302AF4  COND: 00000004  
01302AF4  COND: 0000000C  
01302AF4  COND: 00000002  
01302AF4  COND: 00000010  
01302AF4  COND: 0000001C  
01302AF4  COND: 0000000A  
01302AF4  COND: 0000000A  
01302AF4  COND: 0000000C  
01302AF4  COND: 00000010  
01302B01  断点位于 PltInsta.01302B01  
```

注意这个表中第七位重复了，去掉。



4. 通过上面的表还原注册码，根据这个函数知道：

```asm
i>

01302B30    8A4C24 04       MOV     CL, [ESP+4]                     ; 重点是这几个函数的值将作为指针
01302B34    8A15 A4983001   MOV     DL, [13098A4]                   ; A 到 Z 之间值处理
01302B3A    33C0            XOR     EAX, EAX
01302B3C    3ACA            CMP     CL, DL
01302B3E    7C 13           JL      SHORT 01302B53
01302B40    3A0D A5983001   CMP     CL, [13098A5]
01302B46    7F 0B           JG      SHORT 01302B53
01302B48    0FBED2          MOVSX   EDX, DL
01302B4B    0FBEC1          MOVSX   EAX, CL
01302B4E    2BC2            SUB     EAX, EDX                        ; 值-A
01302B50    C2 0400         RETN    4
```

这个函数的返回值为 0 到 19

```asm
01302B53    3A0D A8983001   CMP     CL, [13098A8]                   ; 数字 2 的处理
01302B59    75 0F           JNZ     SHORT 01302B6A
01302B5B    0FBE05 A6983001 MOVSX   EAX, BYTE PTR [13098A6]         ; DS:[010598A6]=4F ('O')
01302B62    0FBECA          MOVSX   ECX, DL
01302B65    2BC1            SUB     EAX, ECX                        ; DS:[010598A6]=4F ('O')-41('A')=E
01302B67    C2 0400         RETN    4
```

这个函数的返回值是 `E`，如果上面表中值是 `E` 那么注册码就是 `2`

```asm
01302B6A    3A0D A9983001   CMP     CL, [13098A9]                   ; 数字 3 的处理
01302B70    75 0F           JNZ     SHORT 01302B81
01302B72    0FBE05 A7983001 MOVSX   EAX, BYTE PTR [13098A7]         ; DS:[010598A7]=49 ('I')
01302B79    0FBED2          MOVSX   EDX, DL
01302B7C    2BC2            SUB     EAX, EDX                        ; DS:[010598A7]=49 ('I')-41('A')=8
01302B7E    C2 0400         RETN    4
```

这个函数的返回值是 `8`，如果上面表中的值是 `8` 那么注册码就是 `3`

```asm
01302B81    8A15 AA983001   MOV     DL, [13098AA]                   ; 数字 4 到 9 的处理  DS:[013098AA]=34 ('4')
01302B87    3ACA            CMP     CL, DL
01302B89    7C 13           JL      SHORT 01302B9E
01302B8B    3A0D AB983001   CMP     CL, [13098AB]                   ; DS:[013098AB]=39 ('9')
01302B91    7F 0B           JG      SHORT 01302B9E
01302B93    0FBED2          MOVSX   EDX, DL
01302B96    0FBEC1          MOVSX   EAX, CL
01302B99    2BC2            SUB     EAX, EDX                        ; 4~9-4=0~5
01302B9B    83C0 1A         ADD     EAX, 1A
01302B9E    C2 0400         RETN    4
```

这个函数的返回值最小是 `1A`，如果上面表中的值大于等于 `1A` 的注册码就是 `4-9` 是多少就根据表中大于 `1A` 的值-`1A` 所得的值加上 `4`

一组可以使用的注册码：`S6SSK-8CQUN-EQQEK-MCQ6N-KKMQA`

好了，愿自己心情好起来。

`fxyang`
`2006.6.9`

`4dd466d0e1e0cc123afbe504da2462ca`
