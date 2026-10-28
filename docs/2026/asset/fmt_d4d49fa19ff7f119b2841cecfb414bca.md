
# Patch 说明

## 第四个解码循环

```
0052F3D9    66:81E3 510A       and bx, 0A51
0052F3DE    81FA D8FAFFFF      cmp edx, -528
0052F3E4  ^ 0F85 ADFFFFFF      jnz chordpic.0052F397
```

解码循环 Patch 点 ★

```
0052F3EA    66:8BCF            mov cx, di
```

改成：

```
0052F3E4  - E9 92700200     jmp clide.0055647B
0052F3E9    90              nop
```

Patch代码：

```
0055647B  - 0F85 168FFDFF      jnz clide.0052F397
```

`//0052F3E4` 代码挪这里执行

```
00556481    C705 E4F35200 0F85ADFF     mov dword ptr ds:[52F3E4], FFAD850F
0055648B    66:C705 E8F35200 FFFF      mov word ptr ds:[52F3E8], 0FFFF
```

还原`0052F3e4`处修改的代码

```
00556494    C705 51F55200 E9566F02     mov dword ptr ds:[0052F5AA], 26F56E9
```

Patch `0052F5AA`处

```
0055649E  - E9 478FFDFF                 jmp clide.0052F3EA
```

返回原处继续

## 第五个点：获取壳代码重定位基址

```
0052F5A4    8B5E 08                     mov ebx, dword ptr ds:[esi+8]
0052F5A7    03DF                        add ebx, edi
0052F5A9    53                          push ebx
0052F5AA    68 00800000                 push 8000
```

此时，edi中存放着基址 Patch 点 ★

```
0052F5AF    6A 00                       push 0
0052F5B1    56                          push esi
```

改成:

```
0052F5AA  - E9 F46E0200     jmp clide.005564A3
```

Patch代码：

```
005564A3    C705 AAF55200 68008000      mov dword ptr ds:[52F5AA], 800068
```

还原`0052F5AA`处修改的代码

```
005564AD    893D 00655500               mov dword ptr ds:[556700], edi
```

找个位置存放壳的基址，我这里选择`556700`

```
005564B3    60                          pushad
005564B4    9C                          pushfd
005564B5    A1 00655500                 mov eax, dword ptr ds:[556700]
005564BA    C780 F3100300 E9DA5350      mov dword ptr ds:[eax+310F3], 5053DAE9
```

每台机器由于生成的壳的基址不同，所以像这样`jmp xxxx`修改，每台机器有可能都不同，需要个别对待

```
005564C4    C680 F7100300 FF            mov byte ptr ds:[eax+310F7], 0FF
```

Patch `010510F3` 处，当执行到这里时，程序就已经完全解码了

```
005564CB    9D                          popfd
005564CC    61                          popad
005564CD  - E9 D890FDFF                 jmp clide.0052F5AA
```

## 第六个点：程序已经完全解码

```
010510EB    F3:A4           rep movs byte ptr es:[edi], b>
010510ED    8B85 75294400   mov eax, dword ptr ss:[ebp+44>
010510F3    68 00800000     push 8000
```

承上启下 Patch 点 ★
我选择在这里patch，用于在得到基址后，而在壳代码完全解压前的一个patch点，承上启下。

```
010510F8    6A 00           push 0
010510FA    50              push eax
```

改成：

```
010510F3  - E9 DA5350FF     jmp clide.005564D2
```

Patch代码：

```
005564D2    60                        pushad                                  ; 保存现场
005564D3    9C                        pushfd
005564D4    A1 00675500               mov eax, dword ptr ds:[556700]          ; 取得基址
005564D9    C780 F3100300 68008000    mov dword ptr ds:[eax+310F3], 800068    ; 还原Patch点代码
005564E3    C680 F7100300 00          mov byte ptr ds:[eax+310F7], 0
005564EA    C780 C1150300 68146555    mov dword ptr ds:[eax+315C1], 55651468  ; Patch 010515C1处的代码
005564F4    C680 C5150300 00          mov byte ptr ds:[eax+315C5], 0
005564FB    C680 C6150300 C3          mov byte ptr ds:[eax+315C6], 0C3
00556502    05 F3100300               add eax, 310F3                          ; 取得Patch点地址
00556507    A3 0F655500               mov dword ptr ds:[55650F], eax          ; 修改下面Push的地址
0055650C    9D                        popfd                                   ; 恢复现场
0055650D    61                        popad
0055650E    68 00000000               push 0                                  ; Push 返回地址，由00556507处设定
00556513    C3                        retn
```

## 第七个点

```
00F415C1    61              popad
```

patch点，此时壳已经完全解码了

```
00F415C2    75 08           jnz short 00F415CC
00F415C4    B8 01000000     mov eax, 1
00F415C9    C2 0C00         retn 0C
00F415CC    68 00000000     push 0
00F415D1    C3              retn
```

改成:

```
00F415C1    68 14655500     push 556514
00F415C6    C3              retn
```

Patch代码：

```
00556514    60                           pushad                                  ; 保存现场
00556515    9C                           pushfd
00556516    A1 00675500                  mov eax, dword ptr ds:[556700]          ; 取得基址
0055651B    C780 C1150300 617508B8       mov dword ptr ds:[eax+315C1], B8087561  ; 还原Patch点代码
00556525    C680 C5150300 01             mov byte ptr ds:[eax+315C5], 1
0055652C    C680 C6150300 00             mov byte ptr ds:[eax+315C6], 0
00556533    C680 69860100 01             mov byte ptr ds:[eax+18669], 1          ; 修改01038668处的Push 4为Push 1，使内存映
像可写
0055653A    C780 7A860100 68646555       mov dword ptr ds:[eax+1867A], 55656468  ; Patch 0103867A处的代码
00556544    C680 7E860100 00             mov byte ptr ds:[eax+1867E], 0
0055654B    C680 7F860100 C3             mov byte ptr ds:[eax+1867F], 0C3
00556552    05 C1150300                  add eax, 315C1                          ; 取得Patch点地址
00556557    A3 5F655500                  mov dword ptr ds:[55655F], eax          ; 修改下面Push的地址
0055655C    9D                           popfd                                   ; 恢复现场
0055655D    61                           popad
0055655E    68 00000000                  push 0                                  ; Push 返回地址，由00556557处设定
00556563    C3                           retn
```

## 第八个点

```
01038668    6A 01              push 1
0103866A    A1 14B40401        mov eax, dword ptr ds:[104B414]
0103866F    50                 push eax
01038670    A1 E4970401        mov eax, dword ptr ds:[10497E4]
01038675    8B40 08            mov eax, dword ptr ds:[eax+8]
01038678    FFD0               call eax
0103867A    8BD8               mov ebx, eax                         ; 返回到这里
```

patch点，在执行完 `MapViewOfFileEx`后，恢复这前的push 4

改成:

```
01038668    6A 04              push 4
```

Patch代码：

```
00556564    C780 6A510800 0F851E00    mov dword ptr ds:[eax+8516A], 1E850F       ; 还原0052F16A 在内存映像中相应位置的代码,
即是之前直接修改文件中偏移:0008516A的代码
0055656E    66:C780 6E510800 0000     mov word ptr ds:[eax+8516E], 0
00556577    C680 99030000 00          mov byte ptr ds:[eax+399], 0               ; 还原内存映像中.adata段的RSize为0，这两句
是为了逃过CRC校验
0055657E    60                        pushad
0055657F    9C                        pushfd
00556580    A1 00675500               mov eax, dword ptr ds:[556700]             ; 取得基址
00556585    C680 69860100 04          mov byte ptr ds:[eax+18669], 4             ; 还原01038668处的代码
0055658C    C780 7A860100 8BD850E8    mov dword ptr ds:[eax+1867A], E850D88B     ; 还原0103867A的代码
00556596    66:C780 7E860100 4A01     mov word ptr ds:[eax+1867E], 14A
0055659F    C780 B9CA0100 E9069B62    mov dword ptr ds:[eax+1CAB9], 629B06E9     ; Patch 0103CAB9，即Pre-dip前的位置
```

每台机器由于生成的壳的基址不同，所以像这样`jmp xxxx`修改，每台机器有可能都不同，需要个别对待

```
005565A9    66:C780 BDCA0100 FF90     mov word ptr ds:[eax+1CABD], 90FF
005565B2    05 7A860100               add eax, 1867A                             ; 取得Patch点地址
005565B7    A3 BF655500               mov dword ptr ds:[5565BF], eax             ; 修改下面Push的地址
005565BC    9D                        popfd                                      ; 恢复现场
005565BD    61                        popad
005565BE    68 00000000               push 0                                     ; Push 返回地址，由005565B7处设定
005565C3    C3                        retn
```

## 第九个点：Pre-dip前的位置
