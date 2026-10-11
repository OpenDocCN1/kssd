
# Micro Bridge

> 原文：[`www.pediy.com/kssd/pediy03/forum529.htm`](https://www.pediy.com/kssd/pediy03/forum529.htm)

`Micro Bridge`的免光盘注册程序

`http://bridgeworld.myetang.com/soft/lang.zip` [111K]

`Micro Bridge` 的 Key Code 防护措施相当变态，在移动软件目录或重装该软件后所申请的(10$)Key Code 失效。

这个程序的 `KeyCode` 似乎跟机器无关，我没有那个软件，无法知道它是否还检查了 `KeyCode` 的低位。下面的程序生成的 `KeyCode` 可以通过它的 `lang.exe` 的检查：

```c
#include <windows.h>
#include <stdio.h>

void main(void)
{
    long KeyCode;
    long Shift[] = { 0, 4, 8, 16, 20, 24, 28 };
    long High;
    int i;

    KeyCode = GetTickCount() & 0x03FFFFFFL;
    High = 0;
    for(i = 0; i <= 6; i++)
    {
        High ^= (KeyCode >> Shift[i]) & 0x0F;
    }
    High += 0x10;
    KeyCode |= (High << 0x1A);
    printf("Your key code is: %lu\n", KeyCode);
}
```

这个软件的主程序 `mb9.exe` 启动时从 `mb.ini` 中读入 `KeyCode`，然后判断。显然 `KeyCode` 和 `Windows` 目录的属性有关。

判断完了之后设置标志 `DS:[4D08]` 和 `DS:[4CF1]`，这两个标志是等价的，为 1 则可免 CD 运行。`KeyCode` 的算法并不是很复杂。

```asm
352F:1F97  C606084D00           MOV       BYTE PTR [4D08],00    //初始化标志
352F:1F9C  8B46B8               MOV       AX,[BP-48]            //KeyCode 的低位
352F:1F9F  0B46BA               OR        AX,[BP-46]            //KeyCode 的高位
352F:1FA2  7503                 JNZ       1FA7
352F:1FA4  E9D002               JMP       2277
352F:1FA7  C746C07930           MOV       WORD PTR [BP-40],3079
352F:1FAC  C746C22A0F           MOV       WORD PTR [BP-3E],0F2A
352F:1FB1  8DBEA4FE             LEA       DI,[BP+FEA4]
352F:1FB5  16                   PUSH      SS
352F:1FB6  57                   PUSH      DI
352F:1FB7  BFB82F               MOV       DI,2FB8
352F:1FBA  1E                   PUSH      DS
352F:1FBB  57                   PUSH      DI
352F:1FBC  9A0F1A471A           CALL      1A47:1A0F
352F:1FC1  BFD60A               MOV       DI,0AD6
352F:1FC4  0E                   PUSH      CS
352F:1FC5  57                   PUSH      DI
352F:1FC6  9A8E1A471A           CALL      1A47:1A8E
352F:1FCB  6A11                 PUSH      11
352F:1FCD  8D7ED0               LEA       DI,[BP-30]
352F:1FD0  16                   PUSH      SS
352F:1FD1  57                   PUSH      DI
352F:1FD2  9ABA0ACF42           CALL      42CF:0ABA
352F:1FD7  8B46E6               MOV       AX,[BP-1A]
352F:1FDA  8B56E8               MOV       DX,[BP-18]
352F:1FDD  8946A8               MOV       [BP-58],AX
352F:1FE0  8956AA               MOV       [BP-56],DX
352F:1FE3  8D7ED0               LEA       DI,[BP-30]
352F:1FE6  16                   PUSH      SS
352F:1FE7  57                   PUSH      DI
352F:1FE8  9A270BCF42           CALL      42CF:0B27
352F:1FED  FF36FE4C             PUSH      WORD PTR [4CFE]
352F:1FF1  FF36FC4C             PUSH      WORD PTR [4CFC]
352F:1FF5  68FF00               PUSH      00FF
352F:1FF8  9A55016701           CALL      KERNEL!GETWINDOWSDIRECTORY
352F:1FFD  8DBEA4FE             LEA       DI,[BP+FEA4]
352F:2001  16                   PUSH      SS
352F:2002  57                   PUSH      DI
352F:2003  FF36FE4C             PUSH      WORD PTR [4CFE]
352F:2007  FF36FC4C             PUSH      WORD PTR [4CFC]
352F:200B  9A150ECF42           CALL      42CF:0E15
352F:2010  BFD80A               MOV       DI,0AD8
352F:2013  0E                   PUSH      CS
352F:2014  57                   PUSH      DI
352F:2015  9A8E1A471A           CALL      1A47:1A8E
352F:201A  6A11                 PUSH      11
352F:201C  8D7ED0               LEA       DI,[BP-30]
352F:201F  16                   PUSH      SS
352F:2020  57                   PUSH      DI
352F:2021  9ABA0ACF42           CALL      42CF:0ABA
352F:2026  8B46E6               MOV       AX,[BP-1A]
352F:2029  8B56E8               MOV       DX,[BP-18]
352F:202C  8946A4               MOV       [BP-5C],AX
352F:202F  8956A6               MOV       [BP-5A],DX
352F:2032  8D7ED0               LEA       DI,[BP-30]
352F:2035  16                   PUSH      SS
352F:2036  57                   PUSH      DI
352F:2037  9A270BCF42           CALL      42CF:0B27
352F:203C  8B46A8               MOV       AX,[BP-58]
352F:203F  8B56AA               MOV       DX,[BP-56]
352F:2042  2B46A4               SUB       AX,[BP-5C]
352F:2045  1B56A6               SBB       DX,[BP-5A]
352F:2048  8BC8                 MOV       CX,AX
352F:204A  8BDA                 MOV       BX,DX
352F:204C  8B46A8               MOV       AX,[BP-58]
352F:204F  8B56AA               MOV       DX,[BP-56]
352F:2052  2B46A4               SUB       AX,[BP-5C]
352F:2055  1B56A6               SBB       DX,[BP-5A]
352F:2058  9AE618471A           CALL      1A47:18E6
352F:205D  B90400               MOV       CX,0004
352F:2060  31DB                 XOR       BX,BX
352F:2062  9AC919471A           CALL      1A47:19C9
352F:2067  3346C0               XOR       AX,[BP-40]
352F:206A  3356C2               XOR       DX,[BP-3E]
352F:206D  8946C0               MOV       [BP-40],AX
352F:2070  8956C2               MOV       [BP-3E],DX
352F:2073  8B46C0               MOV       AX,[BP-40]
352F:2076  8B56C2               MOV       DX,[BP-3E]
352F:2079  25FFFF               AND       AX,FFFF
352F:207C  81E2FF03             AND       DX,03FF
352F:2080  8946C0               MOV       [BP-40],AX
352F:2083  8956C2               MOV       [BP-3E],DX
352F:2086  8B46C0               MOV       AX,[BP-40]
352F:2089  8B56C2               MOV       DX,[BP-3E]
352F:208C  25FF0F               AND       AX,0FFF
352F:208F  81E20000             AND       DX,0000
352F:2093  8946B4               MOV       [BP-4C],AX
352F:2096  8956B6               MOV       [BP-4A],DX
352F:2099  8B46C0               MOV       AX,[BP-40]
352F:209C  8B56C2               MOV       DX,[BP-3E]
352F:209F  B90C00               MOV       CX,000C
352F:20A2  31DB                 XOR       BX,BX
352F:20A4  9AC919471A           CALL      1A47:19C9
352F:20A9  25FF0F               AND       AX,0FFF
352F:20AC  81E20000             AND       DX,0000
352F:20B0  8946B0               MOV       [BP-50],AX
352F:20B3  8956B2               MOV       [BP-4E],DX
352F:20B6  8B46B4               MOV       AX,[BP-4C]
352F:20B9  8B56B6               MOV       DX,[BP-4A]
352F:20BC  8B4EB0               MOV       CX,[BP-50]
352F:20BF  8B5EB2               MOV       BX,[BP-4E]
352F:20C2  9AE618471A           CALL      1A47:18E6
352F:20C7  B90600               MOV       CX,0006
352F:20CA  31DB                 XOR       BX,BX
352F:20CC  9AEC19471A           CALL      1A47:19EC
352F:20D1  8946AC               MOV       [BP-54],AX
352F:20D4  8956AE               MOV       [BP-52],DX
352F:20D7  8B46C0               MOV       AX,[BP-40]
352F:20DA  8B56C2               MOV       DX,[BP-3E]
352F:20DD  B90200               MOV       CX,0002
352F:20E0  31DB                 XOR       BX,BX
352F:20E2  9AC919471A           CALL      1A47:19C9
352F:20E7  25FF0F               AND       AX,0FFF
352F:20EA  81E20000             AND       DX,0000
352F:20EE  050100               ADD       AX,0001
352F:20F1  83D200               ADC       DX,00
352F:20F4  8946B4               MOV       [BP-4C],AX
352F:20F7  8956B6               MOV       [BP-4A],DX
352F:20FA  8B46C0               MOV       AX,[BP-40]
352F:20FD  8B56C2               MOV       DX,[BP-3E]
352F:2100  B90E00               MOV       CX,000E
352F:2103  31DB                 XOR       BX,BX
352F:2105  9AC919471A           CALL      1A47:19C9
352F:210A  25FF0F               AND       AX,0FFF
352F:210D  81E20000             AND       DX,0000
```



```
352F:2111  050100                   ADD       AX,0001   

352F:2114  83D200                   ADC       DX,00   

352F:2117  8946B0                   MOV       [BP-50],AX   

352F:211A  8956B2                   MOV       [BP-4E],DX   

352F:211D  8B46B4                   MOV       AX,[BP-4C]   

352F:2120  8B56B6                   MOV       DX,[BP-4A]   

352F:2123  8B4EB0                   MOV       CX,[BP-50]   

352F:2126  8B5EB2                   MOV       BX,[BP-4E]   

352F:2129  9AE618471A               CALL      1A47:18E6   

352F:212E  3346AC                   XOR       AX,[BP-54]   

352F:2131  3356AE                   XOR       DX,[BP-52]   

352F:2134  8946BC                   MOV       [BP-44],AX   

352F:2137  8956BE                   MOV       [BP-42],DX   

352F:213A  8B46BC                   MOV       AX,[BP-44]   

352F:213D  8B56BE                   MOV       DX,[BP-42]   

352F:2140  25FFFF                   AND       AX,FFFF   

352F:2143  81E2FF03                 AND       DX,03FF   

352F:2147  8946BC                   MOV       [BP-44],AX   

352F:214A  8956BE                   MOV       [BP-42],DX   

352F:214D  31C0                     XOR       AX,AX   

352F:214F  8946C4                   MOV       [BP-3C],AX   

352F:2152  8946C6                   MOV       [BP-3A],AX   

352F:2155  8B46BC                   MOV       AX,[BP-44]   

352F:2158  8B56BE                   MOV       DX,[BP-42]   

352F:215B  250F00                   AND       AX,000F   

352F:215E  81E20000                 AND       DX,0000   

352F:2162  3346C4                   XOR       AX,[BP-3C]   

352F:2165  3356C6                   XOR       DX,[BP-3A]   

352F:2168  8946C4                   MOV       [BP-3C],AX   

352F:216B  8956C6                   MOV       [BP-3A],DX   

352F:216E  8B46BC                   MOV       AX,[BP-44]   

352F:2171  8B56BE                   MOV       DX,[BP-42]   

352F:2174  B90400                   MOV       CX,0004   

352F:2177  31DB                     XOR       BX,BX   

352F:2179  9AC919471A               CALL      1A47:19C9   

352F:217E  250F00                   AND       AX,000F   

352F:2181  81E20000                 AND       DX,0000   

352F:2185  3346C4                   XOR       AX,[BP-3C]   

352F:2188  3356C6                   XOR       DX,[BP-3A]   

352F:218B  8946C4                   MOV       [BP-3C],AX   

352F:218E  8956C6                   MOV       [BP-3A],DX   

352F:2191  8B46BC                   MOV       AX,[BP-44]   

352F:2194  8B56BE                   MOV       DX,[BP-42]   

352F:2197  B90800                   MOV       CX,0008   

352F:219A  31DB                     XOR       BX,BX   

352F:219C  9AC919471A               CALL      1A47:19C9   

352F:21A1  250F00                   AND       AX,000F   

352F:21A4  81E20000                 AND       DX,0000   

352F:21A8  3346C4                   XOR       AX,[BP-3C]   

352F:21AB  3356C6                   XOR       DX,[BP-3A]   

352F:21AE  8946C4                   MOV       [BP-3C],AX   

352F:21B1  8956C6                   MOV       [BP-3A],DX   

352F:21B4  8B46BC                   MOV       AX,[BP-44]   

352F:21B7  8B56BE                   MOV       DX,[BP-42]   

352F:21BA  B91000                   MOV       CX,0010   

352F:21BD  31DB                     XOR       BX,BX   

352F:21BF  9AC919471A               CALL      1A47:19C9   

352F:21C4  250F00                   AND       AX,000F   

352F:21C7  81E20000                 AND       DX,0000   

352F:21CB  3346C4                   XOR       AX,[BP-3C]   

352F:21CE  3356C6                   XOR       DX,[BP-3A]   

352F:21D1  8946C4                   MOV       [BP-3C],AX   

352F:21D4  8956C6                   MOV       [BP-3A],DX   

352F:21D7  8B46BC                   MOV       AX,[BP-44]   

352F:21DA  8B56BE                   MOV       DX,[BP-42]   

352F:21DD  B91400                   MOV       CX,0014   

352F:21E0  31DB                     XOR       BX,BX   

352F:21E2  9AC919471A               CALL      1A47:19C9   

352F:21E7  250F00                   AND       AX,000F   

352F:21EA  81E20000                 AND       DX,0000   

352F:21EE  3346C4                   XOR       AX,[BP-3C]   

352F:21F1  3356C6                   XOR       DX,[BP-3A]   

352F:21F4  8946C4                   MOV       [BP-3C],AX   

352F:21F7  8956C6                   MOV       [BP-3A],DX   

352F:21FA  8B46BC                   MOV       AX,[BP-44]   

352F:21FD  8B56BE                   MOV       DX,[BP-42]   

352F:2200  B91800                   MOV       CX,0018   

352F:2203  31DB                     XOR       BX,BX   

352F:2205  9AC919471A               CALL      1A47:19C9   

352F:220A  250F00                   AND       AX,000F   

352F:220D  81E20000                 AND       DX,0000   

352F:2211  3346C4                   XOR       AX,[BP-3C]   

352F:2214  3356C6                   XOR       DX,[BP-3A]   

352F:2217  8946C4                   MOV       [BP-3C],AX   

352F:221A  8956C6                   MOV       [BP-3A],DX   

352F:221D  8B46BC                   MOV       AX,[BP-44]   

352F:2220  8B56BE                   MOV       DX,[BP-42]   

352F:2223  B91C00                   MOV       CX,001C   

352F:2226  31DB                     XOR       BX,BX   

352F:2228  9AC919471A               CALL      1A47:19C9   

352F:222D  250F00                   AND       AX,000F   

352F:2230  81E20000                 AND       DX,0000   

352F:2234  3346C4                   XOR       AX,[BP-3C]   

352F:2237  3356C6                   XOR       DX,[BP-3A]   

352F:223A  8946C4                   MOV       [BP-3C],AX   

352F:223D  8956C6                   MOV       [BP-3A],DX   

352F:2240  8B46C4                   MOV       AX,[BP-3C]   

352F:2243  8B56C6                   MOV       DX,[BP-3A]   

352F:2246  051000                   ADD       AX,0010   

352F:2249  83D200                   ADC       DX,00   

352F:224C  B91A00                   MOV       CX,001A   

352F:224F  31DB                     XOR       BX,BX   

352F:2251  9AEC19471A               CALL      1A47:19EC   

352F:2256  0346BC                   ADD       AX,[BP-44]   

352F:2259  1356BE                   ADC       DX,[BP-42]   

352F:225C  8946BC                   MOV       [BP-44],AX   

352F:225F  8956BE                   MOV       [BP-42],DX   

352F:2262  8B46BC                   MOV       AX,[BP-44]   

352F:2265  8B56BE                   MOV       DX,[BP-42]   

352F:2268  3B56BA                   CMP       DX,[BP-46]           //比较 KeyCode    

352F:226B  750A                     JNZ       2277   

352F:226D  3B46B8                   CMP       AX,[BP-48]           //比较 KeyCode    

352F:2270  7505                     JNZ       2277   

352F:2272  C606084D01               MOV       BYTE PTR [4D08],01   //good guy   

352F:2277  803E960600               CMP       BYTE PTR [0696],00   

352F:227C  7407                     JZ        2285   

352F:227E  803E084D00               CMP       BYTE PTR [4D08],00   

352F:2283  7408                     JZ        228D   

352F:2285  C606F14C01               MOV       BYTE PTR [4CF1],01   //good guy   

352F:228A  E95203                   JMP       25DF 

 4dd466d0e1e0cc123afbe504da2462ca
```
