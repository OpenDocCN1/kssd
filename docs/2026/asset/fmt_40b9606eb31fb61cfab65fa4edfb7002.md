
```asm
004002F3  |.  5E            pop     esi
004002F4  |.  C9            leave
004002F5  \.  C3            retn                                     ; 正常返回0x00400383, 溢出返回其它地址
```

### 3.) 在`004002EE`时的堆栈如下

```text
0012FC50   00000000   // saved_old_edi
0012FC54   008E0000   // saved_old_esi
0012FC58   00000000   // 局部变量,   从XXXXXXXX COPY ECX 个 DWORD 到这
0012FC5C   00000000   // 局部变量
0012FC60   00000000   // 局部变量
0012FC64   00000000   // 局部变量
0012FC68   00000000   // 局部变量
0012FC6C   00000000   // 局部变量
0012FC70   00000000   // 局部变量
0012FC74   00000000   // 局部变量
0012FC78   00000000   // 局部变量
0012FC7C   00000000   // 局部变量
0012FC80   00000000   // 局部变量
0012FC84  /0012FCAC   // saved_old_ebp
0012FC88  |00400383   // 返回地址
0012FC8C  |008E0000   // parameter1, 即动态分配的地址XXXXXXXX
0012FC90  |00000033   // parameter2, 文件长度
0012FC94  |00000111   // 调用程序的栈
```

### 4.) 要溢出成功

要溢出成功，可把返回地址覆盖成`004002F5`，这样，就会返回`004002F5`，再执行`004002F5`的`retn`，就回到动态分配的地址`XXXXXXXX`执行。

先解如下方程：

`ECX = (0x0000001A - mmmmmmmm) * mmmmmmmm - 0x0000009C`

`0x0000000D <= ECX <= 0x0000000F`

用如下 C 程序计算：

```c
unsigned int i, c;
for(i=0;i!=0xffffffff;i++) {
    c = (0x1A - i) * i - 0x9C;
    if (c>=0x0D && C<= 0x0F) {
        printf("mmmmmmmm=%08X ECX=%08X\n", i, c);
    }
}
```

结果：`ECX=0000000D`，`mmmmmmmm=????000D`

即`mmmmmmmm`高 16bit 可任意，低 16bit 为`000D`，结果总是`0000000D`。

### 5.) 再解如下方程

`HHHHHHHHLLLLLLLL = b7b6b5b4b3b2b1b0 * 0x78CC02A869948F1B` // (a)

`HHHHHHHHLLLLLLLL % 0x5BE6FF82A5164785 = MMMMMMMM????000D` // (b)

当`HHHHHHHHLLLLLLLL`为：

- 从`0x0000000000000000`到`0x5BE6FF82A5164784`时，模 = `HHHHHHHHLLLLLLLL`，即`LLLLLLLL = ????000D`
- 从`0x5BE6FF82A5164785`到`0xB7CDFF054A2C8F09`时，模 = `HHHHHHHHLLLLLLLL - 0x5BE6FF82A5164785`，即`LLLLLLLL = ????4792`
- 从`0xB7CDFF054A2C8F0A`到`0xFFFFFFFFFFFFFFFF`时，模 = `HHHHHHHHLLLLLLLL - 0xB7CDFF054A2C8F0A`，即`LLLLLLLL = ????8F17`

结果：`LLLLLLLL`的低 16bit 只能是`0x000D`、`0x4792`或`0x8F17`。

用如下 C 程序计算解方程 (a)：

```c
unsigned int i, l;
for(i=0;i!=0xffffffff;i++) {
    l = (i * 0x69948F1B) & 0xFFFF;
    if (l==0x0D || l==0x4792 || l== 0x8F17) {
        printf("b3b2b1b0=%08X,LLLLLLLL=%08X\n", i, l);
    }
}
```

结果：

```text
b3b2b1b0=????97F7,LLLLLLLL=????000D
b3b2b1b0=????65D6,LLLLLLLL=????4792
b3b2b1b0=????33B5,LLLLLLLL=????8F17
```

即`b0`、`b1`只能是如下三组中的数据之一：`{F7, 97}`、`{D6, 65}`、`{B5, 33}`。

### 5.) 将三组数据进行如下变换

```asm
004002CC  |> /8031 1C       /xor     byte ptr [ecx], 1C
004002CF  |. |8A11          |mov     dl, byte ptr [ecx]
004002D1  |. |3051 01       |xor     byte ptr [ecx+1], dl
004002D4  |. |41            |inc     ecx
004002D5  |. |41            |inc     ecx
004002D6  |. |4F            |dec     edi
004002D7  |.^\75 F3         \jnz     short 004002CC
```

再用 OD 看反汇编结果，如下：

```text
// F7, 97 ==> EB, 7C   // JMP short $+80
// D6, 65 ==> CA, AF   // retf xxxxxxxx
// B5, 33 ==> A9, 9A   // test eax, xxxxxxxx
```

`EB, 7C`将使 file 长度至少为 128 byte，现在只讨论决定`A9, 9A`，使文件最小。

### 6.) 设变换后的代码为

```asm
008E0000    A9 9Axxyyzz    test eax, zzyyxx9A
008E0005    90             nop
008E0006    EB 04          jmp  short $+06
```

先将`zz`定为`90`，这样由变换规则可确定`d0-d7`如下：

`{B5, 33, ??, ??, 8C, 00, F7, EF}`

只有 16bit 未知，编一小程序，循环 65536 次，使等式 (a)、(b) 成立，有一组解就行。

### 7.) 然后就是写代码，显示`OK!`，返回，还能正确执行

代码如下：

```asm
008E0000    A9 9A4C4C90     test    eax, 904C4C9A
008E0005    90              nop
008E0006    EB 04           jmp     short 008E000C
008E0008    4F 4B 21 00                              ; "OK!"
008E000C    E8 00000000     call    008E0011
008E0011    5D              pop     ebp
008E0012    8BC5            mov     eax, ebp
008E0014    83E8 09         sub     eax, 9
008E0017    6A 00           push    0
008E0019    50              push    eax
008E001A    50              push    eax
008E001B    6A 00           push    0
008E001D    FF15 4C024000   call    dword ptr [<&USER32.MessageBoxA>]                 ; USER32.MessageBoxA
008E0023    8BEC            mov     ebp, esp
008E0025    C745 00 9703400>mov     dword ptr [ebp], 400397
008E002C    83C5 1C         add     ebp, 1C
008E002F    C3              retn
008E0030    F5 02 40 00                             ; DWORD 004002F5
```

[\[招生\]科锐逆向工程师培训（2026年11月13日实地，远程教学同时开班, 第57期）](https://bbs.kanxue.com/thread-51839.htm)
