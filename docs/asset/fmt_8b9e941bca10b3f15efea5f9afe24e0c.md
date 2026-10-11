
# 汇编代码分析与注释

## 代码片段 1：sub_49F15D 部分逻辑

```asm
CODE:0049F294              jz      short loc_49F28B

CODE:0049F296              dec     [ebp+var_4]    //找到了第一个非零 bit，当前 bit 位数减一

CODE:0049F299              push    [ebp+pSource]

CODE:0049F29C              push    esi

CODE:0049F29D              call    sub_49E386    //用 pSource 初始化 pDes

CODE:0049F2A2              add     esp, 8

CODE:0049F2A5              shr     bx, 1

CODE:0049F2A8              test    bx, bx

CODE:0049F2AB              jnz     short loc_49F30E --〉跳

CODE:0049F2AD              mov     bx, 8000h

CODE:0049F2B1              sub     edi, 2

CODE:0049F2B4              jmp     short loc_49F30E

CODE:0049F2B6 ; 哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪?

CODE:0049F2B6

CODE:0049F2B6 loc_49F2B6:                     ; CODE XREF: sub_49F15D+1BAj

CODE:0049F2B6              push    esi

CODE:0049F2B7              push    esi

CODE:0049F2B8              lea     eax, [ebp+var_88]

CODE:0049F2BE              push    eax

CODE:0049F2BF              call    sub_49E94F    //call sub_49e94f(tempBuf, pDest,pDest);！！！！！

CODE:0049F2C4              add     esp, 0Ch

CODE:0049F2C7              lea     edx, [ebp+var_88]

CODE:0049F2CD              push    edx

CODE:0049F2CE              push    esi

CODE:0049F2CF              call    sub_49E386    //copy tempBuf 到 pDest

CODE:0049F2D4              add     esp, 8

CODE:0049F2D7              test    [edi], bx        //pBuff 当前 bit 位的值是否为 1？

CODE:0049F2DA              jz      short loc_49F2FF

CODE:0049F2DC              push    [ebp+pSource]    //=1 继续

CODE:0049F2DF              push    esi

CODE:0049F2E0              lea     ecx, [ebp+var_88]

CODE:0049F2E6              push    ecx

CODE:0049F2E7              call    sub_49E94F    //call sub_49e94f(tempBuf, pDest, pSource);！！！！

        //这里竟然还有 pSource 作参数，菜鸟一度晕菜！

CODE:0049F2EC              add     esp, 0Ch

CODE:0049F2EF              lea     eax, [ebp+var_88]

CODE:0049F2F5              push    eax

CODE:0049F2F6              push    esi

CODE:0049F2F7              call    sub_49E386    //copy tempBuf 到 pDest

CODE:0049F2FC              add     esp, 8

CODE:0049F2FF

CODE:0049F2FF loc_49F2FF:                     ; CODE XREF: sub_49F15D+17Dj

CODE:0049F2FF              shr     bx, 1        //bx 移位

CODE:0049F302              test    bx, bx

CODE:0049F305              jnz     short loc_49F30E

CODE:0049F307              mov     bx, 8000h

CODE:0049F30B              sub     edi, 2        //edi 指向 pBuff 的当前字

CODE:0049F30E

CODE:0049F30E loc_49F30E:                     ; CODE XREF: sub_49F15D+14Ej

CODE:0049F30E                             ; sub_49F15D+157j ...

CODE:0049F30E              mov     eax, [ebp+var_4]

CODE:0049F311              add     [ebp+var_4], 0FFFFFFFFh    //当前 bit 位减一

CODE:0049F315              test    eax, eax          //遍历完 pBuff?

CODE:0049F317              jnz     short loc_49F2B6       --〉no 则跳

CODE:0049F319              push    0

CODE:0049F31B              lea     edx, [ebp+var_88]

CODE:0049F321              push    edx

CODE:0049F322              call    sub_49E3AC

CODE:0049F327              add     esp, 8

CODE:0049F32A              call    sub_49F0DA

CODE:0049F32F              mov     cx, [ebp+var_6]

CODE:0049F333              mov     ds:word_4DCF68, cx

CODE:0049F33A              xor     eax, eax

CODE:0049F33C

CODE:0049F33C loc_49F33C:                     ; CODE XREF: sub_49F15D+4Ej

CODE:0049F33C                             ; sub_49F15D+55j ...

CODE:0049F33C              pop     edi

CODE:0049F33D              pop     esi

CODE:0049F33E              pop     ebx

CODE:0049F33F              mov     esp, ebp

CODE:0049F341              pop     ebp

CODE:0049F342              retn

CODE:0049F342 sub_49F15D      endp

CODE:0049F342
```

## 逻辑说明

这里有两个 `call`，`sub_49E8B3` 和 `sub_49e94f`。

先看 `sub_49e8b3`，首先将 `pLongIntZ` 的地址存入 `4DD76C+0`。`4dd76c+4` 开始的空间存放了 16 个地址，那些地址指向的空间用来存放 `pLongIntZ*2^n` (`n=1` 到 `16`)，把 `pLongIntZ*2^n` (`n=0` 到 `16`) 的最高字节存到 `4DD7B0+2*n`，把 `pLongIntZ*2^n` (`n=0` 到 `16`) 的次高字节存到 `4DD7D2+2*n`，便于后面比较大小时使用。

`sub_49e94f`，是最核心的函数了，好长啊，从 `49e94f` 到 `49f0d9`，好在结构还比较简单，呵呵。

来看 `sub_49e94f(char *pTempDest, char *pLongIntX, char *pLongIntY)`

## 代码片段 2：sub_49E94F 函数定义

```asm
CODE:0049E94F sub_49E94F      proc near             ; CODE XREF: sub_49F15D+162p

CODE:0049E94F                             ; sub_49F15D+18Ap

CODE:0049E94F

CODE:0049E94F var_6          = word ptr -6

CODE:0049E94F var_4          = dword ptr -4

CODE:0049E94F arg_0          = dword ptr  8

CODE:0049E94F arg_4          = dword ptr  0Ch

CODE:0049E94F arg_8          = dword ptr  10h

CODE:0049E94F

CODE:0049E94F              push    ebp

CODE:0049E950              mov     ebp, esp

CODE:0049E952              add     esp, 0FFFFFFF8h

CODE:0049E955              push    ebx

CODE:0049E956              push    esi

CODE:0049E957              push    edi

CODE:0049E958              mov     edi, [ebp+arg_8]

CODE:0049E95B              mov     ebx, [ebp+arg_0]

CODE:0049E95E              push    [ebp+arg_4]

CODE:0049E961              push    offset dword_4DDF74

CODE:0049E966              call    sub_49E872    //该函数首先将 pLongIntX 的地址存入 4ddf74。

        4ddf74+4 开始的 15 个 DW 空间存放地址，这些地

        址指向的空间的值为 pLongIntX*2^n,n=1 到 15

CODE:0049E96B              add     esp, 8

CODE:0049E96E              movsx   eax, ds:word_4DCF68

CODE:0049E975              add     eax, eax

CODE:0049E977              add     eax, ebx

CODE:0049E979              add     eax, 0FFFFFFFEh

CODE:0049E97C              mov     [ebp+var_4], eax    //ebp+var4 保存 pTempDest 的最高字地址

CODE:0049E97F              mov     esi, [ebp+var_4]

CODE:0049E982              sub     esi, 2          //esi 保存 pTempDest 次高字地址

CODE:0049E985              push    0

CODE:0049E987              push    ebx

CODE:0049E988              call    sub_49E3AC    //初始化 ptempDest 指向的空间为 0

CODE:0049E98D              add     esp, 8

CODE:0049E990              push    edi

CODE:0049E991              call    sub_49E3DB    //计算 pLongIntY 的字长

CODE:0049E996              pop     ecx

CODE:0049E997              mov     [ebp+var_6], ax    //pLongIntY 字长存入 ebp+bar_6

CODE:0049E99B              cmp     [ebp+var_6], 0

CODE:0049E9A0              jnz     short loc_49E9A9    -->nz 跳

CODE:0049E9A2              xor     eax, eax

CODE:0049E9A4              jmp     loc_49F0D3

CODE:0049E9A9 ; 哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪?

CODE:0049E9A9

CODE:0049E9A9 loc_49E9A9:                     ; CODE XREF: sub_49E94F+51j

CODE:0049E9A9              movsx   edx, [ebp+var_6]

CODE:0049E9AD              add     edx, edx

CODE:0049E9AF              add     edi, edx

CODE:0049E9B1              add     edi, 0FFFFFFFEh    //edi 保存 pLongIntY 的当前字的地址

CODE:0049E9B4              jmp     loc_49F0BF    --〉跳

CODE:0049E9B9 ; 哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪?

CODE:0049E9B9

CODE:0049E9B9 loc_49E9B9:                     ; CODE XREF: sub_49E94F+77Cj

CODE:0049E9B9              push    ebx

CODE:0049E9BA              call    sub_49E83C            //将 pTempDest 移高一个字，也就是

                    将 pTempDest*2¹⁶(这步非常重要噢！）

CODE:0049E9BF              pop     ecx

CODE:0049E9C0              test    byte ptr [edi+1], 80h    //比较 pLongIntY 当前字的第 15bit,
```
