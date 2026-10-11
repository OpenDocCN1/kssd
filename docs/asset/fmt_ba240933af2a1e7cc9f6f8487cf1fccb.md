
# VMP 宏观分析：Handlers 解析

我决定从宏观上去看 VMP。对于 handlers 也是一样，还是提供一些精品 handlers 吧。有时间就来更新这个帖子，没时间就不来了。

要看这个还是要对 `vmp` 的堆栈划分有明确的理解。我把 `ebp` 指向的堆栈部分叫 `ebpstack`，把 `edi` 指向的堆栈部分叫 `edistack`，把 `esp` 指向的堆栈部分叫 `espstack`。

## 核心概念：首次处理的重要性

第一次很重要，所以我决定不是放一个，而是放两个 handlers，特别突出这个 `first` 的重要性。

### the first

#### handler address: `0043DC0A`

**源代码：**

```
CPU Disasm
Address   Hex dump        Command                         Comments

0043DC0A  66:D3D2        RCL  DX,CL
0043DC0D  66:F7DA        NEG  DX
0043DC10  8D14A5 BD3F7DFA LEA EDX,[FA7D3FBD]
0043DC17  39F0            CMP  EAX,ESI
0043DC19  F6D8            NEG  AL                       ; *
0043DC1B  8D96 834403BB   LEA EDX,[ESI+BB034483]
0043DC21  9C              PUSHFD
0043DC22  F8              CLC
0043DC23  66:0FCA        BSWAP DX                       ; Undocumented instruction or encoding
0043DC26  C0C8 07        ROR  AL,7                      ; *
0043DC29  66:F7DA        NEG  DX
0043DC2C  8D92 915411CF   LEA EDX,[EDX+CF115491]
0043DC32  FECA            DEC  DL
0043DC34  2C 20           SUB  AL,20                     ; *
0043DC36  81E2 37007496   AND EDX,96740037
0043DC3C  C0D2 05        RCL  DL,5
0043DC3F  52              PUSH EDX
0043DC40  F9              STC
0043DC41  24 3C           AND  AL,3C                     ; *
0043DC43  8D6424 08       LEA  ESP,[ESP+8]
0043DC47  0F8F E90E0000   JG   0043EB36   ；这里有一个分支，但是两个分支都是一样的，除了有一次走过另一个分支，其他都是跳走了，所以下面的代码我也就不发了，发跳转部分。

0043EB36  0FBDD4            BSR  EDX,ESP
0043EB39  66:81D2 A36F    ADC  DX,6FA3
0043EB3E  E8 3DF5FFFF      CALL 0043E080

0043E080  8B55 00          MOV  EDX,DWORD PTR SS:[EBP]       ; *
0043E083  F6C6 E4          TEST DH,E4
0043E086  83C5 04          ADD  EBP,4                     ; *
0043E089  60              PUSHAD
0043E08A  60              PUSHAD
0043E08B  ^ E9 47F3FFFF    JMP  0043D3D7

0043D3D7  891438          MOV  DWORD PTR DS:[EDI+EAX],EDX    ; *
0043D3DA  FF3424          PUSH DWORD PTR SS:[ESP]
0043D3DD  68 7334B716     PUSH 16B73473
0043D3E2  8D6424 4C       LEA  ESP,[ESP+4C]
0043D3E6  E9 D0120000     JMP  0043E6BB  ；到这里就结束了，下面就回到 Dispatch 部分了
```

估计没有人有兴趣去静态分析和好好的看上面的代码，我还是再贴个去除了 junk code 的简化版本。

```
0043DC19    F6D8         NEG  AL                      ; *
0043DC26    C0C8 07     ROR  AL,7                    ; *
0043DC34    2C 20       SUB  AL,20                   ; *
0043DC41    24 3C       AND  AL,3C                   ; *
0043E080    8B55 00     MOV  EDX,DWORD PTR SS:[EBP]    ; *
0043E086    83C5 04     ADD  EBP,4                   ; *
0043D3D7    891438      MOV  DWORD PTR DS:[EDI+EAX],EDX; *
```

计算 `al`，以 `al` 为索引，执行 `mov [edi+eax], [ebp]`。`al` 是从 bytecode 里读出来的，`vmp` 的 bytecode 压力应该是很大的。首先是在 `Dispatch` 里计算出来 `al`，然后通过 `[EAX*4+43E1DB]` 得到 `handleraddress`，这样了还不不能休息，进入这样的 handler 里面，还要继续参与计算。

这就是一个移动数据，`VMP` 不断的使用这个 handler，把数据从 `ebpstack` 移动到 `edistack`。感觉 `vmp` 挺无聊的，有事的时候调用一下这个，没事的时候也调用一下这个，把数据挪去挪来的。

### 对应的反向移动 Handler

挪去还要挪来，这就有了一个和它相对应的 handler。

#### handler address: `0043DFBE`

**源代码：**

```
CPU Disasm
Address   Hex dump        Command                         Comments

0043DFBE  0FBED3          MOVSX EDX,BL
0043DFC1  F6D8            NEG  AL                       ; *
0043DFC3  66:0FB6D2      MOVZX DX,DL
0043DFC7  66:0FABDA      BTS  DX,BX
0043DFCB  0FCA            BSWAP EDX
0043DFCD  C0C8 07        ROR  AL,7                      ; *
0043DFD0  66:0FABD2      BTS  DX,DX
0043DFD4  9C              PUSHFD
0043DFD5  66:89EA        MOV  DX,BP
0043DFD8  00FA            ADD  DL,BH
0043DFDA  2C 20           SUB  AL,20                     ; *
0043DFDC  F5              CMC
0043DFDD  24 3C           AND  AL,3C                     ; *
0043DFDF  E8 84EEFFFF     CALL 0043CE68

0043CE68  D2E2            SHL  DL,CL
0043CE6A  28FA            SUB  DL,BH
0043CE6C  8B1438          MOV  EDX,DWORD PTR DS:[EDI+EAX]  ; *
0043CE6F  F5              CMC
0043CE70  9C              PUSHFD
0043CE71  83ED 04         SUB  EBP,4                     ; *
0043CE74  E8 00000000     CALL 0043CE79
0043CE79  8955 00         MOV  DWORD PTR SS:[EBP],EDX    ; *
0043CE7C  9C              PUSHFD
0043CE7D  FF3424          PUSH DWORD PTR SS:[ESP]
0043CE80  8D6424 18       LEA  ESP,[ESP+18]
0043CE84  ^ E9 D0FFFFFF   JMP  0043CE59

0043CE59  /98             CWDE
0043CE5A  |8D47 50       LEA  EAX,[EDI+50]
0043CE5D  |F7C2 2BCFEA55 TEST EDX,55EACF2B
0043CE63  |E9 F01F0000   JMP  0043EE58

0043EE58  \\60            PUSHAD
0043EE59  66:0FA3F0      BT   AX,SI
0043EE5D  39C5            CMP  EBP,EAX
0043EE5F  ^ E9 CDE5FFFF   JMP  0043D431

0043D431  /E9 521C0000   JMP  0043F088

0043F088  \\8D6424 20    LEA  ESP,[ESP+20]
0043F08C  ^ 0F87 29F6FFFF JA   0043E6BB
```

以上代码完整粘贴自 OD，简化后的代码如下：

```
0043DFC1    F6D8         NEG  AL                      ; *
0043DFCD    C0C8 07     ROR  AL,7                    ; *
0043DFDA    2C 20       SUB  AL,20                   ; *
0043DFDD    24 3C       AND  AL,3C                   ; *
0043CE6C    8B1438      MOV  EDX,DWORD PTR DS:[EDI+EAX]; *
0043CE71    83ED 04     SUB  EBP,4                   ; *
0043CE79    8955 00     MOV  DWORD PTR SS:[EBP],EDX  ; *
```

这个 handler 和上一个是对应的，这个是计算 `al`，以 `al` 为索引，把数据从 `[edi+eax]` 移动到 `[ebp]`，也就是把数据从 `edistack` 移动到 `ebpstack`。

通过这 2 个 handlers，`VMP` 把数据动来动去，有事时候动一下，没事时候动一下，无聊时候动一下。

### the second

#### handler address: `0043D198`

```
0043D198  98              CWDE
0043D199  66:0FBCD2      BSF  DX,DX
0043D19D  10EA            ADC  DL,CH
0043D19F  66:0FB6C2      MOVZX AX,DL
0043D1A3  8B45 00         MOV  EAX,DWORD PTR SS:[EBP]      ; *
0043D1A6  66:F7C3 3739    TEST BX,3937
0043D1AB  D0D2            RCL  DL,1
0043D1AD  8B55 04         MOV  EDX,DWORD PTR SS:[EBP+4]     ; *
0043D1B0  66:0FBAE5 05    BT   BP,5
0043D1B5  68 3B4E2623     PUSH 23264E3B
0043D1BA  E9 EA0C0000     JMP  0043DEA9

0043DEA9  \\66:0FBAE5 06  BT   BP,6
0043DEAE  F7D0            NOT  EAX                    ; *
0043DEB0  ^ E9 2CFFFFFF   JMP  0043DDE1

0043DDE1  /F7D2           NOT  EDX                    ; *
0043DDE3  |F8             CLC
0043DDE4  |E8 D8EFFFFF    CALL 0043CDC1

0043CDC1  F8              CLC    ; NOTEPAD.0043CDC1(guessed Arg1,Arg2,Arg3,Arg4,Arg5,Arg6,Arg7,Arg8,Arg9,Arg10,Arg11,Arg12,Arg13,Arg14,Arg15,Arg16,Arg17,Arg18,Arg19,Arg20)
0043CDC2  21D0            AND  EAX,EDX                  ; *
0043CDC4  E9 28130000     JMP  0043E0F1

0043E0F1  \\9C            PUSHFD
0043E0F2  60              PUSHAD
0043E0F3  68 E0E4B3D7     PUSH D7B3E4E0
0043E0F8  8945 04         MOV  DWORD PTR SS:[EBP+4],EAX   ; *
0043E0FB  ^ E9 FBEFFFFF   JMP  0043D0FB

0043D0FB  /9C             PUSHFD                         ; *
0043D0FC  |8F4424 2C     POP  DWORD PTR SS:[ESP+2C]     ; *
0043D100  |E9 371B0000   JMP  0043EC3C

0043EC3C  \\C64424 04 8D  MOV  BYTE PTR SS:[ESP+4],8D
0043EC41  55              PUSH EBP
0043EC42  FF7424 0C       PUSH DWORD PTR SS:[ESP+0C]
0043EC46  FF7424 34       PUSH DWORD PTR SS:[ESP+34]     ; *
0043EC4A  8F45 00         POP  DWORD PTR SS:[EBP]        ; *
0043EC4D  9C              PUSHFD
```
