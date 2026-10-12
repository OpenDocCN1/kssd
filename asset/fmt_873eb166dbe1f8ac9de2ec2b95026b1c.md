
### 1. Dispatch 核心逻辑与字节码读取

```
0043E6A9  |.  89F3             MOV EBX,ESI                     ; *
0043E6AB  |.  0FBAF2 05       BTR EDX,5
0043E6AF  |.  F7DA            NEG EDX
0043E6B1  |.  0FCA            BSWAP EDX
0043E6B3  |.  66:0FBAEA 0E    BTS DX,0E
0043E6B8  |.  0375 00         ADD ESI,DWORD PTR SS:[EBP]       ; *
0043E6BB  |>  66:0FA5FA       SHLD DX,DI,CL                   ; Dispatch
0043E6BF  |.  8A46 FF         MOV AL,BYTE PTR DS:[ESI-1]       ; *
```

在 `ebx` 中备份一下 `esi` 的 `0041e381`。`[ebp]` 中的是堆栈 `0013FF90` 里面的 `0`，看来这是一个偏移量，用来修正 bytecode 地址。开始读取 bytecode，看来准备工作完成。它没有读取 `[esi]`，而是 `[esi-1]`。

```
0043E6C2  |.  88FA            MOV DL,BH
0043E6C4  |.  30D8            XOR AL,BL                       ; * 在 ebx 中备份 bytecode 地址；原来是要地址参与到计算中
0043E6C6  |.  FECA            DEC DL
0043E6C8  |.  8D95 88EB93F7   LEA EDX,[EBP+F793EB88]
0043E6CE  |.  F6D0            NOT AL                          ; *
0043E6D0  |.  42              INC EDX
0043E6D1  |.  F7D2            NOT EDX
0043E6D3  |.  0F9FC2          SETG DL
0043E6D6  |.  FEC8            DEC AL                          ; *
0043E6D8  |.  D0CA            ROR DL,1
0043E6DA  |.  C0C8 07         ROR AL,7                        ; * 一直在计算 al 值，算好了
0043E6DD  |.  F5              CMC
0043E6DE  |.  C0E6 06         SHL DH,6
0043E6E1  |.  83EE 01        SUB ESI,1                       ; * 指针减 1
0043E6E4  |.  D0D2            RCL DL,1
0043E6E6  |.  66:0FB6D1       MOVZX DX,CL
0043E6EA  |.  66:D1DA         RCR DX,1
0043E6ED  |.  30C3            XOR BL,AL                       ; * bl 可是参与到计算的，bl 帮 al 算完了，现在 al 来帮 bl 算好下一次的值
0043E6EF  |.  60              PUSHAD
0043E6F0  |.  66:0FBAE1 06    BT CX,6
0043E6F5  |.^ E9 32E9FFFF     JMP 0043D02C
```

#### 地址解密与 Handler 查找

```
0043D02C  |>  /C0DE 07        RCR DH,7
0043D02F  |.  |0FB6C0         MOVZX EAX,AL                  ; *
0043D032  |.  |F5             CMC
0043D033  |.  |E9 E7200000    JMP 0043F11F
0043F11F  |>  \38F5           CMP CH,DH
0043F121  |.  66:FFC2         INC DX
0043F124  |.  8B1485 DBE143   MOV EDX,DWORD PTR DS:[EAX*4+43E1DB]  ; *等你等到我心痛呀
```

终于开始，`DispatchTable` 原来是在 `43E1DB`，获得的是加密的地址，如：`49C4C29F`，数据窗口去 `43E1DB` 看看，茫茫多的地址呀，有 200 多个，不过这个不代表着有 200 多个 handler，应该有一些是指向相同的 handler。

```
0043F12B  |.  F9              STC
0043F12C  |.  84F4            TEST AH,DH
0043F12E  |.  60              PUSHAD
0043F12F  |.^ E9 CCEFFFFF     JMP 0043E100
0043E100  |>  /81C2 6B197FB6  ADD EDX,B67F196B              ; *原来只用一条 ADD 指令就解出来了，上面的 49C4C29F=》0043DC0A，很简单的解密，自己可以去 DispatchTable 计算出所有的
0043E106  |.  |C60424 46     MOV BYTE PTR SS:[ESP],46
0043E10A  |.  |895424 3C     MOV DWORD PTR SS:[ESP+3C],EDX   ; * handler 地址存一下，存到哪里呢？看看堆栈吧
```

堆栈数据示例：
`0013FECC  0043DC0A  .C.  ; 存到这里`
`0013FED0  E5064000  .@   ;这里是 EDI 指向的空间，看一下前面的堆栈划分图`

#### Handler 调用准备

```
0043E10E  |.  |9C             PUSHFD
0043E10F  |.  |68 A42795AD    PUSH AD9527A4
0043E114  |.  |FF7424 0C      PUSH DWORD PTR SS:[ESP+0C]
0043E118  |.  |FF3424         PUSH DWORD PTR SS:[ESP]
0043E11B  |.  |FF7424 4C      PUSH DWORD PTR SS:[ESP+4C]      ; *
0043E11F  |.  |C2 5000       RETN 50                           ; *
```

这两条指令要一起看，第一条 push 的就是 `0013FECC  0043DC0A` 这个 handler 地址，第二条就跑去找 handler。

### 2）handlers 解析

紧接着上文，第一个进入的 handler：

```
; bytecode:[0041E380]=0C ; dispatchtable:[0043E563]=49C4C29F handleraddr:0043DC0A
; 功能（按照流程应该放在 handler 后面，但为了方便，把它放到这里）：把 ebpstack 中，ebp 指向的当前数据，移动到 edistack，具体位置则根据 eax 值变动；移动到[edi+eax]的位置
```

```
0043DC0A  |.  66:D3D2         RCL DX,CL
0043DC0D  |.  66:F7DA         NEG DX
0043DC10  |.  8D14A5 BD3F7D   LEA EDX,[FA7D3FBD]
0043DC17  |.  39F0            CMP EAX,ESI
0043DC19  |.  F6D8            NEG AL                          ; *
0043DC1B  |.  8D96 834403BB  LEA EDX,[ESI+BB034483]
0043DC21  |.  9C              PUSHFD
0043DC22  |.  F8              CLC
0043DC23  |.  66:0FCA         BSWAP DX                        ; Undocumented instruction or encoding
0043DC26  |.  C0C8 07        ROR AL,7                        ; *
0043DC29  |.  66:F7DA        NEG DX
0043DC2C  |.  8D92 915411CF  LEA EDX,[EDX+CF115491]
0043DC32  |.  FECA            DEC DL
0043DC34  |.  2C 20           SUB AL,20                       ; *
0043DC36  |.  81E2 37007496  AND EDX,96740037
0043DC3C  |.  C0D2 05         RCL DL,5
0043DC3F  |.  52              PUSH EDX
0043DC40  |.  F9              STC
0043DC41  |.  24 3C           AND AL,3C                       ; *
0043DC43  |.  8D6424 08       LEA ESP,[ESP+8]
0043DC47  |.  0F8F E90E0000   JG 0043EB36
```

#### 计算 EAX 值

```
0043EB36  />  \0FBDD4         BSR EDX,ESP
0043EB39  |.  66:81D2 A36F    ADC DX,6FA3
0043EB3E  |.  E8 3DF5FFFF     CALL 0043E080                   ; \\NOTEPAD.0043E080
```

#### Handler 入口 (0043E080)

```
0043E080  |$  8B55 00         MOV EDX,DWORD PTR SS:[EBP]      ; *
0043E083  |.  F6C6 E4         TEST DH,E4
0043E086  |.  83C5 04        ADD EBP,4                       ; *
0043E089  |.  60              PUSHAD
0043E08A  |.  60              PUSHAD
0043E08B  \.^ E9 47F3FFFF    JMP 0043D3D7
```

获取 `[ebp]` 数据，指针地址+4。

#### 数据移动至目标位置

```
0043D3D7  />  /891438         MOV DWORD PTR DS:[EDI+EAX],EDX  ; *
0043D3DA  |.  |FF3424         PUSH DWORD PTR SS:[ESP]
0043D3DD  |.  |68 7334B716    PUSH 16B73473
0043D3E2  |.  |8D6424 4C      LEA ESP,[ESP+4C]
0043D3E6  \.  |E9 D0120000    JMP 0043E6BB
```

移动到 `[edi+eax]` 位置。`ebpstack` 的值是那些呢，就是 `context` 和几个基础数据。handler 结束，回到 Dispatch 部分。

#### 返回 Dispatch 循环

```
0043E6BB  |>  >66:0FA5FA      SHLD DX,DI,CL                   ; Dispatch
0043E6BF  |.  |8A46 FF        MOV AL,BYTE PTR DS:[ESI-1]       ; *
0043E6C2  |.  |88FA            MOV DL,BH
0043E6C4  |.  |30D8            XOR AL,BL                       ; *
0043E6C6  |.  |FECA            DEC DL
0043E6C8  |.  |8D95 88EB93F7   LEA EDX,[EBP+F793EB88]
0043E6CE  |.  |F6D0            NOT AL                          ; *
0043E6D0  |.  |42              INC EDX
0043E6D1  |.  |F7D2            NOT EDX
0043E6D3  |.  |0F9FC2          SETG DL
0043E6D6  |.  |FEC8            DEC AL                          ; *
0043E6D8  |.  |D0CA            ROR DL,1
0043E6DA  |.  |C0C8 07        ROR AL,7                        ; *
0043E6DD  |.  |F5              CMC
0043E6DE  |.  |C0E6 06        SHL DH,6
0043E6E1  |.  |83EE 01        SUB ESI,1                       ; *
0043E6E4  |.  |D0D2            RCL DL,1
0043E6E6  |.  |66:0FB6D1       MOVZX DX,CL
0043E6EA  |.  |66:D1DA         RCR DX,1
0043E6ED  |.  |30C3            XOR BL,AL                       ; *
0043E6EF  |.  |60              PUSHAD
0043E6F0  |.  |66:0FBAE1 06    BT CX,6
0043E6F5  |.^|E9 32E9FFFF     JMP 0043D02C
```

#### 地址计算与跳转

```
0043D02C  |>  /C0DE 07        RCR DH,7
0043D02F  |.  |0FB6C0         MOVZX EAX,AL                  ; *
0043D032  |.  |F5             CMC
0043D033  |.  |E9 E7200000    JMP 0043F11F
0043F11F  |>  \38F5           CMP CH,DH
0043F121  |.  66:FFC2         INC DX
0043F124  |.  8B1485 DBE143   MOV EDX,DWORD PTR DS:[EAX*4+43E1DB]  ; *
0043F12B  |.  F9             STC
0043F12C  |.  84F4           TEST AH,DH
0043F12E  |.  60             PUSHAD
0043F12F  |.^ E9 CCEFFFFF    JMP 0043E100
0043E100  |>  /81C2 6B197FB6 ADD EDX,B67F196B              ; *
0043E106  |.  |C60424 46     MOV BYTE PTR SS:[ESP],46
0043E10A  |.  |895424 3C     MOV DWORD PTR SS:[ESP+3C],EDX   ; *
0043E10E  |.  |9C             PUSHFD
0043E10F  |.  |68 A42795AD    PUSH AD9527A4
0043E114  |.  |FF7424 0C      PUSH DWORD PTR SS:[ESP+0C]
0043E118  |.  |FF3424         PUSH DWORD PTR SS:[ESP]
0043E11B  |.  |FF7424 4C      PUSH DWORD PTR SS:[ESP+4C]      ; *
0043E11F  |.  |C2 5000       RETN 50                           ; *
```

我们要开始进入下一个 handler，以上的 dispatch 部分代码，在后面的文章中将不再出现。

### 2. 下一个 Handler
