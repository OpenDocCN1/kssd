
# UPX3.03 外壳段处理过程全程分析

> 原文：[`www.pediy.com/kssd/pediy12/135325.html`](https://www.pediy.com/kssd/pediy12/135325.html)

分析的不是很好，有些东西没表达清楚，大家自己分析一哈，这个程序是我自己写的个进程管理，还没写完。。。。

`UPX3.03` 加壳，加壳时选了加密处理，高级导入保护，模拟系统标准功能，但是再外壳段好像并没见到这几个功能。

先来个 `CALL` 公式，等会要用到。具体见附件，自己分析哈

代码:



## CALL 地址计算公式

`CALL` 目标地址计算公式：`CALL` 下条指令开始地址 + `E8` 后面的绝对偏移 = 目标地址

例如：
- `00401B2C` 是当前地址，`5` 是 `CALL` 机器码的长度；`E8 CF 00 00 00` 是机器码。
- 计算：`00401B2C + 5 + CF = 00401C00`
- `E8 CF 00 00 00` 表示 `CALL 00401C00`

### 示例二

- 指令：`00401B3B |. E8 C0270000 CALL decode.00404300`
- 计算：`00401B3B + 5 + 27C0 = 00404300`
- 目标地址：`CALL 00404300`

### 示例三

- 指令：`00401B5F |. E8 3C1B0000 CALL decode.004036A0`
- 计算：`00401B5F + 5 + 1B3C = 004036A0`
- `00401B5F + 5` 是下条指令开始的地址，向后移动 `1B3C` 就是目标指令的开始地址
- 目标地址：`CALL 004036A0`

## 汇编代码分析

```assembly
00417550  >  $  60               PUSHAD
00417551   .  BE 15604100        MOV ESI,进程管理.00416015
00417556   .  8DBE EBAFFEFF      LEA EDI,DWORD PTR DS:[ESI+FFFEAFEB]      ;  得到代码段的开始地址放在 EDI 里面
0041755C   .  57               PUSH EDI
0041755D   .  EB 0B             JMP SHORT 进程管理.0041756A
0041755F     90               NOP
00417560   >  8A06             MOV AL,BYTE PTR DS:[ESI]                   ;  现在分析得知 00416015 这个地址是我们程序代码段的开始地址，被映射到这里了，取出映射的代码的一字节
00417562   .  46               INC ESI                                    ;  指向我们代码的下个字节
00417563   .  8807             MOV BYTE PTR DS:[EDI],AL                   ;  EDI 现在是从 00401000 开始的，把映射的代码放入原来我们代码的地址，那么现在知道在开始代码还原了
00417565   .  47               INC EDI                                    ;  指向我们代码的下个地址，以便存放下个字节
00417566   >  01DB             ADD EBX,EBX                                ;  EBX+EBX，当 EBX 不等于 0 的时候就跳，那么下面如果进位加法为 0，那么取出下个地址到 EBX
00417568   .  75 07             JNZ SHORT 进程管理.00417571
0041756A   >  8B1E             MOV EBX,DWORD PTR DS:[ESI]                 ;  00416015 这个地址放到 EBX 里面，
0041756C   .  83EE FC          SUB ESI,-4                                 ;  00416015 加 4
0041756F   .  11DB             ADC EBX,EBX                                ;  进位加法器
00417571   >^ 72 ED            JB SHORT 进程管理.00417560                 ;  向上跳，EBX 作为跳不跳的标志，循环处理代码，一个大循环
00417573   .  B8 01000000      MOV EAX,1                                  ;  EAX=1
00417578   >  01DB             ADD EBX,EBX                                ;  EBX 再加 EBX，EBX 在这个代码里面作为跳转标志
0041757A   .  75 07             JNZ SHORT 进程管理.00417583
0041757C   .  8B1E             MOV EBX,DWORD PTR DS:[ESI]                 ;  ESI 指向的地址到 EBX
0041757E   .  83EE FC          SUB ESI,-4                                 ;  ESI 加上 4
00417581   .  11DB             ADC EBX,EBX                                ;  进位加法
00417583   >  11C0             ADC EAX,EAX                                ;  进位加法
00417585   .  01DB             ADD EBX,EBX                                ;  EBX+EBX
00417587   .^ 73 EF            JNB SHORT 进程管理.00417578
00417589   .  75 09             JNZ SHORT 进程管理.00417594                ;  跳转到下面
0041758B   .  8B1E             MOV EBX,DWORD PTR DS:[ESI]
0041758D   .  83EE FC          SUB ESI,-4
00417590   .  11DB             ADC EBX,EBX
00417592   .^ 73 E4            JNB SHORT 进程管理.00417578
00417594   >  31C9             XOR ECX,ECX                                ;  ECX 清零
00417596   .  83E8 03          SUB EAX,3                                  ;  EAX 减去 3
00417599   .  72 0D             JB SHORT 进程管理.004175A8
0041759B   .  C1E0 08          SHL EAX,8                                  ;  EAX 逻辑左移 8 位
0041759E   .  8A06             MOV AL,BYTE PTR DS:[ESI]                   ;  ESI 指向的地址取出一字节到 AL
004175A0   .  46               INC ESI                                    ;  指向我们代码的下个字节
004175A1   .  83F0 FF          XOR EAX,FFFFFFFF                           ;  EAX 和 FFFFFFFF 异或
004175A4   .  74 74             JE SHORT 进程管理.0041761A
004175A6   .  89C5             MOV EBP,EAX                                ;  EAX 放到 EBP 里面
004175A8   >  01DB             ADD EBX,EBX                                ;  EBX 加上 EBX
004175AA   .  75 07             JNZ SHORT 进程管理.004175B3                ;  判断跳转，这次是跳
004175AC   .  8B1E             MOV EBX,DWORD PTR DS:[ESI]
004175AE   .  83EE FC          SUB ESI,-4
004175B1   .  11DB             ADC EBX,EBX
004175B3   >  11C9             ADC ECX,ECX                                ;  进位加法
004175B5   .  01DB             ADD EBX,EBX                                ;  EBX 乘以 2
004175B7   .  75 07             JNZ SHORT 进程管理.004175C0
004175B9   .  8B1E             MOV EBX,DWORD PTR DS:[ESI]
004175BB   .  83EE FC          SUB ESI,-4
004175BE   .  11DB             ADC EBX,EBX
004175C0   >  11C9             ADC ECX,ECX                                ;  进位加法
004175C2   .  75 20             JNZ SHORT 进程管理.004175E4
004175C4   .  41               INC ECX
004175C5   >  01DB             ADD EBX,EBX
004175C7   .  75 07             JNZ SHORT 进程管理.004175D0
004175C9   .  8B1E             MOV EBX,DWORD PTR DS:[ESI]
004175CB   .  83EE FC          SUB ESI,-4
004175CE   .  11DB             ADC EBX,EBX
004175D0   >  11C9             ADC ECX,ECX
004175D2   .  01DB             ADD EBX,EBX
004175D4   .^ 73 EF            JNB SHORT 进程管理.004175C5
004175D6   .  75 09             JNZ SHORT 进程管理.004175E1
004175D8   .  8B1E             MOV EBX,DWORD PTR DS:[ESI]
004175DA   .  83EE FC          SUB ESI,-4
004175DD   .  11DB             ADC EBX,EBX
004175DF   .^ 73 E4            JNB SHORT 进程管理.004175C5
004175E1   >  83C1 02          ADD ECX,2
004175E4   >  81FD 00F3FFFF    CMP EBP,-0D00                              ;  迷惑指令
004175EA   .  83D1 01          ADC ECX,1                                  ;  进位加法
004175ED   .  8D142F          LEA EDX,DWORD PTR DS:[EDI+EBP]             ;  装入 EDI+EBP 的地址到 EDX，现在他们指向我们原来的代码段
004175F0   .  83FD FC          CMP EBP,-4                                 ;  判断跳转标志，EBP 低于等于-4 就跳，也就是 EBP 至少要是-4 才跳
004175F3   .  76 0F             JBE SHORT 进程管理.00417604
004175F5   >  8A02             MOV AL,BYTE PTR DS:[EDX]                   ;  EDX 指向我们的代码段取出一字节，到 AL 里面
004175F7   .  42               INC EDX                                    ;  指向下个字节的地址
004175F8   .  8807             MOV BYTE PTR DS:[EDI],AL                   ;  放入现在我们处理的代码，也就是刚才上面没处理的我们代码段
004175FA   .  47               INC EDI                                    ;  指向下个地址，用于接收下个字节码
004175FB   .  49               DEC ECX                                    ;  计数器，ECX，
004175FC   .^ 75 F7            JNZ SHORT 进程管理.004175F5                ;  ECX 作为计数器的跳转，当 ECX 不为 0 就继续处理
004175FE   .^ E9 63FFFFFF      JMP 进程管理.00417566                      ;  向上跳转，跳到第一个 ADD EBX,EBX 那里
00417603     90               NOP
00417604   >  8B02             MOV EAX,DWORD PTR DS:[EDX]                 ;  这里处理程序中的数据和 CALL，就是输入表
00417606   .  83C2 04          ADD EDX,4                                  ;  指向下个地址
00417609   .  8907             MOV DWORD PTR DS:[EDI],EAX                 ;  取出的代码放到 EDI 里面，EDI 一直指向我们没处理完的代码的地址
0041760B   .  83C7 04          ADD EDI,4                                  ;  EDI 加上 4，代码接收段的地址+4
0041760E   .  83E9 04          SUB ECX,4                                  ;  ECX 减去 4
00417611   .^ 77 F1            JA SHORT 进程管理.00417604
00417613   .  01CF             ADD EDI,ECX                                ;  EDI 加上 ECX，指向前刚才接收代码的地址的前字节，也就是最后的一字节
00417615   .^ E9 4CFFFFFF      JMP 进程管理.00417566                      ;  又跳到刚才的第一个 ADD EBX,EBX
0041761A   >  5E               POP ESI
0041761B   .  89F7             MOV EDI,ESI
0041761D   .  B9 4B000000      MOV ECX,4B
00417622   >  8A07             MOV AL,BYTE PTR DS:[EDI]                   ;  指向我们原来代码段的代码，取出到 AL 里面，下面开始处理 CALL,这种 CALL 就是调用一个地址那种，下面计算它的绝对偏移
00417624   .  47               INC EDI                                    ;  指向下个字节
00417625   .  2C E8            SUB AL,0E8                                 ;  AL 减去 E8，判断是否为 E8，也就是 CALL，这里处理 CALL
00417627   >  3C 01            CMP AL,1                                   ;  比较 AL 是否高于 1
00417629   .^ 77 F7            JA SHORT 进程管理.00417622                 ;  这里循环，一直到碰到下个 CALL 的字节，CALL 就是 E8 嘛
0041762B   .  803F 00          CMP BYTE PTR DS:[EDI],0                    ;  判断 EDI 是否等于 0，也就是处理 CALL，下面就计算 CALL 的绝对地址，就是 E8 后面那个字节码，不是处理函数那种 CALL，大家看下我写的那个 CALL 公式
0041762E   .^ 75 F2            JNZ SHORT 进程管理.00417622
00417630   .  8B07             MOV EAX,DWORD PTR DS:[EDI]                 ;  取出里面的地址，里面的地址是定位 CALL 的绝对地址要用到的
00417632   .  8A5F 04          MOV BL,BYTE PTR DS:[EDI+4]                 ;  得到下条地址的开始字节到 AL 里面，CALL 绝对地址就是下条指令开始+刚才上面取出的那个数字
00417635   .  66:C1E8 08       SHR AX,8                                   ;  AX 逻辑右移 8 位
00417639   .  C1C0 10          ROL EAX,10                                 ;  EAX 算术左移 8 位
0041763C   .  86C4             XCHG AH,AL                                 ;  交换他们的内容
0041763E   .  29F8             SUB EAX,EDI                                  ;  EAX 减去这个地址 EDI
00417640   .  80EB E8          SUB BL,0E8                                 ;  再减去 E8
00417643   .  01F0             ADD EAX,ESI                                  ;  EAX 加上 ESI,ESI 这里是代码段开始的地址
00417645   .  8907             MOV DWORD PTR DS:[EDI],EAX                 ;  这里处理 CALL 的地址，算出 CALL 的偏移到 EDI 里面，当 CALL 得到偏移之后自动计算地址
00417647   .  83C7 05          ADD EDI,5                                  ;  EDI 加上 5，表示 CALL 已经被处理，指向 CALL 的后面
0041764A   .  88D8             MOV AL,BL                                    ;  BL 的数据放到 AL 里面
0041764C   .^ E2 D9            LOOPD SHORT 进程管理.00417627              ;  向上跳转，处理下个 CALL ,ECX 作为计数器，就是后面这种 CALL 的数量就比如 CALL 00401000,这种 CALL
0041764E   .  8DBE 00500100    LEA EDI,DWORD PTR DS:[ESI+15000]           ;  CALL 00401000 这种的绝对地址计算处理完毕，现在代码段开始地址+15000 的地址到 EDI 里面
00417654   >  8B07             MOV EAX,DWORD PTR DS:[EDI]                 ;  现在 EDI 指向我们的代码的输入表
00417656   .  09C0             OR EAX,EAX                                   ;  判断 EAX 是否为 0
00417658   .  74 45             JE SHORT 进程管理.0041769F
0041765A   .  8B5F 04          MOV EBX,DWORD PTR DS:[EDI+4]               ;  取得这个地址的数据到 EBX 里面
0041765D   .  8D8430 C47802>   LEA EAX,DWORD PTR DS:[EAX+ESI+278C4]      ;  取得外壳段的 KERNEL32.DLL 的地址到 EAX 里面
00417664   .  01F3             ADD EBX,ESI                                  ;  我们代码段的开始地址加上刚才取出的那个数据
00417666   .  50               PUSH EAX                                     ;  KERNEL32.DLL 的地址
00417667   .  83C7 08          ADD EDI,8                                  ;  EDI 加上 8
0041766A   .  FF96 28790200    CALL DWORD PTR DS:[ESI+27928]              ;  装载 KERNEL32.DLL
00417670   .  95               XCHG EAX,EBP                               ;  EAX 的值和 EBX 交换，现在 EAX 是指向 KERNEL32.DLL 的基址
00417671   >  8A07             MOV AL,BYTE PTR DS:[EDI]                   ;  取得现在的 EDI 的地址指向的数据到 AL 里面，每个函数名字前面有个 01，以便判断获得函数的名字
00417673   .  47               INC EDI                                    ;  指向下个字节的地址，下个地址就是我们的函数名字了
00417674   .  08C0             OR AL,AL                                   ;  判断 AL 是否为 0
00417676   .^ 74 DC            JE SHORT 进程管理.00417654
00417678   .  89F9             MOV ECX,EDI                                ;  我们的函数的名字放到 ECX 里面
0041767A   .  79 07             JNS SHORT 进程管理.00417683                ;  SF 符号为正就跳
0041767C   .  0FB707           MOVZX EAX,WORD PTR DS:[EDI]
0041767F   .  47               INC EDI
00417680   .  50               PUSH EAX
00417681   .  47               INC EDI
00417682     B9               DB B9
00417683   .  57               PUSH EDI                                     ;  我们的函数名字
00417684   .  48               DEC EAX                                    ;  EAX 减去 1
00417685   .  F2:AE            REPNE SCAS BYTE PTR ES:[EDI]
00417687   .  55               PUSH EBP                                     ;  KERNEL32 的基址
00417688   .  FF96 2C790200    CALL DWORD PTR DS:[ESI+2792C]              ;  外壳段的 GetProcAddress
0041768E   .  09C0             OR EAX,EAX                                   ;  得到函数的地址
00417690   .  74 07             JE SHORT 进程管理.00417699
00417692   .  8903             MOV DWORD PTR DS:[EBX],EAX                 ;  放到我们的外壳段中的函数调用的地址，也就是处理输入表
00417694   .  83C3 04          ADD EBX,4                                  ;  指向下个输入表中的地址
00417697   .^ EB D8            JMP SHORT 进程管理.00417671
00417699   >  FF96 3C790200    CALL DWORD PTR DS:[ESI+2793C]
0041769F   >  8BAE 30790200    MOV EBP,DWORD PTR DS:[ESI+27930]           ;  VirtualProtect 的地址放到 EBP 里面
004176A5   .  8DBE 00F0FFFF    LEA EDI,DWORD PTR DS:[ESI-1000]            ;  指向 PE 头的地址，也就是映像基址
004176AB   .  BB 00100000      MOV EBX,1000                               ;  EBX 等于 1000
004176B0   .  50               PUSH EAX
004176B1   .  54               PUSH ESP
004176B2   .  6A 04            PUSH 4
004176B4   .  53               PUSH EBX
004176B5   .  57               PUSH EDI
004176B6   .  FFD5             CALL EBP                                     ;  改变属性
004176B8   .  8D87 9F010000    LEA EAX,DWORD PTR DS:[EDI+19F]             ;  现在 EAX 指向 PE 头中区段的偏移开始
004176BE   .  8020 7F          AND BYTE PTR DS:[EAX],7F                   ;  改写区段名字
004176C1   .  8060 28 7F       AND BYTE PTR DS:[EAX+28],7F                ;  改写区块属性第一个区块的属性
004176C5   .  58               POP EAX
004176C6   .  50               PUSH EAX
004176C7   .  54               PUSH ESP
004176C8   .  50               PUSH EAX
004176C9   .  53               PUSH EBX
004176CA   .  57               PUSH EDI
004176CB   .  FFD5             CALL EBP
004176CD   .  58               POP EAX
```



后面就完了，还几行代码

上传的附件

| ![](img/620251b33676a8e4b98ca0ece915ee8d.jpg) || `UPX3.03 分析笔记.zip`  
|
