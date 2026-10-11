
### 第二个解码循环

```
0052F263  50            push eax
0052F264  68 0FBCAC15   push 15ACBC0F
0052F269  0FB7CF        movzx ecx, di
0052F26C  59            pop ecx
0052F26D  5B            pop ebx
0052F26E  E9 23000000   jmp chordpic.0052F296
```

//解码循环 Patch 点 ★

改成:

```
0052F26E - E9 B5710200   jmp clide.00556428
```

**Patch 代码：**

```
00556428  C705 6FF25200 2300> mov dword ptr ds:[52F26F], 23
```

//还原 `0052F26E` 处修改的代码

```
00556432  C705 36F35200 E90F> mov dword ptr ds:[52F336], 2710FE9
0055643C  66:C705 3AF35200 0> mov word ptr ds:[52F33A], 9000
```

////Patch `0052F336` 处跳转

```
00556445 - E9 248EFDFF   jmp clide.0052F26E
```

// 返回原处继续

### 第三个解码循环

```
0052F311  BE 530C100D   mov esi, 0D100C53
0052F316  81C6 BCDE271F add esi, 1F27DEBC
0052F31C  81EA 9A892B02 sub edx, 22B899A
0052F322  66:8BCB       mov cx, bx
0052F325  81C2 96892B02 add edx, 22B8996
0052F32B  66:81E7 9F0D  and di, 0D9F
0052F330  81FA 34FAFFFF cmp edx, -5CC
0052F336  ^ 0F85 8DFFFFFF jnz chordpic.0052F2C9
```

//解码循环 Patch 点 ★

```
0052F33C  53            push ebx
```

改成：

```
//0052F336 - E9 0F710200   jmp clide.0055644A
//0052F33B    90            nop
```

**Patch 代码：**

```
0055644A - 0F85 798EFDFF   jnz clide.0052F2C9
```

//00052F336 代码挪这里执行

```
00556450  C705 36F35200 0F85> mov dword ptr ds:[52F336], FF8D850F
0055645A  66:C705 3AF35200 F> mov word ptr ds:[52F33A], 0FFFF
```

//还原 `0052F336` 处修改的代码

```
00556432  C705 36F35200 E90F> mov dword ptr ds:[52F3e4], 27092E9
0055643C  66:C705 3AF35200 0> mov word ptr ds:[52F3e8], 9000
```

//Patch `0052F3e4` 处跳转

```
00556476 - E9 C18EFDFF   jmp clide.0052F33C
```

// 返回原处继续

### 第四个解码循环:

```
0052F3D9  66:81E3 510A  and bx, 0A51
0052F3DE  81FA D8FAFFFF  cmp edx, -528
0052F3E4  ^ 0F85 ADFFFFFF jnz chordpic.0052F397
```

//解码循环 Patch 点 ★

```
0052F3EA  66:8BCF       mov cx, di
```

//改成：

```
0052F3E4 - E9 92700200   jmp clide.0055647B
0052F3E9    90            nop
```

**Patch 代码：**

```
0055647B - 0F85 168FFDFF   jnz clide.0052F397
```

//0052F3E4 代码挪这里执行

```
00556481  C705 E4F35200 0F85ADFF   mov dword ptr ds:[52F3E4], FFAD850F
0055648B  66:C705 E8F35200 FFFF     mov word ptr ds:[52F3E8], 0FFFF
```

//还原 `0052F3e4` 处修改的代码

```
00556494  C705 51F55200 E9566F02   mov dword ptr ds:[0052F5AA], 26F56E9
```

//Patch  `0052F5AA` 处

```
0055649E - E9 478FFDFF   jmp clide.0052F3EA
```

// 返回原处继续

### 第五个点：获取壳代码重定位基址

```
0052F5A4  8B5E 08           mov ebx, dword ptr ds:[esi+8]
0052F5A7  03DF              add ebx, edi
0052F5A9  53                push ebx
0052F5AA  68 00800000       push 8000
```

//此时，`edi` 中存放着基址 Patch 点 ★

```
0052F5AF  6A 00             push 0
0052F5B1  56                push esi
```

改成:

```
0052F5AA - E9 F46E0200   jmp clide.005564A3
```

**Patch 代码：**

```
005564A3  C705 AAF55200 68008000   mov dword ptr ds:[52F5AA], 800068
```

//还原 `0052F5AA` 处修改的代码

```
005564AD  893D 00655500   mov dword ptr ds:[556700], edi
```

//找个位置存放壳的基址，我这里选择 `556700`

```
005564B3  60                pushad
005564B4  9C                pushfd
005564B5  A1 00655500       mov eax, dword ptr ds:[556700]
005564BA  C780 F3100300 E9DA5350   mov dword ptr ds:[eax+310F3], 5053DAE9
```

//; 每台机器由于生成的壳的基址不同，所以像这样 `jmp xxxx` 修改，每台机器有可能都不同，需要个别对待

```
005564C4  C680 F7100300 FF  mov byte ptr ds:[eax+310F7], 0FF
```

//Patch `010510F3` 处，当执行到这里时，程序就已经完全解码了

```
005564CB  9D                popfd
005564CC  61                popad
005564CD - E9 D890FDFF      jmp clide.0052F5AA
```

### 第六个点：程序已经完全解码

```
010510EB  F3:A4             rep movs byte ptr es:[edi], b>
010510ED  8B85 75294400     mov eax, dword ptr ss:[ebp+44>
010510F3  68 00800000       push 8000
```

//承上启下 Patch 点 ★

//我选择在这里 patch，用于在得到基址后，而在壳代码完全解压前的一个 patch 点，承上启下。

```
010510F8  6A 00             push 0
010510FA  50               push eax
```

改成：

```
010510F3 - E9 DA5350FF   jmp clide.005564D2
```

**Patch 代码：**

```
005564D2  60                pushad                            ; 保存现场
005564D3  9C                pushfd
005564D4  A1 00675500       mov eax, dword ptr ds:[556700]     ; 取得基址
005564D9  C780 F3100300 68008000   mov dword ptr ds:[eax+310F3], 800068    ; 还原 Patch 点代码
005564E3  C680 F7100300 00   mov byte ptr ds:[eax+310F7], 0
005564EA  C780 C1150300 68146555  mov dword ptr ds:[eax+315C1], 55651468  ; Patch 010515C1 处的代码
005564F4  C680 C5150300 00   mov byte ptr ds:[eax+315C5], 0
005564FB  C680 C6150300 C3   mov byte ptr ds:[eax+315C6], 0C3
00556502  05 F3100300         add eax, 310F3                    ; 取得 Patch 点地址
00556507  A3 0F655500         mov dword ptr ds:[55650F], eax    ; 修改下面 Push 的地址
0055650C  9D                  popfd                             ; 恢复现场
0055650D  61                  popad
0055650E  68 00000000         push 0                            ; Push 返回地址，由 00556507 处设定
00556513  C3                  retn
```

### 第七个点:

```
00F415C1  61                popad
```

//patch 点，此时壳已经完全解码了

```
00F415C2  75 08             jnz short 00F415CC
00F415C4  B8 01000000       mov eax, 1
00F415C9  C2 0C00           retn 0C
00F415CC  68 00000000       push 0
00F415D1  C3                retn
```

改成:

```
00F415C1  68 14655500       push 556514
00F415C6  C3                retn
```

**Patch 代码：**

```
00556514  60                  pushad                              ; 保存现场
00556515  9C                  pushfd
00556516  A1 00675500         mov eax, dword ptr ds:[556700]       ; 取得基址
0055651B  C780 C1150300 617508B8    mov dword ptr ds:[eax+315C1], B8087561  ; 还原 Patch 点代码
00556525  C680 C5150300 01      mov byte ptr ds:[eax+315C5], 1
0055652C  C680 C6150300 00      mov byte ptr ds:[eax+315C6], 0
00556533  C680 69860100 01     mov byte ptr ds:[eax+18669], 1       ; 修改 01038668 处的 Push 4 为 Push 1，使内存映像可写
0055653A  C780 7A860100 68646555 mov dword ptr ds:[eax+1867A], 55656468  ; Patch 0103867A 处的代码
00556544  C680 7E860100 00     mov byte ptr ds:[eax+1867E], 0
0055654B  C680 7F860100 C3     mov byte ptr ds:[eax+1867F], 0C3
00556552  05 C1150300           add eax, 315C1                      ; 取得 Patch 点地址
00556557  A3 5F655500           mov dword ptr ds:[55655F], eax      ; 修改下面 Push 的地址
0055655C  9D                    popfd                               ; 恢复现场
0055655D  61                    popad
0055655E  68 00000000           push 0                              ; Push 返回地址，由 00556557 处设定
00556563  C3                    retn
```

### 第八个点：

```
01038668  6A 01             push 1
0103866A  A1 14B40401       mov eax, dword ptr ds:[104B414]
0103866F  50                push eax
01038670  A1 E4970401       mov eax, dword ptr ds:[10497E4]
01038675  8B40 08           mov eax, dword ptr ds:[eax+8]
01038678  FFD0              call eax
0103867A  8BD8              mov ebx, eax                  ; 返回到这里
```

//patch 点，在执行完 `MapViewOfFileEx` 后，恢复这前的 `push 4`

改成:

```
01038668  6A 04             push 4
```

**Patch 代码：**
