
# 一个 8086 汇编程序的反汇编

> 原文：[`www.pediy.com/kssd/pediy12/130509.html`](https://www.pediy.com/kssd/pediy12/130509.html)

【文章标题】：一个 8086 汇编程序的反汇编

【文章作者】：nanlingcg

【作者邮箱】：1097519717@qq.com

【作者 QQ 号】：1097519717

【软件名称】：8086 变化七彩 HELLO WORLD!

【软件大小】：724 字节

【下载地址】：无

【加壳方式】：无

【保护方式】：无

【编写语言】：8086 汇编

【使用工具】：8086 编译器

【操作平台】：DOS

【软件介绍】：可单变和连续变色

【作者声明】：初学汇编,希望能获得个邀请码.^-^

--------------------------------------------------------------------------------

## 【详细过程】

这个 8086 汇编小程序很好玩,运行后,输出经典的 HELLO WORLD!按 T 可以单色变,按 P 可以连续变色,按 Q 就退出了.

小程序附后,先来反汇编.

IDA 反汇编如下：

```
dseg:0000	;
dseg:0000	;	+-------------------------------------------------------------------------+
dseg:0000	;	|     This file is generated by The Interactive Disassembler (IDA)        |
dseg:0000	;	|     Copyright (c) 2007 by DataRescue sa/nv,         |
dseg:0000	;	| Licensed to: Mach EDV Dienstleistungen, Jan Mach, 1 user, adv, 11/2007  |
dseg:0000	;	+-------------------------------------------------------------------------+
dseg:0000	;
dseg:0000	; Input MD5   : DAE8B672F3ED9BAEAE2398C07F4F160F
dseg:0000
dseg:0000	; File Name   : C:\\Documents and Settings\\Administrator\\桌面\\变幻七.exe
dseg:0000	; Format      : MS-DOS executable (EXE)
dseg:0000	; Base Address: 0h Range: 0h-D4h Loaded length: D4h
dseg:0000	; Entry Point : 3:0
dseg:0000
dseg:0000                 .386
dseg:0000                 .model large
dseg:0000
dseg:0000	; ===========================================================================
dseg:0000
dseg:0000	; Segment type: Pure data
dseg:0000 dseg            segment para stack 'DATA' use16
dseg:0000                 assume cs:dseg
dseg:0000                 db  48h ; H
dseg:0001                 db  45h ; E
dseg:0002                 db  4Ch ; L
dseg:0003                 db  4Ch ; L
dseg:0004                 db  4Fh ; O
dseg:0005                 db  20h
dseg:0006                 db  57h ; W
dseg:0007                 db  4Fh ; O
dseg:0008                 db  52h ; R
dseg:0009                 db  4Ch ; L
dseg:000A                 db  44h ; D
dseg:000B                 db  21h ; !
dseg:000C word_C          dw 0                    ; DATA XREF: start+Fw
dseg:000C                                         ; start+8Dr
dseg:000E word_E          dw 0                    ; DATA XREF: start+18w
dseg:000E                                         ; start+96r
dseg:0010 unk_10          db  1Eh                 ; DATA XREF: start+1Do
dseg:0011                 dd 3B850h
dseg:0015 aOFV            db '',0
dseg:001C                 db  9Ch ; ?
dseg:001D                 db  2Eh ; .
dseg:001E                 db 0FFh
dseg:001F                 db  1Eh
dseg:0020                 db  0Ch
dseg:0021                 db    0
dseg:0022                 db  58h ; X
dseg:0023                 db  1Fh
dseg:0024 word_24         dw 0CFh                 ; DATA XREF: start+Ar
dseg:0024                                         ; start+1Dw ...
dseg:0026 word_26         dw 0                    ; DATA XREF: start+13r
dseg:0026                                         ; start+24w ...
dseg:0028                 align 10h
dseg:0028 dseg            ends
dseg:0028
seg001:0000	; ===========================================================================
seg001:0000
seg001:0000	; Segment type: Pure code
seg001:0000 seg001          segment byte public 'CODE' use16
seg001:0000                 assume cs:seg001
seg001:0000                 assume es:nothing, ss:dseg, ds:nothing, fs:nothing, gs:nothing
seg001:0000
seg001:0000	; =============== S U B R O U T I N E =======================================
seg001:0000
seg001:0000	; Attributes: noreturn
seg001:0000
seg001:0000                 public start
seg001:0000 start           proc near
seg001:0000                 mov     ax, seg dseg
seg001:0003                 mov     ds, ax
seg001:0005                 assume ds:dseg
seg001:0005                 mov     ax, 0
seg001:0008                 mov     es, ax
seg001:000A                 assume es:dseg
seg001:000A                 push    es:word_24
seg001:000F                 pop     word_C
seg001:0013                 push    es:word_26
seg001:0018                 pop     word_E
seg001:001C                 cli
seg001:001D                 mov     es:word_24, offset unk_10
seg001:0024                 mov     es:word_26, ds
seg001:0029                 sti
seg001:002A
seg001:002A loc_5A:                                 ; CODE XREF: start+66j
seg001:002A                                         ; start+7Cj
seg001:002A                 mov     ax, 0B800h
seg001:002D                 mov     es, ax
seg001:002F                 assume es:nothing
seg001:002F                 mov     di, 7C4h
seg001:0032                 mov     si, 0
seg001:0035                 mov     ah, cs:byte_98
seg001:003A                 mov     cx, 0Ch
seg001:003D
seg001:003D loc_6D:                                 ; CODE XREF: start+46j
seg001:003D                 mov     al, [si]
seg001:003F                 mov     es:[di], ax
seg001:0042                 inc     si
seg001:0043                 add     di, 2
seg001:0046                 loop    loc_6D
seg001:0048                 mov     cx, 0
seg001:004B
seg001:004B loc_7B:                                 ; CODE XREF: start:loc_7Bj
seg001:004B                 loop    loc_7B
seg001:004D
seg001:004D loc_7D:                                 ; CODE XREF: start+85j
seg001:004D                 cmp     cs:byte_AE, 94h ; '?
seg001:0053                 jnz     short loc_99
seg001:0055                 inc     cs:byte_98
seg001:005A                 and     cs:byte_98, 0Fh
seg001:0060                 mov     cs:byte_AE, 0
seg001:0066                 jmp     short loc_5A
seg001:0066	; ---------------------------------------------------------------------------
seg001:0068 byte_98         db 0Fh                  ; DATA XREF: start+35r
seg001:0068                                         ; start+55w ...
seg001:0069	; ---------------------------------------------------------------------------
seg001:0069
seg001:0069 loc_99:                                 ; CODE XREF: start+53j
seg001:0069                 cmp     cs:byte_AE, 99h ; '?
seg001:006F                 jnz     short loc_AF
seg001:0071                 inc     cs:byte_98
seg001:0076                 and     cs:byte_98, 0Fh
seg001:007C                 jmp     short loc_5A
seg001:007C	; ---------------------------------------------------------------------------
seg001:007E byte_AE         db 0                    ; DATA XREF: start:loc_7Dr
seg001:007E                                         ; start+60w ...
seg001:007F	; ---------------------------------------------------------------------------
seg001:007F
seg001:007F loc_AF:                                 ; CODE XREF: start+6Fj
seg001:007F                 cmp     cs:byte_AE, 90h ; '?
seg001:0085                 jnz     short loc_7D
seg001:0087                 mov     ax, 0
seg001:008A                 mov     es, ax
seg001:008C                 assume es:dseg
seg001:008C                 cli
seg001:008D                 push    word_C
seg001:0091                 pop     es:word_24
seg001:0096                 push    word_E
seg001:009A                 pop     es:word_26
```



`seg001:009F`                 `sti`

`seg001:00A0`                 `mov`     `ah`, `4Ch`

`seg001:00A2`                 `int`     `21h`             ; DOS - 2+ - QUIT WITH EXIT CODE (EXIT)

`seg001:00A2` `start`           `endp`                    ; `AL` = exit code

`seg001:00A2`

`seg001:00A2` `seg001`          `ends`

`seg001:00A2`

`seg001:00A2`

`seg001:00A2`                 `end` `start`

通过 IDA 的反汇编,手工分析在前一段很不得要领.好在是 8086 程序,就用 DEBUG 再反汇编一次.

如下:

```
C:\DOCUME~1\ADMINI~1>debug e:\abc\abc1\7c.exe
-u
0D13:0000	B8100D        MOV     AX,0D10
0D13:0003 8ED8          MOV     DS,AX
0D13:0005 B80000        MOV     AX,0000
0D13:0008 8EC0          MOV     ES,AX
0D13:000A 26            ES:
0D13:000B FF362400      PUSH    [0024]
0D13:000F 8F060C00      POP     [000C]
0D13:0013 26            ES:
0D13:0014 FF362600      PUSH    [0026]
0D13:0018 8F060E00      POP     [000E]
0D13:001C FA            CLI
0D13:001D 26            ES:
0D13:001E C70624001000  MOV     WORD PTR [0024],0010
0D13:0024 26            ES:
0D13:0025 8C1E2600      MOV     [0026],DS
0D13:0029 FB            STI
0D13:002A B800B8        MOV     AX,B800
0D13:002D 8EC0          MOV     ES,AX
0D13:002F BFC407        MOV     DI,07C4
0D13:0032 BE0000        MOV     SI,0000
0D13:0035 2E            CS:
0D13:0036 8A266800      MOV     AH,[0068]
0D13:003A B90C00        MOV     CX,000C
0D13:003D 8A04          MOV     AL,[SI]
0D13:003F 26            ES:
0D13:0040 8905          MOV     [DI],AX
0D13:0042 46            INC     SI
0D13:0043 83C702        ADD     DI,+02
0D13:0046 E2F5          LOOP    003D
0D13:0048 B90000        MOV     CX,0000
0D13:004B E2FE          LOOP    004B
0D13:004D 2E            CS:
0D13:004E 803E7E0094    CMP     BYTE PTR [007E],94
0D13:0053 7514          JNZ     0069
0D13:0055 2E            CS:
0D13:0056 FE066800      INC     BYTE PTR [0068]
0D13:005A 2E            CS:
0D13:005B 802668000F    AND     BYTE PTR [0068],0F
0D13:0060 2E            CS:
0D13:0061 C6067E0000    MOV     BYTE PTR [007E],00
0D13:0066 EBC2          JMP     002A
0D13:0068 0F            DB      0F
0D13:0069 2E            CS:
0D13:006A 803E7E0099    CMP     BYTE PTR [007E],99
0D13:006F 750E          JNZ     007F
0D13:0071 2E            CS:
0D13:0072 FE066800      INC     BYTE PTR [0068]
0D13:0076 2E            CS:
0D13:0077 802668000F    AND     BYTE PTR [0068],0F
0D13:007C EBAC          JMP     002A
0D13:007E 002E803E      ADD     [3E80],CH
0D13:0082 7E00          JLE     0084
0D13:0084 90            NOP
0D13:0085 75C6          JNZ     004D
0D13:0087 B80000        MOV     AX,0000
0D13:008A 8EC0          MOV     ES,AX
0D13:008C FA            CLI
0D13:008D FF360C00      PUSH    [000C]
0D13:0091 26            ES:
0D13:0092 8F062400      POP     [0024]
0D13:0096 FF360E00      PUSH    [000E]
0D13:009A 26            ES:
0D13:009B 8F062600      POP     [0026]
0D13:009F FB            STI
0D13:00A0 B44C          MOV     AH,4C
0D13:00A2 CD21          INT     21
```

到这里看到 `INT 21H`(4CH)了.该结束了.(IDA 也就解析到这).

两相比照,在代码段 2A 到 7C 段基本一致,所不同的是 2A 以前和 7C 以后,互相比对:首先看 2A 以前,DEBUG 更合理,

```
0D13:0000 B8100D        MOV     AX,0D10
0D13:0003 8ED8          MOV     DS,AX
0D13:0005 B80000        MOV     AX,0000
0D13:0008 8EC0          MOV     ES,AX              <--确定 DS 和 ES
0D13:000A 26            ES:
0D13:000B FF362400      PUSH    [0024]
0D13:000F 8F060C00      POP     [000C]
0D13:0013 26            ES:
0D13:0014 FF362600      PUSH    [0026]
0D13:0018 8F060E00      POP     [000E]  
0D13:001C FA            CLI                        <--保存 0:24 和 0:26 进 DS:0C 和 DS:0E
0D13:001D 26            ES:
0D13:001E C70624001000  MOV     WORD PTR [0024],0010
0D13:0024 26            ES:
0D13:0025 8C1E2600      MOV     [0026],DS          <--10 和 DS 进 0:24 和 0:26
0D13:0029 FB            STI
```

很显然,这是在修改 `INT 9`.

再看 2A-7C 一节:

```
0D13:0029 FB            STI
0D13:002A B800B8        MOV     AX,B800           <--修改 ES 指向文本模式显存映射.
0D13:002D 8EC0          MOV     ES,AX
0D13:002F BFC407        MOV     DI,07C4
0D13:0032 BE0000        MOV     SI,0000        
0D13:0035 2E            CS:
0D13:0036 8A266800      MOV     AH,[0068]        <--等下说
0D13:003A B90C00        MOV     CX,000C
0D13:003D 8A04          MOV     AL,[SI]          <--DS:[SI]=DS:[0]进 AL,等下反汇编 DS 段,或者参看 IDA 的反汇编,其实指向存在 DS 段的"HELLO WORLD!"ASCII
0D13:003F 26            ES:
0D13:0040 8905          MOV     [DI],AX
0D13:0042 46            INC     SI               <--SI 自加 1,
0D13:0043 83C702        ADD     DI,+02           <--DI 自加 2,指向显存映射的.
0D13:0046 E2F5          LOOP    003D             <--循环执行 3D 到这共 CX(0C)次,其实就是单字符向屏幕输出 12 个字符.正是 HELLO WORLD!,抓了图的,就略了.
0D13:0048 B90000        MOV     CX,0000
0D13:004B E2FE          LOOP    004B             <--延时一下
0D13:004D 2E            CS:
0D13:004E 803E7E0094    CMP     BYTE PTR [007E],94
0D13:0053 7514          JNZ     0069
0D13:0055 2E            CS:
0D13:0056 FE066800      INC     BYTE PTR [0068]
0D13:005A 2E            CS:
0D13:005B 802668000F    AND     BYTE PTR [0068],0F
0D13:0060 2E            CS:
0D13:0061 C6067E0000    MOV     BYTE PTR [007E],00
0D13:0066 EBC2          JMP     002A
0D13:0068 0F            DB      0F
0D13:0069 2E            CS:
0D13:006A 803E7E0099    CMP     BYTE PTR [007E],99
0D13:006F 750E          JNZ     007F
0D13:0071 2E            CS:
0D13:0072 FE066800      INC     BYTE PTR [0068]
0D13:0076 2E            CS:
0D13:0077 802668000F    AND     BYTE PTR [0068],0F
0D13:007C EBAC          JMP     002A            <--这段是一个多分支检测,用 BYTE PTR CS:[007E]来分别和某个数比较以修改 CS:[68],并确定程序走向.(从简).
```

再看下面从 7E 开始,IDA 和 DEBUG 解析出了不同结果.经比较分析,IDA 的解析更可靠,IDA 把 7E 解析为一个字节的数据而不是代码,参考 IDA 用 DEBUG 反汇编 7F 以后一段.(7E 为一个数据了).

```
-u 7F
0D13:007F 2E            CS:
0D13:0080 803E7E0090    CMP     BYTE PTR [007E],90   <--依然是分支比较.
0D13:0085 75C6          JNZ     004D              
0D13:0087 B80000        MOV     AX,0000              <--下面修改 ES 及和代码段开头相对应的用 DS:0C 和 DS:0E 修改 0:24 和 0:26
0D13:008A 8EC0          MOV     ES,AX
0D13:008C FA            CLI
0D13:008D FF360C00      PUSH    [000C]
0D13:0091 26            ES:
0D13:0092 8F062400      POP     [0024]
0D13:0096 FF360E00      PUSH    [000E]
0D13:009A 26            ES:
0D13:009B 8F062600      POP     [0026]
0D13:009F FB            STI                          <--修改是回复 INT 9
0D13:00A0 B44C          MOV     AH,
0D13:00A2 CD21          INT     21                   <--程序退出,返回到系统.
```

到这里是不是该借书了呢?不?还有疑问,`CS[68]`和 `CS:[7E]`到底是什么?`INT 9` 是修改成什么了?

```
0D13:001E C70624001000  MOV     WORD PTR [0024],0010
0D13:0024 26            ES:
0D13:0025 8C1E2600      MOV     [0026],DS
0D13:0029 FB            STI
```

这节已告诉我们 `INT 9` 的向量被修改为 `DS:0010` 了.那么我门再反汇编 `DS:0010` 一段.

```
-u ds:0010
0D10:0010 1E            PUSH    DS
0D10:0011 50            PUSH    AX
0D10:0012 B8130D        MOV     AX,0D13
0D10:0015 8ED8          MOV     DS,AX
0D10:0017 E460          IN      AL,60
0D10:0019 A27E00        MOV     [007E],AL    <--这,开始修改 7E 处了.
```



`0D10:001C` `9C` `PUSHF`

`0D10:001D` `2E` `CS:`

`0D10:001E` `FF1E0C00` `CALL` `FAR` `[000C]`

`0D10:0022` `58` `POP` `AX`

`0D10:0023` `1F` `POP` `DS`

`0D10:0024` `CF` `IRET` <--到这中断程序返回了,后面的是没用的.

`0D10:0025` `0000` `ADD` `[BX+SI],AL`

`0D10:0027` `0000` `ADD` `[BX+SI],AL`

`0D10:0029` `0000` `ADD` `[BX+SI],AL`

`0D10:002B` `0000` `ADD` `[BX+SI],AL`

`0D10:002D` `0000` `ADD` `[BX+SI],AL`

其实这个新 `INT 9` 只有这是有意义的

`0D10:0017` `E460` `IN` `AL,60`

`0D10:0019` `A27E00` `MOV` `[007E],AL` <--这,开始修改 `7E` 处了.

读 `60` 端口,取按键扫描码进 `CS:[7E]`.

`CS:[7E]`的用处终于知道了,是用来存键盘扫描码的.那么毫无疑问上面把 `7E` 当数据处理是正确的.`CS:68` 被处理成数据经分析也无不妥.

那么.我们就大概复原下这个程序的大概面貌.

`DS:`

```
0D10:0000  DB   "HELLO WORLD!"           <--用来向屏幕输出的.
0D10:0000  DB 00,00,00,00                <--用来保存 0;24 和 0:26(即 INT 9 向量)的.
0D10:0010 1E            PUSH    DS       <--用来修改为 INT 9 的中断程序代码
0D10:0011 50            PUSH    AX
0D10:0012 B8130D        MOV     AX,0D13
0D10:0015 8ED8          MOV     DS,AX
0D10:0017 E460          IN      AL,60
0D10:0019 A27E00        MOV     [007E],AL     
0D10:001C 9C            PUSHF
0D10:001D 2E            CS:
0D10:001E FF1E0C00      CALL    FAR [000C]
0D10:0022 58            POP     AX
0D10:0023 1F            POP     DS
0D10:0024 CF            IRET
```

`CS:`

```
0D13:0000 B8100D        MOV     AX,0D10
0D13:0003 8ED8          MOV     DS,AX          <--确定 DS
0D13:0005 B80000        MOV     AX,0000
0D13:0008 8EC0          MOV     ES,AX          <--保存和修改 INT 9 向量
0D13:000A 26            ES:
0D13:000B FF362400      PUSH    [0024]
0D13:000F 8F060C00      POP     [000C]
0D13:0013 26            ES:
0D13:0014 FF362600      PUSH    [0026]
0D13:0018 8F060E00      POP     [000E]
0D13:001C FA            CLI
0D13:001D 26            ES:
0D13:001E C70624001000  MOV     WORD PTR [0024],0010
0D13:0024 26            ES:
0D13:0025 8C1E2600      MOV     [0026],DS
0D13:0029 FB            STI
0D13:002A B800B8        MOV     AX,B800          <--ES 和 DI 指向屏幕显存地址
0D13:002D 8EC0          MOV     ES,AX
0D13:002F BFC407        MOV     DI,07C4
0D13:0032 BE0000        MOV     SI,0000
0D13:0035 2E            CS:
0D13:0036 8A266800      MOV     AH,[0068]       <--6E 为颜色码
0D13:003A B90C00        MOV     CX,000C
0D13:003D 8A04          MOV     AL,[SI]
0D13:003F 26            ES:
0D13:0040 8905          MOV     [DI],AX
0D13:0042 46            INC     SI
0D13:0043 83C702        ADD     DI,+02
0D13:0046 E2F5          LOOP    003D
0D13:0048 B90000        MOV     CX,0000
0D13:004B E2FE          LOOP    004B            <--循环 12 次一字节输出 HELLO WORLD!
0D13:004D 2E            CS:
0D13:004E 803E7E0094    CMP     BYTE PTR [007E],94   <--键盘扫描码比较,以确定是否修改 6E(颜色码)或跳走.
0D13:0053 7514          JNZ     0069
0D13:0055 2E            CS:
0D13:0056 FE066800      INC     BYTE PTR [0068]
0D13:005A 2E            CS:
0D13:005B 802668000F    AND     BYTE PTR [0068],0F
0D13:0060 2E            CS:
0D13:0061 C6067E0000    MOV     BYTE PTR [007E],00
0D13:0066 EBC2          JMP     002A               <--键盘扫描码为 94,修改 6E 后跳到 2A 处执行(改变颜色,再向屏幕输出 HELLO WORLD!)
0D13:0068 0F            DB      0F
0D13:0069 2E            CS:
0D13:006A 803E7E0099    CMP     BYTE PTR [007E],99   <--键盘扫描码比较,以确定是否修改 6E(颜色码)或跳走.
0D13:006F 750E          JNZ     007F
0D13:0071 2E            CS:
0D13:0072 FE066800      INC     BYTE PTR [0068]
0D13:0076 2E            CS:
0D13:0077 802668000F    AND     BYTE PTR [0068],0F
0D13:007C EBAC          JMP     002A                  <--键盘扫描码为 99,修改 6E 后跳到 2A 处执行(改变颜色,再向屏幕输出 HELLO WORLD!)
0D13:007E 00            DB      00
0D13:007F 2E            CS:
0D13:0080 803E7E0090    CMP     BYTE PTR [007E],90  <--键盘扫描码比较,不是就继续跳到此为止 D 处继续比较键盘扫描码,是则就执行后面的恢复 INT 9 和退出整个程序
0D13:0085 75C6          JNZ     004D
0D13:0087 B80000        MOV     AX,0000             <--恢复 INT 9
0D13:008A 8EC0          MOV     ES,AX
0D13:008C FA            CLI
0D13:008D FF360C00      PUSH    [000C]
0D13:0091 26            ES:
0D13:0092 8F062400      POP     [0024]
0D13:0096 FF360E00      PUSH    [000E]
0D13:009A 26            ES:
0D13:009B 8F062600      POP     [0026]
0D13:009F FB            STI
0D13:00A0 B44C          MOV     AH,                  ,--退出整个程序,返回系统.
0D13:00A2 CD21          INT     21
```

--------------------------------------------------------------------------------

【经验总结】

整个程序总体来看很简单,但,在程序中于代码段中加入了数据,数据段中加入了代码,使反汇编难度加大.单凭 `IDA` 或 `DEBUG` 完全反汇编有一定难度.

鉴于初学汇编,谨以此文贴出,借以希望获得"看雪"邀请码一个,以便更好学习.

--------------------------------------------------------------------------------

2011 年 03 月 08 日 CG 11:02:32

上传的附件

| ![](img/67af4f1ef9c98996f43e89e749c7dcf5.jpg) || 变幻七彩 HELLO WORLD！.rar |
