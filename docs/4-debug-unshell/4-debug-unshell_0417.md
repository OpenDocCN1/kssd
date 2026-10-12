
# ASTONSHELL 1.90 脱壳 [ASPR 1.24 RC4]

> 原文：[`www.pediy.com/kssd/pediy08/pediy8-549.htm`](https://www.pediy.com/kssd/pediy08/pediy8-549.htm)

`astonshell 1.90`

`www.astonshell.com`

这是个很好的 `windows` 外壳替换工具，可惜是英文版的，我以前脱了 `1.90` 的壳，也做了个不完全的汉化（很懒），现在看到有同志要做汉化，所以把 `1.90` 的脱壳纪要发出来，算不上教程，呵呵，`1.91` 的我也脱不了，可能用了 `SDK`，期待高人！

`a-msater.exe` `v1.9` `ASPR 1.24 RC4` 壳

再跟踪一遍，整理：

\=======

`OD`-`ollscript`，专门找 `ASPR STOLEN BYTES` 的脚本跟，或者手动跟，注意去掉内存访问异常

```
0042CA5B    0000             ADD BYTE PTR DS:[EAX],AL
0042CA5D    0000             ADD BYTE PTR DS:[EAX],AL
0042CA5F    E8 AC46FDFF      CALL A-MAST~2.00401110 >>>
0042CA64    E8 EB18FEFF      CALL A-MAST~2.0040E354 ;返回这里,假 OEP
0042CA69    E8 9E45FDFF      CALL A-MAST~2.0040100C
```

\========

```
00401110    31C0             XOR EAX,EAX <<<   //od 可以跟到这里,
///实际这是 aspr 改变 oep 附近的一个 CALL kernel32.GetModuleHandleA ,所以这里就应该 DUMP 了.
00401112    50               PUSH EAX
00401113    E8 E8FFFFFF      CALL A-MAST~2.00401100 >>>
00401118    8905 08004300    MOV DWORD PTR DS:[430008],EAX
0040111E    C3               RETN
00401100  - FF25 08114400    JMP DWORD PTR DS:[441108] <<<..>>>
00401106    8BC0             MOV EAX,EAX
00401108  - FF25 04114400    JMP DWORD PTR DS:[441104]
0040110E    8BC0             MOV EAX,EAX
00401110    31C0             XOR EAX,EAX
009B1C64    55               PUSH EBP  <<<<
009B1C65    8BEC             MOV EBP,ESP
009B1C67    8B45 08          MOV EAX,DWORD PTR SS:[EBP+8]
009B1C6A    85C0             TEST EAX,EAX
009B1C6C    75 13            JNZ SHORT 009B1C81
009B1C6E    813D A47A9B00 0>CMP DWORD PTR DS:[9B7AA4],400000         ; ASCII "MZP"
009B1C78    75 07            JNZ SHORT 009B1C81
009B1C7A    A1 A47A9B00      MOV EAX,DWORD PTR DS:[9B7AA4]
009B1C7F    EB 06            JMP SHORT 009B1C87
009B1C81    50               PUSH EAX
009B1C82    E8 3135FFFF      CALL 009A51B8                        ; JMP to kernel32.GetModuleHandleA
009B1C87    5D               POP EBP
009B1C88    C2 0400          RETN 4
```

\===

```
009C6C92    55               PUSH EBP ;像 STOLEN BYTES,就是 oep 处的代码,计算一下 OEP 应该是 42CA54
009C6C93    8BEC             MOV EBP,ESP
009C6C95    53               PUSH EBX
009C6C96    56               PUSH ESI
009C6C97    8B75 0C          MOV ESI,DWORD PTR SS:[EBP+C]
009C6C9A    8B5D 08          MOV EBX,DWORD PTR SS:[EBP+8] ;//到这里为止
009C6C9D    EB 11            JMP SHORT 009C6CB0
```

\=====

\===

```
0042C6D0   .  B8 ACE84200    MOV EAX,ADUMP-~3.0042E8AC ;42e8ac 开始的 13 字节被加密.

密文:0042E8AC  5A 5A 88 44 07 8D 07 F7 39 85 0B F3 EE
解密:0042E8AC  5B 58 8B 40 02 8B 00 FF 30 8F 00 FF E3

//通过 0042C6D5      3018          XOR BYTE PTR DS:[EAX],BL 解密,BL 是逐渐加 1 的.

0042C6D5      EB 07          JMP SHORT ADUMP-~3.0042C6DE ;修改后的指令,下面的解密代码不执行,可将密文直接还原,因为我们还要修改解密的代码.

0042C6D7   .  43             INC EBX
0042C6D8   .  40             INC EAX
0042C6D9   .  83FB 0E        CMP EBX,0E
0042C6DC   .^ 75 F7          JNZ SHORT ADUMP-~3.0042C6D5
0042C6DE   .  33C0           XOR EAX,EAX    ;跳到这里
0042C6E0   .  55             PUSH EBP
0042C6E1   .  68 07C74200    PUSH ADUMP-~3.0042C707
0042C6E6   .  64:FF30        PUSH DWORD PTR FS:[EAX]
```

\---

又一个加密段

```
0042C727    E8 8C49FDFF      CALL A-MAST~2.004010B8
0042C72C    BB 01000000      MOV EBX,1
0042C731    B8 ACE84200      MOV EAX,A-MAST~2.0042E8AC ;这里 42E8AC
0042C736    3018             XOR BYTE PTR DS:[EAX],BL
0042C738    43               INC EBX
0042C739    40               INC EAX
0042C73A    83FB 0E          CMP EBX,0E
0042C73D  ^ 75 F7            JNZ SHORT A-MAST~2.0042C736
```

\---

```
0042E8B3    FF30             PUSH DWORD PTR DS:[EAX] ;EAX,就是 GETSYSTEMTIME 的地址...注意这段代码在 42C6D0 动态解密
0042E8B5    8F00             POP DWORD PTR DS:[EAX] ;这里如果脱壳 POP 到 GETSYSTEMTIME 函数地址造成异常,这里可能是检测是否脱壳.
0042E8B7    FFE3             JMP EBX
```

\----

```
0042C48C  |.  8B00           MOV EAX,DWORD PTR DS:[EAX]               ; |
0042C48E  |.  50             PUSH EAX                                 ; |hModule
0042C48F  |.  E8 844DFDFF    CALL <JMP.&kernel32.GetProcAddress>      ; \\GetProcAddress
0042C494  |.  FFD0           CALL EAX  ///异常,加壳程序 EAX 返回 1B ,哈哈,其实是剩余天数了.
///可以这样改 0042C494      B0 FF         MOV AL,0FF
0042C496  |.  8B15 C0EE4200 MOV EDX,DWORD PTR DS:[42EEC0]            ;  A-MAST~4.00440E7C
0042C49C  |.  8902           MOV DWORD PTR DS:[EDX],EAX
0042C49E  |.  A1 C0EE4200    MOV EAX,DWORD PTR DS:[42EEC0]
0042C4A3  |.  8338 00        CMP DWORD PTR DS:[EAX],0
0042C4A6  |.  74 02          JE SHORT A-MAST~4.0042C4AA
0042C4A8  |.  B3 01          MOV BL,1
```

这样修复 `IAT` 后 `ABOUT` 窗不闪烁了。但 `9X` 下肯定出问题，因为 `IAT` 不同

`98` 修复 `IAT`，运行正常。`IAT` 附后，根据以往经验，`98` 下修复的 `IAT` 可在 `2K`,`XP` 下运行。

\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*

\===

首次跟踪::>>

`ASPRDBGR` 的调试信息:

```
AsprDbgr v1.0beta (:P) Made by me... Manko.

  iEP=401000 (C:\temp\1\1\A-master19.exe)
  GST returns to: 992667
    Trick aspr GST... (EAX=12121212h)
  GV returns to: 9A1A61
    IAT Start: 441104   //IAT 开始的地方
          End: 4414FC
       Length: 3F8   //IAT 的大小,可以用 IMPORTREC 重建 IAT 表了.
      IATentry 441108 = 9A1C64 resolved as GetModuleHandleA
      IATentry 441178 = 9A17A4 resolved as GetProcAddress
      IATentry 44117C = 9A1C64 resolved as GetModuleHandleA
      IATentry 441194 = 9A1CD8 resolved as GetCommandLineA
      IATentry 44133C = 9A1D14 resolved as DialogBoxParamA
    11 invalid entries erased.
  Dip-Table at adress: 9A7AB4
    0 412F20 0 0 0 0 0 42C514 40E2F4 40E344 0 0 0 0
  Last SEH passed. Searching for signatures. Singlestepping to OEP!
    Call + OEP-jump-setup at: 9B73D9 ( Code: E8000000 5D81ED )
    Mutated, stolen bytes at: 9B7424 ( Code: 61F2EB01 9A2EEB01 )
    Erase of stolen bytes at: 9B7388 ( Code: 9CFCBFC7 739B00B9 )
      Repz ... found. Skipping erase of stolen bytes. ;)
  Dip from pre-OEP: 401100 (Reached from: 9B7399)
  Sugested tempOEP at: 7FFDEFFB
DebugProcess ended. (??)
```

\====

```
009A1C64    55               PUSH EBP  ;这里可能是 OEP 的原是代码.
009A1C65    8BEC             MOV EBP,ESP
009A1C67    8B45 08          MOV EAX,DWORD PTR SS:[EBP+8]
009A1C6A    85C0             TEST EAX,EAX
009A1C6C    75 13            JNZ SHORT 009A1C81
009A1C6E    813D A47A9A00 0>CMP DWORD PTR DS:[9A7AA4],400000         ; ASCII "MZP"
009A1C78    75 07            JNZ SHORT 009A1C81
009A1C7A    A1 A47A9A00      MOV EAX,DWORD PTR DS:[9A7AA4]  ;这里不久就会返回真正 OEP
```

\---

猜想 `OEP` 代码(`delphi`)

```
push ebp
mov ebp,esp
add esp,-0c
mov eax,4203f8
```

\---

修改后 `OEP` 处的代码:

```
0042CA53 > $  55             PUSH EBP
0042CA54   .  8BEC           MOV EBP,ESP
0042CA56   .  83C4 F4        ADD ESP,-0C
0042CA59   .  B8 44C94200    MOV EAX,ADUMP-~4.0042C944
0042CA5E   .  90             NOP ;//其实 OEP 应该在 42CA54,因为这里多了一字节,不过这样也行
0042CA5F   .  E8 AC46FDFF    CALL ADUMP-~4.00401110
0042CA64   .  E8 EB18FEFF    CALL ADUMP-~4.0040E354 //加壳程序会返回这里
0042CA69   .  E8 9E45FDFF    CALL ADUMP-~4.0040100C
0042CA6E   .  8BC0           MOV EAX,EAX
0042CA70   .  0000           ADD BYTE PTR DS:[EAX],AL
0042CA72   .  0000           ADD BYTE PTR DS:[EAX],AL
```

\====

```
0042C6D0   .  B8 ACE84200    MOV EAX,ADUMP-~3.0042E8AC ;42e8ac 开始的 13 字节被加密.

密文:0042E8AC  5A 5A 88 44 07 8D 07 F7 39 85 0B F3 EE
解密:0042E8AC  5B 58 8B 40 02 8B 00 FF 30 8F 00 FF E3

//通过 0042C6D5      3018          XOR BYTE PTR DS:[EAX],BL 解密,BL 是逐渐加 1 的.

0042C6D5      EB 07          JMP SHORT ADUMP-~3.0042C6DE ;修改后的指令,下面的解密代码不执行,可将密文直接还原,因为我们还要修改解密的代码.

0042C6D7   .  43             INC EBX
0042C6D8   .  40             INC EAX
0042C6D9   .  83FB 0E        CMP EBX,0E
0042C6DC   .^ 75 F7          JNZ SHORT ADUMP-~3.0042C6D5
0042C6DE   .  33C0           XOR EAX,EAX    ;跳到这里
0042C6E0   .  55             PUSH EBP
```



## ADUMP-~3.0042C6E1

```
0042C6E1   .  68 07C74200    PUSH ADUMP-~3.0042C707
0042C6E6   .  64:FF30        PUSH DWORD PTR FS:[EAX]
```

\====

## 加密代码异常点

```
0042E8B1   8B00             MOV EAX,DWORD PTR DS:[EAX];这里即为加密代码
0042E8B3   EB 02            JMP SHORT ADUMP-~4.0042E8B7 ;这是修改后的指令,因为这里会产生非法指令
0042E8B5   8F00             POP DWORD PTR DS:[EAX]
0042E8B7   FFE3             JMP EBX
0042E8B9   8D40 00          LEA EAX,DWORD PTR DS:[EAX]
```

///这个异常

\===

## ADUMP-~4.0042C744

```
0042C744   .  E8 17CCFEFF   CALL ADUMP-~4.00419360
0042C749   .  EB 03         JMP SHORT ADUMP-~4.0042C74E ;这里异常,直接跳过.
0042C74B      90            NOP
0042C74C      90            NOP
0042C74D      90            NOP
0042C74E   >  84C0          TEST AL,AL
0042C750   .  0F84 8C010000 JE ADUMP-~4.0042C8E2
0042C756   .  A1 44EE4200   MOV EAX,DWORD PTR DS:[42EE44]
0042C75B   .  8B00          MOV EAX,DWORD PTR DS:[EAX]
```

现在 `DUMP` 的程序可以运行了,但是不知是否安全,这只是在 `XP` 下.

但是点 `ABOUT` 会导致屏幕闪烁.

\*\*\*\*\*\*\*\*\*\*\*

## `aston.exe`

`oep:4193a0`

```
00985CC9   /EB 01          JMP SHORT 00985CCC
00985CCB   |9A 51579CFC    CALL FAR 0DBF:FC9C5751             ; 远距呼叫
00985CD2    5D             POP EBP
00985CD3    98             CWDE
00985CD4    00B9 5E140000  ADD BYTE PTR DS:[ECX+145E],BH
00985CDA    F3:AA          REP STOS BYTE PTR ES:[EDI]
00985CDC    9D             POPFD
00985CDD    5F             POP EDI
00985CDE    59             POP ECX
00985CDF    C3             RETN
```

```
00985CE0    55             PUSH EBP  ;//oep 处代码
00985CE1    8BEC           MOV EBP,ESP
00985CE3    53             PUSH EBX
00985CE4    56             PUSH ESI
00985CE5    8B75 0C        MOV ESI,DWORD PTR SS:[EBP+C]
00985CE8    8B5D 08        MOV EBX,DWORD PTR SS:[EBP+8]
00985CEB    EB 11          JMP SHORT 00985CFE
00985CED    0FB703         MOVZX EAX,WORD PTR DS:[EBX]
```

问题和 `a-master.exe` 一样

## 解密循环 (004034C9)

```
004034C9   .  83C2 0F       ADD EDX,0F
004034CC   >  8B0D D0A54100 MOV ECX,DWORD PTR DS:[41A5D0] ;  解密 ECX=41A2C8
004034D2   .  8A0C01        MOV CL,BYTE PTR DS:[ECX+EAX]
004034D5   .  300A          XOR BYTE PTR DS:[EDX],CL
004034D7   .  48            DEC EAX
004034D8   .  4A            DEC EDX  ;//EDX 是字符位置,注意这里是重后往前解密
004034D9   .  85C0          TEST EAX,EAX
004034DB   .^ 75 EF        JNZ SHORT ASTON1~2.004034CC
004034DD   .  33C0          XOR EAX,EAX
```

密文:`0041A2E8 E7 7A 3C 28 4C 90 10 5D 8C 8F AA FF A3 00 00 2E`

解密:`0041A2E8 5B 58 8B 40 02 8B 00 FF 30 8F 00 FF E3 47 70 FC`

改成解密后的代码后,再改 `4034cc` 指令为:

```
004034CC      EB 0F         JMP SHORT ASTON1~2.004034DD
```

让他跳走,因为它已经没有用了.

## 解密后代码验证

```
0041A2E8    5B             POP EBX ; ASTON1~2.004034F7,这些指令是刚刚动态解密的.
0041A2E9    58             POP EAX
0041A2EA    8B40 02        MOV EAX,DWORD PTR DS:[EAX+2]
0041A2ED    8B00           MOV EAX,DWORD PTR DS:[EAX]
0041A2EF    FF30           PUSH DWORD PTR DS:[EAX] ;又来这一招,异常
0041A2F1    8F00           POP DWORD PTR DS:[EAX]  ;这里 pop 异常
0041A2F3    FFE3           JMP EBX
```

改为明文后,这样改:

```
0041A2EF    EB 02          JMP SHORT ASTON1~2.0041A2F3
```

\---

## 重新加密循环 (00403532)

```
00403532   .  83C2 0F       ADD EDX,0F                          ;  再加密刚才解密的指令,这个不必管他
00403535   >  8B0D D0A54100 MOV ECX,DWORD PTR DS:[41A5D0]       ;  ASTON1~2.0041A2C8
0040353B   .  8A0C01        MOV CL,BYTE PTR DS:[ECX+EAX]
0040353E   .  300A          XOR BYTE PTR DS:[EDX],CL
00403540   .  48            DEC EAX
00403541   .  4A            DEC EDX
00403542   .  85C0          TEST EAX,EAX
00403544   .^ 75 EF        JNZ SHORT ASTON1~2.00403535
00403546   .  5F            POP EDI
```

\===

## 解密代码 (00408738)

```
00408738   .  BA 80A34100   MOV EDX,ASTON1~2.0041A380           ;  解密
0040873D   >  8B0D 30A94100 MOV ECX,DWORD PTR DS:[41A930]       ;  ASTON1~2.0041A284 //OP:EB16
00408743   .  8B09          MOV ECX,DWORD PTR DS:[ECX]
00408745   .  8A0C01        MOV CL,BYTE PTR DS:[ECX+EAX]
00408748   .  300A          XOR BYTE PTR DS:[EDX],CL
0040874A   .  48            DEC EAX
0040874B   .  4A            DEC EDX
0040874C   .  85C0          TEST EAX,EAX
0040874E   .^ 75 ED        JNZ SHORT ASTON1~2.0040873D
00408750   .  33C0          XOR EAX,EAX
```

密文:`0041A374 34 3E FF 37 63 F9 65 A3 7D E6 63 8D 8C`

明文:`0041A374 5B 58 8B 40 02 8B 00 FF 30 8F 00 FF E3`

\---

## 解密后代码验证 (0041A374)

```
0041A374    5B             POP EBX
0041A375    58             POP EAX
0041A376    8B40 02        MOV EAX,DWORD PTR DS:[EAX+2]
0041A379    8B00           MOV EAX,DWORD PTR DS:[EAX]
0041A37B    FF30           PUSH DWORD PTR DS:[EAX]
0041A37D    8F00           POP DWORD PTR DS:[EAX] ;异常
0041A37F    FFE3           JMP EBX
```

\--

## 重新加密循环 (00408799)

```
00408799   .  BA 80A34100  MOV EDX,ASTON1~2.0041A380    ;再加密
0040879E   >  8B0D 30A94100 MOV ECX,DWORD PTR DS:[41A930]       ;  ASTON1~2.0041A284
004087A4   .  8B09          MOV ECX,DWORD PTR DS:[ECX]
004087A6   .  8A0C01        MOV CL,BYTE PTR DS:[ECX+EAX]
004087A9   .  300A          XOR BYTE PTR DS:[EDX],CL
004087AB   .  48            DEC EAX
004087AC   .  4A            DEC EDX
004087AD   .  85C0          TEST EAX,EAX
004087AF   .^ 75 ED        JNZ SHORT ASTON1~2.0040879E
```

\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*

## 注册用户名显示问题

右下角按钮,关于窗口里注册用户名显示问题,这些都是密文,直接搜索是找不到的.

用 `WINHEX` 内存编辑功能搜索注册名,找到偏移,再用 `OD` 下内存访问,找到以下代码:

\=================

## 注册名解密 (0041212D)

```
0041212D   > \57             PUSH EDI
0041212E   .  89C7           MOV EDI,EAX
00412130   .  BA 48214100   MOV EDX,ASTON.00412148 ;注册名加密存放在 412148
00412135   .  B1 14         MOV CL,14       ;放 CL 值用于解密,cl 就是总字符个数
00412137   >  8A02          MOV AL,BYTE PTR DS:[EDX]   ;逐个取字符
00412139   .  30C8          XOR AL,CL       ;解密
0041213B   .  8807          MOV BYTE PTR DS:[EDI],AL  ;放回[edi]=41c4e0
0041213D   .  42            INC EDX
0041213E   .  47            INC EDI
0041213F   .  FEC9          DEC CL      ;改变 CL 值
00412141   .^ 75 F4        JNZ SHORT ASTON.00412137  ;跳回解密下一个字符
00412143   .  C607 00       MOV BYTE PTR DS:[EDI],0
00412146   .  5F            POP EDI
00412147   .  C3            RETN
```

## 加密注册名数据 (00412148)

```
00412148      50             DB 50                          ;  CHAR 'P' 这里就是加密的注册名,直接改为明文
00412149      7E             DB 7E                          ;  CHAR '~'
0041214A      7B             DB 7B                          ;  CHAR '{'
0041214B      65             DB 65                          ;  CHAR 'e'
0041214C      62             DB 62                          ;  CHAR 'b'
0041214D      76             DB 76                          ;  CHAR 'v'
0041214E      21             DB 21                          ;  CHAR '!'
0041214F      58             DB 58                          ;  CHAR 'X'
00412150      62             DB 62                          ;  CHAR 'b'
00412151      6F             DB 6F                          ;  CHAR 'o'
00412152      6F             DB 6F                          ;  CHAR 'o'
00412153      7B             DB 7B                          ;  CHAR '{'
00412154      58             DB 58                          ;  CHAR 'X'
00412155      6B             DB 6B                          ;  CHAR 'k'
00412156      06             DB 06
00412157      05             DB 05
00412158      56             DB 56                          ;  CHAR 'V'
00412159      46             DB 46                          ;  CHAR 'F'
0041215A      46             DB 46                          ;  CHAR 'F'
0041215B      28             DB 28                          ;  CHAR '('
0041215C   .  C3            RETN
```

\==`zzhzihui@163.net` 16bytes

## 动态写入注册名 (00400080)

```
00400080     C705 48214100 5>MOV DWORD PTR DS:[412148],657B7E50 ;注意这里会动态写入 412148 内容,就是加密的注册名
```



0040008A `C705 4C214100 6>` `MOV DWORD PTR DS:[41214C],58217662`  

00400094 `C705 50214100 6>` `MOV DWORD PTR DS:[412150],7B6F6F62`  

0040009E `C705 54214100 5>` `MOV DWORD PTR DS:[412154],05066B58` ; 我们要用到这里,所以从 `400080` 要跳到 `4000d0`  

004000A8 `C705 342F4100 4>` `MOV DWORD PTR DS:[412F34],7F65604A` ; 这里动态写入 ABOUT 窗的注册名,所以也要跳过  

004000B2 `C705 382F4100 7>` `MOV DWORD PTR DS:[412F38],52277078`  

004000BC `C705 3C2F4100 6>` `MOV DWORD PTR DS:[412F3C],71616168`  

004000C6 `C705 402F4100 5>` `MOV DWORD PTR DS:[412F40],16176D52`  

004000D0 `C605 E8334100 C>` `MOV BYTE PTR DS:[4133E8],0C3`  

004000D7 `C605 DB344100 8>` `MOV BYTE PTR DS:[4134DB],84`  

### 修改

`00400080` `/EB 4E` `JMP SHORT ASTON.004000D0`  

### 412f42 (165h)

```
00412D1E    89F9            MOV ECX,EDI  ;ECX=165h, 357bytes 加密的 ABOUT 框文本
00412D20    BE DD2D4100     MOV ESI,aston.00412DDD ;这里 412ddd 密文开始
00412D25    89C7            MOV EDI,EAX    ;EDI=1bFC68 明文开始,EDI 是随机变化的
00412D27    89C2            MOV EDX,EAX    ;明文开始,用于计数器
00412D29    8A06            MOV AL,BYTE PTR DS:[ESI] ;循环解密
00412D2B    30C8            XOR AL,CL    ;逐字节解密(nop)
00412D2D    8802            MOV BYTE PTR DS:[EDX],AL ;逐字节放入 1BFC68
00412D2F    46              INC ESI
00412D30    42              INC EDX
00412D31    49              DEC ECX
00412D32   ^ 75 F5          JNZ SHORT aston.00412D29 ;跳回继续解密
00412D34    57              PUSH EDI
00412D35    53              PUSH EBX
....
00412D42   .  89C7          MOV EDI,EAX    ;这是解密字符 YOU HAVE ??? DAYS LEFT
00412D44   .  89C3          MOV EBX,EAX
00412D46   .  BA 422F4100   MOV EDX,ASTON.00412F42
00412D4B   .  B9 17000000   MOV ECX,17
00412D50   >  8A02          MOV AL,BYTE PTR DS:[EDX]
00412D52   .  30C8          XOR AL,CL ;(nop)
00412D54   .  8807          MOV BYTE PTR DS:[EDI],AL
00412D56   .  42            INC EDX
00412D57   .  47            INC EDI
00412D58   .  49            DEC ECX
00412D59   .^ 75 F5        JNZ SHORT ASTON.00412D50
00412D5B   .  C607 00       MOV BYTE PTR DS:[EDI],0
```

### 密文: 412ddd-412f58

```
00412DDD  24 44 30 42 35 40 10 7E 13 7C 6A 74 60 76 67 5B  $D0B5@~|jt\`vg\[
00412DED  5F 17 3C 22 28 22 26 29 25 38 6B E3 69 79 7E 7F  \_<"("&)%8k 鉯 y~
00412DFD  7C 69 71 72 71 73 1F 79 51 5D 5F 53 58 4C 58 44  |iqrqsyQ\]\_SXLXD
00412E0D  46 14 60 5D 57 44 58 4F 5F 49 05 27 23 69 4B 4A  F\`\]WDXO\_I'#iKJ
00412E1D  05 56 4A 45 49 54 6C 3E 6F 79 68 7F 6B 6E 72 72  VJEITl>oyhknrr
00412E2D  18 1E 56 3F 7C 71 66 62 37 2C 78 7F 79 78 68 74  V?|qfb7,xyxht
00412E3D  71 44 62 71 75 6F 91 8D 95 99 97 96 D7 9B 98 9B  qDbquo 憤暀棖讻槢
00412E4D  F8 FE BB 9D 9C 95 CF 9E 8C 8B 8E D0 C9 80 93 92  粷湑蠟寢幮蓘搾
00412E5D  95 DE CC CD 96 97 A8 F0 BC AF AF B5 B7 AB BF B3  曓掏枟集帆砍
00412E6D  B9 B8 FD B1 BE BD C2 C4 C0 C6 9B B8 A6 AF B5 A7  垢窘履榔浉Ο掸
00412E7D  A8 A9 AA AC A6 FA 9F F1 D1 D9 DC 9A FB CD DB CF  ī燅奄軞巯
00412E8D  D6 DC DC C4 9D 90 E1 C7 CE C7 8B EF CE C7 D5 C9  周苣潗崆吻嬶吻丈
00412E9D  D3 A9 A9 E3 C5 C4 F6 EA F4 F3 F5 FB F5 B8 C4 E3  萤┿拍鲫趔觖醺你
00412EAD  E5 E4 FC E0 E5 AA AF CA E0 E5 FF F8 F0 A8 D7 F4  邃濯噱ㄗ
00412EBD  EA F7 F5 EB F3 EE 16 10 51 71 71 2A 18 0E 12 1A  犄蹼箢 Qqq\*
00412ECD  55 3F 1C 00 1C 19 1C 06 04 02 47 4A 39 09 12 0A  U?.GJ9..
00412EDD  45 2A 0A 09 0E 0E 30 38 3B 70 56 50 1C 2E 30 33  E\*..08;pVP.03
00412EED  3B 3B 25 72 1F 39 24 27 39 2D 67 6A 1D 29 35 27  ;;%r9$'9-gj)5'
00412EFD  36 64 01 30 28 3A 56 4A 4E 57 42 16 34 32 73 5B  6d0(:VJNWB42s\[
00412F0D  5C 40 41 4B 11 63 5A 46 42 48 44 4F 5F 04 07 62  \\@AKcZFBHDO\_b
00412F1D  0B 68 0D 02 6E 4C 7A 6D 7E 74 36 4D 70 74 7B 7F  h.nLzm~t6Mpt{
00412F2D  74 79 60 1F 1B 1D 05 4A 60 65 7F 78 70 27 52 68  ty\`J\`exp'Rh
00412F3D  64 65 72 50 6C 00 79 60 34 7B 73 67 75 2F 2B 78  derPl.y\`4{sgu/+x
00412F4D  2C 6F 6B 70 20 74 2F 25 68 66 64 75              ,okp t/%hfdu
```

### 明文

```
001BFC68  41 20 53 20 54 20 4F 20 4E 20 31 2E 39 2E 30 0D  A S T O N 1.9.0.
001BFC78  0A 43 6F 70 79 72 69 67 68 74 20 A9 20 31 39 39  .Copyright ?199
001BFC88  39 2D 32 30 30 33 20 47 6C 61 64 69 61 74 6F 72  9-2003 Gladiator
001BFC98  73 20 53 6F 66 74 77 61 72 65 2E 0D 0A 41 6C 6C  s Software...All
001BFCA8  20 72 69 67 68 74 73 20 72 65 73 65 72 76 65 64   rights reserved
001BFCB8  0D 0A 45 2D 6D 61 69 6C 3A 20 73 75 70 70 6F 72  ..E-mail: suppor
001BFCC8  74 40 61 73 74 6F 6E 73 68 65 6C 6C 2E 63 6F 6D  t@astonshell.com
001BFCD8  0D 0A 48 6F 6D 65 20 70 61 67 65 3A 20 68 74 74  ..Home page: htt
001BFCF8  70 3A 2F 2F 77 77 77 2E 61 73 74 6F 6E 73 68 65  p://www.astonshe
001BFCF8  6C 6C 2E 63 6F 6D 0D 0A 0D 0A 50 72 6F 67 72 61  ll.com....Progra
001BFD08  6D 6D 69 6E 67 3A 20 4F 6C 65 67 20 42 75 6C 79  mming: Oleg Buly
001BFD18  63 68 6F 76 2C 20 4E 69 63 6B 20 45 67 6F 72 6F  chov, Nick Egoro
001BFD28  76 0D 0A 41 64 64 69 74 69 6F 6E 61 6C 20 53 75  v..Additional Su
001BFD38  70 70 6F 72 74 3A 20 44 6D 69 74 72 79 20 50 72  pport: Dmitry Pr
001BFD48  6F 73 76 69 72 6E 69 6E 2C 0D 0A 50 61 76 65 6C  osvirnin,..Pavel
001BFD58  20 4B 6F 72 6D 69 73 68 69 6E 2C 20 50 61 75 6C   Kormishin, Paul
001BFD68  20 4E 69 6B 6F 6E 6F 66 66 2C 0D 0A 45 76 67 65   Nikonoff,..Evge
001BFD78  6E 6F 76 20 4E 69 6B 69 74 61 2C 20 54 61 72 61  nov Nikita, Tara
001BFD88  73 20 42 72 69 7A 69 74 73 6B 79 2C 0D 0A 44 6D  s Brizitsky,..Dm
001BFD98  69 74 72 79 20 53 75 68 6F 64 6F 65 76 2C 20 44  itry Suhodoev, D
001BFDA8  2E 4C 2E 20 4F 6C 65 73 63 68 2D 57 69 6C 6C 69  .L. Olesch-Willi
001BFDB8  61 6D 73 0D 0A 0D 0A 44 6D 69 74 72 79 2F 55 6E  ams....Dmitry/Un
001BFDC8  64 65 72 50 6C 00                              derPl.
```

### 方法

将密文通过 `WINHEX` 直接改为明文,将动态写入的部分跳过,将 `XOR AL,CL` 这段解密代码 `NOP`

当然也可以根据要显示的字符通过逆算法(`xor al,cl`)算出密文,这样只需要直接改一小部分密文即可

### OEP: 414d9c

```
00995EB7    FC              CLD
00995EB8    BF F55E9900     MOV EDI,995EF5
00995EBD    B9 5E140000     MOV ECX,145E
00995EC2    F3:AA          REP STOS BYTE PTR ES:[EDI]
00995EC4    9D              POPFD
00995EC5    5F              POP EDI
00995EC6    59              POP ECX
00995EC7    C3              RETN
00995EC8    55              PUSH EBP //这里 STOLEN BYTES,OEP 代码
00995EC9    8BEC            MOV EBP,ESP
00995ECB    53              PUSH EBX
00995ECC    56              PUSH ESI
00995ECD    8B75 0C         MOV ESI,DWORD PTR SS:[EBP+C]
00995ED0    8B5D 08         MOV EBX,DWORD PTR SS:[EBP+8]
00995ED3    EB 11           JMP SHORT 00995EE6
00995ED5    0FB703          MOVZX EAX,WORD PTR DS:[EBX]
```

---

```
004010F4  - FF25 1C314200  JMP DWORD PTR DS:[42311C] //tc eip<900000,停在这里,DUMP
004010FA    8BC0           MOV EAX,EAX
004010FC  - FF25 18314200  JMP DWORD PTR DS:[423118]
00401102    8BC0           MOV EAX,EAX
00401104    31C0           XOR EAX,EAX
00401106    50             PUSH EAX
00401107    E8 E8FFFFFF    CALL THWIZARD.004010F4
```

---

修复 `IAT`,无其他异常.

### 注

这里可能修改了某些信息,但是只是作为个人使用,未对外发放,请原作者见谅.

贴子很乱(垃圾贴)

需要的朋友可以搜索 `aston1.90` 下载原版,然后脱壳,如果需要 `98` 下的 `IAT` 表,可以向我询问.

`zzhzihui@tom.com`
