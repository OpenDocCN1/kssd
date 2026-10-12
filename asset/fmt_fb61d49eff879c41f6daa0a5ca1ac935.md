
此时, `ebx` 是运算得到的函数地址, 而 `edx` 则决定了 `MOV EXX [004XXXXX]` 中的 `EXX` 是 `EAX` 还是 `EBX` 或者 `EDX`、`EDI`、`ESI`、`EBP`。

从上到下的顺序是按照寄存器的顺序生成的：

*   `EAX` `7FFDF000`
*   `ECX` `0012FA04`
*   `EDX` `00009164`
*   `EBX` `00000000`
*   `ESP` `0012F9E8`
*   `EBP` `0045991A` `FractalP.0045991A`
*   `ESI` `00B300D1`
*   `EDI` `0012FD6C`

```assembly
0045B9AC    5A              pop     edx
0045B9AD    8D9402 0A000000 lea     edx, dword ptr [edx+eax+A]
0045B9B4    FFE2            jmp     edx
0045B9B6    895C24 1C       mov     dword ptr [esp+1C], ebx
0045B9BA    EB 27           jmp     short 0045B9E3
0045B9BC    895C24 18       mov     dword ptr [esp+18], ebx
0045B9C0    EB 21           jmp     short 0045B9E3
0045B9C2    895C24 14       mov     dword ptr [esp+14], ebx
0045B9C6    EB 1B           jmp     short 0045B9E3
0045B9C8    895C24 10       mov     dword ptr [esp+10], ebx
0045B9CC    EB 15           jmp     short 0045B9E3
0045B9CE    895C24 0C       mov     dword ptr [esp+C], ebx
0045B9D2    EB 0F           jmp     short 0045B9E3
0045B9D4    895C24 08       mov     dword ptr [esp+8], ebx
0045B9D8    EB 09           jmp     short 0045B9E3
0045B9DA    895C24 04       mov     dword ptr [esp+4], ebx
0045B9DE    EB 03           jmp     short 0045B9E3
0045B9E0    891C24          mov     dword ptr [esp], ebx
0045B9E3    61              popad
0045B9E4    FF4424 04       inc     dword ptr [esp+4]
0045B9E8    9D              popfd
```

现在可以在 `0045B9B4 FFE2 jmp edx` 下断点然后根据 `edx` 和 `ebx` 的值来修复了, 或者改成 `jmp 008D0010` 到代码处, 自动的修复:

例子代码如下：

```assembly
008D0000    A1 FC018D00     mov     eax, dword ptr [8D01FC]
```

这里的 `8D01FC` 处的地址为存放所有存在这种变形 `call` 的地址的第一个地址-4

```assembly
008D0005    83C0 04         add     eax, 4
008D0008    A3 FC018D00     mov     dword ptr [8D01FC], eax
008D000D    FF20            jmp     dword ptr [eax]
008D000F    90              nop
008D0010    B9 00500310     mov     ecx, 10035000       ; 这里 10035000 是 IAT 起始地址，不是前面说的程序的
008D0015    8B01            mov     eax, dword ptr [ecx]
008D0017    3BC3            cmp     eax, ebx
008D0019    90              nop
008D001A    74 0C           je      short 008D0028
008D001C    83C1 04         add     ecx, 4
008D001F    81F9 B8530310   cmp     ecx, 100353B8       ; 这里 100353B8 是 IAT 结束地址+4，不是前面说的程序的
008D0025    75 EE           jnz     short 008D0015
008D0027    90              nop                           ; 这里下断，防止壳产生 IAT 里没有的函数，不过不会用到。
```

下面开始根据 `edx` 的值判断 `mov EXX` 的修复，其中的值是另外程序的，不是前面的地址，可以根据顺序改过来。

```assembly
008D0028    81FA 9C250B10   cmp     edx, 100B259C
008D002E    75 0F           jnz     short 008D003F
008D0030    A1 FC018D00     mov     eax, dword ptr [8D01FC]
008D0035    8B00            mov     eax, dword ptr [eax]
008D0037    C600 A1         mov     byte ptr [eax], 0A1
008D003A    8948 01         mov     dword ptr [eax+1], ecx
008D003D    EB C1           jmp     short 008D0000

008D003F    81FA A2250B10   cmp     edx, 100B25A2
008D0045    75 11           jnz     short 008D0058
008D0047    A1 FC018D00     mov     eax, dword ptr [8D01FC]
008D004C    8B00            mov     eax, dword ptr [eax]
008D004E    66:C700 8B0D    mov     word ptr [eax], 0D8B
008D0053    8948 02         mov     dword ptr [eax+2], ecx
008D0056    EB A8           jmp     short 008D0000

008D0058    81FA A8250B10   cmp     edx, 100B25A8
008D005E    75 11           jnz     short 008D0071
008D0060    A1 FC018D00     mov     eax, dword ptr [8D01FC]
008D0065    8B00            mov     eax, dword ptr [eax]
008D0067    66:C700 8B15    mov     word ptr [eax], 158B
008D006C    8948 02         mov     dword ptr [eax+2], ecx
008D006F    EB 8F           jmp     short 008D0000

008D0071    81FA AE250B10   cmp     edx, 100B25AE
008D0077    75 11           jnz     short 008D008A
008D0079    A1 FC018D00     mov     eax, dword ptr [8D01FC]
008D007E    8B00            mov     eax, dword ptr [eax]
008D0080    66:C700 8B1D    mov     word ptr [eax], 1D8B
008D0085    8948 02         mov     dword ptr [eax+2], ecx
008D0088    EB E5           jmp     short 008D006F

008D008A    81FA B4250B10   cmp     edx, 100B25B4
008D0090    75 11           jnz     short 008D00A3
008D0092    A1 FC018D00     mov     eax, dword ptr [8D01FC]
008D0097    8B00            mov     eax, dword ptr [eax]
008D0099    66:C700 8B25    mov     word ptr [eax], 258B
008D009E    8948 02         mov     dword ptr [eax+2], ecx
008D00A1    EB E5           jmp     short 008D0088

008D00A3    81FA BA250B10   cmp     edx, 100B25BA
008D00A9    75 11           jnz     short 008D00BC
008D00AB    A1 FC018D00     mov     eax, dword ptr [8D01FC]
008D00B0    8B00            mov     eax, dword ptr [eax]
008D00B2    66:C700 8B2D    mov     word ptr [eax], 2D8B
008D00B7    8948 02         mov     dword ptr [eax+2], ecx
008D00BA    EB E5           jmp     short 008D00A1

008D00BC    81FA C0250B10   cmp     edx, 100B25C0
008D00C2    75 11           jnz     short 008D00D5
008D00C4    A1 FC018D00     mov     eax, dword ptr [8D01FC]
008D00C9    8B00            mov     eax, dword ptr [eax]
008D00CB    66:C700 8B35    mov     word ptr [eax], 358B
008D00D0    8948 02         mov     dword ptr [eax+2], ecx
008D00D3    EB E5           jmp     short 008D00BA

008D00D5    81FA C6250B10   cmp     edx, 100B25C6
008D00DB    75 11           jnz     short 008D00EE
008D00DD    A1 FC018D00     mov     eax, dword ptr [8D01FC]
008D00E2    8B00            mov     eax, dword ptr [eax]
008D00E4    66:C700 8B3D    mov     word ptr [eax], 3D8B
008D00E9    8948 02         mov     dword ptr [eax+2], ecx
008D00EC    EB E5           jmp     short 008D00D3

008D00EE    EB FE           jmp     short 008D00EE      ; 如果没有成功会跳到这里死掉。
```

运行后，程序可能暂时运行无相应，另外这段代码里没有判断是否读取完所有的变形的 `call` 的地址，所以直接用到最后会提示 `0000000` 无法读取。

### 第三种

`call` 到壳里的某个地址，其地址和第二种的接近，但是不可执行，应该是中间 `CCCCCCCCC` 代码的变形，不用理它。

如果程序过期, 无法运行并脱壳, 可以在这里跳开:

```assembly
0045BB9B    F3:AB           rep     stos dword ptr es:[edi]
0045BB9D    53              push    ebx
0045BB9E    57              push    edi
0045BB9F    56              push    esi
0045BBA0    FF73 08         push    dword ptr [ebx+8]
0045BBA3    56              push    esi
0045BBA4    50              push    eax
0045BBA5    FFD2            call    edx                 ; 进入这里
0045BBA7    83C4 0C         add     esp, 0C
0045BBAA    8985 E9220000   mov     dword ptr [ebp+22E9], eax
0045BBB0    5E              pop     esi
0045BBB1    5F              pop     edi
0045BBB2    5B              pop     ebx

10001934    55              push    ebp                 ; FractalP.0045991A
10001935    8BEC            mov     ebp, esp
10001937    83EC 10         sub     esp, 10
1000193A    8B45 10         mov     eax, dword ptr [ebp+10]
1000193D    FF05 E88B0410   inc     dword ptr [10048BE8]
10001943    833D E88B0410 0 cmp     dword ptr [10048BE8], 2
1000194A    53              push    ebx
1000194B    56              push    esi
1000194C    57              push    edi
1000194D    A3 90B50410     mov     dword ptr [1004B590], eax
10001952    7C 56           jl      short 100019AA
```

上面 `NOP` 掉, 没有任何提示, 程序就运行了, 壳的注册功能已经被废掉了

```assembly
10001954    E8 286A0200     call    10028381
10001959    8BF0            mov     esi, eax
```

比较乱套，凑合着看吧。辛苦了。

![](img/45777b5dbd6b901752ab7df42fce6892.jpg) ![](img/45777b5dbd6b901752ab7df42fce6892.jpg)

`4dd466d0e1e0cc123afbe504da2462ca`
