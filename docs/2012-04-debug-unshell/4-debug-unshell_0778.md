
# NtkrnlProtector 简明脱壳笔记

> 原文：[`www.pediy.com/kssd/pediy09/pediy09-509.htm`](https://www.pediy.com/kssd/pediy09/pediy09-509.htm)

> 【脱壳对象】: NtkrnlProtector v0.1 加壳的 NotePad from q3 watcher ![](img/7c79845954f5fbf9e981f570ed213a8d.jpg "扮鬼脸")

> 【脱文作者】: cyclotron

> 【附属声明】: 太久没有脱壳了，有点手生，所以趁着周末闲暇捏个软柿子。只是感兴趣，没有其他目的。失误之处敬请诸位大侠赐教。

> 【脱壳笔记】:

首先载入 `NotePad.exe` 至 `LordPE`，修改 `NumOfRVAandSizes` 为 10，以便 `OllyDbg` 正常加载。

忽略所有异常，`OllyDbg` 载入文件，`F9` 运行，来到这里：

代码:

```
004D0000    C3            RETN
004D0001    0000          ADD     BYTE PTR DS:\[EAX\],AL
004D0003    0000          ADD     BYTE PTR DS:\[EAX\],AL
004D0005    0000          ADD     BYTE PTR DS:\[EAX\],AL
004D0007    0000          ADD     BYTE PTR DS:\[EAX\],AL
004D0009    0000          ADD     BYTE PTR DS:\[EAX\],AL
004D000B    0000          ADD     BYTE PTR DS:\[EAX\],AL
004D000D    0000          ADD     BYTE PTR DS:\[EAX\],AL
```

cyclotron 请大家注意这里 `Olly` 由于内存访问断点而停下，事实上我们并没有在这里设置断点，所以属于 `NtkrnlProtector` 的 Anti trick。注意到此时的堆栈已经准备好了 `SEH Handler`，修改这行为 `int 3`（thx 2bpx![](img/c367a558167d303a470cbda3563c57f3.jpg "使眼色")），强行产生异常，并在 `GetProcAddress` 上下断点，`Shift+F9` 运行返回：

代码:

```
01021445    52            PUSH    EDX
01021446    57            PUSH    EDI
01021447    E8 E4000000   CALL    01021530                  ; JMP to kernel32.GetProcAddress
0102144C    60            PUSHAD    ; 返回到这里
0102144D    8BF8          MOV     EDI,EAX
0102144F    B9 04000000   MOV     ECX,4
01021454    B8 60060000   MOV     EAX,660
01021459    C1E8 03       SHR     EAX,3
0102145C    F2:AE         REPNE   SCAS BYTE PTR ES:\[EDI\]
0102145E    85C9          TEST    ECX,ECX
01021460    74 04         JE      SHORT 01021466
01021462    834D EC 01    OR      DWORD PTR SS:\[EBP-14\],1
01021466    61            POPAD
01021467    FF75 F4       PUSH    DWORD PTR SS:\[EBP-C\]
0102146A    837D F0 01    CMP     DWORD PTR SS:\[EBP-10\],1
0102146E    74 05         JE      SHORT 01021475
01021470    E8 96000000   CALL    0102150B
01021475    8B4D FC       MOV     ECX,DWORD PTR SS:\[EBP-4\]
01021478    8901          MOV     DWORD PTR DS:\[ECX\],EAX
0102147A    8345 FC 04    ADD     DWORD PTR SS:\[EBP-4\],4
0102147E    8345 F8 04    ADD     DWORD PTR SS:\[EBP-8\],4
01021482  ^ EB 93         JMP     SHORT 01021417
01021484    83C3 14       ADD     EBX,14
01021487  ^ E9 51FFFFFF  JMP     010213DD
0102148C    8B5D 08       MOV     EBX,DWORD PTR SS:\[EBP+8\]
0102148F    035B 3C       ADD     EBX,DWORD PTR DS:\[EBX+3C\]
01021492    C783 80000000 0>MOV    DWORD PTR DS:\[EBX+80\],0
0102149C    C783 84000000 0>MOV    DWORD PTR DS:\[EBX+84\],0
010214A6    8B45 EC       MOV     EAX,DWORD PTR SS:\[EBP-14\]
010214A9    8BE5          MOV     ESP,EBP
010214AB    5D            POP     EBP
010214AC    C2 0400       RETN    4       ; 移动光标到这里按 F4
```

下断点 `bp VirtualProtect`，`F9` 运行，返回用户代码来到：

代码:

```
00B4B915    59            POP     ECX
00B4B916    C3            RETN

00B4BA72   /E9 00000000  JMP     00B4BA77
00B4BA77   \8B0D 3CA0B500  MOV    ECX,DWORD PTR DS:\[B5A03C\]
00B4BA7D    56            PUSH    ESI
00B4BA7E    57            PUSH    EDI
00B4BA7F    FF7424 0C     PUSH    DWORD PTR SS:\[ESP+C\]
00B4BA83    E8 DBE7FFFF   CALL    00B4A263
00B4BA88    8B35 3CA0B500 MOV    ESI,DWORD PTR DS:\[B5A03C\]
00B4BA8E    8B46 0C       MOV     EAX,DWORD PTR DS:\[ESI+C\]
00B4BA91    8B40 0C       MOV     EAX,DWORD PTR DS:\[EAX+C\]
00B4BA94    034424 0C     ADD     EAX,DWORD PTR SS:\[ESP+C\]
00B4BA98    8BFE          MOV     EDI,ESI
00B4BA9A    A3 48A0B500   MOV     DWORD PTR DS:\[B5A048\],EAX
00B4BA9F    E8 4FE7FFFF   CALL    00B4A1F3
00B4BAA4    57            PUSH    EDI
00B4BAA5    E8 6F650000   CALL    00B52019
00B4BAAA    59            POP     ECX
00B4BAAB    5F            POP     EDI
00B4BAAC    5E            POP     ESI
00B4BAAD    E9 00000000   JMP     00B4BAB2
00B4BAB2    55            PUSH    EBP
00B4BAB3    8BEC          MOV     EBP,ESP
00B4BAB5    53            PUSH    EBX
00B4BAB6    56            PUSH    ESI
00B4BAB7    57            PUSH    EDI
00B4BAB8    8B4C24 F4     MOV     ECX,DWORD PTR SS:\[ESP-C\]
00B4BABC    64:890D 0000000>MOV   DWORD PTR FS:\[0\],ECX
00B4BAC3    5F            POP     EDI
00B4BAC4    5E            POP     ESI
00B4BAC5    5B            POP     EBX
00B4BAC6    C9            LEAVE
00B4BAC7    83C4 3C       ADD     ESP,3C
00B4BACA    A1 48A0B500   MOV     EAX,DWORD PTR DS:\[B5A048\]
00B4BACF    894424 1C     MOV     DWORD PTR SS:\[ESP+1C\],EAX
00B4BAD3    61            POPAD
00B4BAD4    FFE0          JMP     EAX       ; 从这里飞向 OEP
```

注意返回到 `00B4BA72` 以后先不要往下走，为方便起见，我们先把输入表问题一并处理掉。下断点 `bp VirtualAlloc`，然后运行返回至用户代码：

代码:



```
00B4979C    FFD7            CALL    EDI        ; 取消断点，从这里返回
00B4979E    85C0            TEST    EAX,EAX
00B497A0    8945 D4         MOV     DWORD PTR SS:[EBP-2C],EAX
00B497A3    0F84 83010000   JE      00B4992C
00B497A9    6A 40           PUSH    40
00B497AB    68 00100000     PUSH    1000
00B497B0    FF75 D0         PUSH    DWORD PTR SS:[EBP-30]
00B497B3    6A 00           PUSH    0
00B497B5    FFD7            CALL    EDI
00B497B7    85C0            TEST    EAX,EAX
00B497B9    8945 C8         MOV     DWORD PTR SS:[EBP-38],EAX
00B497BC    0F84 6A010000   JE      00B4992C
00B497C2    6A 40           PUSH    40
00B497C4    68 00100000     PUSH    1000
00B497C9    53              PUSH    EBX
00B497CA    6A 00           PUSH    0
00B497CC    FFD7            CALL    EDI
00B497CE    33DB            XOR     EBX,EBX
00B497D0    3BC3            CMP     EAX,EBX
00B497D2    8945 E0         MOV     DWORD PTR SS:[EBP-20],EAX
00B497D5    0F84 51010000   JE      00B4992C
00B497DB    68 7C7DB400     PUSH    0B47D7C                       ; ASCII "kernel32.dll"
00B497E0    FF15 E010B400   CALL    DWORD PTR DS:[B410E0]         ; kernel32.LoadLibraryA
00B497E6    68 0C7FB400     PUSH    0B47F0C                       ; ASCII "GetProcAddress"
00B497EB    50              PUSH    EAX
00B497EC    FF15 DC10B400   CALL    DWORD PTR DS:[B410DC]         ; kernel32.GetProcAddress
00B497F2    8945 EC         MOV     DWORD PTR SS:[EBP-14],EAX
00B497F5    8B46 04         MOV     EAX,DWORD PTR DS:[ESI+4]
00B497F8    8B08            MOV     ECX,DWORD PTR DS:[EAX]
00B497FA    895D E4         MOV     DWORD PTR SS:[EBP-1C],EBX
00B497FD    895D CC         MOV     DWORD PTR SS:[EBP-34],EBX
00B49800    895D D8         MOV     DWORD PTR SS:[EBP-28],EBX
00B49803    8948 04         MOV     DWORD PTR DS:[EAX+4],ECX
00B49806    8B46 04         MOV     EAX,DWORD PTR DS:[ESI+4]
00B49809    8B40 04         MOV     EAX,DWORD PTR DS:[EAX+4]
00B4980C    8B38            MOV     EDI,DWORD PTR DS:[EAX]
00B4980E    037D 08         ADD     EDI,DWORD PTR SS:[EBP+8]
00B49811    8D4D FC         LEA     ECX,DWORD PTR SS:[EBP-4]
00B49814    51              PUSH    ECX
00B49815    E8 81FCFFFF     CALL    00B4949B
00B4981A    83F8 FF         CMP     EAX,-1
00B4981D    0F84 09010000   JE      00B4992C
00B49823    FF75 FC         PUSH    DWORD PTR SS:[EBP-4]
00B49826    FF15 E010B400   CALL    DWORD PTR DS:[B410E0]         ; kernel32.LoadLibraryA
00B4982C    3BC3            CMP     EAX,EBX
00B4982E    8945 F8         MOV     DWORD PTR SS:[EBP-8],EAX
00B49831    0F84 F5000000   JE      00B4992C
00B49837    8B46 04         MOV     EAX,DWORD PTR DS:[ESI+4]
00B4983A    8B40 04         MOV     EAX,DWORD PTR DS:[EAX+4]
00B4983D    8B40 18         MOV     EAX,DWORD PTR DS:[EAX+18]
00B49840    8B08            MOV     ECX,DWORD PTR DS:[EAX]
00B49842    8948 04         MOV     DWORD PTR DS:[EAX+4],ECX
00B49845    8B46 04         MOV     EAX,DWORD PTR DS:[ESI+4]
00B49848    8B40 04         MOV     EAX,DWORD PTR DS:[EAX+4]
00B4984B    8B40 18         MOV     EAX,DWORD PTR DS:[EAX+18]
00B4984E    8B40 04         MOV     EAX,DWORD PTR DS:[EAX+4]
00B49851    8B40 04         MOV     EAX,DWORD PTR DS:[EAX+4]
00B49854    3BC3            CMP     EAX,EBX
00B49856    75 04           JNZ     SHORT 00B4985C
00B49858    33C0            XOR     EAX,EAX
00B4985A    EB 0B           JMP     SHORT 00B49867
00B4985C    25 FFFFFF7F     AND     EAX,7FFFFFFF
00B49861    8945 F0         MOV     DWORD PTR SS:[EBP-10],EAX
00B49864    33C0            XOR     EAX,EAX
00B49866    40              INC     EAX
00B49867    2BC3            SUB     EAX,EBX
00B49869    74 13           JE      SHORT 00B4987E
00B4986B    48              DEC     EAX
00B4986C    75 76           JNZ     SHORT 00B498E4
00B4986E    FF75 F0         PUSH    DWORD PTR SS:[EBP-10]
00B49871    FF75 F8         PUSH    DWORD PTR SS:[EBP-8]
00B49874    FF15 DC10B400   CALL    DWORD PTR DS:[B410DC]         ; kernel32.GetProcAddress
00B4987A    8907            MOV     DWORD PTR DS:[EDI],EAX
00B4987C    EB 66           JMP     SHORT 00B498E4
00B4987E    FF75 FC         PUSH    DWORD PTR SS:[EBP-4]
00B49881    E8 A1FDFFFF     CALL    00B49627
00B49886    85C0            TEST    EAX,EAX
00B49888    8B46 04         MOV     EAX,DWORD PTR DS:[ESI+4]
00B4988B    8B40 04         MOV     EAX,DWORD PTR DS:[EAX+4]
00B4988E    74 28           JE      SHORT 00B498B8      ; Magic jmp，这里改为 jmp
00B49890    8D4D E0         LEA     ECX,DWORD PTR SS:[EBP-20]
00B49893    51              PUSH    ECX
00B49894    8D4D C8         LEA     ECX,DWORD PTR SS:[EBP-38]
00B49897    51              PUSH    ECX
00B49898    8D4D D4         LEA     ECX,DWORD PTR SS:[EBP-2C]
```

跳过上面的 `IAT` 加密，现在我们可以安心 `F9` 直达 `OEP` 了。

代码：

```
01001000  77DA6FC8  ADVAPI32.RegQueryValueExW
01001004  77DA6BF0  ADVAPI32.RegCloseKey
01001008  77DC8F7D  ADVAPI32.RegCreateKeyW
0100100C  77DCD5FD  ADVAPI32.IsTextUnicode
01001010  77DA7883  ADVAPI32.RegQueryValueExA
01001014  77DA761B  ADVAPI32.RegOpenKeyExA
01001018  77DAD7CC  ADVAPI32.RegSetValueExW
0100101C  00000000
01001020  7718D2ED  comctl32.7718D2ED
01001024  00000000
01001028  77F05923  GDI32.77F05923
0100102C  77F23412  GDI32.77F23412
01001030  77F05BB1  GDI32.77F05BB1
01001034  77EF6CA6  ASCII "s("
01001038  77F06AA6  GDI32.77F06AA6
```

拿出 `ImportRec` 完美提取 `API` 地址生成新的输入表，`Fix Dump`。

附件包含了一个 `Dll` 是从 `NtkrnlPacker` 外壳中提取出来用作处理 `License` 的，外壳在执行的过程中直接将该 `Dll` 映射到进程的临时地址空间，`Demo Ver` 中调用了其中的一段代码借 `DialogParamBoxA` 生成一段小小的 `Nag`，有兴趣的朋友不妨研究一下。

`cyclotron`

`2007.5.12` 晚

`4dd466d0e1e0cc123afbe504da2462ca`
